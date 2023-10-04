# Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol?plain=1#L174
```
        for (uint256 i; i < weeksToClaim.length; ++i) {
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol?plain=1#L266
```
        for (uint256 i; i < weeksToClaim.length; ++i) {
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol?plain=1#L299
```
        for (uint i = 0; i < dirs.length; ++i) {
```

## Recommended Mitigation Steps
```solidity
uint256 len = weeksToClaim.length
for (uint256 i; i < len; ++i) {
   ...
}
```