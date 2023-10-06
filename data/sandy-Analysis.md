# Canto Protocol - Analysis 

|Head |Details|
|:----------------|:------|
|Description| A brief introduction to the audit protocol and major contracts
| Approach taken in evaluating the codebase | What is unique? How are the existing patterns used? |
|Codebase quality analysis| Its structure, readability, maintainability, and adherence to best practices|
|Architecture recommendations| The design of the protocol|
|Centralization risks| power, control, or decision-making authority is concentrated in a single entity|
|Other recommendations| Recommendations for improving the quality of your codebase|
|Conclusion| Takeaway from this protocol, recommendations and final review|
|Time spent| Total time spent during auditing and reviewing the codebase |

## Description
Canto Protocol is releasing a new liquidity mining feature, built specifically for Ambient Finance and the feature is being implemented as a sidecar contract that plugs into Ambient using it's proxy contract pattern.

The key contracts of the protocol for this Audit are:
1. LiquidityMiningPath.sol: This contract provides interface for users to interact with the contract.

2. LiquidityMining.sol: This contract contains all the logic for liquidity accruing, and rewards calculation and distribution.


## Approach taken in evaluating the codebase

Steps:

- ``Using a static code analysis tool``: Static code analysis tools can scan the code for potential bugs and vulnerabilities. These tools can be used to identify a wide range of issues, including:

    - Insecure coding practices
    - Common vulnerabilities
    - Code that is not compliant with security standards

- ``Reading the documentation``: The documentation provided by the protocol for this audit helped to provide a detailed overview of the protocol and its code base. It also helped to to understand the purpose of the code and to identify potential areas of concern.

- ``Scoping the analysis``: Once you have a basic understanding of the protocol and its code base, you can start to scope the analysis. This involves identifying the specific areas of code that you want to focus on. For example, you may want to focus on the code that handles user input, the code that interacts with external calls, or the code that changes sensitive state.

- ``Manually reviewing the code``: Once you have scoped the analysis, you can start to manually review the code. This involves reading the code line-by-line and looking for potential problems. Some of the things you should look for include:

   - Unvalidated user input
   - Unsafe external calls
   - Unexpected state changes
   - Lack of Access Control

- ``Marking vulnerable code parts with @audit tags``: Once you have identified any potential vulnerabilities, you should mark them with @audit tags. This will help you to identify the vulnerable code parts later on. 

- ``Digging deep into vulnerable code parts and compare with documentations``:  For each vulnerable code part, you should dig deep to understand how it works and why it is vulnerable. You should also compare the code with the documentation to see if there are any discrepancies.

- ``Performing a series of tests``: Once you have finished reviewing the code, you should perform a series of tests to ensure that it works as intended. These tests should cover a wide range of scenarios, including:

  - Valid and invalid user input
  - Different types of attack vectors
  - Complex state changes

- ``Reporting any problems found``:  If you find any problems with the code, you should report them to the developers of Canto protocol. The developers will then be able to fix the problems and release a new version of the protocol.

## Codebase quality analysis
There were 2 contracts in scope for this Audit with total nSloc of 145 and 2 external imports. Hence, the code base was pretty small. But this code base packed a lot of advanced functionalities. There was only 75% test coverage for the code but as this code base directly deals with reward calculation and distribution, more test coverage is recommended . All the complex functionalities were explained with NatSpec comments and thus, the quality of the code base is pretty good. There is a need to understand a separate part of the codebase / get context in order to audit this  the protocol but it was also well documented and easy to comprehend. This codebase used the concept of concentrated liquidity like Uniswap V3 which was very unique to audit. 

The only recommendation for this codebase is to use smart contract coding best practices like using named imports, custom errors with if statements instead of reverts, and so on

## Architecture recommendations
The user can only interact with ``LiquidityMiningPath.sol`` contract using ``protocolCmd()`` function to claim concentrated and ambient rewards. There is no  interaction between user and the protocol expect this.

The code base is very small and thus not much architectural requirement is there.

## Centralization risks
Centralization risks are pretty high for this code base as the ``setConcRewards`` and ``setAmbRewards`` function can change the rewards without any time lock or delay meaning malicious owner/role can cause some serious loss to the protocol. 

Thus, It is recommended to use timelock or delay for critical functions.


## Other recommendations

- Regular code reviews and adherence to best practices
- Conduct external audits by security experts
- Consider open sourcing the contract for community review
- Maintain comprehensive security documentation
- Establish a responsible disclosure policy for vulnerabilities
- Implement continuous monitoring for unusual activity
- Educate users about risks and best practices

## Conclusion
The Canto protocol released a new liquidity mining feature and liquidity is accrued normal for ambient and concentrated UniswapV3 style which is a unique functionality. This protocol also used sidecars and proxy pattern. The code base is very secure, well-written and test coverage was okay.

## Time-spent 

15 Hours 

### Time spent:
15 hours