+++
title = "Transaction announcements reconciliation"
date = 2019-09-25
weight = 330
in_search_index = true

[taxonomies]
authors = ["Gleb Naumenko", "Pieter Wuille"]
status = ["Draft"]

[extra]
bip = 330
status = ["Draft"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0330.mediawiki"
+++

``` 
  BIP: 330
  Layer: Peer Services
  Title: Transaction announcements reconciliation
  Author: Gleb Naumenko <naumenko.gs@gmail.com>
          Pieter Wuille <pieter.wuille@gmail.com>
  Comments-Summary: No comments yet.
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0330
  Status: Draft
  Type: Standards Track
  Created: 2019-09-25
  License: CC0-1.0
  License-Code: MIT
```

## Abstract

This document specifies a P2P protocol extension for reconciliation of
transaction announcements <b>between 2 nodes</b>, which is a building
block for efficient transaction relay protocols (e.g.,
[Erlay](https://arxiv.org/pdf/1905.10518.pdf)). This is a step towards
increasing the connectivity of the network for almost no bandwidth cost.

## Motivation

Currently in the Bitcoin network, every 32-byte transaction ID is
announced in at least one direction between every pair of connected
peers, via INV messages. This results in high cost of announcing
transactions: *O(nodes \* connections\_per\_node)*.

A <b>reconciliation-based protocol</b> which uses the technique
suggested in this document can have better scaling properties than
INV-based flooding.

Increasing the connectivity of the network makes the network more robust
to partitioning attacks; thus, improving the bandwidth scaling of
transaction relay to *O(nodes)* (and without a high constant overhead)
would allow us to improve the security of the network by increasing
connectivity. It would also reduce the bandwidth required to run a
Bitcoin node and potentially enable more users to run full nodes.

### Erlay

[Erlay](https://arxiv.org/pdf/1905.10518.pdf) is an example of a
high-level transaction relay protocol which employs set reconciliation
for bandwidth efficiency.

Erlay uses both flooding (announcing using INV messages to all peers)
and reconciliation to announce transactions. Flooding is expensive, so
Erlay seeks to use it sparingly and in strategic locations - only
well-connected publicly reachable nodes flood transactions to other
publicly reachable nodes via outbound connections. Since every
unreachable node is directly connected to several reachable nodes, this
policy ensures that a transaction is quickly propagated to be within one
hop from most of the nodes in the network.

All transactions not propagated through flooding are propagated through
efficient set reconciliation. To do this, every node keeps a
reconciliation set for each peer, in which transactions are placed which
would have been announced using INV messages absent this protocol. Every
2 seconds every node chooses a peer from its outbound connections in a
predetermined order to reconcile with, resulting in both sides learning
the transactions known to the other side. After every reconciliation
round, the corresponding reconciliation set is cleared. A more detailed
description of a set reconciliation round and other implementation
details can be found in the paper.

Erlay allows us to:

  - save 40% of the bandwidth consumed by a node, given typical network
    connectivity as of July 2019.
  - achieve similar latency
  - increase network connectivity for almost no bandwidth or latency
    cost
  - improves privacy as a side-effect

This document proposes a P2P-layer extension which is required to enable
efficient reconciliation-based protocols (like Erlay) for transaction
relay.

## Specification

### New data structures

Several new data structures are introduced to the P2P protocol first, to
aid with efficient transaction relay.

#### 32-bit short transaction IDs

During reconciliation, significantly abbreviated transaction IDs are
used of just 32 bits in size. To prevent attackers from constructing
sets of transactions that cause network-wide collisions, the short ID
computation is salted on a per-link basis using 64 bits of entropy
contributed by both communication partners.

Short IDs are computed as follows:

  - Let *salt<sub>1</sub>* and *salt<sub>2</sub>* be the entropy
    contributed by both sides; see the "sendrecon" message further for
    details how they are exchanged.
  - Sort the two salts such that *salt<sub>1</sub> ≤ salt<sub>2</sub>*
    (which side sent what doesn't matter).
  - Compute *h = SHA256("Tx Relay Salting" || salt<sub>1</sub> ||
    salt<sub>2</sub>)*, where the two salts are encoded in 64-bit
    little-endian byte order.
  - Let *k<sub>0</sub>* be the 64-bit integer obtained by interpreting
    the first 8 bytes of *h* in little-endian byte order.
  - Let *k<sub>1</sub>* be the 64-bit integer obtained by interpreting
    the second 8 bytes of *h* in little-endian byte order.
  - Let *s = SipHash-2-4((k<sub>0</sub>,k<sub>1</sub>),wtxid)*, where
    *wtxid* is the transaction hash including witness data as defined by
    BIP141.
  - The short ID is equal to *1 + (s mod 0xFFFFFFFF)*.

This results in approximately uniformly distributed IDs in the range
*\[1..0xFFFFFFFF\]*, which is a requirement for using them as elements
in 32-bit sketches. See the next paragraph for details.

#### Short transaction ID sketches

Reconciliation-based relay uses
[PinSketch](https://www.cs.bu.edu/~reyzin/code/fuzzy.html) BCH-based
secure sketches as introduced by the [Fuzzy Extractors
paper](https://www.cs.bu.edu/~reyzin/fuzzy.html). They are a form of set
checksums with the following properties:

  - Sketches have a predetermined capacity, and when the number of
    elements in the set does not exceed the capacity, it is always
    possible to recover the entire set from the sketch by decoding the
    sketch. A sketch of nonzero b-bit elements with capacity c can be
    stored in bc bits.
  - A sketch of the [symmetric
    difference](https://en.wikipedia.org/wiki/Symmetric_difference)
    between the two sets (i.e., all elements that occur in one but not
    both input sets), can be obtained by combining the sketches of those
    sets.

The sketches used here consists of elements of the [finite
field](https://en.wikipedia.org/wiki/Finite_field) *GF(2<sup>32</sup>)*.
Specifically, we represent finite field elements as polynomials in *x*
over *GF(2)* modulo *x<sup>32</sup>7</sup> + x<sup>3</sup> +
x<sup>2</sup> + 1*. To map integers to finite field elements, simply
treat each bit *i* (with value *2<sup>i</sup>*) in the integer as the
coefficient of *x<sup>i</sup>* in the polynomial representation. For
example the integer *101 = 2<sup>6</sup> + 2<sup>5</sup> + 2<sup>2</sup>
+ 1* is mapped to field element *x<sup>6</sup> + x<sup>5</sup> +
x<sup>2</sup> + 1*. These field elements can be added and multiplied
together, but the specifics of that are out of scope for this document.

A short ID sketch with capacity *c* consists of a sequence of *c* field
elements. The first is the sum of all short IDs in the set, the second
is the sum of the 3rd powers of all short IDs, the third is the sum of
the 5th powers etc., up to the last element with is the sum of the
*(2c-1)*th powers. These elements are then encoded as 32-bit integers in
little endian byte order, resulting in a *4c*-byte serialization.

The following Python 3.2+ code implements the creation of sketches:

    FIELD_BITS = 32
    FIELD_MODULUS = (1 << FIELD_BITS) + 0b10001101
    
    def mul2(x):
        """Compute 2*x in GF(2^FIELD_BITS)"""
        return (x << 1) ^ (FIELD_MODULUS if x.bit_length() >= FIELD_BITS else 0)
    
    def mul(x, y):
        """Compute x*y in GF(2^FIELD_BITS)"""
        ret = 0
        for bit in [(x >> i) & 1 for i in range(x.bit_length())]:
            ret, y = ret ^ bit * y, mul2(y)
        return ret
    
    def create_sketch(shortids, capacity):
        """Compute the bytes of a sketch for given shortids and given capacity."""
        odd_sums = [0 for _ in range(capacity)]
        for shortid in shortids:
            squared = mul(shortid, shortid)
            for i in range(capacity):
                odd_sums[i] ^= shortid
                shortid = mul(shortid, squared)
        return b''.join(elem.to_bytes(4, 'little') for elem in odd_sums)

The [minisketch](https://github.com/sipa/minisketch/) library implements
the construction, merging, and decoding of these sketches efficiently.

#### Truncated transaction IDs

For announcing and relaying transaction outside of reconciliation, we
need an unambiguous, unsalted way to refer to transactions to
deduplicate transaction requests. As we're introducing a new scheme
anyway, this is a good opportunity to switch to wtxid-based requests
rather than txid-based ones. While using full 256-bit wtxids is
possible, this is overkill as they contribute significantly to the total
bandwidth as well. Instead, we truncate the wtxid to just their first
128 bits. These are referred to as truncated IDs.

### Intended Protocol Flow

Set reconciliation primarily consists of the transmission and decoding
of a reconciliation set sketch upon request.

[framed|center|Set reconciliation protocol
flow](File:bip-0330/recon_scheme_merged.png "wikilink")

#### Bisection

If a node is unable to reconstruct the set difference from the received
sketch, the node then makes an additional reconciliation request,
similar to the initial one, but this request is applied to only a
fraction of possible transactions (e.g., in the range 0x0–0x8). Because
of the linearity of sketches, a sketch of a subset of transactions would
allow the node to compute a sketch for the remainder, which saves
bandwidth.

[framed|300px|center|Bisection](File:bip-0330/bisection.png "wikilink")

### New messages

Several new protocol messages are added: sendrecon, reqreconcil, sketch,
reqbisec, reconcildiff, invtx, gettx. This section describes their
serialization, contents, and semantics.

In what follows, all integers are serialized in little-endian byte
order. Boolean values are encoded as a single byte that must be 0 or 1
exactly. Arrays are serialized with the CompactSize prefix that encodes
their length, as is common in other P2P messages.

#### sendrecon

The sendrecon message announces support for the reconciliation protocol.
It is expected to be only sent once, and ignored by nodes that don't
support it.

Its payload consists of:

| Data type | Name      | Description                                                                        |
| --------- | --------- | ---------------------------------------------------------------------------------- |
| bool      | sender    | Indicates whether the sender will send "reqreconcil" message                       |
| bool      | responder | Indicates whether the sender will respond to "reqreconcil" messages.               |
| uint32    | version   | Sender must set this to 1 currently, otherwise receiver should ignore the message. |
| uint64    | salt      | The salt used in the short transaction ID computation.                             |

"reqreconcil" messages can only be sent if the sender has sent a
"sendrecon" message with sender=true, and the receiver has sent a
"sendrecon" message with responder=true.

#### reqreconcil

The reqreconcil message initiates a reconciliation round.

| Data type | Name      | Description                                                                                                                                     |
| --------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| uint16    | set\_size | Size of the sender's reconciliation set, used to estimate set difference.                                                                       |
| uint8     | q         | Coefficient used to estimate set difference. Multiplied by PRECISION=2^6 and rounded up by the sender and divided by PRECISION by the receiver. |

Upon receipt of a "reqreconcil" message, the receiver:

  - Constructs and sends a "sketch" message (see below), with a sketch
    of capacity computed as *|set\_size - local\_set\_size| + q \*
    (set\_size + local\_set\_size) + c*, where *local\_set\_size*
    represents size of the receiver's reconciliation set.
  - Makes a snapshot of their current reconciliation set, and clears the
    set itself. The snapshot is kept until a "reconcildiff" message is
    received by the node.

It is suggested to use *c=1* to avoid sending empty sketches and reduce
the overhead caused by under-estimations.

Intuitively, *q* represents the discrepancy in sets: the closer the sets
are, the lower optimal *q* is. As suggested by Erlay, *q* should be
derived as an optimal *q* value for the previous reconciliation with a
given peer, once the actual set sizes and set difference are known.
Alternatively, *q=0.1* should be used as a default value. For example,
if in previous round *set\_size=30* and *local\_set\_size=20*, and the
\*actual\* difference was *4*, then a node should compute *q* as
following: *q=(|30-20| - 1) / (30+20)=0.18* The derivation of *q* can be
changed according to the version of the protocol.

No new "reqreconcil" message can be sent until a "reconcildiff" message
is sent.

#### sketch

The sketch message is used to communicate a sketch required to perform
set reconciliation.

| Data type | Name   | Description                                        |
| --------- | ------ | -------------------------------------------------- |
| byte\[\]  | skdata | The sketch of the sender's reconciliation snapshot |

Upon receipt of a "sketch" message, a node computes the set difference
by combining the receiver sketch with a sketch computed locally for a
corresponding reconciliation set. If this is the 2nd time for this round
a "sketch" message was received, the bisection approach is used, and by
combining the new sketch with the previous one, two difference sketches
are obtained, one for the first half and one for the second half of the
short id range. The receiving node then tries to decode this sketch (or
sketches), and based on the result:

  - If decoding fails, a "reconcildiff" message is sent with the failure
    flag set (success=false). If this was the first "sketch" in the
    round, a "reqbisec" message may be sent instead.
  - If decoding succeeds, a "reconcildiff" message is sent with the
    truncated IDs of all locally known transactions that appear in the
    decode result, and the short IDs of the unrecognized ones.

The receiver also makes snapshot of their current reconciliation set,
and clears the set itself. The snapshot is kept until a "reconcildiff"
message is sent by the node.

#### reqbisec

The reqbisec message is used to signal that set reconciliation has
failed and an extra sketch is needed to find set difference.

It has an empty payload.

Upon receipt of a "reqbisec" message, a node responds to it with a
"sketch" message, which contains a sketch of a subset of corresponding
reconciliation set <b>snapshot</b> (stored when "reqreconcil" message
for the current round was processed) (values in range *\[0..(2^31)\]*).

#### reconcildiff

The reconcildiff message is used to announce transactions which are
found to be missing during set reconciliation on the sender's side.

| Data type  | Name          | Description                                                                   |
| ---------- | ------------- | ----------------------------------------------------------------------------- |
| uint8      | success       | Indicates whether sender of the message succeeded at set difference decoding. |
| uint32\[\] | ask\_shortids | The short IDs that the sender did not have.                                   |

Upon receipt a "reconcildiff" message with *success=1*, a node sends a
"invtx" message for the transactions requested by 32-bit IDs (first
vector) containing their 128-bit truncated IDs (with parent transactions
occuring before their dependencies), and can request announced
transactions (second vector) it does not have via a "gettx" message.
Otherwise if *success=0*, receiver should request bisection via
*reqbisec* (if failure happened for the first time). If failure happened
for the second time, receiver should announce the transactions from the
reconciliation set via an "invtx" message, excluding the transactions
announced from the sender.

The <b>snapshot</b> of the corresponding reconciliation set is cleared
by the sender and the receiver of the message.

The sender should also send their own "invtx" message along with the
reconcildiff message to announce transactions which are missing on the
receiver's side.

#### invtx

The invtx message is used to announce transactions (both along with
reconcildiff message and as a response to the reconcildiff message). It
is the truncated ID analogue of "inv" (which cannot be used because it
has 256-bit elements).

| Data type   | Name          | Description                                                                       |
| ----------- | ------------- | --------------------------------------------------------------------------------- |
| uint128\[\] | inv\_truncids | The truncated IDs of transactions the sender believes the receiver does not have. |

Upon receipt a "invtx" message, a node requests announced transactions
it does not have. The <b>snapshot</b> of the corresponding
reconciliation set is cleared by the sender of the message.

#### gettx

The gettx message is used to request transactions by 128-bit truncated
IDs. It is the truncated ID analogue of "getdata".

| Data type   | Name          | Description                                                                       |
| ----------- | ------------- | --------------------------------------------------------------------------------- |
| uint128\[\] | ask\_truncids | The truncated IDs of transactions the sender wants the full transaction data for. |

Upon receipt a "gettx" message, a node sends "tx" messages for the
requested transactions.

## Local state

This BIP suggests a stateful protocol and it requires storing several
variables at every node to operate properly.

#### Reconciliation sets

Every node stores a set of 128-bit truncated IDs for every peer which
supports transaction reconciliation, representing the transactions which
would have been sent according to the regular flooding protocol.
Incoming transactions are added to sets when those transactions are
received (if they satisfy the policies such as minimum fee set by a
peer). A reconciliation set is moved to the corresponding set snapshot
after the transmission of the initial sketch.

#### Reconciliation set snapshot

After the transmitting of the initial sketch (either sending or
receiving of reconcildiff message), every node should store the snapshot
of the current reconciliation set, and clear the set. This is important
to make bisection more stable during the reconciliation round (bisection
should be applied to the snapshot). The snapshot is also used to
efficiently lookup the transactions requested by short ID. The snapshot
is cleared after the end of the reconciliation round (sending or
receiving of the reconcildiff message).

#### q-coefficient

The q value should be stored to make efficient difference estimation. It
is shared across peers and changed after every reconciliation.
q-coefficient represents the discrepancy in sets: the closer the sets
are, the lower optimal *q* is. In future implementations, q could vary
across different peers or become static.

## Backward compatibility

Older clients remain fully compatible and interoperable after this
change.

Clients which do not implement this protocol remain fully compatible
after this change using existing protocols, because transaction
announcement reconciliation is used only for peers that negotiate
support for it.

## Rationale

#### Why using PinSketch for set reconciliation?

PinSketch is more bandwidth efficient than IBLT, especially for the
small differences in sets we expect to operate over. PinSketch is as
bandwidth efficient as CPISync, but PinSketch has quadratic decoding
complexity, while CPISync have cubic decoding complexity. This makes
PinSketch significantly faster.

#### Why using 32-bit short transaction IDs?

To use Minisketch in practice, transaction IDs should be shortened
(ideally, not more than 64 bits per element). Small number of bits per
transaction also allows to save extra bandwidth and make operations over
sketches faster. According to our estimates, 32 bits provides low
collision rate in a non-adversarial model (which is enabled by using
independent salts per-link).

#### Why using 128-bit short IDs?

To avoid problems caused by the delays in the network, our protocol
requires extra round of announcing unsalted transaction IDs.
[Erlay](https://arxiv.org/pdf/1905.10518.pdf) protocol on top of this
work also requires announcing unsalted transaction IDs for flooding.
Both of these measures allow to deduplicate transaction announcements
across the peers. However, using full 256-bit IDs to uniquely identify
transactions seems to be an overkill. 128 is the highest power of 2
which provides good enough collision-resistance in an adversarial model,
and trivially saves a significant portion of the bandwidth related to
these announcements.

#### Why using bisection instead of extending the sketch?

Unlike extended sketches, bisection does not require operating over
sketches of higher order. This allows to avoid the high computational
cost caused by quadratic decoding complexity.

## Implementation

TODO

## Acknowledgments

A large fraction of this proposal was done during designing Erlay with
Gregory Maxwell, Sasha Fedorova and Ivan Beschastnikh. We would like to
thank Suhas Daftuar for contributions to the design and BIP structure.
We would like to thank Ben Woosley for contributions to the high-level
description of the idea.

## Copyright

This document is licensed under the Creative Commons CC0 1.0 Universal
license.
