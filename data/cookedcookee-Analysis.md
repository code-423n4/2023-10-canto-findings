### Analysis

Overall both `LiquidityMiningPath.sol` and `LiquidityMining.sol` are well written, efficient, and logical contracts. The logic employed in the `accrueConcentratedPositionTimeWeightedLiquidity()` function, in particular, is clean and very thoroughly considered given the complexity involved. The way in which the novel contracts integrate into the pre-existing architecture was impressive and very well done also. This speaks to both the quality of the novel contracts and the pre-existing contracts.

With all of that being said, one dimension that the current design fails to properly consider is how to handle unclaimed and unclaimable liquidity mining rewards. Unclaimed liquidity mining rewards are locked in the DEX contract, and there are conditions under which some percentage of the liquidity mining rewards are unclaimable by anyone. More details regarding these issues can be found in the accompanying submitted reports. It is recognised that novel sidecar contracts and/or upgrades can be built to allow governance actors to retrieve the unclaimed and unclaimable funds, however, a simpler and more thorough approach is to build this functionality into the liquidity mining contracts initially and directly.

Additionally, many variables of the current design are hard-coded into the contracts. Examples include the `WEEK` constant, and the tick range of plus or minus 10 ticks in `LiquidityMining,sol`. While taking this approach has the clear benefit of saving gas and reducing code complexity, it also limits the utility of these contracts to be used in other circumstances. For instance, if a wider minimum range of in range liquidity is desired a fork changing the relevant values will need to be created, deployed, and integrated into the DEX. Depending on the intended scope and utility of these contracts, it may prove beneficial to add protocol command functions to allow governance to change some or all of these hard-coded variables.

Finally, implicit in much of this discussion, and the overall design of the Ambient DEX architecture, is a high degree of centralisation and trust in the governance structure. While this is acknowledged and mitigated by the [governance team](https://docs.ambient.finance/governance-and-policy/policy), it is worth emphasising that users must trust the designers and maintainers of this code. Governance can, for instance, reset or remove the liquidity rewards after the liquidity period has expired, but before the rewards have been claimed. This could be used to exploit liquidity providers, and it is important for users to be cognizant of this risk.

The auditing approach I employed involved a detailed manual review, followed by targeted hardhat testing of key functionality.

Thank you for your time!

### Time spent:
15 hours