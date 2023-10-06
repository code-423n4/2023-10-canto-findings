## Summary

### Gas Optimizations


|ID|Title|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [GAS-01](#gas-01-avoid-unnecessary-recomputations)| Avoid unnecessary recomputations | 3 | 9 |


Total: 3 instances over 1 issues with 9 gas saved

## Gas Optimizations

### [GAS-01] Avoid unnecessary recomputations
In the following loops `upperTick - 10` is computed on every iteration of the loop. Since the value doesn't change it could have been saved to a variable and then reused. 

*There are 3 instances of this issue*

```solidity
File: canto_ambient/contracts/mixins/LiquidityMining.sol

88:                    for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) { 


139:                    for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) { 


184:                        for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j) { 

```

*GitHub*: [88](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L88-#L88), [139](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L139-#L139), [184](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L184-#L184)