## [L-01] no input validation in accrueConcentratedPositionTimeWeightedLiquidity

In the accrueConcentratedPositionTimeWeightedLiquidity function of the contract, there is a no input validation for the lowerTick and upperTick parameters. These parameters are used to specify the tick range, and there are no checks to ensure that lowerTick is less than or equal to upperTick. This oversight could lead to unexpected behavior or errors if lowerTick is greater than upperTick.

```solidity

156  function claimConcentratedRewards(
157        address payable owner,
158        bytes32 poolIdx,
159        int24 lowerTick,
160        int24 upperTick,
161        uint32[] memory weeksToClaim
162    ) internal {

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L156C1-L162C17

To prevent potential issues related to the tick range, you should add input validation to ensure that lowerTick is less than or equal to upperTick

## [L-02] no input Validation in claimConcentratedRewards

In the claimConcentratedRewards function takes an array of weeks to claim rewards for and processes them without validating the input.

```solidity

161        uint32[] memory weeksToClaim

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L161

While you check that the claimed week is not in the future and has not been claimed before, there is no validation to ensure that the weeks provided in the weeksToClaim array are in ascending order or that there are no duplicate entries. This can lead to unexpected behavior or errors if the input array contains out-of-sequence or duplicate weeks.

## [L-03] Redundant Comment in crossTicks Function

In the crossTicks function, there is a redundant comment that does not provide additional information beyond what is already apparent from the function name and code.

The comment, "@notice Keeps track of the tick crossings," essentially repeats the function name's purpose, which is to "crossTicks." Since the code and function name are self-explanatory, this comment does not provide additional value and can be considered redundant.

```solidity
22    /// @notice Keeps track of the tick crossings

```
