## Gas Optimizations

## Sumary

|   No | ISSUE  | INSTANCE  |
|------|--------|-----------|
|[G-01]|Use uint256(1)/uint256(2) instead for true and false boolean states|3
|[G-02]|+= costs more gas than = + for state variables|10
|[G-03]|Counting down in for statements is more gas efficient|1
|[G-04]|Use do while loops instead of for loops|6
|[G-05]|Not using the named return variable when a function returns, wastes deployment gas|1
|[G-06]|Public Functions To External|7
|[G-07]|Use nested if statements instead of &&|5
|[G-08]|Use != 0 instead of > 0 for unsigned integer comparison|7
|[G-09]|Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)|1
|[G-10]|Use assembly to perform efficient back-to-back calls|2
|[G-11]|Tightly pack storage variables/optimize the order of variable declaration|6
|[G-12]|Using a positive conditional flow to save a NOT opcode|3
|[G-13]|Using a positive conditional flow to save a NOT opcode|3
|[G-14]|Empty blocks should be removed or emit something|1


## [G-1]  Use uint256(1)/uint256(2) instead for true and false boolean states
If you don’t use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.


```solidity

file: mixins/LiquidityMining.sol

190   concLiquidityRewardsClaimed_[poolIdx][posKey][week] = true;

283 ambLiquidityRewardsClaimed_[poolIdx][posKey][week] = true;
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L190
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L283

```solidity

file: mixins/TradeMatcher.sol

394   bool doMore = true;
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L394

## [G-2]  += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: callpaths/LiquidityMiningPath.sol

70 weekFrom += uint32(WEEK);

79   weekFrom += uint32(WEEK);
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L70
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L79

```solidity

file: mixins/LiquidityMining.sol#

58  timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * liquidity;
59                time += dt;

129   timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +=

132   time += dt;

185  inRangeLiquidityOfPosition += timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];

188  rewardsToSend += inRangeLiquidityOfPosition * concRewardPerWeek_[poolIdx][week] / overallInRangeLiquidity;

215    timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;
216                time += dt;

247    += dt * liquidity;
248                time += dt;

281 rewardsToSend += rewardsForWeek;
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L58-L59 
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L129
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L132
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L185
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L188
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L215-L216 
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L247-248
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L281

## [G-3] Counting down in for statements is more gas efficient
Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

```solidity

file: mixins/MarketSequencer.sol

299 for (uint i = 0; i < dirs.length; ++i) 

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L299

## [G-4] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity

file: mixins/LiquidityMining.sol

88 or (int24 i = lowerTick + 10; i <= upperTick - 10; ++i)

139  for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i)

174  for (uint256 i; i < weeksToClaim.length; ++i)

184 for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j)

266    for (uint256 i; i < weeksToClaim.length; ++i)
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L88
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L139
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L174
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L184
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L266

```solidity

file: mixins/MarketSequencer.sol

299  for (uint i = 0; i < dirs.length; ++i) 

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L299

## [G-5] Not using the named return variable when a function returns, wastes deployment gas

```solidity

file: mixins/TradeMatcher.sol

500   return bumpTick;
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L500

## [G-6] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: callpaths/LiquidityMiningPath.sol

26   function protocolCmd(bytes calldata cmd) public virtual

41  function userCmd(bytes calldata input) public payable

54   function claimConcentratedRewards(bytes32 poolIdx, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim)
        public
        payable

61  function claimAmbientRewards(bytes32 poolIdx, uint32[] memory weeksToClaim) public payable

65  function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable 

74  function setAmbRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable 

85   function acceptCrocProxyRole(address, uint16 slot) public pure returns (bool) 
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L26
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L41
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L54
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L61
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L65
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L74
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L85

## [G-07] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity

file: LiquidityMiningPath.sol

67   require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

76  require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L67
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L76

```solidity

file: mixins/LiquidityMining.sol

94   while (time < block.timestamp && tickTrackingIndex < numTickTracking) 
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L94

```solidity

file: mixins/MarketSequencer.sol

