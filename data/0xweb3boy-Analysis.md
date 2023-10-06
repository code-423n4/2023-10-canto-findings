# Canto Ambient Liquidity Mining
Audit contest: `2023-10-canto`

## Table of Contents

- 1. Executive Summary
- 2. Code Audit Approach
  - 2.1 Audit Documentation and Scope
  - 2.2 Code review
  - 2.3 Threat Modelling
  - 2.4 Exploitation and Proofs of Concept
  - 2.5 Report Issues
- 3. Architecture overview
  - 3.1 Overview of Liquidity Mining Feature by Canto for Ambient Finance
  - 3.2  Key Contracts Introduced
  - 3.3 Incentive Mechanism
  - 3.4 Liquidity Mining Rewards
  - 3.5 Reward Distribution Example
  - 3.6 Setting Weekly Reward Rate
- 4. Implementation Notes
  - 4.1 General Impressions
  - 4.2 Composition over Inheritance
  - 4.3 Comments
  - 4.4 Solidity Versions
- 5. Conclusion

## 1. Executive Summary

In focusing on the ongoing audit contest 2023-10-canto, my analysis starts by delineating the code audit methodology applied to the contracts within the defined scope. Subsequently, I provide insights into the architectural aspects, offering my perspective. Finally, I offer observations pertaining to the code implementation.


I want to emphasize that unless expressly specified, any potential architectural risks or implementation concerns discussed in this document should not be construed as vulnerabilities or suggestions to modify the architecture or code based solely on this analysis. As an auditor, I recognize the necessity of a comprehensive evaluation of design choices in intricate projects, considering risks as only one component of a larger evaluative process. It's essential to acknowledge that the project team may have already evaluated these risks and established the most appropriate approach to mitigate or coexist with them.

## 2. Code Audit Approach

Time spent: 16 hours

### 2.1 Audit Documentation and Scope
Commencing the analysis, the first phase entailed a thorough review of the https://github.com/code-423n4/2023-10-canto/tree/main to fully grasp the fundamental concepts and limitations of the audit. This initial step was crucial in guiding the prioritization of my audit efforts. Notably, the README associated with this audit contest stands out for its excellent quality, offering valuable insights and actionable guidance that significantly streamline the onboarding process for auditors.


### 2.2 Code review


Initiating the code review, the starting point involved gaining a comprehensive understanding of "LiquidityMining.sol," a pivotal component responsible for implementing a liquidity mining protocol for Ambient. Canto's strategic intention is to leverage this sidecar mechanism to incentivize liquidity provision for Ambient pools deployed on their platform. This understanding of the core pattern significantly facilitated the comprehension of the protocol contracts and their interconnections. During this phase, meticulous documentation of observations and the formulation of pertinent questions regarding potential exploits were undertaken, striking a balance between depth and breadth of analysis.

### 2.3 Threat Modelling

The initial step involved crafting precise assumptions that, if breached, could present notable security risks to the system. This approach serves to guide the identification of optimal exploitation strategies. Although not an exhaustive threat modeling exercise, it closely aligns with the essence of such an analysis.

### 2.4 Exploitation and Proofs of Concept

Progressing from this juncture, the primary methodology took the form of a cyclic process, conditionally encompassing steps 2.2, 2.3, and 2.4. This involved iterative attempts at exploitation and the subsequent creation of proofs of concept, occasionally aided by available documentation or the helpful community on Discord. The key focus during this phase was to challenge fundamental assumptions, generate novel ones in the process, and refine the approach by utilizing coded proofs of concept to hasten the development of successful exploits.

### 2.5 Report Issues

While this particular stage might initially appear straightforward, it harbors subtleties worth considering. Hastily reporting vulnerabilities and subsequently overlooking them is not a prudent course of action. The optimal approach to augment the value delivered to sponsors (and ideally, to auditors as well) entails thoroughly documenting the potential gains from exploiting each vulnerability. This comprehensive assessment aids in the determination of whether these exploits could be strategically amalgamated to generate a more substantial impact on the system's security. It's important to recognize that seemingly minor and moderate issues, when skillfully leveraged, can compound into a critical vulnerability. This assessment must be weighed against the risks that users might encounter. Within the realm of Code4rena audit contests, a heightened level of caution and an expedited reporting channel are accorded to zero-day vulnerabilities or highly sensitive bugs impacting deployed contracts.

## 3. Architecture overview


