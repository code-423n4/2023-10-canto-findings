# Low 

# Summary
|      | issue  | insteance  |
|------|--------|------------|
|[L-01]| Unprotected Initializer in LiquidityMining.initTickTracking(bytes32,int24)|1|
|[L-02]| Uninitialized Local Variable in LiquidityMining.claimConcentratedRewards|1|
|[L-03]| Dubious Typecast in LiquidityMining.accrueConcentratedGlobalTimeWeightedLiquidity(bytes32,CurveMath.CurveState)|3|
|[L-04]| Low Level Call to Custom Address in LiquidityMining.claimConcentratedRewards|2|
|[L-05]| Dangerous Usage of block.timestamp in LiquidityMining.accrueAmbientPositionTimeWeightedLiquidity|8|
|[L-06]| Lack of Access Control|4|
|[L-07]| Lack of Self-Destruct Function|~|

## [L-01] Unprotected Initializer in LiquidityMining.initTickTracking(bytes32,int24)
- The function 'initTickTracking' is an initializer function that is not protected. This could potentially allow unauthorized or unintended calls to this function, which could lead to incorrect initialization of state variables.

- Initializer functions are used to set initial state for a contract or a specific function. If these functions are not properly protected, they can be called by anyone, potentially leading to incorrect state initialization or other unexpected behavior.

```solidity
    function initTickTracking(bytes32 poolIdx, int24 tick) internal {
        StorageLayout.TickTracking memory tickTrackingData = StorageLayout
            .TickTracking(uint32(block.timestamp), 0);
        tickTracking_[poolIdx][tick].push(tickTrackingData);
    }
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L16-L20

`Recommendation`: It is recommended to protect initializer functions with appropriate access control modifiers or require statements to ensure that they can only be called by authorized entities and under the correct conditions.



## [L-02] Uninitialized Local Variable in LiquidityMining.claimConcentratedRewards
- The local variable 'rewardsToSend' is never initialized in the function 'claimConcentratedRewards'. This could potentially lead to unexpected behavior or vulnerabilities in the contract.

- Uninitialized local variables can lead to unexpected behavior as they may contain arbitrary values. In this case, the 'rewardsToSend' variable is used without being initialized, which could lead to unexpected results.


```solidity
173          uint256 rewardsToSend;
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L173

`Recommendation`: It is recommended to initialize all local variables to prevent any unexpected behavior. If a variable is meant to be initialized to zero, it should be explicitly set to zero to improve code readability and ensure the correct behavior of the function.

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L265

## [L-03] Dubious Typecast in LiquidityMining.accrueConcentratedGlobalTimeWeightedLiquidity(bytes32,CurveMath.CurveState)
- The function 'accrueConcentratedGlobalTimeWeightedLiquidity' contains dubious typecasts from uint256 to uint32 in the expressions 'timeWeightedWeeklyGlobalConcLiquidityLastSet_[poolIdx] = uint32(block.timestamp)' and 'dt = uint32(block.timestamp - time)'. These could potentially lead to unexpected behavior or vulnerabilities in the contract.

- Typecasting is a way to convert a variable from one data type to another. However, if not done carefully, it can lead to unexpected results, especially when casting between different integer sizes. In this case, the typecasts from uint256 to uint32 could potentially lead to overflow if the original values are greater than 2^32 - 1.

