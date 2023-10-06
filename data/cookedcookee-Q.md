## [L-01] Claimable week elements are not forced to conform to the week standard used throughout

The `uint32[] memory weeksToClaim` input variable in both the `claimConcentratedRewards()` and `claimAmbientRewards()` functions do not enforce the week normalisation standard used throughout the contract. This exacerbates the unbounded input array issue highlighted already.

Github [link](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L175)

## Recommended Mitigation

Consider requiring that every element in `uint32[] memory weeksToClaim` satisfies an additional require check in the respective for-loops as follows.

```solidity
File: contracts/mixins/LiquidityMining.sol
175:             uint32 week = weeksToClaim[i];
176:             require(week % WEEK == 0, "Invalid week");

268:             uint32 week = weeksToClaim[i];
269:             require(week % WEEK == 0, "Invalid week");
```

## [N-01] Add clarifying comment to storage variable

The `tickTrackingIndexAccruedUpTo_` storage variable lacks a clarifying comment, such as those found with all other equally complex storage variables.

Github [link](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/StorageLayout.sol#L192)

## Recommended Mitigation

Consider adding a clarifying comment to the `tickTrackingIndexAccruedUpTo_` storage variable.

```solidity
File: contracts/mixins/StorageLayout.sol
191:     // Pool -> Position -> Tick -> Index
192:     mapping(bytes32 => mapping(bytes32 => mapping(int32 => uint32))) tickTrackingIndexAccruedUpTo_;
```

## [N-02] Remove commented out logic

No longer used logic remains as commented out code.

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol
66:         // require(msg.sender == governance_, "Only callable by governance");

75:         // require(msg.sender == governance_, "Only callable by governance");
```

Github [link](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L66)

## Recommended Mitigation

Remove lines 66 and 75 from `LiquidityMiningPath.sol`.