### 3.1 Overview of Liquidity Mining Feature by Canto for Ambient Finance
Introduction to the Feature: Canto, in its pursuit of enhancing liquidity dynamics, has introduced a new liquidity mining feature tailored specifically for Ambient Finance, a targeted approach to bolster the platform's liquidity infrastructure.

Integration through Sidecar Contract: The implementation strategy involves the creation of a sidecar contract meticulously designed to integrate into the Ambient ecosystem. This integration is achieved through the utilization of Ambient's proxy contract pattern, a structured approach to seamlessly amalgamate the new liquidity mining feature.

### 3.2 Key Contracts Introduced:

LiquidityMiningPath.sol: This contract is pivotal in providing the essential interfaces, enabling seamless interactions for users interacting with the liquidity mining protocol.
LiquidityMining.sol: This contract forms the operational backbone, encapsulating the logic and functionalities necessary for the effective operation of the liquidity mining feature.
Understanding the LiquidityMining Sidecar
Objective of the Sidecar: The LiquidityMining sidecar is meticulously engineered to realize a robust liquidity mining protocol within the Ambient ecosystem. Canto envisions utilizing this sidecar to stimulate and incentivize liquidity contributions to the Ambient pools hosted on Canto's platform.

### 3.3 Incentive Mechanism:

The sidecar employs an incentivization mechanism targeting a specific width of liquidity, primarily based on the current tick.
Focused on stabilizing liquidity pools, the incentivization range spans from the current tick minus 10 to the current tick plus 10.
To qualify for incentives, a user's liquidity range must encompass at least 21 ticks, inclusive of the current tick and 10 ticks on each side.
Additionally, users are advised to maintain a slight buffer on either side of the stipulated range to ensure uninterrupted rewards, particularly in response to minor price fluctuations.

### 3.4 Liquidity Mining Rewards:

The core idea behind the liquidity mining sidecar centers on tracking the time-weighted liquidity across global and per-user levels.
This tracking is conducted for both ambient and concentrated positions on a per-tick basis.
The protocol calculates the user's proportion of in-range liquidity over a specific time span, subsequently disbursing this percentage of the global rewards to the user.

### 3.5 Reward Distribution Example:

For illustrative purposes, if the weekly rewards amount to 10 CANTO and only LP A is contributing liquidity, LP A will receive the entire 10 CANTO as their reward.
However, if there are multiple liquidity providers, such as LP A and LP B, who contribute liquidity throughout the week, the rewards will be shared evenly. In this scenario, each provider will receive 5 CANTO.
Implementation Insights

### 3.6 Setting Weekly Reward Rate:

The liquidity mining sidecar incorporates functions dedicated to setting the weekly reward rate.
The reward rates are determined by establishing a total disbursement amount per week, providing the governing body the flexibility to choose the number of weeks for which the reward rate will be set.
Focus on LiquidityMining.sol:

The critical aspect of the implementation lies within the LiquidityMining.sol contract, housing the core logic for reward accumulation and claim processes.
Auditors and wardens are strongly advised to direct their primary focus towards this segment of the codebase, recognizing its pivotal role in the successful operation of the liquidity mining feature.

## 4. Implementation Notes

During the course of the audit, several noteworthy implementation details were identified, and among these, a significant subset holds potential value for the ongoing analysis.

### 4.1 General Impressions

# Overview of Ambient

Ambient Finance: Ambient is a single-contract decentralized exchange (dex) designed to facilitate liquidity provision in a flexible manner. It allows liquidity providers to deposit liquidity in either the "ambient" style (similar to uniV2) or the "concentrated" style (similar to uniV3) into any token pair.

Main Contract: The primary contract in Ambient Finance is called CrocSwapDex. Users interact with this contract, and it is the main interface for all actions related to the protocol.

### Sidecar Contracts
Ambient utilizes modular proxy contracts called "sidecars," each responsible for a unique function within the protocol. Here are the sidecar contracts:

BootPath: Special sidecar used to install other sidecar contracts.
ColdPath: Handles the creation of new pools.
WarmPath: Manages liquidity operations like minting and burning.
LiquidityMiningPath: New sidecar developed by Canto to handle liquidity mining for both ambient and concentrated liquidity.
KnockoutPath: Manages logic for knockout liquidity.
LongPath: Contains logic for parsing and executing arbitrarily long compound orders.
MicroPaths: Contains functions related to single atomic actions within a longer compound action on a pre-loaded pool's liquidity curve.
SafeModePath: Reserved for emergency mode.

