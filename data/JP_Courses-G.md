0. GAS: LiquidityMining::accrueConcentratedPositionTimeWeightedLiquidity() - L81: calls method `encodePosKey()` twice during this main function call(via `lookupPosition()` too), chowing unnecessary gas.

https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/mixins/LiquidityMining.sol#L81

Within a single main function call the `encodePosKey()` method is executed twice to generate the exact same hash `posKey`. Consider modifying the implementation so that this method can be called only once but its result used twice. It should reduce gas consumption during function execution quite noticably.

This is executed twice during `accrueConcentratedPositionTimeWeightedLiquidity()` call:
`encodePosKey(owner, poolIdx, lowerTick, upperTick);`

1. GAS: LiquidityMining::claimConcentratedRewards - L174: use unchecked block for `++i` at end of `for` loop.

https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/mixins/LiquidityMining.sol#L174

Current:
```solidity
for (uint256 i; i < weeksToClaim.length; ++i) {
```

Suggested:
```solidity
for (uint256 i; i < weeksToClaim.length;) {
    // for loop logic
    {
        i++;
    }
}
```

2. GAS: LiquidityMining::claimConcentratedRewards() - L182: suggest to use `!= 0` instead of `> 0`.

https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/mixins/LiquidityMining.sol#L182

Current:
```solidity
if (overallInRangeLiquidity > 0) {
```

Suggested:
```solidity
if (overallInRangeLiquidity != 0) {
```

3. GAS: LiquidityMining::claimConcentratedRewards - L192: suggest to use `!= 0` instead of `> 0`.

https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/mixins/LiquidityMining.sol#L192

Current:
```solidity
if (rewardsToSend > 0) {
```

Suggested:
```solidity
if (rewardsToSend != 0) {
```


