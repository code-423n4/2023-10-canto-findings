# Missing event and or timelock for critical parameter change
Events help non-contract tools to track changes, and timelocks prevent users from being surprised by changes
There are 1 instances of this issue:
```solidity
File: contracts/mixins/LiquidityCurve.sol

    function commitCurve (bytes32 poolIdx, CurveMath.CurveState memory curve)
        internal {
        curves_[poolIdx] = curve;
    }
```

## Recommendation
```diff
function commitCurve (bytes32 poolIdx, CurveMath.CurveState memory curve)
        internal {
        curves_[poolIdx] = curve;
+       Curves(poolIdx, block.timestamp, curve);
    }
```

