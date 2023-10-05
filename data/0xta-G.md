# Gas Optimization

# Summary

| Number | Gas Optimization                                                                             | Context |
| :----: | :------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Using Storage instead of memory for structs/arrays saves gas                                 |    5    |
| [G-02] | Use calldata instead of memory for function arguments that do not get mutated                |    4    |
| [G-03] | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS          |    1    |
| [G-04] | Can make the variable outside the loop to save gas                                           |   10    |
| [G-05] | Use solidity version 0.8.20 to gain some gas boost                                           |    2    |
| [G-06] | Add unchecked {} for subtraction                                                             |    4    |
| [G-07] | Use ++i instead of i++ to increment                                                          |    1    |
| [G-08] | Using assembly to revert with an error message                                               |    8    |
| [G-09] | State variables should be cached in stack variables rather than re-reading them from storage |   12    |

## [G-01] Using Storage instead of memory for structs/arrays saves gas

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
It is only beneficial to use reduced-size arguments if you are dealing with storage values because the compiler will pack multiple elements into one storage slot, and thus, combine multiple reads or writes into a single operation.>=

When dealing with function arguments or memory values, there is no inherent benefit because the compiler does not pack these values.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

```solidity
File: contracts/mixins/LiquidityMining.sol

17        StorageLayout.TickTracking memory tickTrackingData = StorageLayout

32        StorageLayout.TickTracking memory tickTrackingData = StorageLayout

95                    TickTracking memory tickTracking = tickTracking_[poolIdx][i][tickTrackingIndex];

169        CurveMath.CurveState memory curve = curves_[poolIdx];

261        CurveMath.CurveState memory curve = curves_[poolIdx];

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol

## [G-02] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity
File: contracts/mixins/LiquidityMining.sol

161        uint32[] memory weeksToClaim

259        uint32[] memory weeksToClaim

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol


54    function claimConcentratedRewards(bytes32 poolIdx, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim)

61    function claimAmbientRewards(bytes32 poolIdx, uint32[] memory weeksToClaim) public payable {

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L54

## [G-03] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol

86        return slot == CrocSlots.LIQUIDITY_MINING_PROXY_IDX;

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L86

## [G-04] Can make the variable outside the loop to save gas

When you declare a variable inside a loop the variable is reinitialized and assigned memory or storage space during each iteration of the loop. This can use more gas, leading to higher transaction costs. To save gas, it's often recommended to declare the variable outside the loop so that it is only initialized once and reused throughout the loop.

```solidity
File: contracts/mixins/LiquidityMining.sol

89                uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i];
90                uint32 origIndex = tickTrackingIndex;
91                uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
92                uint32 time = lastAccrued;

96                  uint32 currWeek = uint32((time / WEEK) * WEEK);
97                    uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

140                uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);

175            uint32 week = weeksToClaim[i];

181            uint256 overallInRangeLiquidity = timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][week];

267            uint32 week = weeksToClaim[i];

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L89

## [G-05] Use solidity version 0.8.20 to gain some gas boost

Upgrade to the latest solidity version 0.8.20 to get additional gas savings. See latest release for reference:

https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/

```solidity
File:

3   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L3

```solidity
File: contracts/mixins/LiquidityMining.sol

3   pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L3

## [G-06] Add unchecked {} for subtraction

where the operands can't underflow because of a previous while-statement

```solidity
File: contracts/mixins/LiquidityMining.sol

56                        : block.timestamp - time

101                      : block.timestamp - time

213                        : block.timestamp - time

243                        : block.timestamp - time
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L56

## [G-07] Use ++i instead of i++ to increment

The reason behind this is in way ++i and i++ are evaluated by the compiler.

i++ returns i(its old value) before incrementing i to a new value. This means that 2 values are stored on the stack for usage whether you wish to use it or not. ++i on the other hand, evaluates the ++ operation on i (i.e it increments i) then returns i (its incremented value) which means that only one item needs to be stored on the stack.

```solidity
File: contracts/mixins/LiquidityMining.sol

122                                tickTrackingIndex++;

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L122

## [G-08] Using assembly to revert with an error message

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol


67        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

76        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L67

```solidity
File: contracts/mixins/LiquidityMining.sol

176            require(week + WEEK < block.timestamp, "Week not over yet");

177            require(

194            require(sent, "Sending rewards failed");

268            require(week + WEEK < block.timestamp, "Week not over yet");

269            require(

287            require(sent, "Sending rewards failed");

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L176

## [G-09] State variables should be cached in stack variables rather than re-reading them from storage

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol

67        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

76        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L67

```solidity
File: contracts/mixins/LiquidityMining.sol


51               uint32 currWeek = uint32((time / WEEK) * WEEK);
52                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);



96                    uint32 currWeek = uint32((time / WEEK) * WEEK);
97                    uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);


176            require(week + WEEK < block.timestamp, "Week not over yet");

208                uint32 currWeek = uint32((time / WEEK) * WEEK);
209                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);


238                uint32 currWeek = uint32((time / WEEK) * WEEK);
239                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);


268            require(week + WEEK < block.timestamp, "Week not over yet");

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L51