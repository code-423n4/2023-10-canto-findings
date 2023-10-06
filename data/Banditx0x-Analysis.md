## Contextualising my Findings

### A Nuance in Ambient Liquidity Causes Incorrect Reward Accounting - My Favourite Finding

The CANTO scope was short in terms of lines of code but large in terms of background knowledge. The ambient finance contracts are not well known by auditors, as mentioned by the sponsor. When I read into the documentation I noticed a key fact:

_"The first advantage is that instead of accumulating fees in a separate side-pocket, fees accumulated by ambient LP positions auto-compound back into the original position without any manual management"_

The CANTO rewards system is designed to accrue interest before every change in liquidity amount, such as before minting and burning functions. However, due to fees compounding and increasing the liquidity position the invariant that the liquidity amount between different accrual periods is broken. This is further elucidated in my issue:

_"Fee Compounding is Over-rewarded in Ambient Liquidity Positions"_

### Gas Problems

There are problems with the gas consumption of the contract. The calls to functions like `crossTick` when a swap is a cost which affects every swap. This cost is imposed every time the price impact causes a price tick is crossed. With a price tick of only 1 bps, this will happen extremely frequently. The first consequence is that the gas for every swap over a liquidity pool is increased slightly. The second consequence is explained in one of my issues:

_"Array Length of `tickTracking_ ` Can be Purposely Increased to Brick Minting and Burning of Most Users' Liquidity Positions"_

This issue concerns the fact that every time a tick is entered, the array for `tickTrading_` increases by one. By repeatedly making small swaps back and forth across a tick, an attacker can cause the array for a particular tick to grow large. This array is iterated over every time other users call mint and burn, and making the array very large will cause these critical functions to revert due to out of gas.

The second seperate issue with gas concerns the iteration over many ticks for "wide" liquidity positions. The program actually loops through every tick in a liquidity position, except the first 10 and last 10. There can be over 160000 ticks in a liquidity position! Iterating over a loop so many times will cause an out of gas error which is my second gas issue:

_"Gas Limit Exceeded Due to Iterrating Through Almost Every Tick"_

### Accounting Issues

There were 2 accounting issues:

_"Calculation For Overall Liquidity Includes Ineligible Positions and Does Not Correspond With the Calculation For Individual Liquidity Positions"_

This concerns the `overallLiquidity` tracking including ineligible positions when only eligible positions should be counted towards the global liquidity. 

The next issue:

_"Concentrated Pool Rewards are Proportional to The Entire Liquidity Amount For Any Eligible Positions, Even if That Liquidity Is Out of The Reward Range"_

This concerns the entire liquidity amount being used for reward calculation rather than just tick based liquidity or in-reward-range liquidity.

## Architechtural considerations:

**Accounting**

Here are three suggested changes to the accounting architecture. This is to address some of the problems raised in the issue submissions.

1. Only eligible liquidity positions should be included for the globalLiquidity used for rewards distribution. 

2. The global liquidity calculation should be adjusted to ensure that the `global eligible liquidity` is greater or equal to `the sum of all liquidity positions`. Otherwise the rewards allocated would exceed the weekly allocated rewards, leading to reward insolvency.

3. Rewards for individual positions should be adjusted for tick size. `pos.liquidity` should be divided by `upperTick - lowerTick`

**Gas**

There is a larger overall design issue which is the decision to do on-chain liquidity mining by adding calls to reward accounting functions to every swap, mint and burn for a liquidity pool. This is extremely gas intensive and could affect the normal functioning of the protocol. It may even cause important functions to revert. 

One simple solution can be done by off-chain rewarding. Perpetual Protocol, which is a concentrated liquidity Perpetual AMM built on-top of Uniswap v3, used off-chain calculations for their liquidity mining program: [LINK].(https://perpprotocol.mirror.xyz/0C9AYnB93z5AZ4c-phO8Wg7jXRFYLt3e9TiZ2VH3DLw) This has huge centralisation downsides, which may go against CATNO's ethos but is the simplest gas solution. Obviously, this is not a smart contract solution so may automatically be unacceptable.

The gas problems may have to be addressed through a complete architectural redesign rather than just normal gas optimisation. Here are some suggestions: 

One immediate step would to `pop()` tickTrackingData as soon as the exitTimestamp == entryTimestamp. This happens to the last element of the array when `crossTicks` is called. Tracking this is unnecessary, because the tick was active for 0 blocks, and therefore the time delta and hence allocated rewards is zero.

The documentation stated that CANTO rewards are meant to be distributed for stable pools for this codebase. The term "stable" could have different interpretations, but this reccomendation assumes that this refers to stablecoin-like or pegged asset pairs such as stETH/WETH, USDT/USDC etc.

Instead of iterating through every tick, one could assume a range where the stable assets could lie and then reward all positions that lie within the specified range - in this case +- 10 ticks of the price tick. 

This makes an assumption that these "stable assets" will actually stay pegged to each other. However, the current accounting architecture has multiple problems:

- Given the high number of loops required by the current accounting mechanism, there are multiple reasons that gas could run out. This includes iterating through too many ticks or having too many tick entries/exits

- The current mechanism increases the gas costs of all minting, burning and harvesting

- DOS attacks like the one described in this issue are possible.

Assuming a stable price has the downside of misallocating rewards if the stable assets depeg from each other.







### Time spent:
25 hours