+++
title = "Dynamic Hierarchical Deterministic Key Trees"
date = 2015-11-16
weight = 83
in_search_index = true

[taxonomies]
authors = ["Eric Lombrozo"]
status = ["Rejected"]

[extra]
bip = 83
status = ["Rejected"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0083.mediawiki"
+++

``` 
  BIP: 83
  Layer: Applications
  Title: Dynamic Hierarchical Deterministic Key Trees
  Author: Eric Lombrozo <eric@ciphrex.com>
  Comments-Summary: No comments yet.
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0083
  Status: Rejected
  Type: Standards Track
  Created: 2015-11-16
  License: PD
```

## Abstract

This BIP defines a scheme for key derivation that allows for dynamic
creation of key hierarchies based on the algorithm described in BIP32.

## Motivation

Several proposals have been made to try to standardize a structure for
hierarchical deterministic wallets for the sake of interoperability
(reference BIP32, BIP44, BIP45). However, all proposals to date have
tried to impose a specific structure upfront without providing any
flexibility for dynamic creation of new hierarchical levels with
different semantics or mapping between different applications that use
distinct structures.

Instead of attempting to impose a specific structure upfront, this BIP
proposes that we design the derivation in such a way that we can
continue extending hierarchies arbitrarily and indefinitely.

## Specification

BIP32 provides a hierarchical derivation scheme where every node in the
tree can be either used to derive child nodes or used as a signing key
for ECDSA. This means that as soon as we choose to use a node as a
signing key, we can no longer derive children from that node. To draw an
analogy to file systems, each node is either a file or a directory but
never both. However, given the need to predictably know the location of
new children, it is generally not a good idea to mix both signing keys
and parent nodes at the same level in the hierarchy. This means that as
soon as we've decided that a particular level in the hierarchy is to be
used for signing keys, we've lost the ability to nest deeper levels into
the tree.

At every level of the hierarchy, we reserve the child with index 0 to
allow further nesting, and for signing key parent nodes use child
indices 1 to MAX\_INDEX (2<sup>31</sup> - 1) for signing keys. We can
use either hardened or nonhardened derivation.

Let p denote a specific signing key parent node and k be an index
greater than 0. The children signing keys are then:

p / k

with k \> 0.

To create sublevels, we derive the nested nodes:

p / 0 / n

with n ≥ 0.

Each of these nodes can now contain signing key children of their own,
and again we reserve index 0 to allow deeper nesting.

## Notation

We propose the following shorthand for writing nested node derivations:

p // n instead of p / 0 / n

p //' n instead of p / 0' / n

## Mappings

Rather than specifying upfront which path is to be used for a specific
purpose (i.e. external invoicing vs. internal change), different
applications can specify arbitrary parent nodes and derivation paths.
This allows for nesting of sublevels to arbitrary depth with
application-specified semantics. Rather than trying to specify use cases
upfront, we leave the design completely open-ended. Different
applications can exchange these mappings for interoperability.
Eventually, if certain mappings become popular, application user
interfaces can provide convenient shortcuts or use them as defaults.

Note that BIP32 suggests reserving child 0 for the derivation of signing
keys rather than sublevels. It is not really necessary to reserve
signing key parents, however, as each key's parent's path can be
explicitly stated. But unless we reserve a child for sublevel
derivation, we lose the ability to nest deeper levels into the
hierarchy. While we could reserve any arbitrary index for nesting
sublevels, reserving child 0 seems simplest to implement, leaving all
indices \> 0 for contiguously indexed signing keys. We could also use
MAX\_INDEX (2<sup>31</sup> - 1) for this purpose. However, we believe
doing so introduces more ideosyncracies into the semantics and will
present a problem if we ever decide to extend the scheme to use indices
larger than 31 bits.

## Use Cases

### Account Hierarchies

For all that follows, we assume that key indices k \> 0 and parent node
indices n ≥ 0.

From a master seed m, we can construct a default account using the
following derivations for nonhardened signing keys:

m / 1 / k (for change/internal outputs)

m / 2 / k (for invoice/external outputs)

To create subaccount a<sub>n</sub>, we use:

a<sub>n</sub> = m // n

To generate keys for subaccount a<sub>n</sub>, we use:

a<sub>n</sub> / 1 / k (for change/internal outputs)

a<sub>n</sub> / 2 / k (for invoice/external outputs)

We can continue creating subaccounts indefinitely using this scheme.

### Bidirectional Payment Channels

In order to create a bidirectional payment channel, it is necessary that
previous commitments be revokable. In order to revoke previous
commitments, each party reveals a secret to the other that would allow
them to steal the funds in the channel if a transaction for a previous
commitment is inserted into the blockchain.

By allowing for arbitrary nesting of sublevels, we can construct
decision trees of arbitrary depth and revoke an entire branch by
revealing a parent node used to derive all the children.

## References

  - [BIP32 - Hierarchical Deterministic
    Wallets](bip-0032.mediawiki "wikilink")
  - [Lightning Network
    Whitepaper](https://lightning.network/lightning-network-paper.pdf "wikilink")

## Copyright

This document is placed in the public domain.
