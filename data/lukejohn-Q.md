QA1. LiquidMiningPath.protocolCmd() lacks the modifier ``payable``. As a result, nobody can fund the contract by calling this function since it will always revert. Note that the function call setConcRewards() and setAmbRewards(), which have the payable modifier. 

[https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L26-L37](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L26-L37)

Mitigation: add a modifier to the LiquidMiningPath.protocolCmd() function.

