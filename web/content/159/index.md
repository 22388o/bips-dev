+++
title = "NODE_NETWORK_LIMITED service bit"
date = 2017-05-11
weight = 159
in_search_index = true

[taxonomies]
authors = ["Jonas Schnelli"]
status = ["Draft"]

[extra]
bip = 159
status = ["Draft"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0159.mediawiki"
+++

``` 
  BIP: 159
  Layer: Peer Services
  Title: NODE_NETWORK_LIMITED service bit
  Author: Jonas Schnelli <dev@jonasschnelli.ch>
  Comments-Summary: No comments yet.
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0159
  Status: Draft
  Type: Standards Track
  Created: 2017-05-11
  License: BSD-2-Clause
```

## Abstract

Define a service bit that allow pruned peers to signal their limited
services

## Motivation

Pruned peers can offer the same services as traditional peer except of
serving all historical blocks. Bitcoin right now only offers the
NODE\_NETWORK service bit which indicates that a peer can serve all
historical blocks.

1.  Pruned peers can relay blocks, headers, transactions, addresses and
    can serve a limited number of historical blocks, thus they should
    have a way how to announce their service(s)
2.  Peers no longer in initial block download should consider connecting
    some of its outbound connections to pruned peers to allow other
    peers to bootstrap from non-pruned peers

## Specification

### New service bit

This BIP proposes a new service bit

|                        |                |                                                                                                  |
| ---------------------- | -------------- | ------------------------------------------------------------------------------------------------ |
| NODE\_NETWORK\_LIMITED | bit 10 (0x400) | If signaled, the peer <I>MUST</I> be capable of serving at least the last 288 blocks (\~2 days). |

A safety buffer of 144 blocks to handle chain reorganizations
<I>SHOULD</I> be taken into account when connecting to a peer signaling
the `NODE_NETWORK_LIMITED` service bit.

### Address relay

Full nodes following this BIP <I>SHOULD</I> relay address/services
(`addr` message) from peers they would connect to (including peers
signaling `NODE_NETWORK_LIMITED`).

### Counter-measures for peer fingerprinting

Peers may have different prune depths (depending on the peers
configuration, disk space, etc.) which can result in a fingerprinting
weakness (finding the prune depth through getdata requests).
NODE\_NETWORK\_LIMITED supporting peers <I>SHOULD</I> avoid leaking the
prune depth and therefore not serve blocks deeper than the signaled
`NODE_NETWORK_LIMITED` threshold (288 blocks).

### Risks

Pruned peers following this BIP may consume more outbound bandwidth.

Light clients (and such) who are not checking the `nServiceFlags`
(service bits) from a relayed `addr`-message may unwillingly connect to
a pruned peer and ask for (filtered) blocks at a depth below their
pruned depth. Light clients should therefore check the service bits (and
eventually connect to peers signaling `NODE_NETWORK_LIMITED` if they
require \[filtered\] blocks around the tip). Light clients obtaining
peer IPs though DNS seed should use the DNS filtering option.

## Compatibility

This proposal is backward compatible.

## Reference implementation

  - <https://github.com/bitcoin/bitcoin/pull/11740> (signaling)
  - <https://github.com/bitcoin/bitcoin/pull/10387> (connection and
    relay)

## Copyright

This BIP is licensed under the 2-clause BSD license.
