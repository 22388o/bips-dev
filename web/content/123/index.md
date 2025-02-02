+++
title = "BIP Classification"
date = 2015-08-26
weight = 123
in_search_index = true

[taxonomies]
authors = ["Eric Lombrozo"]
status = ["Active"]

[extra]
bip = 123
status = ["Active"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0123.mediawiki"
+++

``` 
  BIP: 123
  Title: BIP Classification
  Author: Eric Lombrozo <elombrozo@gmail.com>
  Comments-Summary: No comments yet.
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0123
  Status: Active
  Type: Process
  Created: 2015-08-26
  License: CC0-1.0
           GNU-All-Permissive
```

## Abstract

This document describes a classification scheme for BIPs.

BIPs are classified by system layers with lower numbered layers
involving more intricate interoperability requirements.

The specification defines the layers and sets forth specific criteria
for deciding to which layer a particular standards BIP belongs.

## Copyright

This BIP is dual-licensed under the Creative Commons CC0 1.0 Universal
and GNU All-Permissive licenses.

## Motivation

Bitcoin is a system involving a number of different standards. Some
standards are absolute requirements for interoperability while others
can be considered optional, giving implementors a choice of whether to
support them.

In order to have a BIP process which more closely reflects the
interoperability requirements, it is necessary to categorize BIPs
accordingly. Lower layers present considerably greater challenges in
getting standards accepted and deployed.

## Specification

Standards BIPs are placed in one of four layers:

1.  Consensus
2.  Peer Services
3.  API/RPC
4.  Applications

Non-standards BIPs may be placed in these layers, or none at all.

### 1\. Consensus Layer

The consensus layer defines cryptographic commitment structures. Its
purpose is ensuring that anyone can locally evaluate whether a
particular state and history is valid, providing settlement guarantees,
and assuring eventual convergence.

The consensus layer is not concerned with how messages are propagated on
a network.

Disagreements over the consensus layer can result in network
partitioning, or forks, where different nodes might end up accepting
different incompatible histories. We further subdivide consensus layer
changes into soft forks and hard forks.

#### Soft Forks

In a soft fork, some structures that were valid under the old rules are
no longer valid under the new rules. Structures that were invalid under
the old rules continue to be invalid under the new rules.

#### Hard Forks

In a hard fork, structures that were invalid under the old rules become
valid under the new rules.

### 2\. Peer Services Layer

The peer services layer specifies how nodes find each other and
propagate messages.

Only a subset of all specified peer services are required for basic node
interoperability. Nodes can support further optional extensions.

It is always possible to add new services without breaking compatibility
with existing services, then gradually deprecate older services. In this
manner, the entire network can be upgraded without serious risks of
service disruption.

### 3\. API/RPC Layer

The API/RPC layer specifies higher level calls accessible to
applications. Support for these BIPs is not required for basic network
interoperability but might be expected by some client applications.

There's room at this layer to allow for competing standards without
breaking basic network interoperability.

### 4\. Applications Layer

The applications layer specifies high level structures, abstractions,
and conventions that allow different applications to support similar
features and share data.

## Classification of existing BIPs

| Number                               | Layer                 | Title                                                                                 | Owner                                                    | Type          | Status    |
| ------------------------------------ | --------------------- | ------------------------------------------------------------------------------------- | -------------------------------------------------------- | ------------- | --------- |
| [1](bip-0001.mediawiki "wikilink")   |                       | BIP Purpose and Guidelines                                                            | Amir Taaki                                               | Process       | Active    |
| [2](bip-0002.mediawiki "wikilink")   |                       | BIP process, revised                                                                  | Luke Dashjr                                              | Process       | Draft     |
| [9](bip-0009.mediawiki "wikilink")   |                       | Version bits with timeout and delay                                                   | Pieter Wuille, Peter Todd, Greg Maxwell, Rusty Russell   | Informational | Final     |
| [10](bip-0010.mediawiki "wikilink")  | Applications          | Multi-Sig Transaction Distribution                                                    | Alan Reiner                                              | Informational | Withdrawn |
| [11](bip-0011.mediawiki "wikilink")  | Applications          | M-of-N Standard Transactions                                                          | Gavin Andresen                                           | Standard      | Final     |
| [12](bip-0012.mediawiki "wikilink")  | Consensus (soft fork) | OP\_EVAL                                                                              | Gavin Andresen                                           | Standard      | Withdrawn |
| [13](bip-0013.mediawiki "wikilink")  | Applications          | Address Format for pay-to-script-hash                                                 | Gavin Andresen                                           | Standard      | Final     |
| [14](bip-0014.mediawiki "wikilink")  | Peer Services         | Protocol Version and User Agent                                                       | Amir Taaki, Patrick Strateman                            | Standard      | Final     |
| [15](bip-0015.mediawiki "wikilink")  | Applications          | Aliases                                                                               | Amir Taaki                                               | Standard      | Deferred  |
| [16](bip-0016.mediawiki "wikilink")  | Consensus (soft fork) | Pay to Script Hash                                                                    | Gavin Andresen                                           | Standard      | Final     |
| [17](bip-0017.mediawiki "wikilink")  | Consensus (soft fork) | OP\_CHECKHASHVERIFY (CHV)                                                             | Luke Dashjr                                              | Standard      | Withdrawn |
| [18](bip-0018.mediawiki "wikilink")  | Consensus (soft fork) | hashScriptCheck                                                                       | Luke Dashjr                                              | Standard      | Accepted  |
| [19](bip-0019.mediawiki "wikilink")  | Applications          | M-of-N Standard Transactions (Low SigOp)                                              | Luke Dashjr                                              | Standard      | Draft     |
| [20](bip-0020.mediawiki "wikilink")  | Applications          | URI Scheme                                                                            | Luke Dashjr                                              | Standard      | Replaced  |
| [21](bip-0021.mediawiki "wikilink")  | Applications          | URI Scheme                                                                            | Nils Schneider, Matt Corallo                             | Standard      | Final     |
| [22](bip-0022.mediawiki "wikilink")  | API/RPC               | getblocktemplate - Fundamentals                                                       | Luke Dashjr                                              | Standard      | Final     |
| [23](bip-0023.mediawiki "wikilink")  | API/RPC               | getblocktemplate - Pooled Mining                                                      | Luke Dashjr                                              | Standard      | Final     |
| [30](bip-0030.mediawiki "wikilink")  | Consensus (soft fork) | Duplicate transactions                                                                | Pieter Wuille                                            | Standard      | Final     |
| [31](bip-0031.mediawiki "wikilink")  | Peer Services         | Pong message                                                                          | Mike Hearn                                               | Standard      | Final     |
| [32](bip-0032.mediawiki "wikilink")  | Applications          | Hierarchical Deterministic Wallets                                                    | Pieter Wuille                                            | Informational | Final     |
| [33](bip-0033.mediawiki "wikilink")  | Peer Services         | Stratized Nodes                                                                       | Amir Taaki                                               | Standard      | Draft     |
| [34](bip-0034.mediawiki "wikilink")  | Consensus (soft fork) | Block v2, Height in Coinbase                                                          | Gavin Andresen                                           | Standard      | Final     |
| [35](bip-0035.mediawiki "wikilink")  | Peer Services         | mempool message                                                                       | Jeff Garzik                                              | Standard      | Final     |
| [36](bip-0036.mediawiki "wikilink")  | Peer Services         | Custom Services                                                                       | Stefan Thomas                                            | Standard      | Draft     |
| [37](bip-0037.mediawiki "wikilink")  | Peer Services         | Connection Bloom filtering                                                            | Mike Hearn, Matt Corallo                                 | Standard      | Final     |
| [38](bip-0038.mediawiki "wikilink")  | Applications          | Passphrase-protected private key                                                      | Mike Caldwell, Aaron Voisine                             | Standard      | Draft     |
| [39](bip-0039.mediawiki "wikilink")  | Applications          | Mnemonic code for generating deterministic keys                                       | Marek Palatinus, Pavol Rusnak, Aaron Voisine, Sean Bowe  | Standard      | Accepted  |
| [42](bip-0042.mediawiki "wikilink")  | Consensus (soft fork) | A finite monetary supply for Bitcoin                                                  | Pieter Wuille                                            | Standard      | Draft     |
| [43](bip-0043.mediawiki "wikilink")  | Applications          | Purpose Field for Deterministic Wallets                                               | Marek Palatinus, Pavol Rusnak                            | Informational | Draft     |
| [44](bip-0044.mediawiki "wikilink")  | Applications          | Multi-Account Hierarchy for Deterministic Wallets                                     | Marek Palatinus, Pavol Rusnak                            | Standard      | Accepted  |
| [45](bip-0045.mediawiki "wikilink")  | Applications          | Structure for Deterministic P2SH Multisignature Wallets                               | Manuel Araoz, Ryan X. Charles, Matias Alejo Garcia       | Standard      | Accepted  |
| [47](bip-0047.mediawiki "wikilink")  | Applications          | Reusable Payment Codes for Hierarchical Deterministic Wallets                         | Justus Ranvier                                           | Informational | Draft     |
| [49](bip-0049.mediawiki "wikilink")  | Applications          | Derivation scheme for P2WPKH-nested-in-P2SH based accounts                            | Daniel Weigl                                             | Informational | Draft     |
| [50](bip-0050.mediawiki "wikilink")  |                       | March 2013 Chain Fork Post-Mortem                                                     | Gavin Andresen                                           | Informational | Final     |
| [60](bip-0060.mediawiki "wikilink")  | Peer Services         | Fixed Length "version" Message (Relay-Transactions Field)                             | Amir Taaki                                               | Standard      | Draft     |
| [61](bip-0061.mediawiki "wikilink")  | Peer Services         | Reject P2P message                                                                    | Gavin Andresen                                           | Standard      | Final     |
| [62](bip-0062.mediawiki "wikilink")  | Consensus (soft fork) | Dealing with malleability                                                             | Pieter Wuille                                            | Standard      | Withdrawn |
| [64](bip-0064.mediawiki "wikilink")  | Peer Services         | getutxo message                                                                       | Mike Hearn                                               | Standard      | Draft     |
| [65](bip-0065.mediawiki "wikilink")  | Consensus (soft fork) | OP\_CHECKLOCKTIMEVERIFY                                                               | Peter Todd                                               | Standard      | Final     |
| [66](bip-0066.mediawiki "wikilink")  | Consensus (soft fork) | Strict DER signatures                                                                 | Pieter Wuille                                            | Standard      | Final     |
| [67](bip-0067.mediawiki "wikilink")  | Applications          | Deterministic Pay-to-script-hash multi-signature addresses through public key sorting | Thomas Kerin, Jean-Pierre Rupp, Ruben de Vries           | Standard      | Accepted  |
| [68](bip-0068.mediawiki "wikilink")  | Consensus (soft fork) | Relative lock-time using consensus-enforced sequence numbers                          | Mark Friedenbach, BtcDrak, Nicolas Dorier, kinoshitajona | Standard      | Final     |
| [69](bip-0069.mediawiki "wikilink")  | Applications          | Lexicographical Indexing of Transaction Inputs and Outputs                            | Kristov Atlas                                            | Informational | Accepted  |
| [70](bip-0070.mediawiki "wikilink")  | Applications          | Payment Protocol                                                                      | Gavin Andresen, Mike Hearn                               | Standard      | Final     |
| [71](bip-0071.mediawiki "wikilink")  | Applications          | Payment Protocol MIME types                                                           | Gavin Andresen                                           | Standard      | Final     |
| [72](bip-0072.mediawiki "wikilink")  | Applications          | bitcoin: uri extensions for Payment Protocol                                          | Gavin Andresen                                           | Standard      | Final     |
| [73](bip-0073.mediawiki "wikilink")  | Applications          | Use "Accept" header for response type negotiation with Payment Request URLs           | Stephen Pair                                             | Standard      | Final     |
| [74](bip-0074.mediawiki "wikilink")  | Applications          | Allow zero value OP\_RETURN in Payment Protocol                                       | Toby Padilla                                             | Standard      | Draft     |
| [75](bip-0075.mediawiki "wikilink")  | Applications          | Out of Band Address Exchange using Payment Protocol Encryption                        | Justin Newton, Matt David, Aaron Voisine, James MacWhyte | Standard      | Draft     |
| [80](bip-0080.mediawiki "wikilink")  |                       | Hierarchy for Non-Colored Voting Pool Deterministic Multisig Wallets                  | Justus Ranvier, Jimmy Song                               | Informational | Deferred  |
| [81](bip-0081.mediawiki "wikilink")  |                       | Hierarchy for Colored Voting Pool Deterministic Multisig Wallets                      | Justus Ranvier, Jimmy Song                               | Informational | Deferred  |
| [83](bip-0083.mediawiki "wikilink")  | Applications          | Dynamic Hierarchical Deterministic Key Trees                                          | Eric Lombrozo                                            | Standard      | Draft     |
| [99](bip-0099.mediawiki "wikilink")  |                       | Motivation and deployment of consensus rule changes (\[soft/hard\]forks)              | Jorge Timón                                              | Informational | Draft     |
| [101](bip-0101.mediawiki "wikilink") | Consensus (hard fork) | Increase maximum block size                                                           | Gavin Andresen                                           | Standard      | Withdrawn |
| [102](bip-0102.mediawiki "wikilink") | Consensus (hard fork) | Block size increase to 2MB                                                            | Jeff Garzik                                              | Standard      | Draft     |
| [103](bip-0103.mediawiki "wikilink") | Consensus (hard fork) | Block size following technological growth                                             | Pieter Wuille                                            | Standard      | Draft     |
| [105](bip-0105.mediawiki "wikilink") | Consensus (hard fork) | Consensus based block size retargeting algorithm                                      | BtcDrak                                                  | Standard      | Draft     |
| [106](bip-0106.mediawiki "wikilink") | Consensus (hard fork) | Dynamically Controlled Bitcoin Block Size Max Cap                                     | Upal Chakraborty                                         | Standard      | Draft     |
| [107](bip-0107.mediawiki "wikilink") | Consensus (hard fork) | Dynamic limit on the block size                                                       | Washington Y. Sanchez                                    | Standard      | Draft     |
| [109](bip-0109.mediawiki "wikilink") | Consensus (hard fork) | Two million byte size limit with sigop and sighash limits                             | Gavin Andresen                                           | Standard      | Draft     |
| [111](bip-0111.mediawiki "wikilink") | Peer Services         | NODE\_BLOOM service bit                                                               | Matt Corallo, Peter Todd                                 | Standard      | Accepted  |
| [112](bip-0112.mediawiki "wikilink") | Consensus (soft fork) | CHECKSEQUENCEVERIFY                                                                   | BtcDrak, Mark Friedenbach, Eric Lombrozo                 | Standard      | Final     |
| [113](bip-0113.mediawiki "wikilink") | Consensus (soft fork) | Median time-past as endpoint for lock-time calculations                               | Thomas Kerin, Mark Friedenbach                           | Standard      | Final     |
| [114](bip-0114.mediawiki "wikilink") | Consensus (soft fork) | Merkelized Abstract Syntax Tree                                                       | Johnson Lau                                              | Standard      | Draft     |
| [120](bip-0120.mediawiki "wikilink") | Applications          | Proof of Payment                                                                      | Kalle Rosenbaum                                          | Standard      | Draft     |
| [121](bip-0121.mediawiki "wikilink") | Applications          | Proof of Payment URI scheme                                                           | Kalle Rosenbaum                                          | Standard      | Draft     |
| [122](bip-0122.mediawiki "wikilink") | Applications          | URI scheme for Blockchain references / exploration                                    | Marco Pontello                                           | Standard      | Draft     |
| [123](bip-0123.mediawiki "wikilink") |                       | BIP Classification                                                                    | Eric Lombrozo                                            | Process       | Draft     |
| [124](bip-0124.mediawiki "wikilink") | Applications          | Hierarchical Deterministic Script Templates                                           | Eric Lombrozo, William Swanson                           | Informational | Draft     |
| [125](bip-0125.mediawiki "wikilink") | Applications          | Opt-in Full Replace-by-Fee Signaling                                                  | David A. Harding, Peter Todd                             | Standard      | Accepted  |
| [126](bip-0126.mediawiki "wikilink") |                       | Best Practices for Heterogeneous Input Script Transactions                            | Kristov Atlas                                            | Informational | Draft     |
| [130](bip-0130.mediawiki "wikilink") | Peer Services         | sendheaders message                                                                   | Suhas Daftuar                                            | Standard      | Accepted  |
| [131](bip-0131.mediawiki "wikilink") | Consensus (hard fork) | "Coalescing Transaction" Specification (wildcard inputs)                              | Chris Priest                                             | Standard      | Draft     |
| [132](bip-0132.mediawiki "wikilink") |                       | Committee-based BIP Acceptance Process                                                | Andy Chase                                               | Process       | Withdrawn |
| [133](bip-0133.mediawiki "wikilink") | Peer Services         | feefilter message                                                                     | Alex Morcos                                              | Standard      | Draft     |
| [134](bip-0134.mediawiki "wikilink") | Consensus (hard fork) | Flexible Transactions                                                                 | Tom Zander                                               | Standard      | Draft     |
| [140](bip-0140.mediawiki "wikilink") | Consensus (soft fork) | Normalized TXID                                                                       | Christian Decker                                         | Standard      | Draft     |
| [141](bip-0141.mediawiki "wikilink") | Consensus (soft fork) | Segregated Witness (Consensus layer)                                                  | Eric Lombrozo, Johnson Lau, Pieter Wuille                | Standard      | Draft     |
| [142](bip-0142.mediawiki "wikilink") | Applications          | Address Format for Segregated Witness                                                 | Johnson Lau                                              | Standard      | Deferred  |
| [143](bip-0143.mediawiki "wikilink") | Consensus (soft fork) | Transaction Signature Verification for Version 0 Witness Program                      | Johnson Lau, Pieter Wuille                               | Standard      | Draft     |
| [144](bip-0144.mediawiki "wikilink") | Peer Services         | Segregated Witness (Peer Services)                                                    | Eric Lombrozo, Pieter Wuille                             | Standard      | Draft     |
| [145](bip-0145.mediawiki "wikilink") | API/RPC               | getblocktemplate Updates for Segregated Witness                                       | Luke Dashjr                                              | Standard      | Draft     |
| [146](bip-0146.mediawiki "wikilink") | Consensus (soft fork) | Dealing with signature encoding malleability                                          | Johnson Lau, Pieter Wuille                               | Standard      | Draft     |
| [147](bip-0147.mediawiki "wikilink") | Consensus (soft fork) | Dealing with dummy stack element malleability                                         | Johnson Lau                                              | Standard      | Draft     |
| [150](bip-0150.mediawiki "wikilink") | Peer Services         | Peer Authentication                                                                   | Jonas Schnelli                                           | Standard      | Draft     |
| [151](bip-0151.mediawiki "wikilink") | Peer Services         | Peer-to-Peer Communication Encryption                                                 | Jonas Schnelli                                           | Standard      | Draft     |
| [152](bip-0152.mediawiki "wikilink") | Peer Services         | Compact Block Relay                                                                   | Matt Corallo                                             | Standard      | Draft     |
