# Loss of precision

Multiplication should always be performed before division to avoid loss of precision.

https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L96C21-L97C77

https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L208C17-L209C73



# Impact 

Because of `nextWeek` effects  time-weighted concentrated liquidity , this could cause of miscalculation.

# Recommended Mitigation Steps

```solidity
-- uint32 currWeek = uint32((time / WEEK) * WEEK);
-- uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

++ uint32 currWeek = uint32((time * WEEK) / WEEK);
++ uint32 nextWeek = uint32(((time + WEEK) * WEEK) / WEEK);
```