### Interacting with Ambient Contracts
User and Protocol Commands: Interaction with Ambient contracts is facilitated through two main functions:

`userCmd(uint16 callpath, bytes calldata cmd)`: Used by users for various actions.
`protocolCmd(uint16 callpath, bytes calldata cmd, bool sudo)`: Utilized for governance-related actions.
`Callpath Parameter`: The callpath parameter determines which sidecar contract receives the command. Different values correspond to different sidecar contracts, each serving a specific purpose.

`Command Encoding (cmd)`: The cmd parameter is ABI encoded calldata fed to the specified sidecar contract. The code parameter within cmd specifies the function to be called within the sidecar contract based on the desired action.



### 4.2 Composition over Inheritance

Canto has used inheritance over composition because inheritance is a  prevalent choice due to the need for standardized behaviors and code reusability. Smart contracts often adhere to established standards such as ERC-20 or ERC-721, and inheritance facilitates the straightforward integration of these standards, promoting interoperability and adherence to well-defined interfaces. By centralizing common functionalities in a base contract and allowing other contracts to inherit these functionalities, developers can streamline deployment, reduce redundancy, and simplify interactions. In addition, this approach aids in efficient gas usage, aligning with the limitations imposed by the Ethereum Virtual Machine (EVM) contract size. Moreover, inheritance supports logical code organization and maintenance, enabling easier updates and enhancing readability by segregating related functionalities into distinct contracts.


### 4.3 Comments

#### Importance of Comments for Clarity:

Comments serve a pivotal role in enhancing the understandability of the codebase. While the code is generally clean and logically structured, judiciously placed comments can provide valuable insights into the functionalities and intentions behind the code. They contribute to a better comprehension of the code's purpose, especially for auditors and developers involved in the analysis.

#### Strategic Comment Placement:

The codebase would greatly benefit from an increased presence of comments, particularly within the LiquidityMining.sol contract. Strategic placement of comments within this essential contract can significantly aid auditors in comprehending the implementation details. These comments should elucidate the logic, processes, and methodologies employed, promoting a seamless audit experience.

#### Facilitating Audits and Code Readability:

Comprehensive comments in the LiquidityMining.sol contract can expedite the auditing process by allowing auditors to swiftly grasp the intended functionality of the code. This, in turn, enhances the readability of the functions and methods, making it easier for auditors to identify any inconsistencies or deviations between the documented intentions and the actual code implementation.

#### Detecting Discrepancies in Code Intentions:

An important aspect of code review is discerning any mismatches between the documented intentions in the comments and the actual code implementation. Comments that accurately reflect the code's purpose are crucial for auditors, as discrepancies between the two can be indicators of potential vulnerabilities or errors. The act of aligning comments with the true code behavior is fundamental to ensuring the reliability and security of the smart contract.


### 4.4 Solidity Versions

Though there are valid arguments both in support of and against adopting the latest Solidity version, I find that this discussion bears little significance for the current state of the project. Without a doubt, choosing the most up-to-date version is a far superior decision when compared to the potential risks associated with outdated versions.


## 5. Conclusion

### Positive Audit Experience:

The process of auditing this codebase and evaluating its architectural choices has been thoroughly enjoyable and enriching. Navigating through the intricacies and nuances of the project has been enlightening, presenting an opportunity to delve into the complexities of the system.

### Strategic Simplifications for Complexity Management:

Inherent complexity is a characteristic of many systems, especially those in the realm of blockchain and smart contracts. The strategic introduction of simplifications within this project has proven to be a valuable approach. These simplifications are well-thought-out and strategically implemented, demonstrating an understanding of how to manage complexity effectively.

### Achieving a Harmonious Balance:

A notable achievement of this project is striking a harmonious balance between the imperative for simplicity and the challenge of managing inherent complexity. This equilibrium is crucial in ensuring that the codebase remains comprehensible, maintainable, and scalable, even as the system becomes more intricate.

### Importance of Methodology Overview:

The overview provided regarding the methodology employed during the audit of the contracts within the defined scope is invaluable. It offers a clear and structured insight into the analytical approach undertaken, shedding light on the depth and rigor of the evaluation process.

### Relevance for Project Team and Stakeholders:

The insights presented are not only beneficial for the project team but also extend to any party with an interest in analyzing this codebase. The detailed observations, considerations, and recommendations have the potential to guide and inform decision-making, contributing to the project's overall improvement and security.


### Time spent:
16 hours

### Time spent:
16 hours