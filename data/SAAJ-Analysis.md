# Introduction
This analysis is based on reviewing of Canto contest related to its launching of a liquidity mining feature developed for Ambient Finance. 

Liquidity mining involves a sidecar contract integrating with Ambient through its proxy contract pattern. The contracts in scope are ```LiquidityMiningPath.sol``` and ```LiquidityMining.sol``` that are developed to facilitate user interactions and implement the logic for pool integration and implementation.

# Mechanism review
The LiquidityMining sidecar implementation is based on drivinga liquidity mining protocol for Ambient.  Canto's objective is to utilize this sidecar to incentivize liquidity within Ambient pools as the  mechanism revolves around returns for a specific width of liquidity based on the current tick, focusing on stable pools. Liquidity providers are encouraged to maintain a range of ticks around the current tick to receive incentives.

The rewards for liquidity mining are calculated based on the time-weighted liquidity provided, both globally and per-user, considering ambient and concentrated positions per tick. The system distributes rewards weekly based on the total reward amount set by governance, this enables fair distribution of rewards based on the percentage of in-range liquidity provided by a user over a specified time span.


This implementation of Ambient pools enhances liquidity provision and fosters a fair and sustainable reward distribution system for liquidity providers on the Canto ecosystem.

# Codebase Review 
There are 2 smart contracts that are in scope and reviewed with additional ambient hooks which are called by the main contract i.e., in scope of this audit analysis.

- **LiquidityMining.sol**:

  - The ```LiquidityMining``` smart contract is used for managing liquidity mining for canto dapp and its implementation for ambient pools for mining.
  - The function **`initTickTracking`** initializes tick tracking for a specific pool and tick. initTickTracking function also keep record of timestamp when the tracking begins and the number of times the tick has been tracked.
  - **`crossTicks`** function keeps track of tick crossings within a pool with recording exit and entry timestamps for a tick whenever a tick is crossed.
  - **`accrueConcentratedGlobalTimeWeightedLiquidity`** function accrues the global time-weighted concentrated liquidity for a specified pool. It  carries out calculation and updates the time-weighted weekly global concentrated liquidity based on the provided curve state.
  - **`accrueConcentratedPositionTimeWeightedLiquidity`** function accrues time-weighted concentrated liquidity for a specific position within a pool. The consideration is tick entry and exit history to calculate the time-weighted liquidity for the given position.
  - **`claimConcentratedRewards`** function allows user to claim concentrated rewards for a specified position by calculating rewards based on the position's liquidity and the provided weeks to claim, then sends the rewards to the owner.
  - **`accrueAmbientGlobalTimeWeightedLiquidity`** function accrues the global time-weighted ambient liquidity for a specified pool. Calculation and updating of the time-weighted weekly global ambient liquidity based on the provided curve state.
  - **`accrueAmbientPositionTimeWeightedLiquidity`** function accrues the time-weighted ambient liquidity for a specific position within a pool. Ambient seeds and calculates the time-weighted liquidity for the given position.
  - **`claimAmbientRewards`** function allows an address to claim ambient rewards for a specified position. Rewards are based on the position's ambient seeds and the provided weeks to claim then sends the rewards to the owner.




- **LiquidityMiningPath.sol**:
    - The ```LiquidityMiningPath``` contract serves as a proxy sidecar for the CrocSwapDex smart contract, specifically handling liquidity mining-related function.
    - The contract acts as a proxy, allowing code separation to avoid Ethereum’s contract code size limit. Delegate call to the main CrocSwapDex contract using ```DELEGATECALL```.
    - Two types of rewards are supported i.e.; Concentrated and Ambient rewards.
        - ```Concentrated Rewards``` are users specific to a pool index, tick range, and weeks to claim.
        - ```Ambient Rewards``` are based on user to specify a pool index and weeks to claim.
    - ``` acceptCrocProxyRole``` as a validation function ensures that it is used correctly during upgrades.

# Systemic risk
The risk identified from the assets in scope are summarised as below:
1. **Timestamp**
   - The system heavily relies on timestamps to calculate time-weighted rewards and track liquidity changes. Timestamp-based logic are very likely susceptible to manipulation or inaccuracies, leading to potential exploits or incorrect reward calculations. 
2. **Complexity**
   - The functions described involve complex calculations and data processing, which could result in high gas costs for users interacting with the contracts.
3. **Lack of Upgradeability mechanism**
   - There's no evident consideration for contract upgradability as a robust smart contract system it must have consideration for future upgrades without disrupting the existing functionality or data.
4. **Delegate Call Usage**
   - The use of ```DELEGATECALL``` to delegate functionality to another contract ```CrocSwapDex``` could introduce potential vulnerabilities if not handled cautiously , allowing the delegated contract to have unintended control over the proxy contract's state and operations.


# Recommendation
The recommendation is made for the contract of ```LiquidityMiningPath``` that serves as a proxy sidecar for the CrocSwapDex smart contract, specifically handling liquidity mining-related functionality. The contract allows for code separation to avoid Ethereum’s contract code size limit.

The recommendation are related to improving the overall architecture in areas of:

 - Breaking down the proxy sidecar into smaller, more focused components as each component should handle a specific aspect of liquidity mining (e.g., rewards calculation, user interactions, governance).
- Creating separate contracts for different functionalities like  rewards distribution, staking logic, governance), making it easier to maintain and upgrade each part independently.
-  Different proxy patterns beyond the basic delegatecall-based proxy used like Transparent, UUPS and Beacon proxy.
    - **Transparent** proxies provide facilitation to forward all calls transparently to the implementation contract without altering storage. They provide a cleaner separation between proxy and implementation.
    - **UUPS (Universal Upgradeable Proxy Standard)** proxies allow for more efficient upgrades by separating storage from logic. They reduce gas costs during upgrades.
    -  **Beacon** proxies separate storage and logic even further, allowing multiple implementations to share the same storage.

# Conclusion
Canto liquidity mining for Ambient pool will be effective only when it is implemented with best practice and open consideration for upgradability in reference to future developments.

Currently, the implementation must follow safe and secure method to overcome the complexity of cross pool liquidity mining.

**Note to Judge**

I spent around 10 hours on this analysis which provides me consideration for areas of improvement that must be made for securing the platform from potential vulnerabilities identified in systemic risk that can cause direct loss of funds for protocol and its users.


### Time spent:
10 hours