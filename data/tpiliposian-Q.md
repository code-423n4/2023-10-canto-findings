# Code Optimization Suggestion

## Description

In the `LiquidityMiningPath.sol` two functions `setConcRewards` and `setAmbRewards` share common requirements. While this doesn't represent a security issue, it's a potential area for code optimization and readability improvement.

Currently, both functions perform the same checks: they validate that `weekFrom` and `weekTo` are divisible by `WEEK`, and (maybe) they both have a governance check that is commented out. This repetition not only makes the code longer but also increases the chances of errors when maintaining the contract.

## Code Snippet

https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L65-L81

```solidity
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

## Remediation 

To optimize the code and improve readability, consider creating a custom modifier, to centralize the governance and `weekFrom % WEEK == 0 && weekTo % WEEK == 0` checks. Then, apply this modifier to both functions. This change will reduce code duplication and make it easier to manage access control requirements in the future.
