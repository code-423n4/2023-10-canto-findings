# gas optimizm

# Summary
|  no    | issue  | insteance  |
|------|--------|------------|
|[G-01]|Use Modifiers Instead of Functions To Save Gas|1|
|[G-02]|Use assembly in place of abi.decode to extract calldata values more efficiently|2|
|[G-03]|Can Make The Variable Outside The Loop To Save Gas |15|
|[G-04]|>=/<= costs less gas than >/<|2|
|[G-05]|Uncheck arithmetics operations that can’t underflow/overflow|1|
|[G-06]|Use do while loops instead of for loops|2|



## [G-01] Use Modifiers Instead of Functions To Save Gas

By using modifiers  you can keep track of gas consumption and perform gas-saving operations based on your contract's specific requirements. 


```solidity
File:   contracts/callpaths/LiquidityMiningPath.sol
85   function acceptCrocProxyRole(address, uint16 slot) public pure returns (bool) {
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L85


## [G-02] Use assembly in place of abi.decode to extract calldata values more efficiently


Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```solidity
File:  contracts/callpaths/LiquidityMiningPath.sol
28    abi.decode(cmd, (uint8, bytes32, uint32, uint32, uint64));

43    abi.decode(input, (uint8, bytes32, int24, int24, uint32[]));
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L28
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L43




## [G-03] Can Make The Variable Outside The Loop To Save Gas 

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract.

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





## [G-04] >=/<= costs less gas than >/<

The compiler uses opcodes GT and ISZERO for code that uses >, but only requires LT for >=. A similar behaviour applies for >, which uses opcodes LT and ISZERO, but only requires GT for <=.

```solidity
File:  canto_ambient/contracts/mixins/LiquidityMining.sol
105   if (tickTracking.enterTimestamp < nextWeek) {

107   if (tickTracking.enterTimestamp < time) {    
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L105


## [G-05] Uncheck arithmetics operations that can’t underflow/overflow

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block

```solidity
123   dt = tickActiveEnd - tickActiveStart;
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L123


## [G-06] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
File:  contracts/mixins/LiquidityMining.sol
174  for (uint256 i; i < weeksToClaim.length; ++i) {


266  for (uint256 i; i < weeksToClaim.length; ++i) {                
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L174
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L266


