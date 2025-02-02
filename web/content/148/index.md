+++
title = "Mandatory activation of segwit deployment"
date = 2017-03-12
weight = 148
in_search_index = true

[taxonomies]
authors = ["Shaolin Fry"]
status = ["Final"]

[extra]
bip = 148
status = ["Final"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0148.mediawiki"
+++

``` 
  BIP: 148
  Layer: Consensus (soft fork)
  Title: Mandatory activation of segwit deployment
  Author: Shaolin Fry <shaolinfry@protonmail.ch>
  Comments-Summary: No comments yet.
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0148
  Status: Final
  Type: Standards Track
  Created: 2017-03-12
  License: BSD-3-Clause
           CC0-1.0
```

## Abstract

This document specifies a BIP16 like soft fork flag day activation of
the segregated witness BIP9 deployment known as "segwit".

## Definitions

"existing segwit deployment" refer to the BIP9 "segwit" deployment using
bit 1, between November 15th 2016 and November 15th 2017 to activate
BIP141, BIP143 and BIP147.

## Motivation

Segwit increases the blocksize, fixes transaction malleability, and
makes scripting easier to upgrade as well as bringing many other
[benefits](https://bitcoincore.org/en/2016/01/26/segwit-benefits/).

It is hoped that miners will respond to this BIP by activating segwit
early, before this BIP takes effect. Otherwise this BIP will cause the
mandatory activation of the existing segwit deployment before the end of
midnight November 15th 2017.

## Specification

All times are specified according to median past time.

This BIP will be active between midnight August 1st 2017 (epoch time
1501545600) and midnight November 15th 2017 (epoch time 1510704000) if
the existing segwit deployment is not locked-in or activated before
epoch time 1501545600. This BIP will cease to be active when segwit is
locked-in.

While this BIP is active, all blocks must set the nVersion header top 3
bits to 001 together with bit field (1\<\<1) (according to the existing
segwit deployment). Blocks that do not signal as required will be
rejected.

### Reference implementation

    // Check if Segregated Witness is Locked In
    bool IsWitnessLockedIn(const CBlockIndex* pindexPrev, const Consensus::Params& params)
    {
        LOCK(cs_main);
        return (VersionBitsState(pindexPrev, params, Consensus::DEPLOYMENT_SEGWIT, versionbitscache) == THRESHOLD_LOCKED_IN);
    }
    
    // BIP148 mandatory segwit signalling.
    int64_t nMedianTimePast = pindex->GetMedianTimePast();
    if ( (nMedianTimePast >= 1501545600) &&  // Tue 01 Aug 2017 00:00:00 UTC
         (nMedianTimePast <= 1510704000) &&  // Wed 15 Nov 2017 00:00:00 UTC
         (!IsWitnessLockedIn(pindex->pprev, chainparams.GetConsensus()) &&  // Segwit is not locked in
          !IsWitnessEnabled(pindex->pprev, chainparams.GetConsensus())) )   // and is not active.
    {
        bool fVersionBits = (pindex->nVersion & VERSIONBITS_TOP_MASK) == VERSIONBITS_TOP_BITS;
        bool fSegbit = (pindex->nVersion & VersionBitsMask(chainparams.GetConsensus(), Consensus::DEPLOYMENT_SEGWIT)) != 0;
        if (!(fVersionBits && fSegbit)) {
            return state.DoS(0, error("ConnectBlock(): relayed block must signal for segwit, please upgrade"), REJECT_INVALID, "bad-no-segwit");
        }
    }

<https://github.com/bitcoin/bitcoin/compare/master>...shaolinfry:bip-segwit-flagday

## Backwards Compatibility

This deployment is compatible with the existing "segwit" bit 1
deployment scheduled between midnight November 15th, 2016 and midnight
November 15th, 2017.

## Rationale

Historically, the P2SH soft fork (BIP16) was activated using a
predetermined flag day where nodes began enforcing the new rules. P2SH
was successfully activated with relatively few issues

By orphaning non-signalling blocks during the last month of the BIP9 bit
1 "segwit" deployment, this BIP can cause the existing "segwit"
deployment to activate without needing to release a new deployment.

## References

  - [Mailing list
    discussion](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2017-March/013714.html)
  - [P2SH flag day
    activation](https://github.com/bitcoin/bitcoin/blob/v0.6.0/src/main.cpp#L1281-L1283)
  - [BIP9 Version bits with timeout and
    delay](bip-0009.mediawiki "wikilink")
  - [BIP16 Pay to Script Hash](bip-0016.mediawiki "wikilink")
  - [BIP141 Segregated Witness (Consensus
    layer)](bip-0141.mediawiki "wikilink")
  - [BIP143 Transaction Signature Verification for Version 0 Witness
    Program](bip-0143.mediawiki "wikilink")
  - [BIP147 Dealing with dummy stack element
    malleability](bip-0147.mediawiki "wikilink")
  - [Segwit
    benefits](https://bitcoincore.org/en/2016/01/26/segwit-benefits/)

## Copyright

This document is dual licensed as BSD 3-clause, and Creative Commons CC0
1.0 Universal.
