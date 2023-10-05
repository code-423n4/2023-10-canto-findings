## Summary

### Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[L&#x2011;01](#l01-missing-events)] | Missing Event for critical functions or parameters change | 4 |

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-remove-commented-code)] | Remove commented codes | 2 |
| [[N&#x2011;02](#n02-follow-solidity-standard)] | Follow Solidity standard naming conventions (internal function style rule) for functions | 8 |


## Low Risk Issues


### [L&#x2011;01] Missing Event for critical functions or parameters change

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

*There is 4 instance of this issue:*

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol

54:    function claimConcentratedRewards(bytes32 poolIdx, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim)

61:    function claimAmbientRewards(bytes32 poolIdx, uint32[] memory weeksToClaim) public payable {

65:    function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {

74:    function setAmbRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {

```
*GitHub*: [54](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L54), [61](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L61C1-L61C97), [65](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L65C1-L65C115), [74](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L74C1-L74C114)

```solidity
File: contracts/mixins/LiquidityMining.sol

16:     function initTickTracking(bytes32 poolIdx, int24 tick) internal {

24:     function crossTicks(

```

*GitHub*: [16](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L16), [24](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L24C1-L24C25)

## Non-critical Issues

### [N&#x2011;01] Remove commented codes

*There are 2 instances of this issue:*

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol

66:         // require(msg.sender == governance_, "Only callable by governance");

75:         // require(msg.sender == governance_, "Only callable by governance");

```
*GitHub*: [66](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L66), [75](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L75)

### [N&#x2011;02] Follow Solidity standard naming conventions (internal function style rule) for functions

The below codes don’t follow Solidity’s standard naming convention,

internal functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

*There are 8 instances of this issue:*

```solidity
File: contracts/mixins/LiquidityMining.sol

16:     function initTickTracking(bytes32 poolIdx, int24 tick) internal {

24:     function crossTicks(

39:     function accrueConcentratedGlobalTimeWeightedLiquidity(

69:     function accrueConcentratedPositionTimeWeightedLiquidity(

156:    function claimConcentratedRewards(

198:    function accrueAmbientGlobalTimeWeightedLiquidity(

224:    function accrueAmbientPositionTimeWeightedLiquidity(

256:    function claimAmbientRewards(

```
*GitHub*: [16](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L16C1-L16C70), [24](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L24C1-L24C25), [39](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L39C1-L39C60), [69](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L69), [156](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L156), [198](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L198C1-L198C55), [224](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L224C1-L224C57), [256](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L256C1-L256C34)