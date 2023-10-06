## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |Internal functions that are not called by the contract should be removed to save deployment gas | |15| | 
| [G-02] |Counting down in for statements is more gas efficient  | |1| | 
| [G-03] | The result of a function call should be cached rather than re-calling the function   | |2| | 
| [G-04] | Use uint256(1)/uint256(2) instead for true and false boolean states | |1| | 
| [G-05] |Use assembly to make more efficient back-to-back calls | |1| | 
| [G-06] | Use != 0 instead of > 0 for unsigned integer comparison | |2| | 
| [G-07] |Cache external calls outside of loop to avoid re-calling function on each iteration  | |1| | 
| [G-08] |Duplicated require()/if() checks should be refactored to a modifier or function | |3| | 
| [G-09] |Use Modifiers Instead of Functions To Save Gas | |2| | 
| [G-10] | x += y costs more gas than x = x + y for state variables  | |1| | 
| [G-11] |Don't initialize variables with default value  | |3| | 
| [G-12] |Use calldata instead of memory for function arguments that do not get mutated   | |2| | 






## Gas Optimizations  

## [G-1] Internal functions that are not called by the contract should be removed to save deployment gas

Internal functions in Solidity are only intended to be invoked within the contract or by other internal functions. If an internal function is not called anywhere within the contract, it serves no purpose and contributes unnecessary overhead during deployment. Removing such functions can lead to substantial gas savings.


```solidity
file:   contracts/mixins/TradeMatcher.sol

63      function mintAmbient (CurveMath.CurveState memory curve, uint128 liqAdded, 
                          bytes32 poolHash, address lpOwner)
        internal returns (int128 baseFlow, int128 quoteFlow) {

79     function lockAmbient (CurveMath.CurveState memory curve, uint128 liqAdded)
        internal pure returns (int128, int128) {
        (uint128 base, uint128 quote) = liquidityReceivable(curve, liqAdded);
        return signMintFlow(base, quote);        
    }

100     function burnAmbient (CurveMath.CurveState memory curve, uint128 liqBurned, 
                          bytes32 poolHash, address lpOwner)
        internal returns (int128, int128) {

132     function mintRange (CurveMath.CurveState memory curve, int24 priceTick,
                        int24 lowTick, int24 highTick, uint128 liquidity,
                        bytes32 poolHash, address lpOwner)
        internal returns (int128 baseFlow, int128 quoteFlow) {

170     function burnRange (CurveMath.CurveState memory curve, int24 priceTick,
                        int24 lowTick, int24 highTick, uint128 liquidity,
                        bytes32 poolHash, address lpOwner)
        internal returns (int128, int128) {

240      function mintKnockout (CurveMath.CurveState memory curve, int24 priceTick,
                           KnockoutLiq.KnockoutPosLoc memory loc,
                           uint128 liquidity, bytes32 poolHash, uint8 knockoutBits)
        internal returns (int128 baseFlow, int128 quoteFlow) {

263     function burnKnockout (CurveMath.CurveState memory curve, int24 priceTick,
                           KnockoutLiq.KnockoutPosLoc memory loc,
                           uint128 liquidity, bytes32 poolHash)
        internal returns (int128 baseFlow, int128 quoteFlow) {

287     function claimKnockout (CurveMath.CurveState memory curve, 
                            KnockoutLiq.KnockoutPosLoc memory loc,
                            uint160 root, uint256[] memory proof, bytes32 poolHash)
        internal returns (int128 baseFlow, int128 quoteFlow) {

308     function recoverKnockout (KnockoutLiq.KnockoutPosLoc memory loc,
                              uint32 pivotTime, bytes32 poolHash)
        internal returns (int128 baseFlow, int128 quoteFlow) {

333     function harvestRange (CurveMath.CurveState memory curve, int24 priceTick,
                           int24 lowTick, int24 highTick, bytes32 poolHash,
                           address lpOwner)
        internal returns (int128, int128) {

384     function sweepSwapLiq (Chaining.PairFlow memory accum,
                           CurveMath.CurveState memory curve, int24 midTick,
                           Directives.SwapDirective memory swap,
                           PoolSpecs.PoolCursor memory pool) internal {
            

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L63-65


```solidity
file:   contracts/mixins/MarketSequencer.sol

