### Report 1:
tickTrackingIndexAccruedUpTo_ mapping used in [L89](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L89), [L135](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L135), [L144](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L144) & [L146](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L146) has no mapping description.
Its mapping description should be added to L192 @ 
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/StorageLayout.sol#L192
```solidity
+++ //Pool -> Position -> Tick -> TrackingIndex 
    mapping(bytes32 => mapping(bytes32 => mapping(int32 => uint32))) tickTrackingIndexAccruedUpTo_;
```
###  Report 2:
timeWeightedWeeklyPositionConcLiquidityLastSet_[poolIdx][posKey] at [L151](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L151) of LiquidityMining.sol contract can be updated to block.timestamp even if difference between lowerTick & UpperTick from [L72-L73](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L72-L73) of same contract is less than 20. A deep analysis of the accrueConcentratedPositionTimeWeightedLiquidity(...) function and it use case at [163](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L163) shows that the whole function setup is not relevant if " UpperTick - lowerTick is less than 20 " due to the subtraction operations in the loop at [L88](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L88), [L139](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L139) & [L184](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L184).
Therefore a validation check is necessary if "UpperTick - lowerTick is less than 20" instead of just giving room for timeWeightedWeeklyPositionConcLiquidityLastSet_[poolIdx][posKey]  update when it is not 
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L151
```solidity
 function accrueConcentratedPositionTimeWeightedLiquidity(
        address payable owner,
        bytes32 poolIdx,
        int24 lowerTick,
        int24 upperTick
    ) internal {
+++ require ( UpperTick - lowerTick < 20, "InvalidTickRange" )
...
```