


# Adanced Analysis report of Canto Ambient Liquidity Mining 



## Overview

Canto introduce a new liquidity mining feature tailored specifically for Ambient Finance. This innovative addition is structured as a sidecar contract, seamlessly integrated with Ambient through their proxy contract pattern. The implementation of this liquidity mining feature encompasses two distinct contracts



## Scope Contract

1 **LiquidityMiningPath.sol**


**Structs and Enums:** There are several structs (Chaining.PairFlow, Directives.PoolDirective, etc.) used to organize and pass around data. It's important to understand the purpose and content of each struct.

**Functions:**

- The contract contains multiple functions (tradeOverPool, swapOverPool, mintOverPool, burnOverPool, harvestOverPool, initCurve, applyToCurve, applySwap, applyAmbient, applyConcentrated) which are responsible for various aspects of liquidity mining and trading.

- These functions are well-commented, which is a good practice for code readability and understanding.

- Each function performs a specific action related to liquidity mining, such as trading, swapping, minting, burning, or harvesting.


2 **LiquidityMining.sol**

**Contract Purpose:** This contract acts as a proxy sidecar, housing code related to liquidity mining for the CrocSwapDex protocol. It's important to note that this contract should only be interacted with using DELEGATECALL and not directly or externally.

**Functionality:**

- **protocolCmd**: This function processes protocol control-related commands. It interprets the command and triggers the corresponding action related to setting reward rates.

- **userCmd:** This function handles user commands, primarily for claiming liquidity mining rewards. It decodes the input and calls the relevant claim function.

- **claimConcentratedRewards**: This function allows a user to claim concentrated rewards for a specific pool and tick range. It takes the user's address, pool index, tick range, and an array of weeks to claim.

- **claimAmbientRewards**: This function allows a user to claim ambient rewards for a specific pool. It takes the user's address, pool index, and an array of weeks to claim.

- **setConcRewards**: This function sets concentrated rewards for a specific pool and time range. It takes the pool index, starting week, ending week, and the weekly reward amount.

- **setAmbRewards**: This function sets ambient rewards for a specific pool and time range. It takes the pool index, starting week, ending week, and the weekly reward amount.

- **acceptCrocProxyRole**: This function is used during an upgrade to verify that the contract is a valid Croc sidecar proxy and is in the correct slot.


## Contract Graphical Representation


1 **LiquidityMining.sol**
![LiquidityMining.sol](https://github.com/kaveyjoe/Work/blob/main/Function%20Graph%20-%20LiquidityMining.sol.png)


2 **LiquidityMiningPath.sol**

![LiquidityMiningpath.sol](https://github.com/kaveyjoe/Work/blob/main/Function%20Graph%20-%20MarketSequencer.sol.png)

## Key Components and Mechanisms
**1. Liquidity Incentive Mechanism**

The LiquidityMining sidecar employs an incentive mechanism to stimulate liquidity provision within a specified tick range. To be eligible for rewards, users must provide liquidity within the range of currentTick-10 to currentTick+10. This implies that users must cover at least 21 ticks to receive incentives, with a recommended buffer to accommodate small price movements.

**2. Liquidity Mining Rewards**

The system tracks time-weighted liquidity for both ambient and concentrated positions. This allows the protocol to determine the percentage of in-range liquidity provided by a user over a specific period. Subsequently, it distributes a proportionate share of the global rewards to the user.


## Centralization Risks

**Proxy Contract Design:**

- Reliance on an externally owned admin key for changes to underlying logic.
- Potential centralization of control if governance key is controlled by a single entity or small group.

**Governance Control:**

- Functions setConcRewards and setAmbRewards have governance-only restrictions (currently commented-out).
- Centralized governance can concentrate decision-making power over reward rates.

**Rewards Distribution:**

- Functions claimConcentratedRewards and claimAmbientRewards handle rewards distribution.
- Centralized distribution mechanism or governance can lead to unfair or centralized control over rewards.

**Upgradeability:**

- Proxy contracts support upgrades.
- Centralized or non-transparent upgrade process can centralize control over contract updates.

## Systematic Risks

**Timestamp Dependency:**

- Reliance on block timestamps for time-dependent operations.
- Vulnerability to potential manipulation by miners, impacting reward calculations.

**Oracles and External Data:**

- Reliance on external oracles for critical data (e.g., token prices).
- Risks associated with oracle manipulation or failures, affecting contract behavior.

**Liquidity Risk:**

- Liquidity mining involves rewarding users for providing liquidity.
- Insufficient liquidity or sudden large withdrawals can impact protocol stability and functionality.

**Gas usage**
- Complex operations like reward distribution and liquidity tracking can have unpredictable gas costs.
- High gas costs may make some operations economically unfeasible for users.

**External Dependencies:**

- Contracts import libraries and mixins from external sources.
- Changes or compromise of these dependencies can impact contract functionality.


## Codebase Quality analysis

- **Innovative Approach:** Canto has demonstrated innovation in introducing a liquidity mining feature tailored for Ambient Finance. The seamless integration with Ambient via a proxy contract pattern showcases a thoughtful approach to protocol development.

- **Structural Clarity:** The contracts, LiquidityMining.sol and LiquidityMiningPath.sol, exhibit a well-organized structure with clearly defined structs, enums, and functions. This organized layout fosters a clear understanding of the codebase.

- **Emphasis on Documentation:** The codebase is notably well-commented, a commendable practice that enhances code comprehensibility. This will be valuable for any future developers interacting with or modifying these contracts.

- **Strategic Incentive Design:** The liquidity incentive mechanism, emphasizing a specific tick range for rewards eligibility, showcases a thoughtful approach to incentivizing meaningful liquidity provision.

- **Fair Reward Allocation:** The system's ability to track time-weighted liquidity for both ambient and concentrated positions underscores a fair and transparent method for distributing rewards based on user contributions.

- **Identified Centralization Risks:** The reliance on admin keys and certain governance restrictions raise potential concerns about centralization. Key areas include the proxy contract design and governance-only functions.

- **Upgradeability and Transparency Considerations:** The support for contract upgrades is a positive feature. However, the potential for a centralized or non-transparent upgrade process necessitates vigilance in maintaining transparency and control.

- **Mitigating Systematic Risks:** Awareness of timestamp dependency and reliance on external oracles for critical data is crucial. These aspects present potential vulnerabilities that must be addressed to ensure the protocol's security.

- **Proactive Liquidity Risk Management:** Acknowledging the risks tied to liquidity mining, particularly in scenarios of insufficient liquidity or abrupt large withdrawals, demonstrates a keen awareness of protocol stability.

- **Gas Usage Optimization Awareness:** The recognition of potentially unpredictable gas costs for complex operations highlights a consideration for user experience and economic feasibility.

- **Dependency Monitoring:** The acknowledgement of importing libraries and mixins from external sources underlines a prudent approach to external dependencies. Vigilance regarding potential changes or compromises in these dependencies is key.




### Time spent:
15 hours