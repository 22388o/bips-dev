+++
title = "Dynamically Controlled Bitcoin Block Size Max Cap"
date = 2015-08-24
weight = 106
in_search_index = true

[taxonomies]
authors = ["Upal Chakraborty"]
status = ["Rejected"]

[extra]
bip = 106
status = ["Rejected"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0106.mediawiki"
+++

``` 
  BIP: 106
  Layer: Consensus (hard fork)
  Title: Dynamically Controlled Bitcoin Block Size Max Cap
  Author: Upal Chakraborty <bitcoin@upalc.com>
  Comments-Summary: No comments yet.
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0106
  Status: Rejected
  Type: Standards Track
  Created: 2015-08-24
```

## Abstract

This BIP proposes replacing the fixed one megabyte maximum block size
with a dynamically controlled maximum block size that may increase or
decrease with difficulty change depending on various network factors. I
have two proposals regarding this...

i. Depending only on previous block size calculation.

ii. Depending on previous block size calculation and previous Tx fee
collected by miners.

## Motivation

With increased adoption, transaction volume on bitcoin network is bound
to grow. If the one megabyte max cap is not changed to a flexible one
which changes itself with changing network demand, then adoption will
hamper and bitcoin's growth may choke up. Following graph shows the
change in average block size since inception...

<https://blockchain.info/charts/avg-block-size?timespan=all&showDataPoints=false&daysAverageString=1&show_header=true&scale=0&address=>

## Specification

### Proposal 1 : Depending only on previous block size calculation

` If more than 50% of block's size, found in the first 2000 of the last difficulty period, is more than 90% MaxBlockSize`  
`     Double MaxBlockSize`  
` Else if more than 90% of block's size, found in the first 2000 of the last difficulty period, is less than 50% MaxBlockSize`  
`     Half MaxBlockSize`  
` Else`  
`     Keep the same MaxBlockSize`

### Proposal 2 : Depending on previous block size calculation and previous Tx fee collected by miners

` TotalBlockSizeInLastButOneDifficulty = Sum of all Block size of first 2008 blocks in last 2 difficulty period`  
` TotalBlockSizeInLastDifficulty = Sum of all Block size of second 2008 blocks in last 2 difficulty period (This actually includes 8 blocks from last but one difficulty)`  
` `  
` TotalTxFeeInLastButOneDifficulty = Sum of all Tx fees of first 2008 blocks in last 2 difficulty period`  
` TotalTxFeeInLastDifficulty = Sum of all Tx fees of second 2008 blocks in last 2 difficulty period (This actually includes 8 blocks from last but one difficulty)`  
` `  
` If ( ( (Sum of first 4016 block size in last 2 difficulty period)/4016 > 50% MaxBlockSize) AND (TotalTxFeeInLastDifficulty > TotalTxFeeInLastButOneDifficulty) AND (TotalBlockSizeInLastDifficulty > TotalBlockSizeInLastButOneDifficulty) )`  
`     MaxBlockSize = TotalBlockSizeInLastDifficulty * MaxBlockSize / TotalBlockSizeInLastButOneDifficulty`  
` Else If ( ( (Sum of first 4016 block size in last 2 difficulty period)/4016 < 50% MaxBlockSize) AND (TotalTxFeeInLastDifficulty < TotalTxFeeInLastButOneDifficulty) AND (TotalBlockSizeInLastDifficulty < TotalBlockSizeInLastButOneDifficulty) )`  
`     MaxBlockSize = TotalBlockSizeInLastDifficulty * MaxBlockSize / TotalBlockSizeInLastButOneDifficulty`  
` Else`  
`     Keep the same MaxBlockSize`

## Rationale

These two proposals have been derived after discussion on
[BitcoinTalk](https://bitcointalk.org/index.php?topic=1154536.0) and
[bitcoin-dev mailing
list](http://lists.linuxfoundation.org/pipermail/bitcoin-dev/2015-August/010285.html).
The original idea and its evolution in the light of various arguments
can be found [here](http://upalc.com/maxblocksize.php).

### Proposal 1 : Depending only on previous block size calculation

This solution is derived directly from the indication of the problem. If
transaction volume increases, then we will naturally see bigger blocks.
On the contrary, if there are not enough transaction volume, but maximum
block size is high, then only few blocks may sweep the mempool. Hence,
if block size is itself taken into consideration, then maximum block
size can most rationally be derived. Moreover, this solution not only
increases, but also decreases the maximum block size, just like
difficulty.

### Proposal 2 : Depending on previous block size calculation and previous Tx fee collected by miners

This solution takes care of stable mining subsidy. It will not increase
maximum block size, if Tx fee collection is not increasing and thereby
creating a Tx fee pressure on the market. On the other hand, though the
block size max cap is dynamically controlled, it is very difficult to
game by any party because the increase or decrease of block size max cap
will take place in the same ratio of average block size increase or
decrease.

## Compatibility

This is a hard-forking change to the Bitcoin protocol; anybody running
code that fully validates blocks must upgrade before the activation time
or they will risk rejecting a chain containing larger-than-one-megabyte
blocks.

## Other solutions considered

[Making Decentralized Economic
Policy](http://gtf.org/garzik/bitcoin/BIP100-blocksizechangeproposal.pdf)
- by Jeff Garzik

[Elastic block cap with rollover
penalties](https://bitcointalk.org/index.php?topic=1078521.0) - by Meni
Rosenfeld

[Increase maximum block
size](https://github.com/bitcoin/bips/blob/master/bip-0101.mediawiki) -
by Gavin Andresen

[Block size following technological
growth](https://gist.github.com/sipa/c65665fc360ca7a176a6) - by Pieter
Wuille

[The Bitcoin Lightning Network: Scalable Off-Chain Instant
Payments](https://lightning.network/lightning-network-paper.pdf) - by
Joseph Poon & Thaddeus Dryja

## Deployment

If consensus is achieved, deployment can be made at a future block
number at which difficulty will change.
