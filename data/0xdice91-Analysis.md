# Analysis - Canto Liquidity Mining Protocol Contest

## Description
Canto Liquidity Mining is a novel feature introduced to the canto chain, mainly for incentivizing liquidity on Ambient Finance, it makes use of an Incentive mechanism and a nice liquidity mining scheme that incentivizes LPs to provide liquidity to the aimed protocol. 
## Approach taken in evaluating the codebase
During This audit, I focused on getting a full understanding of the mechanisms in the protocol, the functions to be called by a user and the ones to be called by the governance, I focused on invariants that could be broken and security guidelines followed by the protocol.
# Architecture recommendations
The Overall Architecture of the protocol is well designed and user flows are properly implemented as the interactions between the contracts and the mechanisms implemented all align with Canto's aimed goals.
# Codebase quality analysis
The Codebase was really short and all the more easier to understand, I would mark is as Good all though there wasns't a full coverage of test on the codebase and Natspecs for certain functions were missing.
| Codebase Quality Categories  | Comments |
| --- | --- |
| **Unit Testing**  | The Codebase was actually well-teste, but the coverage being 75% wasn't enough, and Hardhat was use, I strongly recommend the use of Foundry|
| **Code Comments**  | Incomplete Comments and Natspecs on critical functions that were heuristic but due to the SLOC of the codebase I can see why the devs didn't put much time into that, but for a good and well audited codebase I recommend more and detailed comments and Natspecs be added to the 2 contracts|
| **Documentation** | The codebase was well described in the contest page of the contest, however I recommend that a detailed documentation of the codebase should be made and more should be done explaining how the functions are supposed to work|
| **Organization** | The Codebase was actually so easy and simple removing complexities that made it look mature and well organized with clear distinctions between the contracts, and how they interact with each other to help aid their functionalities |
# Centralization risks
Due to Canto's Governance I think the issue of centralization was well mitigated in the codebase 
# Mechanism review
The audit cover a total of 2 contracts.
1. LiquidityMiningPath.sol
2. LiquidityMining.sol

Mechanism where to set the rewards which is to be done by the Governance and claiming of rewards. i believe the mechanisms in the codebase were properly implemented and user flows go as planned.
# Systemic risks
The Systematic risk have already been admitted by the developers in the Ambient docs.


### Time spent:
12 hours