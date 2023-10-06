## Use `memory` instead of `storage`
In the instance below, instead of `storage`, `memory` would save some gas.

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L75

        RangePosition72 storage pos = lookupPosition(
            owner,
            poolIdx,
            lowerTick,
            upperTick
        );
