## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:|
| [G-01] | Use `calldata` instead of `memory` for function arguments that do not get mutate | 5 |
| [G-02] | Use `assembly` in place of `abi.decode` to extract calldata values more efficiently | 2 |
| [G-03] | Duplicated `require`/`if` checks should be refactored to a modifier or function | 2 |
| [G-04] | Using `ternary` operator instead of if else saves gas | 2 |


## [G-01] Use `calldata` instead of `memory` for function arguments that do not get mutated.

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

**_There are 5 instances of this issue_**

```solidity
File : contracts/callpaths/LiquidityMiningPath.sol

41: function userCmd(bytes calldata input) public payable {
42:      (uint8 code, bytes32 poolHash, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim) =
43:          abi.decode(input, (uint8, bytes32, int24, int24, uint32[]));
44:
45:      if (code == UserCmd.CLAIM_CONC_REWARDS_CODE) {
46:          claimConcentratedRewards(poolHash, lowerTick, upperTick, weeksToClaim);
47:      } else if (code == UserCmd.CLAIM_AMB_REWARDS_CODE) {
48:          claimAmbientRewards(poolHash, weeksToClaim);
49:      } else {
50:          revert("Invalid user command");
51:      }
52:  }

54: function claimConcentratedRewards(bytes32 poolIdx, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim)
55:        public
56:        payable
57:    {
58:        claimConcentratedRewards(payable(msg.sender), poolIdx, lowerTick, upperTick, weeksToClaim);
59:    }

61: function claimAmbientRewards(bytes32 poolIdx, uint32[] memory weeksToClaim) public payable {
62:        claimAmbientRewards(payable(msg.sender), poolIdx, weeksToClaim);
63:    }


```
[41-63](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L41C4-L63C6)

```solidity
File : contracts/mixins/LiquidityMining.sol

156: function claimConcentratedRewards(
157:    address payable owner,
158:    bytes32 poolIdx,
159:    int24 lowerTick,
160:    int24 upperTick,
161:    uint32[] memory weeksToClaim
162:  ) internal {

256: function claimAmbientRewards(
257:    address owner,
258:    bytes32 poolIdx,
259:    uint32[] memory weeksToClaim
260:  ) internal {    

```
[156-162](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L156C5-L162C17), [256-260](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L256C5-L260C17)


## [G-02] Use `assembly` in place of `abi.decode` to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

**_There are 2 instances of this issue_**

```solidity
File : contracts/callpaths/LiquidityMiningPath.sol

26: function protocolCmd(bytes calldata cmd) public virtual {
27:     (uint8 code, bytes32 poolHash, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) =
28:     abi.decode(cmd, (uint8, bytes32, uint32, uint32, uint64));

41: function userCmd(bytes calldata input) public payable {
42:      (uint8 code, bytes32 poolHash, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim) =
43:      abi.decode(input, (uint8, bytes32, int24, int24, uint32[]));

```
[26-28](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L26C5-L28C71), [41-43](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L41C5-L43C73)


## [G-03] Duplicated `require`/`if` checks should be refactored to a modifier or function

In development process, if you changed one of them you should find all of other to change and for large and complicated projects possible this change will be missed.

**_There are 2 instances of this issue_**

```solidity
File : contracts/callpaths/LiquidityMiningPath.sol

     /// @audit this require is duplicated on line 76
67:  require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

```
[67](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L67), [76](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L76)

```solidity
File : contracts/mixins/LiquidityMining.sol

     /// @audit this if is duplicated on line 285
192: if (rewardsToSend > 0) {

```
[192](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L192),[285](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L285)


## [G-04] Using `ternary` operator instead of single line if else saves gas

When using the if-else construct, Solidity generates bytecode that includes a JUMP operation. The JUMP operation requires the EVM to change the program counter to a different location in the bytecode, resulting in additional gas costs. On the other hand, the ternary operator doesn't involve a JUMP operation, as it generates bytecode that utilizes conditional stack manipulation.
The exact amount of gas saved by using the ternary operator instead of the if-else construct in Solidity can vary depending on the specific scenario, the complexity of the surrounding code, and the conditions involved.

**_There are 2 instances of this issue_**

```solidity
File : contracts/mixins/LiquidityMining.sol

107:  if (tickTracking.enterTimestamp < time) {
108:    // Tick was already active when last claim happened, only accrue from last claim timestamp
109:    tickActiveStart = time;
110:   } else {
111:        // Tick has become active this week
112:       tickActiveStart = tickTracking.enterTimestamp;

142: if (tickTracking_[poolIdx][i][numTickTracking - 1].exitTimestamp == 0) {
143:         // Tick currently active
144:        tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking - 1;
145:      } else {
146:         tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking;
147:         }

```
[107-112](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L107C25-L112C75), [142-147](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L142C21-L147C22)
