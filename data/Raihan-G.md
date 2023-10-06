# gas

# Summary
|      | issue  | insteance  |
|------|--------|------------|
|[G-01]| Can Make The Variable Outside The Loop To Save Gas |15|
|[G-02]| >=/<= costs less gas than >/<|2|
|[G-03]| Uncheck arithmetics operations that can’t underflow/overflow|1|
|[G-04]| Use do while loops instead of for loops|2|
|[G-05]| Use Modifiers Instead of Functions To Save Gas|1|
|[G-06]| Use assembly in place of abi.decode to extract calldata values more efficiently|2|

## [G-01] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```
There are 15 instances of this issue:
```solidity
File:  contracts/mixins/LiquidityMining.sol
51             uint32 currWeek = uint32((time / WEEK) * WEEK);
52             uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

89               uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i];
                uint32 origIndex = tickTrackingIndex;
                uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
                uint32 time = lastAccrued;
                // Loop through all in-range time spans for the tick or up to the current time (if it is still in range)
                while (time < block.timestamp && tickTrackingIndex < numTickTracking) {
                    TickTracking memory tickTracking = tickTracking_[poolIdx][i][tickTrackingIndex];
                    uint32 currWeek = uint32((time / WEEK) * WEEK);
                    uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
                    uint32 dt = uint32(
                        nextWeek < block.timestamp
                            ? nextWeek - time
                            : block.timestamp - time
                    );
                    uint32 tickActiveStart; // Timestamp to use for the liquidity addition
104                 uint32 tickActiveEnd;

175  uint32 week = weeksToClaim[i];

208  uint32 currWeek = uint32((time / WEEK) * WEEK);
209  uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

238  uint32 currWeek = uint32((time / WEEK) * WEEK);
239  uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

267  uint32 week = weeksToClaim[i];
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L89-L104

## [G-02] >=/<= costs less gas than >/<
The compiler uses opcodes GT and ISZERO for code that uses >, but only requires LT for >=. A similar behaviour applies for >, which uses opcodes LT and ISZERO, but only requires GT for <=.
[Reffrence](https://gist.github.com/CloudEllie/05fda0fdecba2ed7d85f68c709e46c8d#gas-18--costs-less-gas-than-)

There are 2 instances of this issue:
```solidity
File:  canto_ambient/contracts/mixins/LiquidityMining.sol
105   if (tickTracking.enterTimestamp < nextWeek) {

107   if (tickTracking.enterTimestamp < time) {    
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L105


## [G-03] Uncheck arithmetics operations that can’t underflow/overflow
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block

There are 1 instances of this issue:
```solidity
File: contracts/mixins/LiquidityMining.sol
123   dt = tickActiveEnd - tickActiveStart;
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L123


## [G-04] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops)

There are 2 instances of this issue:
```solidity
File:  contracts/mixins/LiquidityMining.sol
174  for (uint256 i; i < weeksToClaim.length; ++i) {


266  for (uint256 i; i < weeksToClaim.length; ++i) {                
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L123


## [G-05] Use Modifiers Instead of Functions To Save Gas
Example of two contracts with modifiers and internal view function:
```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}
```
Differences:
```
Deploy Modifier.sol
108727
Deploy Inlined.sol
110473
Modifier.foo
21532
Inlined.foo
21556
```
This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.
There are 1 instances of this issue:

```solidity
File:   contracts/callpaths/LiquidityMiningPath.sol
85   function acceptCrocProxyRole(address, uint16 slot) public pure returns (bool) {
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L85


## [G-06] Use assembly in place of abi.decode to extract calldata values more efficiently
Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.
https://code4rena.com/reports/2023-08-goodentry#gas-optimizations:~:text=%5BG%2D02%5D%20Use%20assembly%20in%20place%20of%20abi.decode%20to%20extract%20calldata%20values%20more%20efficiently

```solidity
File:  contracts/callpaths/LiquidityMiningPath.sol
28    abi.decode(cmd, (uint8, bytes32, uint32, uint32, uint64));

43    abi.decode(input, (uint8, bytes32, int24, int24, uint32[]));
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L28

