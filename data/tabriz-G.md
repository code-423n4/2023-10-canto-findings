## [G-01] Move storage pointer to top of function to avoid offset calculation
We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

```
 if (lastAccrued != 0) {
            AmbientPosition storage pos = lookupPosition(owner, poolIdx);
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L233C8-L234C74


## [G-02] Using storage instead of memory for structs/arrays saves gas
Using a memory pointer for a storage struct/array will effectively load all the fields of that data type from storage (SLOAD) into memory (MSTORE). Using a storage pointer will allow you to read specific fields from storage as you need them. If you are not going to use all of the fields of your data type then you should use a storage pointer so that you don’t incur extra Gcoldsload (2100 gas) for fields that you will never use.

```
    TickTracking memory tickTracking = tickTracking_[poolIdx][i][tickTrackingIndex];
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L95C17-L95C101

```
  CurveMath.CurveState memory curve = curves_[poolIdx];
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L169C7-L169C62

```
    CurveMath.CurveState memory curve = curves_[poolIdx];
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L261C5-L261C62

## [G-03] Arithmatic operations can be unchecked, since there is no underflow or overflow

```
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
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L50C12-L60C14

```
  while (time < block.timestamp && tickTrackingIndex < numTickTracking) {
                    TickTracking memory tickTracking = tickTracking_[poolIdx][i][tickTrackingIndex];
                    uint32 currWeek = uint32((time / WEEK) * WEEK);
                    uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
                    uint32 dt = uint32(
                        nextWeek < block.timestamp
                            ? nextWeek - time
                            : block.timestamp - time
                    );
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L94C15-L102C23

```
 }
                // Percentage of this weeks overall in range liquidity that was provided by the user times the overall weekly rewards
                rewardsToSend += inRangeLiquidityOfPosition * concRewardPerWeek_[poolIdx][week] / overallInRangeLiquidity;
            }
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L186C16-L189C14

```
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
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L207C13-L217C14

```
 if (overallTimeWeightedLiquidity > 0) {
                uint256 rewardsForWeek = (timeWeightedWeeklyPositionAmbLiquidity_[
                    poolIdx
                ][posKey][week] * ambRewardPerWeek_[poolIdx][week]) /
                    overallTimeWeightedLiquidity;
                rewardsToSend += rewardsForWeek;
            }
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L276C12-L282C14


## [G-04] += Costs More Gas Than =+ For State Variables

```
  timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * liquidity;
                time += dt;
            }
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L58C15-L60C14

```
    timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;
                time += dt;
            }
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L215C13-L217C14

## [G-05] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```
 function protocolCmd(bytes calldata cmd) public virtual {
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L26C4-L26C62

```
    function userCmd(bytes calldata input) public payable {
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L41C1-L41C60

```
function claimConcentratedRewards(bytes32 poolIdx, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim)
        public
        payable
    {
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L54C5-L57C6

```
 function claimAmbientRewards(bytes32 poolIdx, uint32[] memory weeksToClaim) public payable {
        claimAmbientRewards(payable(msg.sender), poolIdx, weeksToClaim);
    }

    function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L61C4-L65C115

```
   function setAmbRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L74C2-L74C114

```
 function acceptCrocProxyRole(address, uint16 slot) public pure returns (bool) {
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L85C4-L85C84
