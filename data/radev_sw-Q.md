|Number|Issue|Instances|
|:-:|:-|:-:|
|[NC-1](#NC-1)|Overwriting set values for `poolIdx` and `weekFrom`|1|
|[NC-2](#NC-2)|Potential gas issues with `setConcRewards`|1|
|[NC-3](#NC-3)|Improvement in `LiquidityMiningPath#claimConcentratedRewards()`|1|
|[NC-4](#NC-4)|Function interruption due to "Already claimed" check|2|
|[NC-5](#NC-5)|DoS in `accrueAmbientPositionTimeWeightedLiquidity()`|1|
|[NC-6](#NC-6)|Skipped weeks issue in `accrueAmbientPositionTimeWeightedLiquidity()`|1|
|[NC-7](#NC-7)|Checks absence on `lookupPosition`|1|
|[NC-8](#NC-8)|Event emitting in `claimAmbientRewards()`|1|
|[NC-9](#NC-9)|Floating point arithmetic concern|1|

---

# NC-1 Overwriting set values for `poolIdx` and `weekFrom`

There's no check to prevent overwriting already set values for a particular `poolIdx` and `weekFrom` combination in the `concRewardPerWeek_` mapping. This might lead to unforeseen results and data integrity issues. 

Example:

- If `concRewardPerWeek_[1][5] = 20;` has already been set,
- And the user (or governance, when the commented out permission check is enabled) calls `setConcRewards` with `poolIdx = 1`, `weekFrom = 5`, and `weeklyReward = 10`,
- Then after the function execution, the value would indeed be overwritten to `concRewardPerWeek_[1][5] = 10;`.
  If there's a desire to prevent overwriting already set values, an additional check would need to be added, such as:

```solidity
require(concRewardPerWeek_[poolIdx][weekFrom] == 0, "Reward already set for this week");
```

This would throw an exception and halt the function if there's already a reward set for the given pool and week.

# NC-2 Potential gas issues with `setConcRewards`

The usage of the `while` loop without bounds can result in high gas consumption, making transactions expensive or even impossible to process in extreme cases. It also opens up possibilities for malicious actors to grief the contract.


**Gas Griefing**:
A malicious actor (or even an unintentional user) could call this function with a very large range between `weekFrom` and `weekTo`, causing the function to execute the `while` loop an excessive number of times. This could result in a transaction that consumes a vast amount of gas, potentially causing the transaction to run out of gas or become prohibitively expensive to execute.

If there are no restrictions on who can call this function (the governance check is commented out), it could be used as an attack vector by malicious users to "grief" the contract, by repeatedly calling the function with large ranges to waste the gas of any monitoring or interacting services.

**Gas Limit Issues**:
If the range between `weekFrom` and `weekTo` is large enough, it could exceed the block gas limit. This would make it impossible for the transaction to be processed, effectively blocking the operation.

**Gas Griefing Explained**:

Gas griefing is when someone purposely makes certain actions on the Ethereum network (like using a function in a smart contract) use up more gas, or computing power, than they should. This can cause problems and inconveniences.

**Impacts**:

1. **Costly Actions**: Making an action on the network can become more expensive for users.
2. **Blocked Functions**: Some actions might become impossible to do if they use up too much gas.
3. **Slower Network**: Too many of these "expensive" actions can make the whole network slow down.
4. **Loss of Trust**: Users might start doubting the reliability of a project if it's often targeted by gas griefing.
5. **Good for Miners, Bad for Users**: Miners, who validate transactions, might earn more because of the higher fees. But this is bad for regular users who have to pay these fees.
6. **Network Traffic Jams**: A lot of these "expensive" actions can cause a traffic jam on the network, making everything slow.
7. **Opens Doors for Other Attacks**: Gas griefing can create opportunities for other harmful actions on the network.
8. **Extra Costs for Projects**: Projects have to spend more to monitor for gas griefing and might need to fix or update their systems if attacked.

In simple words, gas griefing is like someone intentionally causing traffic jams on a highway, making journeys longer and more costly for everyone. It's essential to design roads (or in this case, smart contracts) to prevent these jams and keep everything running smoothly.

**Recommendation**:

1. **Add an Upper Bound**: Limit the difference between `weekFrom` and `weekTo` to a maximum value, e.g.,

   ```solidity
   require(weekTo - weekFrom <= MAX_WEEKS_RANGE, "Range too large");
   ```

   where `MAX_WEEKS_RANGE` is a predefined constant that sets a reasonable maximum number of weeks for which rewards can be set in a single transaction.

2. **Re-enable Governance Check**: If it's intended for only certain addresses (like a governance mechanism) to call this function, ensure that you uncomment and properly configure the `require` check for permissions. This way, you can limit potential abuse by restricting who can set rewards.

# NC-3 Improvement in `LiquidityMiningPath#claimConcentratedRewards()`

The direct usage of `msg.sender` in this function can be optimized. It's generally good practice to provide flexibility and make functions more versatile by allowing parameters instead.

# NC-4 Function interruption due to "Already claimed" check

These checks, though essential for security, might end up stopping the entire function process. An alternative design should be considered, perhaps skipping the loop iteration instead of stopping the whole function.

```solidity
        require(
            !concLiquidityRewardsClaimed_[poolIdx][posKey][week],
            "Already claimed"
        );
```

```solidity
        require(
            !ambLiquidityRewardsClaimed_[poolIdx][posKey][week],
            "Already claimed"
        );
```

# NC-5 DoS in `accrueAmbientPositionTimeWeightedLiquidity()`

A potential Denial of Service could occur if the difference between `lastAccrued` and `block.timestamp` is significantly large, making the function very gas-heavy.

# NC-6 Skipped weeks issue in `accrueAmbientPositionTimeWeightedLiquidity()`

In case this function isn't called for a prolonged period, weeks in between might not be correctly accounted for, potentially resulting in inaccurate reward calculations.

# NC-7 Checks absence on `lookupPosition`

Assuming always correct data without validation might result in unexpected behaviors. Validation checks should be added.

# NC-8 Event emitting in `claimAmbientRewards()`

Emitting events is crucial for dApps to monitor and track. Their omission might make interaction and monitoring difficult.

# NC-9 Floating point arithmetic concern

Solidity doesn't handle floating-point numbers natively. It's essential to be cautious when performing divisions to avoid precision loss.
