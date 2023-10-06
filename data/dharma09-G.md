## Summary

- [No need to save `liquidity` it’s just waste of gas](#)
- [Make Variable Outside For loop for better gas optimization](#)
    - [Declared **`time`** outside of the loop for gas efficiency](#)
    - [Declared `week` outside of the loop for gas efficiency](#)
    - [Moved the calculation of **`overallInRangeLiquidity`** outside the loop to reduce storage reads](#)
    - [Declared **`currWeek`**, **`nextWeek`**, and **`dt`** outside of the loop for improved gas efficiency and reduce storage reads](#)
- [With assembly, .call (bool sent) transfer can be done gas-optimized](#)
- [Use `Do while` loops instead of `for` loops](#)
- [Gas savings to avoid casting in the loop](#)

### No need to save `liquidity` it’s just waste of gas

Under `pos.liquidity_` `liquidity` is saved and used in one place, there is no need for such variable and you can insert the function in the math logic.

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L235
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L87

```diff
-87: uint256 liquidity = pos.liquidity_;
...
-129: timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +=
-130:                            (tickActiveEnd - tickActiveStart) * liquidity;
+129: timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +=
+130:                            (tickActiveEnd - tickActiveStart) * pos.liquidity_;
```

### Make Variable Outside For loop for better gas optimization

Declared **`currWeek`**, **`nextWeek`**, and **`dt`** outside of the loop for improved gas efficiency and reduce storage reads.

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L238C13-L244C19

```solidity
File: canto_ambient/contracts/mixins/LiquidityMining.sol
224: function accrueAmbientPositionTimeWeightedLiquidity(
        address payable owner,
        bytes32 poolIdx
    ) internal {
        bytes32 posKey = encodePosKey(owner, poolIdx);
        uint32 lastAccrued = timeWeightedWeeklyPositionAmbLiquidityLastSet_[
            poolIdx
        ][posKey];
        // Only init time on first call
        if (lastAccrued != 0) {
            AmbientPosition storage pos = lookupPosition(owner, poolIdx);
            uint256 liquidity = pos.seeds_;
            uint32 time = lastAccrued;
            while (time < block.timestamp) {
238:                uint32 currWeek = uint32((time / WEEK) * WEEK); //@audit
239:                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
240:                uint32 dt = uint32(
241:                    nextWeek < block.timestamp
242:                        ? nextWeek - time
243:                        : block.timestamp - time
244:                );
                timeWeightedWeeklyPositionAmbLiquidity_[poolIdx][posKey][
                    currWeek
                ] += dt * liquidity;
                time += dt;
            }
        }
251:        timeWeightedWeeklyPositionAmbLiquidityLastSet_[poolIdx][
252:            posKey
253:        ] = uint32(block.timestamp);
254:    }
```

```diff
File: canto_ambient/contracts/mixins/LiquidityMining.sol
224: function accrueAmbientPositionTimeWeightedLiquidity(
        address payable owner,
        bytes32 poolIdx
    ) internal {
        bytes32 posKey = encodePosKey(owner, poolIdx);
        uint32 lastAccrued = timeWeightedWeeklyPositionAmbLiquidityLastSet_[
            poolIdx
        ][posKey];
        // Only init time on first call
        if (lastAccrued != 0) {
            AmbientPosition storage pos = lookupPosition(owner, poolIdx);
            uint256 liquidity = pos.seeds_;
            uint32 time = lastAccrued;
+           uint32 currWeek;
+           uint32 nextWeek;
+           uint32 dt;
            while (time < block.timestamp) {
-238:                uint32 currWeek = uint32((time / WEEK) * WEEK); //@audit
-239:                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
-240:                uint32 dt = uint32(
-241:                    nextWeek < block.timestamp
-242:                        ? nextWeek - time
-243:                        : block.timestamp - time
-244:                );
+238:            currWeek = uint32((time / WEEK) * WEEK);
+239:            nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
+240:            dt = uint32(nextWeek < block.timestamp ? nextWeek - time : block.timestamp - time);
                timeWeightedWeeklyPositionAmbLiquidity_[poolIdx][posKey][
                    currWeek
                ] += dt * liquidity;
                time += dt;
            }
        }
251:        timeWeightedWeeklyPositionAmbLiquidityLastSet_[poolIdx][
252:            posKey
253:        ] = uint32(block.timestamp);
254:    }
```

Declared **`time`** outside of the loop for gas efficiency.

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L92

```solidity
File: canto_ambient/contracts/mixins/LiquidityMining.sol
92: uint32 time = lastAccrued;
```

Declared `week` outside of the loop for gas efficiency

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L267

```solidity
File: canto_ambient/contracts/mixins/LiquidityMining.sol
267: uint32 week = weeksToClaim[i]; //@audit
```

Moved the calculation of **`overallInRangeLiquidity`** outside the loop to reduce storage reads.

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L181

```solidity
File: canto_ambient/contracts/mixins/LiquidityMining.sol
181: uint256 overallInRangeLiquidity = timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][week]; //@audit
```

Moved the calculation of `overallTimeWeightedLiquidity` outside the loop to reduce storage reads.

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L273

```solidity
File: canto_ambient/contracts/mixins/LiquidityMining.sol
273: uint256 overallTimeWeightedLiquidity = timeWeightedWeeklyGlobalAmbLiquidity_[ //@audit Moved the calculation of overallTimeWeightedLiquidity outside the loop to reduce storage reads.
274:                    poolIdx
275:                ][week];
```

### With assembly, .call (bool sent) transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided. 

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L286
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L193

```solidity
(bool sent,) = dest.call{value:amount}(""); bool success; assembly {
sent := call(gas(), dest, amount, 0, 0) }
```

### Use `Do while` loops instead of `for` loops

A `do while` loop costs less gas in comparison to a `for` loop as the condition is not checked for the first iteration in a `do while`.

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L184C11-L186C18

```solidity
File: canto_ambient/contracts/mixins/LiquidityMining.sol
182: if (overallInRangeLiquidity > 0) {
183:                uint256 inRangeLiquidityOfPosition;
184:                for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j) {
185:                    inRangeLiquidityOfPosition += timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];
186:                }
187:                // Percentage of this weeks overall in range liquidity that was provided by the user times the overall weekly rewards
188:                rewardsToSend += inRangeLiquidityOfPosition * concRewardPerWeek_[poolIdx][week] / overallInRangeLiquidity;
189:            }
```

### Gas savings to avoid casting in the loop

Improve gas spending by reducing unnecessary casting

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L88

```diff
-88: for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
+88: for (int256 i = lowerTick + 10; i <= upperTick - 10; ++i) {
-139: for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
+139: for (int256 i = lowerTick + 10; i <= upperTick - 10; ++i) {
-184: for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j) {
+184: for (int256 j = lowerTick + 10; j <= upperTick - 10; ++j) {

```