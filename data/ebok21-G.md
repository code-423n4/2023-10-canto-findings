## Using += is gas extensive

Suggestion to use a = a+b as it is 10 gas cheaper per operation as compared to a += b.
One down side is that it might sacrifice code readability.

Instances of inefficiency in code-
```solidity
58:    timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * liquidity;
59:    time += dt;
```
https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L58C17-L59C28

```solidity
129:    timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] += (tickActiveEnd - tickActiveStart) * liquidity;

132:    time += dt;
```
https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L129

```solidity
185:    inRangeLiquidityOfPosition += timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];

188:    rewardsToSend += inRangeLiquidityOfPosition * concRewardPerWeek_[poolIdx][week] / overallInRangeLiquidity;
```
https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L185C21-L188C123

```solidity
215:    timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;
216:    time += dt;
```
https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L215C1-L216C28

```solidity
247:    ] += dt * liquidity;
248:    time += dt;
```
https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L247C1-L248C28

```solidity
281:    rewardsToSend += rewardsForWeek;
```
https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L281C17-L281C49