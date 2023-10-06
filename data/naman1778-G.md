## [G-01] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

    File: canto_ambient/contracts/mixins/LiquidityMining.sol	

    29: uint256 numElementsExit = tickTracking_[poolIdx][exitTick].length;

    48: uint256 liquidity = curve.concLiq_;

    87: uint256 liquidity = pos.liquidity_;

    90: uint32 origIndex = tickTrackingIndex;

    169: CurveMath.CurveState memory curve = curves_[poolIdx];

    205: uint256 liquidity = curve.ambientSeeds_;

    235: uint256 liquidity = pos.seeds_;

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol

    diff --git a/canto_ambient/contracts/mixins/LiquidityMining.sol b/canto_ambient/contracts/mixins/LiquidityMining.sol
    index a83c273..b8559f3 100644
    --- a/canto_ambient/contracts/mixins/LiquidityMining.sol
    +++ b/canto_ambient/contracts/mixins/LiquidityMining.sol
    @@ -26,8 +26,7 @@ contract LiquidityMining is PositionRegistrar {
             int24 exitTick,
             int24 entryTick
         ) internal {
    -        uint256 numElementsExit = tickTracking_[poolIdx][exitTick].length;
    -        tickTracking_[poolIdx][exitTick][numElementsExit - 1]
    +        tickTracking_[poolIdx][exitTick][tickTracking_[poolIdx][exitTick].length - 1]
                 .exitTimestamp = uint32(block.timestamp);
             StorageLayout.TickTracking memory tickTrackingData = StorageLayout
                 .TickTracking(uint32(block.timestamp), 0);
    @@ -45,7 +44,6 @@ contract LiquidityMining is PositionRegistrar {
             ];
             // Only set time on first call
             if (lastAccrued != 0) {
    -            uint256 liquidity = curve.concLiq_;
                 uint32 time = lastAccrued;
                 while (time < block.timestamp) {
                     uint32 currWeek = uint32((time / WEEK) * WEEK);
    @@ -55,7 +53,7 @@ contract LiquidityMining is PositionRegistrar {
                             ? nextWeek - time
                             : block.timestamp - time
                     );
    -                timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * liquidity;
    +                timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * curve.concLiq_;
                     time += dt;
                 }
             }
    @@ -84,10 +82,8 @@ contract LiquidityMining is PositionRegistrar {
             ][posKey];
             // Only set time on first call
             if (lastAccrued != 0) {
    -            uint256 liquidity = pos.liquidity_;
                 for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
                     uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i];
    -                uint32 origIndex = tickTrackingIndex;
                     uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
                     uint32 time = lastAccrued;
                     // Loop through all in-range time spans for the tick or up to the current time (if it is still in range)
    @@ -127,11 +123,11 @@ contract LiquidityMining is PositionRegistrar {
                                 }
                             }
                             timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +=
    -                            (tickActiveEnd - tickActiveStart) * liquidity;
    +                            (tickActiveEnd - tickActiveStart) * pos.liquidity_;
                         }
                         time += dt;
                     }
    -                if (tickTrackingIndex != origIndex) {
    +                if (tickTrackingIndex != tickTrackingIndex) {
                         tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = tickTrackingIndex;
                     }
                 }
    @@ -166,9 +162,8 @@ contract LiquidityMining is PositionRegistrar {
                 lowerTick,
                 upperTick
             );
    -        CurveMath.CurveState memory curve = curves_[poolIdx];
             // Need to do a global accrual in case the current tick was already in range for a long time without any modifications that triggered an accrual
    -        accrueConcentratedGlobalTimeWeightedLiquidity(poolIdx, curve);
    +        accrueConcentratedGlobalTimeWeightedLiquidity(poolIdx, curves_[poolIdx]);
             bytes32 posKey = encodePosKey(owner, poolIdx, lowerTick, upperTick);
             uint256 rewardsToSend;
             for (uint256 i; i < weeksToClaim.length; ++i) {
    @@ -202,7 +197,6 @@ contract LiquidityMining is PositionRegistrar {
             uint32 lastAccrued = timeWeightedWeeklyGlobalAmbLiquidityLastSet_[poolIdx];
             // Only set time on first call
             if (lastAccrued != 0) {
    -            uint256 liquidity = curve.ambientSeeds_;
                 uint32 time = lastAccrued;
                 while (time < block.timestamp) {
                     uint32 currWeek = uint32((time / WEEK) * WEEK);
    @@ -212,7 +206,7 @@ contract LiquidityMining is PositionRegistrar {
                             ? nextWeek - time
                             : block.timestamp - time
                     );
    -                timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;
    +                timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * curve.ambientSeeds_;
                     time += dt;
                 }
             }
    @@ -232,7 +226,7 @@ contract LiquidityMining is PositionRegistrar {
             // Only init time on first call
             if (lastAccrued != 0) {
                 AmbientPosition storage pos = lookupPosition(owner, poolIdx);
    -            uint256 liquidity = pos.seeds_;
    +            uint256 liquidity = ;
                 uint32 time = lastAccrued;
                 while (time < block.timestamp) {
                     uint32 currWeek = uint32((time / WEEK) * WEEK);
    @@ -244,7 +238,7 @@ contract LiquidityMining is PositionRegistrar {
                     );
                     timeWeightedWeeklyPositionAmbLiquidity_[poolIdx][posKey][
                         currWeek
    -                ] += dt * liquidity;
    +                ] += dt * pos.seeds_;
                     time += dt;
                 }
             }

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }

    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |


## [G-02] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8). Subtructions act the same way.

    File: canto_ambient/contracts/callpaths/LiquidityMiningPath.sol	

    70: weekFrom += uint32(WEEK);

    79: weekFrom += uint32(WEEK);

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol

    diff --git a/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol b/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol
    index e6c63f7..3ef5508 100644
    --- a/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol
    +++ b/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol
    @@ -67,7 +67,7 @@ contract LiquidityMiningPath is LiquidityMining {
             require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
             while (weekFrom <= weekTo) {
                 concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
    -            weekFrom += uint32(WEEK);
    +            weekFrom = weekFrom + uint32(WEEK);
             }
         }

    @@ -76,7 +76,7 @@ contract LiquidityMiningPath is LiquidityMining {
             require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
             while (weekFrom <= weekTo) {
                 ambRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
    -            weekFrom += uint32(WEEK);
    +            weekFrom = weekFrom + uint32(WEEK);
             }
         }

    File: canto_ambient/contracts/mixins/LiquidityMining.sol	

    58: timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * liquidity;

    59: time += dt;

    129: timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +=
    130:     (tickActiveEnd - tickActiveStart) * liquidity;
    132: time += dt;

    185: inRangeLiquidityOfPosition += timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];

    188: rewardsToSend += inRangeLiquidityOfPosition * concRewardPerWeek_[poolIdx][week] / overallInRangeLiquidity;

    215: timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;
    
    216: time += dt;

    245: timeWeightedWeeklyPositionAmbLiquidity_[poolIdx][posKey][
    246:     currWeek
    247: ] += dt * liquidity;

    248: time += dt;

    281: rewardsToSend += rewardsForWeek;

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol

    diff --git a/canto_ambient/contracts/mixins/LiquidityMining.sol b/canto_ambient/contracts/mixins/LiquidityMining.sol
    index a83c273..caf1f35 100644
    --- a/canto_ambient/contracts/mixins/LiquidityMining.sol
    +++ b/canto_ambient/contracts/mixins/LiquidityMining.sol
    @@ -55,8 +55,8 @@ contract LiquidityMining is PositionRegistrar {
                             ? nextWeek - time
                             : block.timestamp - time
                     );
    -                timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * liquidity;
    -                time += dt;
    +                timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] = timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] + dt * liquidity;
    +                time = time + dt;
                 }
             }
             timeWeightedWeeklyGlobalConcLiquidityLastSet_[poolIdx] = uint32(
    @@ -126,10 +126,10 @@ contract LiquidityMining is PositionRegistrar {
                                     tickActiveEnd = nextWeek;
                                 }
                             }
    -                        timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +=
    +                        timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] = timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +
                                 (tickActiveEnd - tickActiveStart) * liquidity;
                         }
    -                    time += dt;
    +                    time = time + dt;
                     }
                     if (tickTrackingIndex != origIndex) {
                         tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = tickTrackingIndex;
    @@ -182,10 +182,10 @@ contract LiquidityMining is PositionRegistrar {
                 if (overallInRangeLiquidity > 0) {
                     uint256 inRangeLiquidityOfPosition;
                     for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j) {
    -                    inRangeLiquidityOfPosition += timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];
    +                    inRangeLiquidityOfPosition = inRangeLiquidityOfPosition + timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];
                     }
                     // Percentage of this weeks overall in range liquidity that was provided by the user times the overall weekly rewards
    -                rewardsToSend += inRangeLiquidityOfPosition * concRewardPerWeek_[poolIdx][week] / overallInRangeLiquidity;
    +                rewardsToSend = rewardsToSend + inRangeLiquidityOfPosition * concRewardPerWeek_[poolIdx][week] / overallInRangeLiquidity;
                 }
                 concLiquidityRewardsClaimed_[poolIdx][posKey][week] = true;
             }
    @@ -212,8 +212,8 @@ contract LiquidityMining is PositionRegistrar {
                             ? nextWeek - time
                             : block.timestamp - time
                     );
    -                timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;
    -                time += dt;
    +                timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] = timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] + dt * liquidity;
    +                time = time + dt;
                 }
             }
             timeWeightedWeeklyGlobalAmbLiquidityLastSet_[poolIdx] = uint32(
    @@ -242,10 +242,10 @@ contract LiquidityMining is PositionRegistrar {
                             ? nextWeek - time
                             : block.timestamp - time
                     );
    -                timeWeightedWeeklyPositionAmbLiquidity_[poolIdx][posKey][
    +                timeWeightedWeeklyPositionAmbLiquidity_[poolIdx][posKey][currWeek] = timeWeightedWeeklyPositionAmbLiquidity_[poolIdx][posKey][
                         currWeek
    -                ] += dt * liquidity;
    -                time += dt;
    +                ] + (dt * liquidity);
    +                time = time + dt;
                 }
             }
             timeWeightedWeeklyPositionAmbLiquidityLastSet_[poolIdx][
    @@ -278,7 +278,7 @@ contract LiquidityMining is PositionRegistrar {
                         poolIdx
                     ][posKey][week] * ambRewardPerWeek_[poolIdx][week]) /
                         overallTimeWeightedLiquidity;
    -                rewardsToSend += rewardsForWeek;
    +                rewardsToSend = rewardsToSend + rewardsForWeek;
                 }
                 ambLiquidityRewardsClaimed_[poolIdx][posKey][week] = true;
             }

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.add();
            c1.add1();
        }
    }

    contract Contract0 {

        uint8 num1 = 1;

        function add() public{
            num1 += 1;
        }

    }

    contract Contract1 {

        uint8 num1 = 1;

        function add1() public{
            num1 = num1 + 1;
        }

    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 67017                                     | 268             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add                                       | 5405            | 5405 | 5405   | 5405 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 70623                                     | 286             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add1                                      | 5363            | 5363 | 5363   | 5363 | 1       |

## [G-03] Instead of counting down in *for* statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8). Subtructions act the same way.

    File: canto_ambient/contracts/mixins/LiquidityMining.sol	

    88: for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {

    139: for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {

    174: for (uint256 i; i < weeksToClaim.length; ++i) {

    184: for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j) {

    266: for (uint256 i; i < weeksToClaim.length; ++i) {

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol

    diff --git a/canto_ambient/contracts/mixins/LiquidityMining.sol b/canto_ambient/contracts/mixins/LiquidityMining.sol
    index a83c273..d14f75e 100644
    --- a/canto_ambient/contracts/mixins/LiquidityMining.sol
    +++ b/canto_ambient/contracts/mixins/LiquidityMining.sol
    @@ -85,7 +85,7 @@ contract LiquidityMining is PositionRegistrar {
             // Only set time on first call
             if (lastAccrued != 0) {
                 uint256 liquidity = pos.liquidity_;
    -            for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
    +            for (int24 i = upperTick - 10 ; i >= lowerTick + 10 ; --i) {
                     uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i];
                     uint32 origIndex = tickTrackingIndex;
                     uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
    @@ -136,7 +136,7 @@ contract LiquidityMining is PositionRegistrar {
                     }
                 }
             } else {
    -            for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
    +            for (int24 i = upperTick - 10 ; i >= lowerTick + 10 ; --i) {
                     uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
                     if (numTickTracking > 0) {
                         if (tickTracking_[poolIdx][i][numTickTracking - 1].exitTimestamp == 0) {
    @@ -171,7 +171,7 @@ contract LiquidityMining is PositionRegistrar {
             accrueConcentratedGlobalTimeWeightedLiquidity(poolIdx, curve);
             bytes32 posKey = encodePosKey(owner, poolIdx, lowerTick, upperTick);
             uint256 rewardsToSend;
    -        for (uint256 i; i < weeksToClaim.length; ++i) {
    +        for (uint256 i = weeksToClaim.length - 1; i >= 0 ; --i) {
                 uint32 week = weeksToClaim[i];
                 require(week + WEEK < block.timestamp, "Week not over yet");
                 require(
    @@ -181,7 +181,7 @@ contract LiquidityMining is PositionRegistrar {
                 uint256 overallInRangeLiquidity = timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][week];
                 if (overallInRangeLiquidity > 0) {
                     uint256 inRangeLiquidityOfPosition;
    -                for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j) {
    +                for (int24 j = upperTick - 10 ; j >= lowerTick + 10 ; --j) {
                         inRangeLiquidityOfPosition += timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];
                     }
                     // Percentage of this weeks overall in range liquidity that was provided by the user times the overall weekly rewards
    @@ -263,7 +263,7 @@ contract LiquidityMining is PositionRegistrar {
             accrueAmbientGlobalTimeWeightedLiquidity(poolIdx, curve);
             bytes32 posKey = encodePosKey(owner, poolIdx);
             uint256 rewardsToSend;
    -        for (uint256 i; i < weeksToClaim.length; ++i) {
    +        for (uint256 i = weeksToClaim.length - 1; i >= 0  ; --i) {
                 uint32 week = weeksToClaim[i];
                 require(week + WEEK < block.timestamp, "Week not over yet");
                 require(

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }


    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }


    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

