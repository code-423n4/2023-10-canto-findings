## State variables that are used multiple times in a function should be cached in stack variables
Week state variable should be cached in function LiquidityMining.accrueConcentratedGlobalTimeWeightedLiquidity ,LiquidityMining.accrueConcentratedPositionTimeWeightedLiquidity ,LiquidityMining.accrueAmbientGlobalTimeWeightedLiquidity ,LiquidityMining.accrueAmbientPositionTimeWeightedLiquidity (save 2000 gas 21 sload)

```
#L51-L52
uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
#L96-L97
 uint32 currWeek = uint32((time / WEEK) * WEEK);
                    uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
#L208-L209
 uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
#L238-L239
uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
```
LiquidityMiningPath.setConcRewards and LiquidityMiningPath.setAmbRewards

```
#L67-L68
 require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
            weekFrom += uint32(WEEK);
        }

#L76-L77
require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            ambRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
            weekFrom += uint32(WEEK);
        }


```


