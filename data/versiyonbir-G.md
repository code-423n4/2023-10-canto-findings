# FUNCTION SHOULD BE EXTERNAL
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L41-L52
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L26-L37

A function with public visibility modifier was detected that is not called internally.
public and external differs in terms of gas usage. The former use more than the latter when used with large arrays of data. This is due to the fact that Solidity copies arguments to memory on a public function while external read from calldata which a cheaper than memory allocation.

If you know the function you create only allows for external calls, use the external visibility modifier instead of public. It provides performance benefits and you will save on gas.

# SPLITTING REQUIRE STATEMENTS
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L67-L67
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L76-L76
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L319-L321

Require statements when combined using operators in a single statement usually lead to a larger deployment gas cost but with each runtime calls, the whole thing ends up being cheaper by some gas units.

It is recommended to separate the require statements with one statement/validation per line.

# CUSTOM ERRORS TO SAVE GAS
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L35-L35
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L50-L50

The contract was found to be using revert() statements. Since Solidity v0.8.4, custom errors have been introduced which are a better alternative to the revert.
This allows the developers to pass custom errors with dynamic data while reverting the transaction and also making the whole implementation a bit cheaper than using revert.
It is recommended to replace all the instances of revert() statements with error() to save gas.


# CHEAPER INEQUALITIES IN IF()
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L192-L192
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L285-L285
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L283-L283

The contract was found to be doing comparisons using inequalities inside the if statement.
When inside the if statements, non-strict inequalities (>=, <=) are usually cheaper than the strict equalities (>, <).

It is recommended to go through the code logic, and, if possible, modify the strict inequalities with the non-strict ones to save ~3 gas as long as the logic of the code is not affected.

# UNNECESSARY CHECKED ARITHMETIC IN LOOP
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L174-L174
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L184-L184
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L266-L266

Increments inside a loop could never overflow due to the fact that the transaction will run out of gas before the variable reaches its limits. Therefore, it makes no sense to have checked arithmetic in such a place.

It is recommended to have the increment value inside the unchecked block to save some gas.

# CHEAPER CONDITIONAL OPERATORS
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L141-L141
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L192-L192
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L285-L285
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L283-L283

During compilation, x != 0 is cheaper than x > 0 for unsigned integers in solidity inside conditional statements.

Consider using x != 0 in place of x > 0 in uint wherever possible.


# ARRAY LENGTH CACHING
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L174-L191
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L266-L284

During each iteration of the loop, reading the length of the array uses more gas than is necessary. In the most favorable scenario, in which the length is read from a memory variable, storing the array length in the stack can save about 3 gas per iteration. In the least favorable scenario, in which external calls are made during each iteration, the amount of gas wasted can be significant.

Consider storing the array length of the variable before the loop and use the stored length instead of fetching it in each iteration.

# STORAGE VARIABLE CACHING IN MEMORY
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L13-L13

The contract LiquidityMining is using the state variable WEEK multiple times in the function accrueConcentratedGlobalTimeWeightedLiquidity, accrueConcentratedPositionTimeWeightedLiquidity, accrueAmbientGlobalTimeWeightedLiquidity, accrueAmbientPositionTimeWeightedLiquidity.

SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).

