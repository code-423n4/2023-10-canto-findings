Caching `weeksToClaim.length` leads to less gas used:

```diff
L173:         uint256 rewardsToSend;
L174:
+             uint32 weeksToClaimLength = weeksToClaim.length
L175:
-             for (uint256 i; i < weeksToClaim.length; ++i) {
+             for (uint256 i; i < weeksToClaimLength; ++i) {
```

Affected code:
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L174