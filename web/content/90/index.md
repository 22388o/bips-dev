+++
title = "Buried Deployments"
date = 2016-11-08
weight = 90
in_search_index = true

[taxonomies]
authors = ["Suhas Daftuar"]
status = ["Final"]

[extra]
bip = 90
status = ["Final"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0090.mediawiki"
+++

``` 
  BIP: 90
  Title: Buried Deployments
  Author: Suhas Daftuar <sdaftuar@chaincode.com>
  Comments-Summary: Mostly Recommended for implementation, with some Discouragement
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0090
  Status: Final
  Type: Informational
  Created: 2016-11-08
  License: PD
```

## Abstract

Prior soft forks (BIP 34, BIP 65, and BIP 66) were activated via miner
signaling in block version numbers. Now that the chain has long since
passed the blocks at which those consensus rules have triggered, we can
(as a simplification) replace the trigger mechanism by caching the block
heights at which those consensus rules became enforced.

## Motivation

BIPs 34, 65 and 66 were deployed on mainnet using miner signaling using
block version numbers. In short, new consensus rules were proposed for
use in blocks with a higher version number (N+1) than the prevailing
block version (N) in use on the network, and those rules became enforced
under the following conditions:

1.  75% rule: If 750 of the prior 1000 blocks are version N+1 or higher,
    then blocks with version N+1 or higher must correctly enforce the
    new consensus rule.
2.  95% rule: If 950 of the prior 1000 blocks are version N+1 or higher,
    then blocks with version less than N+1 are invalid.

Please see those [BIPs](#References "wikilink") for more details.

Note that this trigger mechanism is dependent on the chain history. To
validate a block, we must test whether the trigger was met by looking at
the previous 1000 blocks in the chain before it, which can be
inefficient.

In addition, this mechanism for code deployments have been deprecated in
favor of BIP 9 deployments, which offer several advantages (please see
[BIP 9](https://github.com/bitcoin/bips/blob/master/bip-0009.mediawiki)).

Thus we propose elimination of the logic implementing these kinds of
deployments, by replacing the test which governs enforcement of BIP 34,
BIP 65, and BIP 66 with simple height checks, which we choose to be the
block height triggering the 95% activation rule on mainnet for each of
those deployments. This simplification of the consensus rules would
reduce the technical debt associated with deployment of those consensus
changes.

## Considerations

It is technically possible for this to be a non-backwards compatible
change. For example, if an alternate chain were created in which BIP
34's 95% activation triggered at a lower height (H') than it did on the
current mainnet chain (H), then older software would enforce that
version 1 blocks were invalid at heights between H' and H, while newer
software implementing this change would not. Similarly, this BIP
proposes doing away with the 75% threshold check altogether, which
means, for example, that a version 2 block forking off of mainnet at
height H-1 which omitted the height in coinbase would be invalid to
older software, while accepted by newer software.

However, while newer software and older software might validate old
blocks differently, that could only cause a consensus split if there
were an extremely large blockchain reorganization onto a chain built off
such a block. As of November 2016, the most recent of these changes (BIP
65, enforced since December 2015) has nearly 50,000 blocks built on top
of it. The occurrence of such a reorg that would cause the activating
block to be disconnected would raise fundamental concerns about the
security assumptions of Bitcoin, a far bigger issue than any
non-backwards compatible change.

So while this proposal could <i>theoretically</i> result in a consensus
split, it is extremely unlikely, and in particular any such
circumstances would be sufficiently damaging to the Bitcoin network to
dwarf any concerns about the effects of this proposed change.

## Specification

The BIP 34, 66, and 65 activation heights are set to 227931, 363725, and
388381, respectively.

The 1000-block lookback test, first described in BIP 34, is no longer
performed during validation of any blocks. Instead, a new check is
added:

`   if((block.nVersion < 2 && nHeight >= consensusParams.BIP34Height) ||`  
`      (block.nVersion < 3 && nHeight >= consensusParams.BIP66Height) ||`  
`      (block.nVersion < 4 && nHeight >= consensusParams.BIP65Height))`  
`           return state.Invalid(false, REJECT_OBSOLETE, strprintf("bad-version(0x%08x)", block.nVersion),`  
`                                strprintf("rejected nVersion=0x%08x block", block.nVersion));`

Furthermore, rather than consider the block versions of the prior 1000
blocks to determine whether to enforce BIP 34, BIP 65, or BIP 66 on a
given block, we instead just compare the height of the block being
validated with the stored activation heights:

`   // Enforce rule that the coinbase starts with serialized block height`  
`   if (nHeight >= consensusParams.BIP34Height)`  
`   {`  
`       CScript expect = CScript() << nHeight;`  
`       if (block.vtx[0].vin[0].scriptSig.size() < expect.size() ||`  
`           !std::equal(expect.begin(), expect.end(), block.vtx[0].vin[0].scriptSig.begin())) {`  
`           return state.DoS(100, false, REJECT_INVALID, "bad-cb-height", false, "block height mismatch in coinbase");`  
`       }`  
`   }`

and

`   // Start enforcing the DERSIG (BIP66) rule`  
`   if (pindex->nHeight >= chainparams.GetConsensus().BIP66Height) {`  
`       flags |= SCRIPT_VERIFY_DERSIG;`  
`   }`

`   // Start enforcing CHECKLOCKTIMEVERIFY (BIP65) rule`  
`   if (pindex->nHeight >= chainparams.GetConsensus().BIP65Height) {`  
`       flags |= SCRIPT_VERIFY_CHECKLOCKTIMEVERIFY;`  
`   }`

Please see the implementation for additional details.

## Implementation

<https://github.com/bitcoin/bitcoin/pull/8391>.

## References

[BIP34 Block v2, Height in
Coinbase](https://github.com/bitcoin/bips/blob/master/bip-0034.mediawiki)

[BIP66 Strict DER
signatures](https://github.com/bitcoin/bips/blob/master/bip-0066.mediawiki)

[BIP65
OP\_CHECKLOCKTIMEVERIFY](https://github.com/bitcoin/bips/blob/master/bip-0065.mediawiki)

[BIP9 Version bits with timeout and
delay](https://github.com/bitcoin/bips/blob/master/bip-0009.mediawiki)

## Acknowledgements

Thanks to Nicolas Dorier for drafting an initial version of this BIP,
and to Alex Morcos, Matt Corallo, and Greg Maxwell for suggestions and
feedback.

## Copyright

This document is placed in the public domain.
