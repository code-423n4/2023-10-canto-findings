![Canto Liquidity Mining Protocol](https://canto.io/assets/logo-a672d2b0.svg)

# Analysis - Canto Liquidity Mining Protocol

## Description

Canto Protocol is introducing a cutting-edge liquidity mining feature, purpose-built for Ambient Finance, marking a significant step forward in the DeFi ecosystem.The protocol's foundation lies in a sophisticated sidecar contract, seamlessly integrating with Ambient Finance through a proxy contract pattern. Canto's liquidity mining feature introduces two crucial contracts: LiquidityMiningPath.sol, which provides user interfaces for interaction, and LiquidityMining.sol, the heart of the system, housing all the essential logic.

**The key contracts of the protocol for this Audit are**:

- **LiquidityMiningPath.sol**:

  This contract serves as the interface that allows the CrocSwapDex contract to interact with the Canto Protocol by making calls using userCmd and protocolCmd.It plays a critical role in facilitating communication and interaction between the Canto Protocol and the CrocSwapDex contract, enabling various commands and actions related to liquidity mining.

- **LiquidityMining.sol**:

  This contract contains the core logic and functionality for the liquidity mining feature within the Canto Protocol. It plays a central role in implementing the mechanics of liquidity mining, including tracking time-weighted liquidity, calculating rewards, and distributing them to users. This contract is crucial for the operation of the liquidity mining feature.

## Approach Taken-in Evaluating The Venus Prime Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Canto Protocol .

    **Main Contracts I Looked At**

                LiquidityMiningPath.sol
                LiquidityMining.so

    I started my analysis by examining the LiquidityMiningPath.sol contract. This contract is a proxy sidecar contract within the Canto Protocol, designed for liquidity mining operations. It contains essential components related to CANTO liquidity mining. This contract serves as a standalone entity and primarily operates on the contract state within the primary CrocSwap contract through DELEGATECALL.

    Then, I turned our attention to the LiquidityMining.sol contract is an integral component of the Canto Protocol, focused on managing liquidity mining operations. This contract includes functions related to tracking, accruing, and claiming liquidity mining rewards for users participating in liquidity provision activities.

2.  **Documentation Review**:

    Then went to Review [this document](https://docs.ambient.finance/) and Learn some basic concept in [Readme](https://code4rena.com/contests/2023-10-canto-liquidity-mining-protocol#top).

3.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Architecture Description and Diagram

Architecture of the key contracts that are part of the Canto Liquidity Mining Protocol:

**LiquidityMiningPath**:- The LiquidityMiningPath contract acts as a vital gateway for users to interact with the liquidity mining feature of Canto's protocol. It ensures that users are properly incentivized for providing liquidity in specified tick ranges and follows a transparent and flexible reward distribution mechanism..

**LiquidityMining**:- This contract is used to facilitate the accrual and distribution of rewards to liquidity providers based on their participation in liquidity provision within specific tick ranges, and the contract keeps track of time-weighted liquidity changes and ensures that rewards are distributed accurately to liquidity providers based on their contributions and the weeks they choose to claim

## Codebase Quality

Overall, I consider the quality of the two contract of the Canto Protocol codebase to be excellent. The code appears to be very mature and well-developed. Details are explained below:

| Codebase Quality Categories       | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Readability**              | The code is well-formatted, with consistent indentation and spacing. Clear and descriptive variable and function names enhance readability. The use of meaningful constants like WEEK also improves code comprehension. purposes.                                                                                                                                                                                                                                                                                                                                                  |
| **Code Comments**                 | The contract has extensive comments and documentation using NatSpec-style comments, which is great for understanding the purpose of different functions and the overall contract. The comments are clear and help explain the contract's functionality.                                                                                                                                                                                                                                                                                                                            |
| **Testing**                       | Codebase has good introducation of how to run tests [README.md](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/README.md).                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| **Code Structure and Formatting** | The code structure and formatting in both LiquidityMiningPath and LiquidityMining are generally well-organized and adhere to common Solidity coding conventions. They follow common Solidity coding conventions and use comments and documentation effectively to explain their functionality. While there's a mention of role-based access control in comments, it's important to ensure that proper access control mechanisms are implemented if required, as indicated in the comments.                                                                                         |
| **Custom Error Types**            | Using custom error types in contracts can be a good practice for several reasons: 1. Clarity: Custom error types provide more descriptive and informative error messages.2. Error Handling: Custom error types allow for more precise error handling. Contracts can specify different error conditions with specific error codes, making it easier to handle different scenarios.3. Readability: When a contract uses custom error types, it makes the code more readable and self-explanatory. Developers can quickly understand what went wrong when a specific error is raised. |

## Systemic & Centralization Risks

1.  **Proxy Sidecar Usage** (Centralization Risk):

    The contract LiquidityMiningPath mentions that it should only be invoked with DELEGATECALL within the primary CrocSwap contract. This implies that the CrocSwap contract is the central point that interacts with this contract. Depending on the permissions and control that the CrocSwap contract has, this could centralize the functionality and control within the CrocSwap contract.

2.  **Time-Weighted Liquidity Calculations**:

    The contract LiquidityMining calculates rewards based on time-weighted liquidity contributions. Depending on how this calculation is implemented, it could lead to centralization if certain users or liquidity providers are favored over others due to the way their contributions are measured.

3.  **External Calls**:

    The LiquidityMining contract has External calls like owner.call{value: rewardsToSend}("") can be risky. Ensure that they are safe and follow best practices for interacting with external contracts.

4.  **Reward Distribution**:

    The contract LiquidityMining has control over distributing rewards. If this process isn't designed in a decentralized manner, it could lead to centralization of rewards.

5.  **Time-Based Logic**:

    The LiquidityMining contract relies heavily on time-based logic (e.g., weekly calculations). If the timing is manipulated, it could impact reward calculations and the overall system's integrity.

6.  **Concentration of Liquidity**:

    The LiquidityMining contract deals with liquidity tracking and rewards. If liquidity is highly concentrated among a few large users, it could centralize control over the system's rewards.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Venus Prime protocol.**

## Gas Optimization

Gas optimization techniques are critical in smart contract development to reduce transaction costs and make contracts more efficient. The Canto protocol, as observed in the provided contract, can benefit from various gas-saving strategies to enhance its performance. Below is a summary of gas optimization techniques followed by categorized findings within the contract:

### Summary

| Finding                                                                                      | Occurrences |
| :------------------------------------------------------------------------------------------- | :---------: |
| Add unchecked {} for subtraction                                                             |      4      |
| Using assembly to revert with an error message                                               |      8      |
| Use solidity version 0.8.20 to gain some gas boost                                           |      2      |
| State variables should be cached in stack variables rather than re-reading them from storage |     27      |
| Avoid contract calls by making the architecture monolithic                                   |      2      |
| Always use Named Returns                                                                     |      1      |
| Can make the variable outside the loop to save gas                                           |     10      |
| Use calldata instead of memory for function arguments that do not get mutated                |      4      |
| Using Storage instead of memory for structs/arrays saves gas                                 |      5      |
| Using > 0 costs more gas than != 0                                                           |      5      |
| Do-While loops are cheaper than for loops                                                    |      5      |
| Use uint256(1)/uint256(2) instead for true and false boolean states                          |      2      |
| Split require statements that have boolean expressions                                       |      2      |
| Use ++i instead of i++ to increment                                                          |      1      |


### Time spent:
9 hours