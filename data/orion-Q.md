# Lines of code

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L130


# Vulnerability details

## Impact
LiquidityMining.accrueConcentratedPositionTimeWeightedLiquidity may unintentionally reverts and make transactions does not succeed

## Proof of Concept
The LiquidityMining.accrueConcentratedPositionTimeWeightedLiquidity function calculates the concentrated position liquidity for specific pool, the logic however under some special conditions may reverts unintentionally via the EVM under/overflow protection, the function at some point subtracts `tickActiveStart` from `tickActiveEnd` at line 160

```
(tickActiveEnd - tickActiveStart) * liquidity;
```

For this situation to be met, the following conditions should met :


1/ Line 105 : tickTracking.enterTimestamp < nextWeek (nextWeek = uint32(((time + WEEK) / WEEK) * WEEK)) 
2/ Line 107 : tickTracking.enterTimestamp < time , here the `tickActiveStart` variant will be : tickActiveStart = time;
3/ Line 114 : then (tickTracking.exitTimestamp > 0) will meet not met, will then be passed to the else condition at line 117 which later be forwarded to the condition (tickTracking.exitTimestamp < nextWeek), when this won't met the `tickActiveEnd` variable will be `tickActiveEnd = nextWeek;` as `nextWeek` here is  `uint32(((time + WEEK) / WEEK) * WEEK);`

Under this situation the `tickActiveEnd` variant is designed to be actually lower than tickActiveStart, but eventually the `(tickActiveEnd - tickActiveStart) * liquidity;` condition will become so more like :

```
uint32(((time + WEEK) / WEEK) * WEEK) - (time) * liquidity;
```

That condition will underflows thus the  LiquidityMining.accrueConcentratedPositionTimeWeightedLiquidity
will reverts 

Due to the shortage of time I wasn't be able to make a clear PoC, to make a work around for this situation to meet

## Tools Used
Manual review

## Recommended Mitigation Steps
In lines 123, and 130 check if `tickActiveEnd > tickActiveStart` then make the substraction, otherwise return 1 or 0 for the timeWeightedWeeklyPositionInRangeConcLiquidity_ definition


## Assessed type

Error