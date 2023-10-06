|     |
| --- |
| \[G-01\] uint8 is not always cheaper than uint256 |
| \[G-02\] Use assembly in place of `abi.decode` to extract calldata values more efficiently |
| \[G-03\] Cheaper input validations should come before expensive operations |
| \[G-04\] Duplicated `require()/if()` checks should be refactored to a modifier or function |
| \[G-05\] DEFAULT VALUE ASSIGNMENT TO VARIABLES CAN BE OMITTED TO SAVE GAS |
| \[G-06\] Use != 0 instead of > 0 |
| \[G-07\] Unbounded Gas Consumption Risk Due to External Call Recipients |

## \[G-01\] uint8 is not always cheaper than uint256

The EVM only operates on 32 bytes/ 256 bits at a time. This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.

```
(uint8 code, bytes32 poolHash, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) =
            abi.decode(cmd, (uint8, bytes32, uint32, uint32, uint64));
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L27C9-L28C71

```
(uint8 code, bytes32 poolHash, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim) =
            abi.decode(input, (uint8, bytes32, int24, int24, uint32[]));
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L42C9-L43C73

## \[G-02\] Use assembly in place of `abi.decode` to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```
(uint8 code, bytes32 poolHash, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) =
            abi.decode(cmd, (uint8, bytes32, uint32, uint32, uint64));
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L27C9-L28C71

```
(uint8 code, bytes32 poolHash, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim) =
            abi.decode(input, (uint8, bytes32, int24, int24, uint32[]));
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L42C9-L43C73

## \[G-03\] Cheaper input validations should come before expensive operations

```
require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L67

```
require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L76

## \[G-04\] Duplicated `require()/if()` checks should be refactored to a modifier or function

to reduce code duplication and improve readability. • Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check. • A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```
File:	callpaths/LiquidityMiningPath.sol

67: 	require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");


76:	require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L67

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L76

## \[G-05\] DEFAULT VALUE ASSIGNMENT TO VARIABLES CAN BE OMITTED TO SAVE GAS

Inside the `for` loop where the default value of `0` assigned to the `i` variable. Since `i` is by default initialzied to `0` it is not required explicitly assign the `0` value to `i` variable inside the `for` loops. This will save gas.

```
for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L88

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L139

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L174

## \[G-06\] Use != 0 instead of > 0

Using `> 0` uses slightly more gas than using `!= 0`. Use `!= 0` when comparing `uint` variables to zero, which cannot hold values below zero.

```
if (numTickTracking > 0) {
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L141

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L192

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L276

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L285

## \[G-07\] Unbounded Gas Consumption Risk Due to External Call Recipients

In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using addr.call{gas: }. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.

```
(bool sent, ) = owner.call{value: rewardsToSend}("");
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L193

In the `claimConcentratedRewards()` function a low-level call was made without specifing the amount of gas to be used in the low-level call. This is bad practice as this function could be called by a contract and then the contract's fallback function (if the contract does not specify a receive function) would get executed and could possibly use up all the gas thereby causing the ``claimConcentratedRewards`()` `` function to revert and the transaction fails. This scenario could lead to bad consumer experience, you should specify the amount of gas to be used in a low-level call as this would prevent such occurences as only a predefined amount of gas can be used by the calling contract.