# Gas Report

**Gas Findings List**

| Number | Issue Detail | Severity |
|-----:|----|-----|
| [`G-01`](#G-01-use-assembly-to-abidecode) | Use `Assembly` to `abi.decode` | Gas |
| [`G-02`](#G-02-removing-internal-functions-that-are-not-called-within-the-contract-helps-save-gas-during-deployment) | Removing Internal functions that are not called within the contract helps save gas during deployment | Gas |
| [`G-03`](#G-02-with-assembly-call-bool-success-transfer-can-be-done-gas-optimized) | With assembly, `.call (bool success)` transfer can be done `gas-optimized` | Gas |
| [`G-04`](#G-02-duplicated-require-if-checks-should-be-refactored-to-a-modifier-or-function) | Duplicated `require()/if()` checks should be refactored to a `modifier` or function | Gas |
| [`G-05`](#G-02-x-y-costs-more-gas-than-x-x-y-for-state-variables) | `x += y` costs more gas than `x = x + y` for `state` variables | Gas |
| [`G-06`](#G-02-replace-state-variable-reads-and-writes-within-loops-with-local-variable-reads-and-writes) | Replace `state` variable `reads` and `writes` within loops with `local` variable `reads` and `writes` | Gas |
| [`G-07`](#G-02-use-assembly-for-math-add-sub-mul-div) | Use assembly for `math (add, sub, mul, div)` | Gas |

> Total ~ 7 Issues

## G-01 Use `Assembly` to `abi.decode`
### Summary 
Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.
```solidity
File : LiquidityMiningPath.sol

27:        (uint8 code, bytes32 poolHash, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) =
28:           abi.decode(cmd, (uint8, bytes32, uint32, uint32, uint64));

42:        (uint8 code, bytes32 poolHash, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim) =
43:            abi.decode(input, (uint8, bytes32, int24, int24, uint32[]));
```
## G-02 Removing Internal functions that are not called within the contract helps save gas during deployment.
```solidity:
File: LiquidityMining.sol
16:    function initTickTracking(bytes32 poolIdx, int24 tick) internal {

24:    function crossTicks(
25:        bytes32 poolIdx,
26:        int24 exitTick,
27:       int24 entryTick
28:    ) internal {
```
## G-03 With assembly `.call (bool success)` transfer can be done `gas-optimized`
### Summary 
Return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided. (bool success,) = dest.call{value:amount}(""); bool success; assembly {success := call(gas(), dest, amount, 0, 0) }
```solidity
File: LiquidityMining.sol
193:            (bool sent, ) = owner.call{value: rewardsToSend}("");
194:            require(sent, "Sending rewards failed");

286:           (bool sent, ) = owner.call{value: rewardsToSend}("");
287:            require(sent, "Sending rewards failed");
```
## G-04  Duplicated `require()/if()` checks should be refactored to a `modifier` or function
### Summary 
It is common to use require() or if() statements to validate certain conditions before executing specific code. However, when the same checks are repeated multiple times within a contract, it can result in redundant code and unnecessary gas consumption. To save gas and improve code readability and maintainability, it is recommended to refactor duplicated checks into modifiers or functions. By doing so, the checks can be abstracted into reusable code blocks that can be applied to multiple functions within the contract.
```solidity
File: LiquidityMiningPath.sol
67:        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

76:        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
```
## G-05 `x += y` costs more gas than `x = x + y` for `state` variables
### Summary 
```solidity
File: LiquidityMining.sol
58:                 timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * liquidity;

129:                        timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +=
130:                           (tickActiveEnd - tickActiveStart) * liquidity;

215:                 timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;

245:                timeWeightedWeeklyPositionAmbLiquidity_[poolIdx][posKey][
246:                  currWeek
247:                ] += dt * liquidity;
```
## G-06 Replace `state` variable `reads` and `writes` within loops with `local` variable `reads` and `writes`
### Summary 
When accessing state variables within loops, each read or write operation incurs additional gas costs due to the storage and memory access operations involved. These costs can accumulate quickly, particularly in loops with a large number of iterations.
```solidity
File: LiquidityMining.sol

187:                // Percentage of this weeks overall in range liquidity that was provided by the user times the overall weekly rewards
188:                rewardsToSend += inRangeLiquidityOfPosition * concRewardPerWeek_[poolIdx][week] / overallInRangeLiquidity;
189:            }

277:              uint256 rewardsForWeek = (timeWeightedWeeklyPositionAmbLiquidity_[
278:                   poolIdx
279:                ][posKey][week] * ambRewardPerWeek_[poolIdx][week]) /
280:                   overallTimeWeightedLiquidity;
```
## G-07 Use assembly for `math (add, sub, mul, div)`
### Summary 
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.
```solidity
 File: LiquidityMining.sol
51:                   uint32 currWeek = uint32((time / WEEK) * WEEK);
 
52:                   uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
 
53                    uint32 dt = uint32(
54                        nextWeek < block.timestamp
55                            ? nextWeek - time
56                            : block.timestamp - time
57:                   );
 
 
96:                       uint32 currWeek = uint32((time / WEEK) * WEEK);
 
97:                       uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

98                        uint32 dt = uint32(
99                            nextWeek < block.timestamp
100                               ? nextWeek - time
101                               : block.timestamp - time
102:                      );
 


208:                  uint32 currWeek = uint32((time / WEEK) * WEEK);
 
209:                  uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
 
210                   uint32 dt = uint32(
211                       nextWeek < block.timestamp
212                           ? nextWeek - time
213                           : block.timestamp - time
214:                  );

 
238:                  uint32 currWeek = uint32((time / WEEK) * WEEK);
 
239:                  uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
 
240                   uint32 dt = uint32(
241                       nextWeek < block.timestamp
242                           ? nextWeek - time
243                           : block.timestamp - time
244:                  );
```
