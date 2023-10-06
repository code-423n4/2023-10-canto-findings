## G1
### Save array length in memory to prevent repetitive checks in a loop.
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L266
``` solidity
 unit weekLength = weeksToClaim.length;
 for (uint256 i; i < weekLength; ++i) {
```