```solidity
    function accrueConcentratedGlobalTimeWeightedLiquidity(
        bytes32 poolIdx,
        CurveMath.CurveState memory curve
    ) internal {
        uint32 lastAccrued = timeWeightedWeeklyGlobalConcLiquidityLastSet_[
            poolIdx
        ];
        // Only set time on first call
        if (lastAccrued != 0) {
            uint256 liquidity = curve.concLiq_;
            uint32 time = lastAccrued;
            while (time < block.timestamp) {
                uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
                uint32 dt = uint32(
                    nextWeek < block.timestamp
                        ? nextWeek - time
                        : block.timestamp - time
                );
                timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * liquidity;
                time += dt;
            }
        }
        timeWeightedWeeklyGlobalConcLiquidityLastSet_[poolIdx] = uint32(
            block.timestamp
        );
    }
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L39-L65

`Recommendation`: It is recommended to use clear constants and ensure that the values being typecast do not exceed the range of the target data type. If the value can potentially be greater than 2^32 - 1, consider using a different data type or add checks to ensure the value is within the acceptable range before performing the typecast.

`Same issue in these line`

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L24-L35

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#65-L154


## [L-04] Low Level Call to Custom Address in LiquidityMining.claimConcentratedRewards
- The function 'claimConcentratedRewards' contains a low level call to a custom address. This could potentially lead to unexpected behavior or vulnerabilities in the contract.

- Low level calls to custom addresses can be risky as they can potentially interact with malicious contracts or functions. This can lead to a variety of security issues such as reentrancy attacks, unexpected state changes, or loss of funds.

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L156-L196

`Recommendation`: It is recommended to avoid low level calls to custom addresses and interact with other contracts through well-defined interfaces and access control mechanisms. If a low level call is necessary, ensure that the address being called is trusted and that the call is protected with appropriate security measures such as reentrancy guards or checks-effects-interactions patterns.

`Same issue in this line`
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L256-L289


## [L-05] Dangerous Usage of block.timestamp in LiquidityMining.accrueAmbientPositionTimeWeightedLiquidity
- The function 'accrueAmbientPositionTimeWeightedLiquidity' uses block.timestamp for comparisons. This could potentially lead to unexpected behavior or vulnerabilities in the contract.

-  The block.timestamp value can be manipulated by miners to some degree. This can lead to a variety of security issues if it is used for critical logic in the contract, such as comparisons or calculations.


```solidity
237   while (time < block.timestamp) {

241   nextWeek < block.timestamp    

// @audit also in this line of code (94,105,107,119,98-102,116)
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L237-L241

Recommendation: It is recommended to avoid relying on block.timestamp for critical contract logic. If time-based logic is necessary, consider using a mechanism that cannot be manipulated by miners, such as block numbers for relative time or an external time oracle for absolute time.

## [L-06] Lack of Access Control
The contract lacks proper access control mechanisms. The protocolCmd, userCmd, setConcRewards, and setAmbRewards functions can be called by any address, which could potentially allow an attacker to manipulate the reward rates or claim rewards that they are not entitled to.

In the setConcRewards and setAmbRewards functions, there are commented out require statements that check if the caller is the governance. These statements should be uncommented to enforce access control and ensure that only the governance can call these functions.

```solidity
26  function protocolCmd(bytes calldata cmd) public virtual {

41  function userCmd(bytes calldata input) public payable {    

65  function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {    

74  function setAmbRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {    
```
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol

`Recommendations:`
- Uncomment the require statements in the setConcRewards and setAmbRewards functions to enforce that only the governance can call these functions.

- Implement similar access control mechanisms in the protocolCmd and userCmd functions to prevent unauthorized manipulation of the contract.

- Consider using the OpenZeppelin's Ownable contract or similar to manage access control in a standardized and secure manner.

- Thoroughly test all changes to ensure that access control is enforced as expected and that there are no regressions in functionality.


## [L-07] Lack of Self-Destruct Function
The contract does not have a self-destruct function. This is a potential security flaw as it means the contract cannot be stopped if a bug or security vulnerability is discovered. A self-destruct function allows the contract owner to stop the contract and send the remaining Ether in the contract to a specified address.

Locations:
Entire LiquidityMiningPath contract

Recommendations:
Implement a self-destruct function that can only be called by the contract owner. This function should use the selfdestruct keyword to destroy the contract and send the remaining Ether to the owner.

Ensure that the self-destruct function can only be called by the contract owner to prevent unauthorized destruction of the contract.

Thoroughly test the self-destruct function to ensure it works as expected and does not introduce any new vulnerabilities.

Consider the implications of adding a self-destruct function. While it can provide a way to stop the contract in case of a vulnerability, it can also be a point of centralization and a potential security risk if the owner's private key is compromised.