## QA Report For Canto Liquidity Mining. 

| ID  |   Title                                                     | Occurrences |
|:---:|:---                                                         | :---:       |
| 1.  | Avoid Casting 2 times the same data.                        | 2           |
| 2   | Use calldata instead of memory in function parameters.      | 6           |
| 3.  | Use unchecked{++I;} in foor loops.                          | 5           |
| 4.  | Cache the length of the array before reading in a for cycle.| 2           |
| 5.  | Use ExcessivelySafeCall for call to untrusted address.      | 2           |
| 6.  | Improve Contracts documentation using Natspec.              | 1           |

**1.- Avoid Casting 2 times the same data.**
   To improve efficiency and gas cost avoid casting 2 times the same data, instead create a variable to save the casting value once and use this variable in the sections needed. 

for example you can improve the **crossTicks** function to save the cast value of block.timestamp and use this variable everywhere the uint32(block.timestamp) is needed.
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L24-L35
```solidity
    function crossTicks(bytes32 poolIdx, int24 exitTick, int24 entryTick) internal {
        uint32 timestamp = uint32(block.timestamp);
        uint256 numElementsExit = tickTracking_[poolIdx][exitTick].length;
        tickTracking_[poolIdx][exitTick][numElementsExit - 1].exitTimestamp = timestamp;
        StorageLayout.TickTracking memory tickTrackingData = StorageLayout.TickTracking(timestamp, 0);
        tickTracking_[poolIdx][entryTick].push(tickTrackingData);
    }
```
**2.- Use calldata instead of memory in function parameters.**
in functions parameters that don't mute use calldata instead of memory to reduce gas costs, you can change "memory" keyword for "calldata" in the following functions.

https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L39-L65
```solidity
    function accrueConcentratedGlobalTimeWeightedLiquidity(
        bytes32 poolIdx,
        CurveMath.CurveState memory curve) 
    internal {}
```
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L161C11-L161C11
```solidity
    function claimConcentratedRewards(
        address payable owner,
        bytes32 poolIdx,
        int24 lowerTick,
        int24 upperTick,
        uint32[] memory weeksToClaim
    ) internal {}
```
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L200
```solidity
    function accrueAmbientGlobalTimeWeightedLiquidity(
        bytes32 poolIdx,
        CurveMath.CurveState memory curve
    ) internal {}
```
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L259
```solidity
    function claimAmbientRewards(
        address owner,
        bytes32 poolIdx,
        uint32[] memory weeksToClaim
    ) internal {}
```
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L54
```solidity
    function claimConcentratedRewards(bytes32 poolIdx, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim)
        public
        payable
    {
        claimConcentratedRewards(payable(msg.sender), poolIdx, lowerTick, upperTick, weeksToClaim);
    }
```
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L61
```solidity
    function claimAmbientRewards(bytes32 poolIdx, uint32[] memory weeksToClaim) public payable {
        claimAmbientRewards(payable(msg.sender), poolIdx, weeksToClaim);
    }
```
**3.- Use unchecked{++i;} in foor loops.**

It’s recommended to use unchecked{++i;} or unchecked{++j;} in for loops, the unchecked increment variable helps to reduce gas costs in the for loop.

You can improve this in the following lines in the LiquidityMining contract.

https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L88C13-L88C71
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L139
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L174
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L266
Example of implementation:

```solidity
for (int24 i = lowerTick + 10; i <= upperTick - 10; ) {
// Code
unchecked{++i;}
}
```
Here you can change this:
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L184
```solidity
for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j) {
    inRangeLiquidityOfPosition += timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];
}
```
For this:
```solidity
for (int24 j = lowerTick + 10; j <= upperTick - 10;) {
    inRangeLiquidityOfPosition += timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];
unchecked{++j;}
}
```

**4.- Cache the length of the array before reading in a for cycle.**

To avoid reading the length of the array every time the for loop iterates we can read the length of the array once before the for loop starts and save the value in a memory variable that would be used in the for loop to check in every iteration.

For example, in the following loop.
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L174-L189
we can rewrite it in this:
```solidity
        for (uint256 i; i < weeksToClaim.length; ++i) {
           ** Code omitted
        }
```
To this:
```solidity
      uint256 length = weeksToClaim.length
      for (uint256 i; i < length; ++i){
           ** Code omitted
      }
```
The same can be applied here:
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L266-L284
```solidity
      uint256 length = weeksToClaim.length
      for (uint256 i; i < length; ++i){
           ** Code omitted
      }
```
**5- Use ExcessivelySafeCall for calls to untrusted addresses.**

It’s recommended to use the library ExcessivelySafeCall to call arbitrary addresses to prevent gas griefing attacks caused by a big return of data from the call to unsafe addresses.

https://github.com/nomad-xyz/ExcessivelySafeCall

You can implement this library in the calls to the owner's address.

https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L193
https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L286
implementation example:

1.- add the library and use it for addresses.
2.- substitute call for excessivelySafeCall.

in Liquidity Mining contract add:
```solidity
+ import {ExcessivelySafeCall} from "libraries/ExcessivelySafeCall.sol";

+ using ExcessivelySafeCall for address;

if (rewardsToSend > 0) {
   (bool sent,) = owner.excessivelySafeCall(
            SOME_GAS,
            rewardsToSend, // Value to sent
            32,  // <-- the magic. Copy no more than 32 bytes to memory
            abi.encodeWithSelector("")
   require(sent, "Sending rewards failed");
}
```
**6- Improve Contracts documentation using Natspec.**

It is recommended to improve documentation in contracts by using Natspec to document the functionality of each function in contracts, the variables they receive, and the values each function returns, so that when other/new engineers on the code team read the code they can fully understand the functionality of each feature in each contract quickly.

https://docs.soliditylang.org/en/latest/natspec-format.html