48       function tradeOverPool (Chaining.PairFlow memory flow,
                            Directives.PoolDirective memory dir,
                            Chaining.ExecCntx memory cntx) internal {

65      function swapOverPool (Directives.SwapDirective memory dir,
                           PoolSpecs.PoolCursor memory pool)
        internal returns (Chaining.PairFlow memory flow) {

156     function harvestOverPool (int24 bidTick, int24 askTick,
                              PoolSpecs.PoolCursor memory pool,
                              uint128 minPrice, uint128 maxPrice, address lpConduit)
        internal returns (int128 baseFlow, int128 quoteFlow) {

239     function initCurve (PoolSpecs.PoolCursor memory pool,
                        uint128 price, uint128 initLiq)
        internal returns (int128 baseFlow, int128 quoteFlow) {
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L48-L50
## [G-2] Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

by changing this logic you can save 12171 gas per one for loop 

Tools used Remix


```solidity
file:  contracts/mixins/MarketSequencer.sol

299    for (uint i = 0; i < dirs.length; ++i) {

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L299
## [G-3]  The result of a function call should be cached rather than re-calling the function

External calls are expensive. Consider caching the following:


### Cache they result of applySwap() external function to save gas 

```solidity
file:  contracts/mixins/MarketSequencer.sol

256    applySwap(flow, dir.swap_, curve, cntx);

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L256


### Cache they result of hasSwapLeft() external function to save gas 

```solidity
file:  contracts/mixins/TradeMatcher.sol

408   hasSwapLeft(curve, swap);

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L408
## [G-4] Use uint256(1)/uint256(2) instead for true and false boolean states

If you don’t use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

```solidity
file:  contracts/mixins/TradeMatcher.sol

394   bool doMore = true;

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L394
## [G-5] Use assembly to make more efficient back-to-back calls

In the instance below, two external calls, both of which take two function parameters, are performed. We can potentially avoid memory expansion costs by using assembly to utilize scratch space + free memory pointer memory offsets for the function calls. We will use the same memory space for both function calls.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls.

```solidity
file:   contracts/mixins/MarketSequencer.sol

285     callMintAmbient(curve, dir.liquidity_, cntx.pool_.hash_) :
        callBurnAmbient(curve, dir.liquidity_, cntx.pool_.hash_);

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L285-L286
## [G-6] Use != 0 instead of > 0 for unsigned integer comparison

It’s generally more gas-efficient to use != 0 instead of > 0 when comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation, while the > 0 comparison requires an additional subtraction operation. As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

```solidity
file:  contracts/mixins/MarketSequencer.sol
 
283   if (dir.liquidity_ > 0) {


```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L283

```solidity
file:   contracts/mixins/TradeMatcher.sol

454    return inLimit && (swap.qty_ > 0);


```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L454
## [G-7] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

```solidity
file:  contracts/mixins/MarketSequencer.sol

302    flow.accumFlow(nextBase, nextQuote);

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L302
## [G-8] Duplicated require()/if() checks should be refactored to a modifier or function

Sign modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.



```solidity
file:  contracts/mixins/TradeMatcher.sol
 

/// @audit this revert is duplicated on line 222

204   if (lpConduit != lockHolder_)

 /// @audit this revert is duplicated on line 225

207   require(doesAccept, "LP");

/// @audit this revert is duplicated on line 435

409   if (doMore)



```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L204
## [G-9] Use Modifiers Instead of Functions To Save Gas

Example of two contracts with modifiers and internal view function:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}

```

```solidity
file:  contracts/mixins/TradeMatcher.sol

350   function signMintFlow (uint128 base, uint128 quote) private pure
        returns (int128, int128) {
        return (base.toInt128Sign(), quote.toInt128Sign());
    }


358      function signBurnFlow (uint128 base, uint128 quote) private pure
        returns (int128, int128){
        return (-(base.toInt128Sign()), -(quote.toInt128Sign()));
    }

```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L350-L353
## [G-10] x += y costs more gas than x = x + y for state variables

```solidity
file:   contracts/mixins/TradeMatcher.sol

495    swap.qty_ -= burnSwap;

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L495
## [G-11] Don't initialize variables with default value


Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with itâ€™s default value costs unnecesary gas.


```solidity
file:   contracts/mixins/TradeMatcher.sol

195     int24 NA_LOW_TICK = 0;

196     int24 NA_HIGH_TICK = 0;

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L195

```solidity
file:  contracts/mixins/MarketSequencer.sol
 
299    for (uint i = 0; i < dirs.length; ++i) {

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L299
## [G-12] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.


```solidity
file:  contracts/mixins/TradeMatcher.sol

289    uint160 root, uint256[] memory proof, bytes32 poolHash)

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/TradeMatcher.sol#L289

```solidity
file:  contracts/mixins/MarketSequencer.sol

295    Directives.ConcentratedDirective[] memory dirs

```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L295


