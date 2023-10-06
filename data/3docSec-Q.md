[L-01] LiquidityMiningPath.protocolCmd should be payable
The function `LiquidityMiningPath.protocolCmd` is called by `CrocSwapDex.protocolCmd` which is payable.
Consider making it payable too, especially because this way it would be possible to enable and fund rewards with one `protocolCmd` call.

[N-01] Consider removing the hardhat console import from production code
[LevelBook.sol](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LevelBook.sol#L9) imports the hardhat console. Consider removing this import before production deployment.