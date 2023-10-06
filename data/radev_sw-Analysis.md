# Canto Analysis Report 

---

## Canto Liquidity Mining Protocol Audit
- **Purpose**: Canto introduces a new liquidity mining feature specifically for Ambient Finance.
- **Implementation**: This feature is a sidecar contract integrated into Ambient using their proxy contract pattern.
- **New Contracts**:

  1. LiquidityMiningPath.sol: Allows user interactions.
  2. LiquidityMining.sol: Contains core logic.

- **About LiquidityMining Sidecar**:

  - This sidecar is designed to implement a liquidity mining protocol for Ambient. Its goal is to incentivize liquidity for Ambient pools on Canto.

- **Incentive Mechanism**:

  - Incentives focus on a width of liquidity based on the current tick, specifically ranging from currentTick-10 to currentTick+10. Users must provide liquidity within this range to qualify for rewards.

- **Rewards**:

  - The protocol tracks time-weighted liquidity (both global and per-user) for various positions, helping calculate user rewards based on their contribution. Rewards are then disbursed proportionately among liquidity providers.

- **Implementation Details**:

  - Reward rates are set weekly, based on a predetermined total disbursement amount. Governance determines the duration of these reward rates.
  - LiquidityMining.sol is central for rewards and needs special attention during audits.

- **Ambient Overview**:

  - Previously known as CrocSwap, Ambient is a dex allowing liquidity providers to deposit either "ambient" liquidity or concentrated liquidity into token pairs.
  - CrocSwapDex is Ambient's main contract that users interact with.
  - Ambient uses modular "sidecar" contracts to manage functionalities not accommodated in CrocSwapDex due to EVM contract limit.

- **Noteworthy Sidecar Contracts**:
  - BootPath: Installs other sidecars.
  - ColdPath: Creates new pools.
  - WarmPath: Manages liquidity.
  - LiquidityMiningPath: Canto's sidecar for liquidity mining.
  - KnockoutPath, LongPath, MicroPaths, SafeModePath: Handle various specific functionalities.

---
---

## Examples & Scenarios

### **Loop Execution in `LiquidityMining.sol#accrueAmbientPositionTimeWeightedLiquidity()`**:
Let's assume:

1. `WEEK` = 7 days = 604800 seconds.
2. `time` (the starting point from which we are updating) = some arbitrary timestamp representing a Wednesday at noon.
3. `block.timestamp` (current Ethereum block's timestamp) = a timestamp representing 9 days later, which is a Friday evening.

This means the loop will have to account for a full week from the starting Wednesday to the next Wednesday, and then an additional 2 days up to Friday evening.

**Initial Data**:

- `time` = 1,678,688,000 (Wednesday, Noon)
- `block.timestamp` = 1,693,568,000 (Friday, 9 days later)
- `liquidity` = 1000 (just an arbitrary value for this example)

**First Iteration**:

- `currWeek` = `time` rounded down to the last Sunday = 1,676,160,000 (Last Sunday)
- `nextWeek` = `currWeek` + `WEEK` = 1,682,960,000 (Next Sunday)
- `dt` = Since `nextWeek` (Next Sunday) is before `block.timestamp` (Friday, 9 days later), we'll take a full week, i.e., `nextWeek - time` = 1,682,960,000 - 1,678,688,000 = 4,272,000 seconds (5 days from Wednesday Noon to Sunday).
- The liquidity for the current week (from the Last Sunday to the Next Sunday) is incremented by `dt * liquidity` = 4,272,000 \* 1000.
- Now, increment `time` by `dt`: `time` = 1,678,688,000 + 4,272,000 = 1,682,960,000 (Next Sunday).

**Second Iteration**:

- `currWeek` = `time` (which is now the Next Sunday) = 1,682,960,000.
- `nextWeek` = `currWeek` + `WEEK` = 1,689,760,000 (Sunday after the Next Sunday).
- `dt` = Since `block.timestamp` (Friday, 9 days later) is before `nextWeek` (Sunday after the Next Sunday), we'll take the difference from the current `time` (Next Sunday) up to `block.timestamp` (Friday, 9 days later) = 1,693,568,000 - 1,682,960,000 = 10,608,000 seconds (2 days from Sunday to Tuesday).
- The liquidity for the current week (from the Next Sunday to the following Sunday) is incremented by `dt * liquidity` = 10,608,000 \* 1000.
- Now, increment `time` by `dt`: `time` = 1,682,960,000 + 10,608,000 = 1,693,568,000 (Friday, 9 days later).

At this point, `time` is equal to `block.timestamp`, so the loop ends.

**Results**:

- For the week starting at the Last Sunday, the time-weighted liquidity was incremented by the equivalent of 5 days of liquidity.
- For the week starting at the Next Sunday, the time-weighted liquidity was incremented by the equivalent of 2 days of liquidity.


### **`claimAmbientRewards()` Function Walkthrough**:
### Scenario:

1. Assume `owner` is `0x1234567890abcdef1234567890abcdef12345678`.
2. The `poolIdx` is a hypothetical index (hash) `0xabcdef1234567890`.
3. There are 2 weeks to claim in `weeksToClaim` array: `[100, 101]` (These are hypothetical week numbers).
4. Let's assume that the `block.timestamp` is `102`.

### Step-by-Step Execution:

1. **Initialization**:

   - `curve` is fetched from `curves_` based on `poolIdx`.
   - The function `accrueAmbientPositionTimeWeightedLiquidity()` and `accrueAmbientGlobalTimeWeightedLiquidity()` are called.
   - The `posKey` (a unique identifier for the user's position) is generated using `encodePosKey()`.

2. **Rewards Calculation**:

   - Start loop for each week in `weeksToClaim`:

     - For `week = 100`:

       - Check if week + WEEK < block.timestamp. Since 100 + WEEK < 102, this is true.
       - Check if the rewards for this week have already been claimed. Assume they haven't.
       - Fetch the overall time-weighted liquidity for this week from `timeWeightedWeeklyGlobalAmbLiquidity_`. Let's assume this is `1000`.
       - If the overall liquidity is greater than 0, calculate the rewards for the week:
         - Get the time-weighted position ambient liquidity for this week for the specific user, assume it's `10`.
         - Fetch the total rewards available for the week (`ambRewardPerWeek_`). Let's assume it's `500`.
         - Calculate rewards: (10 \* 500) / 1000 = `5`. Add this to `rewardsToSend`.
       - Mark the rewards as claimed for this week.

     - For `week = 101`:
       - Same steps repeated. Let's assume the user gets `4` as the reward for this week.
       - Add this to `rewardsToSend`. Now, `rewardsToSend = 9`.

3. **Sending Rewards**:
   - Check if `rewardsToSend` is greater than 0.
   - Send the rewards to the owner. Here, `9` will be sent to the `owner`.

### Outcome:

The owner would receive `9` as rewards for the weeks `[100, 101]` in our hypothetical scenario.

---
---

## Questions & Notes:

### **During Documentation Review**:
1. Who is the intended audience for interaction with the `LiquidityMining` and `LiquidityMiningPath` contracts?
2. What is the correct procedure to engage with Ambient Contracts?
3. Clarification on the term "sidecar contract".

### **During Audit**:
1. Canto is developing a liquidity mining protocol tailored for the Ambient decentralized exchange.
2. Differentiation between Ambient rewards and Concentrated rewards.
3. Insight into the `CurveState` Struct.

### **Potential Attack Vectors**:
1. Possibility of liquidity providers withdrawing more rewards than they have accumulated.
2. Scenario where liquidity providers might receive lesser rewards than they are entitled to.



### Time spent:
20 hours