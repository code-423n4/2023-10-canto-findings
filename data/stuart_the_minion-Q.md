## Would be better to check if weeks in `weeksToClaim` is divisible by `WEEK` in `claimConcentratedRewards()` and `claimAmbientRewards()`

In LiquidityMining, `timeWeightedWeeklyPositionAmbLiquidity_` and `timeWeightedWeeklyPositionInRangeConcLiquidity_` are indexed by multiple of `WEEK`.
But in `claimConcentratedRewards()` and `claimAmbientRewards()`, the checking multiple of `WEEK` is missing.


## Can add lower and upper tick validation in concentrated liquidity functions in LiquidityMining
Checking like `require(upperTick >= 21 + lowerTick)` would prevent unnecessary operations in all concentrated liquidity functions in LiquidityMining.