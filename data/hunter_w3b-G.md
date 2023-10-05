# Gas Optimization

Gas optimization techniques are critical in smart contract development to reduce transaction costs and make contracts more efficient. The Canto protocol, as observed in the provided contract, can benefit from various gas-saving strategies to enhance its performance. Below is a summary of gas optimization techniques followed by categorized findings within the contract

# Summary

# Summary

| Finding                                                                                      | Occurrences |
| :------------------------------------------------------------------------------------------- | :---------: |
| Add unchecked {} for subtraction                                                             |      4      |
| Using assembly to revert with an error message                                               |      8      |
| Use solidity version 0.8.20 to gain some gas boost                                           |      2      |
| State variables should be cached in stack variables rather than re-reading them from storage |     27      |
| Avoid contract calls by making the architecture monolithic                                   |      2      |
| Always use Named Returns                                                                     |      1      |
| Can make the variable outside the loop to save gas                                           |     10      |
| Use calldata instead of memory for function arguments that do not get mutated                |      4      |
| Using Storage instead of memory for structs/arrays saves gas                                 |      5      |
| Using > 0 costs more gas than != 0                                                           |      5      |
| Do-While loops are cheaper than for loops                                                    |      5      |
| Use uint256(1)/uint256(2) instead for true and false boolean states                          |      2      |
| Split require statements that have boolean expressions                                       |      2      |
| Use ++i instead of i++ to increment                                                          |      1      |

## [G-01] Add unchecked {} for subtraction

where the operands can't underflow because of a previous while-statement

```solidity
File: contracts/mixins/LiquidityMining.sol

56                        : block.timestamp - time

101                      : block.timestamp - time

213                        : block.timestamp - time

243                        : block.timestamp - time
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L56

## [G-02] Using assembly to revert with an error message

When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Here’s an example;

```solidity

/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num) external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num) external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(
                    0x40,
                    0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000
                ) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}

```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

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

## [G-03] Use solidity version 0.8.20 to gain some gas boost

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

## [G-04] State variables should be cached in stack variables rather than re-reading them from storage

```solidity
File: contracts/mixins/LiquidityMining.sol

// @audit  WEEK  is state variable should be cached in stack variable
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

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol

67        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

76        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L67

## [G-05] Avoid contract calls by making the architecture monolithic

Contract calls are expensive, and the best way to save gas on them is by not using them at all. There is a natural tradeoff with this, but having several contracts that talk to each other can sometimes increase gas and complexity rather than manage it.

```solidity
File: contracts/mixins/LiquidityMining.sol

193            (bool sent, ) = owner.call{value: rewardsToSend}("");

286            (bool sent, ) = owner.call{value: rewardsToSend}("");

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L193

## [G-06] Always use Named Returns

The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

```solidity

contract NamedReturn {
    function myFunc1(uint256 x, uint256 y) external pure returns (uint256) {
        require(x > 0);
        require(y > 0);

        return x * y;
    }
}

contract NamedReturn2 {
    function myFunc2(uint256 x, uint256 y) external pure returns (uint256 z) {
        require(x > 0);
        require(y > 0);

        z = x * y;
    }
}

```

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol

86        return slot == CrocSlots.LIQUIDITY_MINING_PROXY_IDX;

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L86

## [G-07] Can make the variable outside the loop to save gas

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

## [G-08] Use calldata instead of memory for function arguments that do not get mutated

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

## [G-09] Using Storage instead of memory for structs/arrays saves gas

In Solidity, storage pointers are variables that reference a location in storage of a contract. They are not exactly the same as pointers in languages like C/C++.

It is helpful to know how to use storage pointers efficiently to avoid unnecessary storage reads and perform gas-efficient storage updates.

Here’s an example showing where storage pointers can be helpful.

```solidity
contract StoragePointerUnOptimized {
    struct User {
        uint256 id;
        string name;
        uint256 lastSeen;
    }

    constructor() {
        users[0] = User(0, "John Doe", block.timestamp);
    }

    mapping(uint256 => User) public users;

    function returnLastSeenSecondsAgo(
        uint256 _id
    ) public view returns (uint256) {
        User memory _user = users[_id];
        uint256 lastSeen = block.timestamp - _user.lastSeen;
        return lastSeen;
    }
}

```

Above, we have a function that returns the last seen of a user at a given index. It gets the lastSeen value and subtracts that from the current block.timestamp. Then we copy the whole struct into memory and get the lastSeen which we use in calculating the last seen in seconds ago. This method works well but is not so efficient, this is because we are copying all of the struct from storage into memory including variables we don’t need. Only if there was a way to only read from the lastSeen storage slot (without assembly). That’s where storage pointers come in.

```solidity

