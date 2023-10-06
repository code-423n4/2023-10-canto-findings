https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol

1)Safety and Error Handling: In functions like claimConcentratedRewards and claimAmbientRewards, validate the owner address to prevent unauthorized access.
You can add owner validation to functions like claimConcentratedRewards and claimAmbientRewards to prevent unauthorized access.

modifier onlyOwner(bytes32 poolIdx, address owner) {
    require(owner == msg.sender, "Unauthorized");
    _;
}

function claimConcentratedRewards(
    address payable owner,
    bytes32 poolIdx,
    int24 lowerTick,
    int24 upperTick,
    uint32[] memory weeksToClaim
) internal onlyOwner(poolIdx, owner) {
    // Function logic here
}

function claimAmbientRewards(
    address owner,
    bytes32 poolIdx,
    uint32[] memory weeksToClaim
) internal onlyOwner(poolIdx, owner) {
    // Function logic here
}

In this proof of concept:

We define a modifier called onlyOwner that takes the poolIdx and owner address as parameters.
Inside the modifier, we use require to check if the owner address matches msg.sender. If they don't match, the "Unauthorized" error message is displayed, and the function call is reverted.
We apply the onlyOwner modifier to the claimConcentratedRewards and claimAmbientRewards functions by using the onlyOwner(poolIdx, owner) syntax.
Now, these functions will only execute if the caller's address matches the specified owner, preventing unauthorized access to claim rewards for positions that do not belong to the caller.

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol

1)Payable Functions: Some functions are marked as payable, even though they do not handle msg.value. While this is not a critical issue, it's a good practice to remove the payable modifier from functions that don't interact with Ether to avoid any confusion.

2)Access Control: The setConcRewards and setAmbRewards functions are currently commented out with require statements to check for access control (msg.sender == governance_). You should clarify how access control is implemented and whether these functions should be accessible to specific addresses or roles.