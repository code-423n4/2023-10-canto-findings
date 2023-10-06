
##LOW RISK 1

Zero Reward Check Missing

Lines of Code Affected

    LiquidityMining.sol: claimConcentratedRewards and claimAmbientRewards functions


Impact

Users could incur unnecessary gas costs when claiming zero rewards.
Proof of Concept

```solidity

function claimConcentratedRewards(
    address payable owner,
    bytes32 poolIdx,
    int24 lowerTick,
    int24 upperTick,
    uint32[] memory weeksToClaim
) internal {
    // Missing check for zero rewards
}

```


Recommended Mitigation Steps

Implement a check to exit the function if rewards are zero, thereby saving gas costs.





##LOW RISK 2

Tick Crossing Validation Missing

Lines of Code Affected

    LiquidityMining.sol: crossTicks function

Impact

Absence of validation for exitTick and entryTick could lead to incorrect tick crossing data, affecting reward calculations.


Proof of Concept

```solidity

function crossTicks(
    bytes32 poolIdx,
    int24 exitTick,
    int24 entryTick
) internal {
    // Missing validation for exitTick and entryTick
}

```

Recommended Mitigation Steps

Add validation to ensure exitTick and entryTick are valid sequential ticks.





##LOW RISK 3

Unchecked Tick Initialization


Lines of Code Affected

    LiquidityMining.sol: initTickTracking function

Impact

Failure to check for existing tick tracking data could lead to overwriting, thereby affecting the accuracy of reward calculations.


Proof of Concept

```solidity

function initTickTracking(bytes32 poolIdx, int24 tick) internal {
    // Missing check for existing tick tracking data
    StorageLayout.TickTracking memory tickTrackingData = StorageLayout
        .TickTracking(uint32(block.timestamp), 0);
    tickTracking_[poolIdx][tick].push(tickTrackingData);
}
```

Recommended Mitigation Steps

Add a check if tick tracking already exists for the passed parameters before initializing
Emit an event on initialization so external callers can track init status




