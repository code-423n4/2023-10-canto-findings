`Low Risk & Non Critical Findings 5`
# QA Report

**Low-Risk Findings List**



| Number | Issue Detail | Severity |
|-----:|----|-----|
| [`L-01`](#L-01-multiplication-on-the-result-of-division) | `Multiplication` on the Result of `Division` | Low |
| [`L-02`](#L-02-not-enough-coverage-of-functions-and-contracts-in-the-docs) | Not `Enough` Coverage of `Functions` and `Contracts ` In the `Docs` | Low |
> Total ~ 3 Issues


**Non Critical Issues List**
| Number | Issue Details | Instances |
|-----:|----|-----|
| [`NC-01`](#NC-01-dt-as-a-local-variable-name-causes-confusion) | `dt` as a `Local` variable name causes `confusion` | Non-Critical |
| [`NC-02`](#NC-02-there-should-be-a-min-and-max-weekfrom-weekto-limit-to-be-set) | There should be a `Min` and `Max` `weekFrom/weekTo` Limit to be set. | Non-Critical |
| [`NC-03`](#NC-03-missing-events-for-certain-critical-functions) | `Missing` Events for certain `Critical` functions. | Non-Critical |


# Low Risk Findings.

## L-01 `Multiplication` on the Result of `Division`
### Summary 
 Dividing an integer by another integer will often result in a loss of precision. When the result is multiplied by another number, the loss of precision is magnified, often to material magnitudes. (X / Z) * Y should be re-written as (X * Y) / Z.
 **Note** This wasn't found in the bot report.
### Vulnerability Details 
you can see instances of these issues
```solidity
 File: LiquidityMining.sol
51:                   uint32 currWeek = uint32((time / WEEK) * WEEK);
52:                   uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
............... 
96:                       uint32 currWeek = uint32((time / WEEK) * WEEK);
97:                       uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
................
208:                  uint32 currWeek = uint32((time / WEEK) * WEEK);
209:                  uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
................
238:                  uint32 currWeek = uint32((time / WEEK) * WEEK); 
239:                  uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

```
### Recommended Mitigation Step.
multiply before division. An instance of such mitigation would be
```diff
 File: LiquidityMining.sol
-  51:                   uint32 currWeek = uint32((time / WEEK) * WEEK);
+  51:                   uint32 currWeek = uint32((WEEK / WEEK) * time);
```
## L-02 Not `Enough` Coverage of `Functions` and `Contracts` In the `Docs`
### Summary 
 There isn't enough coverage on the user flows and functions in canto liquidity mining, which can lead to confusion and incorrect assumptions made when auditing the specific codebases.
### Vulnerability Details 
the only place where there is anything said about the protocol is on the contest page on the code4rena site [Link here]() and you can see that it doesn't do a good job of explaining how the contract and the functions logic are supposed to interact and be interacted with.  
### Recommended Mitigation Step
Create good documentation of the contracts and the functions stating and explaining how they are  meant to be performed.
.
# Non Critical Issues List.

## NC-01 `dt` as a `Local` variable name causes `confusion`
### Summary 
 The local variable named `dt` doesn't give a complete and accurate context as to what it is meant for in `LiquidityMining.sol` and can cause confusion.
### Vulnerability Details 
 Local variable dt is declared in the:
 - [`accrueConcentratedGlobalTimeWeightedLiquidity()`](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L53C1-L53C36)
 - [`accrueConcentratedPositionTimeWeightedLiquidity()`](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L98C1-L98C40)
 - [`accrueAmbientGlobalTimeWeightedLiquidity()`](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L210C1-L210C36)
 - [`accrueAmbientPositionTimeWeightedLiquidity()`](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L240C1-L240C36)

Functions respectively, however, the variable name gives little context as to what is stored and nothing is said about it in the `read.md`
### Recommended Mitigation Step
choose a better name for the variable that entails what it is meant for.

## NC-02 There should be a `Min` and `Max` `weekFrom` and  `weekTo` limit to be set
### Summary 
There should be a minimum and maximum amount of weeks that can be set to have weekly rewards in order to help protect governance against any input mistake made.
### Vulnerability Details 
you can see the instance of this issue.
```solidity
File: LiquidityMiningPath.sol
    function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
        // require(msg.sender == governance_, "Only callable by governance");
        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            concRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
            weekFrom += uint32(WEEK);
        }
    }

    function setAmbRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
        // require(msg.sender == governance_, "Only callable by governance");
        require(weekFrom % WEEK == 0 && weekTo % WEEK == 0, "Invalid weeks");
        while (weekFrom <= weekTo) {
            ambRewardPerWeek_[poolIdx][weekFrom] = weeklyReward;
            weekFrom += uint32(WEEK);
        }
    }
```
### Recommended Mitigation Step
add a check to ensure against setting weekly rewards for too many weeks ahead.
## NC-03 `Missing` Events for certain `Critical` functions.
### Summary 
 certain critical functions lack events which are very useful for contracts deployed on the EVM
 **Note** This is not the same as the issue in the bot reports.
### Vulnerability Details 
you can see the Instances of critical functions that do not emit events. when claiming rewards or when accruals are being made.
- [`claimConcentratedRewards()`](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L156C1-L162C17)
- [`claimAmbientRewards()`](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L256C1-L260C17)
- [`accrueAmbientGlobalTimeWeightedLiquidity()`](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L198C1-L201C17)
- [`accrueAmbientPositionTimeWeightedLiquidity()`](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L224C1-L227C17)
- [`accrueConcentratedPositionTimeWeightedLiquidity()`](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L69C1-L74C17)
- [`accrueConcentratedGlobalTimeWeightedLiquidity()`](https://github.com/code-423n4/2023-10-canto/blob/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/mixins/LiquidityMining.sol#L39C1-L42C17)
### Recommended Mitigation Step
emit events for the given functions.