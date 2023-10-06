|Number|Issue|Instances|
|:-:|:-|:-:|
|[NC-1](#NC-1)|Overwriting set values for `poolIdx` and `weekFrom`|1|
|[NC-2](#NC-2)|Potential gas issues with `setConcRewards`|1|
|[NC-3](#NC-3)|Improvement in `LiquidityMiningPath#claimConcentratedRewards()`|1|
|[NC-4](#NC-4)|Function interruption due to "Already claimed" check|2|
|[NC-5](#NC-5)|DoS in `accrueAmbientPositionTimeWeightedLiquidity()`|1|
|[NC-6](#NC-6)|Skipped weeks issue in `accrueAmbientPositionTimeWeightedLiquidity()`|1|
|[NC-7](#NC-7)|Checks absence on `lookupPosition`|1|
|[NC-8](#NC-8)|Gas concerns in `claimAmbientRewards()`|1|
|[NC-9](#NC-9)|Event omission in `claimAmbientRewards()`|1|
|[NC-10](#NC-10)|Floating point arithmetic concern|1|
|[NC-11](#NC-11)|Gas cost concern with `accrueAmbientPositionTimeWeightedLiquidity()`|1|

---

# NC-1 Overwriting set values for `poolIdx` and `weekFrom`

There's no check to prevent overwriting already set values for a particular `poolIdx` and `weekFrom` combination in the `concRewardPerWeek_` mapping. This might lead to unforeseen results and data integrity issues. 

# NC-2 Potential gas issues with `setConcRewards`

The usage of the `while` loop without bounds can result in high gas consumption, making transactions expensive or even impossible to process in extreme cases. It also opens up possibilities for malicious actors to grief the contract.

# NC-3 Improvement in `LiquidityMiningPath#claimConcentratedRewards()`

The direct usage of `msg.sender` in this function can be optimized. It's generally good practice to provide flexibility and make functions more versatile by allowing parameters instead.

# NC-4 Function interruption due to "Already claimed" check

These checks, though essential for security, might end up stopping the entire function process. An alternative design should be considered, perhaps skipping the loop iteration instead of stopping the whole function.

# NC-5 DoS in `accrueAmbientPositionTimeWeightedLiquidity()`

A potential Denial of Service could occur if the difference between `lastAccrued` and `block.timestamp` is significantly large, making the function very gas-heavy.

# NC-6 Skipped weeks issue in `accrueAmbientPositionTimeWeightedLiquidity()`

In case this function isn't called for a prolonged period, weeks in between might not be correctly accounted for, potentially resulting in inaccurate reward calculations.

# NC-7 Checks absence on `lookupPosition`

Assuming always correct data without validation might result in unexpected behaviors. Validation checks should be added.

# NC-8 Gas concerns in `claimAmbientRewards()`

Considering the loop's presence and multiple storage updates, there's a potential risk of the function consuming too much gas when called with a large array of weeks.

# NC-9 Event omission in `claimAmbientRewards()`

Emitting events is crucial for dApps to monitor and track. Their omission might make interaction and monitoring difficult.

# NC-10 Floating point arithmetic concern

Solidity doesn't handle floating-point numbers natively. It's essential to be cautious when performing divisions to avoid precision loss.

# NC-11 Gas cost concern with `accrueAmbientPositionTimeWeightedLiquidity()`

Multiplications, especially with large numbers, might consume more gas, making the function expensive.
