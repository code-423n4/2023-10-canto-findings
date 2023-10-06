https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L161


Since it's reading the inputted parameter

```
uint32[] calldata weeksToClaim
``` 
Using `calldata` to read from the inputted parameter saves gas cost