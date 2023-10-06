## Summary

Total **12 instances** over **3 issues**with **382** gas saved:

|ID|Issue|Instances|Gas|
|:--:|:---|:--:|:--:|
| [[G&#x2011;01]](#g01-use-unchecked-block-for-safe-subtractions) | Use `unchecked` block for safe subtractions | 4 | 340 |
| [[G&#x2011;02]](#g02-do-not-cache-state-variables-that-are-used-only-once) | Do not cache state variables that are used only once | 2 | 6 |
| [[G&#x2011;03]](#g03-using-assembly-to-check-for-zero-can-save-gas) | Using assembly to check for zero can save gas | 6 | 36 |

## Gas Optimizations

### [G&#x2011;01] Use `unchecked` block for safe subtractions


If it can be confirmed that the subtraction operation will not overflow, using an unchecked block can save gas.
For example, `require(x <= y); z = y - x;` can be optimized to `require(x <= y); unchecked { z = y - x; }`.

There are 4 instances:

- *LiquidityMining.sol* ( [#L56](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L56), [#L101](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L101), [#L213](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L213), [#L243](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L243) ):

```solidity
/// @audit checked on line 50
56:                         : block.timestamp - time

/// @audit checked on line 94
101:                             : block.timestamp - time

/// @audit checked on line 207
213:                         : block.timestamp - time

/// @audit checked on line 237
243:                         : block.timestamp - time
```

### [G&#x2011;02] Do not cache state variables that are used only once


It's cheaper to access the state variable directly if it is accessed only once. This can save the 3 gas cost of the extra stack allocation.

There are 2 instances:

- *LiquidityMining.sol* ( [#L169](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L169), [#L261](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L261) ):

```solidity
169:         CurveMath.CurveState memory curve = curves_[poolIdx];

261:         CurveMath.CurveState memory curve = curves_[poolIdx];
```

### [G&#x2011;03] Using assembly to check for zero can save gas


Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

There are 6 instances:

- *LiquidityMining.sol* ( [#L47](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L47), [#L86](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L86), [#L114](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L114), [#L142](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L142), [#L204](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L204), [#L233](https://github.com/code-423n4/2023-10-canto/tree/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L233) ):

```solidity
47:         if (lastAccrued != 0) {

86:         if (lastAccrued != 0) {

114:                         if (tickTracking.exitTimestamp == 0) {

142:                     if (tickTracking_[poolIdx][i][numTickTracking - 1].exitTimestamp == 0) {

204:         if (lastAccrued != 0) {

233:         if (lastAccrued != 0) {
```
