### LOW-1 Arrays lack checks for zero values (0) 

1. https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L54 

2. https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/mixins/LiquidityMining.sol#L259

3. https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/mixins/LiquidityMining.sol#L161


The above has examples of arrays taking in values e.g. weeks to claim for concentrated liquidity. There are no checks that any of the values in the array is != 0. Its possible front ends or user or entry errors can lead to zero values(default for uint) passed in by error.

The above can lead to unexpected behaviours, potential errors etc. Consider 
```javascript
//canto_ambient/contracts/mixins/LiquidityMining.sol
//uint32 week = weeksToClaim[i]; week not checked to not be 0 value 
for (uint256 i; i < weeksToClaim.length; ++i) {
            uint32 week = weeksToClaim[i];
            require(week + WEEK < block.timestamp, "Week not over yet");
            require(
                !concLiquidityRewardsClaimed_[poolIdx][posKey][week],
                "Already claimed"
            );
```

### LOW-2 - Lack sanity checks function inputs values that must != 0 

1. https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L65 uint64 weeklyReward must not be 0 

2. https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L74 uint64 weeklyReward must not be 0 

```
    function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
        // require(msg.sender == governance_, "Only callable by governance");
        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward; // wrong rewards stored 
            weekFrom += uint32(WEEK);
        }
    }
```
weeklyReward may be passed in accidentally, by error, or by default value as 0 leading to the storage of pools weekFrom rewards incorrectly stored as 0 leading to loss of rewards if not picked up 

**Recommendation** -> It is recommended to ensure all cases where inputs of values = 0 in functions are checked and ensure there is revert in such cases so that zero values are not allowed e.g

```
require(weeklyReward != 0, "cant be zero")
```

### LOW-3 - Lack sanity checks function inputs 

There is no check that inputs weekFrom <= weeTo 
setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) 
```
function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
        // require(msg.sender == governance_, "Only callable by governance");
        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
            weekFrom += uint32(WEEK);
        }
    }

    function setAmbRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
        // require(msg.sender == governance_, "Only callable by governance");
        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            ambRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
            weekFrom += uint32(WEEK);
        }
    }
```
If values weekFrom and weekTo are switched by error it can lead to AmbRewards and or ConRewards not being set with a thinking that they have been set as the while loop will not run 
https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L68
https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L77

### LOW-4 - Use Solidity integrated time units 

https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/mixins/LiquidityMining.sol#L13
```javascript
uint256 constant WEEK = 604800; // Week in seconds 604800
```
There exists in solidity time units for days, weeks, hours, years, etc 
Recommended to use solidity integrated time units to increase code readability and avoid calculation error.

### NC-1 - Function arguments/parameters not underscored 

It is considered best practise to underscore function arguments/parameters for readablity, ensure not clash or confuse with state variables etc

All the functions in the code do not make use of this pre or post underscore format 

**Recommendation** -> It is recommended to underscore to functions parameters/arguments e.g as below 
```javascript
 function setConcRewards(bytes32 _poolIdx, uint32 _weekFrom, uint32 _weekTo, uint64 _weeklyReward) public payable {
```

### NC-2 - Internal functions not underscored 

It is considered best practise to underscore internal and private functions e.g 
```
function _crossTicks(bytes32 poolIdx,int24 exitTick, int24 entryTick) internal {
```

All the functions in /canto_ambient/contracts/mixins/LiquidityMining.sol are internal but do not make use of this best practise
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L161

### NC-3 - Make use of latest Solidity version 

The contracts use version 0.8.19 when latest version is 0.8.21. Latest version may have benefited from bug fixes, optimizations, patches, improvements, that still are in 0.8.19 and may put project at risk.

 
### NC-4 - Functions can be better named to improve usability 

Functions especially those used by users to interact with the protocol/project need to have intuitive names that can readily help users/stakeholders have an idea what the function is doing. The following functions are examples of non intuitive naming e.g few examples below 
```
userCmd -> userLiquidityCommands
accrueConcentratedPositionTimeWeightedLiquidity // is too long a name 
```
Consider review all relevant functions and rename appropriately considering meaning, length, appropriateness etc 

### NC-5 - Inconsistent spacing in comments 

https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L19
```
/* @title Liquidity mining callpath sidecar.
 * @notice Defines a proxy sidecar contract that's used to move code outside the 
 *         main contract to avoid Ethereum's contract code size limit. Contains
 *         components related to CANTO liquidity mining.
 * 
 * @dev    This exists as a standalone contract but will only ever contain proxy code,
 *         not state. As such it should never be called directly or externally, and should
 *         only be invoked with DELEGATECALL so that it operates on the contract state
 *         within the primary CrocSwap contract.
 * @dev Since this contract is a proxy sidecar, entrypoints need to be marked
 *      payable even though it doesn't directly handle msg.value. Otherwise it will
 *      fail on any. Because of this, this contract should never be used in any other
 *      context besides a proxy sidecar to CrocSwapDex. */
``` 
The spacing of last @dev comments does not correspond with earlier @dev and @notice 

### NC-6 - Functions sharing common code can be combined or reuse the common code as internal function or modifier 
The following functions share common code canto_ambient/contracts/callpaths/LiquidityMiningPath.sol
```
function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
        // require(msg.sender == governance_, "Only callable by governance");
        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
            weekFrom += uint32(WEEK);
        }
    }

    function setAmbRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
        // require(msg.sender == governance_, "Only callable by governance");
        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            ambRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
            weekFrom += uint32(WEEK);
        }
    }
```
We can see the same update for weeks and same require in functions. An example improvement is below 
```
    function setRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward, bool Amb) public payable {
        // require(msg.sender == governance_, "Only callable by governance");
        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            if(Amb) {ambRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;}
            else {concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;}
            weekFrom += uint32(WEEK);
        }
    }
```

Additional functions sharing coming code are 
```
function accrueConcentratedGlobalTimeWeightedLiquidity(
function accrueAmbientGlobalTimeWeightedLiquidity(
\\ in the parts below that are almost similar 
if (lastAccrued != 0) {
            uint256 liquidity = curve.ambientSeeds_;
            uint32 time = lastAccrued;
            while (time < block.timestamp) {
                uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
                uint32 dt = uint32(
                    nextWeek < block.timestamp
                        ? nextWeek - time
                        : block.timestamp - time
                );
                timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;
                time += dt;
            }
        }
        timeWeightedWeeklyGlobalAmbLiquidityLastSet_[poolIdx] = uint32(
            block.timestamp
        );
```
Consider the appropriate refactors that work best in order to avoid code duplication, reuse, bloat, repetition etc 