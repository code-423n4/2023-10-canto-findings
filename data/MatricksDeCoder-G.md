### GAS-1 Cache multiple mapping accesses in storage pointer 

There are several instances in same function where mappings are reread/reaccessed see below in 
 
```
function crossTicks(
        bytes32 poolIdx,
        int24 exitTick,
        int24 entryTick
    ) internal {
        uint256 numElementsExit = tickTracking_[poolIdx][exitTick].length; //1 
        tickTracking_[poolIdx][exitTick][numElementsExit - 1] //2
            .exitTimestamp = uint32(block.timestamp);
        StorageLayout.TickTracking memory tickTrackingData = StorageLayout
            .TickTracking(uint32(block.timestamp), 0);
        tickTracking_[poolIdx][entryTick].push(tickTrackingData); //3
    }

```
tickTracking_[poolIdx] or tickTracking_[poolIdx][exitTick] can be saved in a storage  pointer 

```
function accrueConcentratedGlobalTimeWeightedLiquidity(
        bytes32 poolIdx,
        CurveMath.CurveState memory curve
    ) internal {
        uint32 lastAccrued = timeWeightedWeeklyGlobalConcLiquidityLastSet_[
            poolIdx
        ]; //1
        // Only set time on first call
        if (lastAccrued != 0) {
            uint256 liquidity = curve.concLiq_;
            uint32 time = lastAccrued;
            while (time < block.timestamp) {
                uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
                uint32 dt = uint32(
                    nextWeek < block.timestamp
                        ? nextWeek - time
                        : block.timestamp - time
                );
                timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * liquidity;
                time += dt;
            }
        }
        timeWeightedWeeklyGlobalConcLiquidityLastSet_[poolIdx] = uint32(
            block.timestamp
        ); //2
    }
```
timeWeightedWeeklyGlobalConcLiquidityLastSet_[poolIdx] can be saved in a storage pointer 

```
function accrueConcentratedPositionTimeWeightedLiquidity(
        ....
        uint32 lastAccrued = timeWeightedWeeklyPositionConcLiquidityLastSet_[
            poolIdx
        ][posKey]; //1  line 82 
        ......
        uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i]; // 1 line 89 
        ....
        tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = tickTrackingIndex; // 2 line 135 
        ....
        tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking - 1; // 3 line 144
        ....
        tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking; //4 line 146
        timeWeightedWeeklyPositionConcLiquidityLastSet_[poolIdx][
            posKey
        ] = uint32(block.timestamp); //2 line 151 
    }
```
timeWeightedWeeklyPositionConcLiquidityLastSet_[poolIdx][posKey] can have a storage pointer created 
tickTrackingIndexAccruedUpTo_[poolIdx][posKey] used many times can be saved in storage pointer 

The above are just examples but functions in LiquidityMining.sol e.g timeWeightedWeeklyPositionAmbLiquidityLastSet_[
            poolIdx
        ][posKey] in function  accrueAmbientGlobalTimeWeightedLiquidity; and may other 
instances in which mappings are not cached or saved in storage pointer in cases where they are used several times in other remaining functions. 

**Impact** -> Caching a mapping's value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. 

**Recommendation** -> Check all functions and Save  a storage pointer for the mapping variable and use it instead of repeatedly fetching the reference in a map

### GAS-2 Avoid code duplication to especially reduce code size 

There are functions with duplicated code parts that can have those parts placed in separate functions or have the functions combined into a single function. 

he following functions share common code canto_ambient/contracts/callpaths/LiquidityMiningPath.sol
```
function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
       // require(msg.sender == governance_, "Only callable by governance");
       require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
       while (weekFrom <= weekTo) {
           concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
           weekFrom += uint32(WEEK);
       }
   }

   function setAmbRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
       // require(msg.sender == governance_, "Only callable by governance");
       require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
       while (weekFrom <= weekTo) {
           ambRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
           weekFrom += uint32(WEEK);
       }
   }
```
We can see the same update for weeks and same require in functions. An example improvement is below
```
   function setRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward, bool Amb) public payable {
       // require(msg.sender == governance_, "Only callable by governance");
       require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
       while (weekFrom <= weekTo) {
           if(Amb) {ambRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;}
           else {concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;}
           weekFrom += uint32(WEEK);
       }
   }
```

Additional functions sharing coming code are
```
function accrueConcentratedGlobalTimeWeightedLiquidity(
function accrueAmbientGlobalTimeWeightedLiquidity(
\\ in the parts below that are almost similar
if (lastAccrued != 0) {
           uint256 liquidity = curve.ambientSeeds_;
           uint32 time = lastAccrued;
           while (time < block.timestamp) {
               uint32 currWeek = uint32((time / WEEK) * WEEK);
               uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
               uint32 dt = uint32(
                   nextWeek < block.timestamp
                       ? nextWeek - time
                       : block.timestamp - time
               );
               timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;
               time += dt;
           }
       }
       timeWeightedWeeklyGlobalAmbLiquidityLastSet_[poolIdx] = uint32(
           block.timestamp
       );
```
Consider the appropriate refactors that work best in order to avoid code duplication, reuse, bloat, repetition etc
```