// This results in approximately 5,000 gas savings compared to the previous version.
contract StoragePointerOptimized {
    struct User {
        uint256 id;
        string name;
        uint256 lastSeen;
    }

    constructor() {
        users[0] = User(0, "John Doe", block.timestamp);
    }

    mapping(uint256 => User) public users;

    function returnLastSeenSecondsAgoOptimized(
        uint256 _id
    ) public view returns (uint256) {
        User storage _user = users[_id];
        uint256 lastSeen = block.timestamp - _user.lastSeen;
        return lastSeen;
    }
}

```

“The above implementation results in approximately 5,000 gas savings compared to the first version”. Why so, the only change here was changing memory to storage and we were told that anything storage is expensive and should be avoided?

Here we store the storage pointer for users[_id] in a fixed sized variable on the stack (the pointer of a struct is basically the storage slot of the start of the struct, in this case, this will be the storage slot of user[_id].id). Since storage pointers are lazy (meaning they only act(read or write) when called or referenced). Next we only access the lastSeen key of the struct. This way we make a single storage load then store it on the stack, instead of 3 or possibly more storage loads and a memory store before taking a small chunk from memory unto the stack.

Note: When using storage pointers, it’s important to be careful not to reference dangling pointers. (Here is a video tutorial on dangling pointers by one of RareSkills' instructors).

```solidity
File: contracts/mixins/LiquidityMining.sol

17        StorageLayout.TickTracking memory tickTrackingData = StorageLayout

32        StorageLayout.TickTracking memory tickTrackingData = StorageLayout

95                    TickTracking memory tickTracking = tickTracking_[poolIdx][i][tickTrackingIndex];

169        CurveMath.CurveState memory curve = curves_[poolIdx];

261        CurveMath.CurveState memory curve = curves_[poolIdx];

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol

## [G-10] Using > 0 costs more gas than != 0

```solidity
File: contracts/mixins/LiquidityMining.sol

141                if (numTickTracking > 0) {

182            if (overallInRangeLiquidity > 0) {

192        if (rewardsToSend > 0) {

276            if (overallTimeWeightedLiquidity > 0) {

285        if (rewardsToSend > 0) {

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L141

## [G-11] Do-While loops are cheaper than for loops

If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.

For-Example:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

// times == 10 in both tests
contract Loop1 {
    function loop(uint256 times) public pure {
        for (uint256 i; i < times; ) {
            unchecked {
                ++i;
            }
        }
    }
}

contract Loop2 {
    function loop(uint256 times) public pure {
        if (times == 0) {
            return;
        }

        uint256 i;

        do {
            unchecked {
                ++i;
            }
        } while (i < times);
    }
}

```

```solidity
File: contracts/mixins/LiquidityMining.sol

88            for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {

139            for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {

174        for (uint256 i; i < weeksToClaim.length; ++i) {

184                for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j) {

266        for (uint256 i; i < weeksToClaim.length; ++i) {


```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L88

## [G-12] Use uint256(1)/uint256(2) instead for true and false boolean states

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

```solidity
File: contracts/mixins/LiquidityMining.sol

190            concLiquidityRewardsClaimed_[poolIdx][posKey][week] = true;

283            ambLiquidityRewardsClaimed_[poolIdx][posKey][week] = true;

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L190

## [G-13] Split require statements that have boolean expressions

When we split require statements, we are essentially saying that each statement must be true for the function to continue executing.

If the first statement evaluates to false, the function will revert immediately and the following require statements will not be examined. This will save the gas cost rather than evaluating the next require statement.

```solidity

// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract Require {
    function dontSplitRequireStatement(
        uint256 x,
        uint256 y
    ) external pure returns (uint256) {
        require(x > 0 && y > 0); // both conditon would be evaluated, before reverting or notreturn x \* y;
    }
}

contract RequireTwo {
    function splitRequireStatement(
        uint256 x,
        uint256 y
    ) external pure returns (uint256) {
        require(x > 0); // if x <= 0, the call reverts and "y > 0" is not checked.
        require(y > 0);

        return x * y;
    }
}

```

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol

67        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

76        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L67

## [G-14] Use ++i instead of i++ to increment

The reason behind this is in way ++i and i++ are evaluated by the compiler.

i++ returns i(its old value) before incrementing i to a new value. This means that 2 values are stored on the stack for usage whether you wish to use it or not. ++i on the other hand, evaluates the ++ operation on i (i.e it increments i) then returns i (its incremented value) which means that only one item needs to be stored on the stack.

```solidity
File: contracts/mixins/LiquidityMining.sol

122                                tickTrackingIndex++;

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L122
