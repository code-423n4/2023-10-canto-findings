# LOW FINDING

## [L‑01] Loss of precision

Multiplication should occur before division to avoid loss of precision

```solidity
File : contracts/mixins/LiquidityMining.sol

51: uint32 currWeek = uint32((time / WEEK) * WEEK);
52: uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

96: uint32 currWeek = uint32((time / WEEK) * WEEK);
97: uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

208: uint32 currWeek = uint32((time / WEEK) * WEEK);
209: uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

238: uint32 currWeek = uint32((time / WEEK) * WEEK);
239: uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

```
[51-52](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L51C17-L52C73), [96-97](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L96C21-L97C77), [208-209](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L208C1-L209C73) [238-239](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L238C17-L239C73)

# NON CRITICAL

## [N-01] Start internal function name with underscore

```solidity

16: function initTickTracking(bytes32 poolIdx, int24 tick) internal {

24: function crossTicks(
25:        bytes32 poolIdx,
26:        int24 exitTick,
27:        int24 entryTick
28:    ) internal {

39: function accrueConcentratedGlobalTimeWeightedLiquidity(
40:        bytes32 poolIdx,
41:        CurveMath.CurveState memory curve
42:    ) internal {

69: function accrueConcentratedPositionTimeWeightedLiquidity(
70:        address payable owner,
71:        bytes32 poolIdx,
72:        int24 lowerTick,
73:        int24 upperTick
74:    ) internal {

156: function claimConcentratedRewards(
157:        address payable owner,
158:        bytes32 poolIdx,
159:        int24 lowerTick,
160:        int24 upperTick,
161:        uint32[] memory weeksToClaim
162:    ) internal {

198: function accrueAmbientGlobalTimeWeightedLiquidity(
199:        bytes32 poolIdx,
200:        CurveMath.CurveState memory curve
201:    ) internal {

224:  function accrueAmbientPositionTimeWeightedLiquidity(
225:        address payable owner,
226:        bytes32 poolIdx
227:    ) internal {

256: function claimAmbientRewards(
257:        address owner,
258:        bytes32 poolIdx,
259:        uint32[] memory weeksToClaim
260:    ) internal {

```
[16](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L16), [24-28](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L24C5-L28C17), [39-42](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L39C5-L42C17), [69-74](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L69C5-L74C17), [156-162](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L156C5-L162C17), [198-201](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L198C5-L201C17), [224-227](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L224C4-L227C17), [256-260](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L256C5-L260C17)