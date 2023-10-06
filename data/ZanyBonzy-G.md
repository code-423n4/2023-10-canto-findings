1. Instead of using abi.decode, assembly can be used to efficiently extract calldata values. That way call data values that will not be used will be ignored.
https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L28
https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L43