319 require((bend.lowTick_ >= TickMath.MIN_TICK) &&

320 (bend.highTick_ <= TickMath.MAX_TICK) &&
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L319-L320

```solidity

file: mixins/TradeMatcher.sol

454  return inLimit && (swap.qty_ > 0);
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L454

## [G-08]Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity

file: mixins/LiquidityMining.sol

141 if (numTickTracking > 0)

182   if (overallInRangeLiquidity > 0) 

192   if (rewardsToSend > 0) 

276  if (overallTimeWeightedLiquidity > 0) 

285  if (rewardsToSend > 0) 
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L141
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L182
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L192
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L276
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L285

```solidity

file: mixins/MarketSequencer.sol

283   if (dir.liquidity_ > 0)
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L283

```solidity

file: mixins/TradeMatcher.sol

454  return inLimit && (swap.qty_ > 0);

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L454

## [G-09] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity

file: callpaths/LiquidityMiningPath.sol

58  claimConcentratedRewards(payable(msg.sender), poolIdx, lowerTick, upperTick, weeksToClaim);

 61 function claimAmbientRewards(bytes32 poolIdx, uint32[] memory weeksToClaim) public payable {
 62       claimAmbientRewards(payable(msg.sender), poolIdx, weeksToClaim);

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L58
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L62

## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: LiquidityMiningPath.sol

54  function claimConcentratedRewards(bytes32 poolIdx, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim)
        public
        payable
    {
        claimConcentratedRewards(payable(msg.sender), poolIdx, lowerTick, upperTick, weeksToClaim);
    }

    function claimAmbientRewards(bytes32 poolIdx, uint32[] memory weeksToClaim) public payable {
        claimAmbientRewards(payable(msg.sender), poolIdx, weeksToClaim);
    }

    function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
        // require(msg.sender == governance_, "Only callable by governance");
        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
            weekFrom += uint32(WEEK);
        }
72    }
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L54-L72

```solidity

file: mixins/TradeMatcher.sol

350   function signMintFlow (uint128 base, uint128 quote) private pure
        returns (int128, int128) {
        return (base.toInt128Sign(), quote.toInt128Sign());
    }

    /* @notice Converts the unsigned flow associated with a burn operation to a pair
     *         net settlement flow. (Will always be negative because a burn requires use
     *         to pay collateral to the pool.) */
    function signBurnFlow (uint128 base, uint128 quote) private pure
        returns (int128, int128){
        return (-(base.toInt128Sign()), -(quote.toInt128Sign()));
361    }
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L350-361

 ### [G-11] Tightly pack storage variables/optimize the order of variable declaration
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions

```solidity

file: mixins/LiquidityMining.sol

173 uint256 rewardsToSend;
        for (uint256 i; i < weeksToClaim.length; ++i) 
175            uint32 week = weeksToClaim[i];

205   uint256 liquidity = curve.ambientSeeds_;
            uint32 time = lastAccrued;
            while (time < block.timestamp) {
                uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
210                uint32 dt = uint32(

235   uint256 liquidity = pos.seeds_;
            uint32 time = lastAccrued;
            while (time < block.timestamp) {
                uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
240                uint32 dt = uint32(  

265  uint256 rewardsToSend;
        for (uint256 i; i < weeksToClaim.length; ++i) {
            uint32 week = weeksToClaim[i];
            require(week + WEEK < block.timestamp, "Week not over yet");
            require(
                !ambLiquidityRewardsClaimed_[poolIdx][posKey][week],
                "Already claimed"
            );
            uint256 overallTimeWeightedLiquidity = timeWeightedWeeklyGlobalAmbLiquidity_[
                    poolIdx
                ][week];
            if (overallTimeWeightedLiquidity > 0) {
277                uint256 rewardsForWeek = (timeWeightedWeeklyPositionAmbLiquidit      
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L173-L175
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L205-210
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L235-L240
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L265-L277

```solidity

file: mixins/MarketSequencer.sol

23  using SafeCast for int256;
    using SafeCast for int128;
    using SafeCast for uint256;
    using SafeCast for uint128;
27  using TickMath for uint128;
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L23

```solidity

file: mixins/TradeMatcher.sol

31  using SafeCast for int256;
    using SafeCast for int128;
    using SafeCast for uint256;
    using SafeCast for uint128;
    using TickMath for uint128;
    using LiquidityMath for uint96;
37  using LiquidityMath for uint128;
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L31-L37

## [G-13] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity

file: mixins/MarketSequencer.sol

255  if (!dir.chain_.swapDefer_) 
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L255

```solidity

file: mixins/TradeMatcher.sol

425  if (!tightSpill) 

485  if (!Bitmaps.isTickFinite(bumpTick)) { return bumpTick; }
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L425
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L485

## [G-14] Empty blocks should be removed or emit something
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).

```solidity

file: callpaths/LiquidityMiningPath.sol

26  function protocolCmd(bytes calldata cmd) public virtual {
        (uint8 code, bytes32 poolHash, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) =
           abi.decode(cmd, (uint8, bytes32, uint32, 

  i     if (code == ProtocolCmd.SET_CONC_REWARDS_CODE) {
            setConcRewards(poolHash, weekFrom, weekTo, weeklyReward);
        } else if (code == ProtocolCmd.SET_AMB_REWARDS_CODE) {
            setAmbRewards(poolHash, weekFrom, weekTo, weeklyReward);
        } else {
            revert("Invalid protocol command");
        }
37    }
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L26-L37
