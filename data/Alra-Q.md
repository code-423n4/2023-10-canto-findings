## Summary
There are portions of commented code in some files.
## QA Reports

The LiquidityMiningPath contract includes commented out lines of code without giving developers enough context on why those lines have been discarded, thus apparently providing them with little to no value at all.

As the purpose of these lines is unclear and may confuse future developers and external contributors, consider removing them from the code base. If they are to provide some sort of documentation of an interface, consider extracting them to a separate document where a deeper and more thorough explanation could be included.

## Instances
[line 66](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L66)
[line 75](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L75)

## Mitigation
Remove commented code