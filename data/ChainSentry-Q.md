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
