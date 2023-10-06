## Would be better to check if weeks in `weeksToClaim` is divisible by `WEEK` in `claimConcentratedRewards()` and `claimAmbientRewards()`

In LiquidityMining, `timeWeightedWeeklyPositionAmbLiquidity_` and `timeWeightedWeeklyPositionInRangeConcLiquidity_` are indexed by multiple of `WEEK`.
But in `claimConcentratedRewards()` and `claimAmbientRewards()`, the checking multiple of `WEEK` is missing.

```
    function claimConcentratedRewards(
        address payable owner,
        bytes32 poolIdx,
        int24 lowerTick,
        int24 upperTick,
        uint32[] memory weeksToClaim
    ) internal {
        ... ...
        for (uint256 i; i < weeksToClaim.length; ++i) {
            uint32 week = weeksToClaim[i];
            require(week + WEEK < block.timestamp, "Week not over yet");
+           require(week % WEEK == 0, "Invalid week"); 
            ... ...
        }
        ... ...
    }
```

## Can add lower and upper tick validation in concentrated liquidity functions in LiquidityMining
Checking like `require(upperTick >= 21 + lowerTick)` would prevent unnecessary operations in all concentrated liquidity functions in LiquidityMining.
```
    function claimConcentratedRewards(
        address payable owner,
        bytes32 poolIdx,
        int24 lowerTick,
        int24 upperTick,
        uint32[] memory weeksToClaim
    ) internal {
+       require(upperTick >= 21 + lowerTick, "Invalid Range");
        ... ...
    }
```