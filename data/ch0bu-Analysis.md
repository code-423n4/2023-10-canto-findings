# Introduction
The goal of this contest is to do a security audit for a new liquidity mining feature that Canto is releasing for Ambient Finance.

Ambient is a DEX protocol that allows for two-sided AMMs combining concentrated and ambient constant-product liquidity on any arbitrary pair of blockchain assets.

# Methodology
The following steps were taken:
Step 1 - read the documentation
Step 2 - read through the code to get a sense of how the protocol works
Step 3 - deep dive into the code and finding vulnerabilities
Step 4: writing reports


# Codebase Quality
The overall quality of the code is good. It is well written, and even though it is complex, it is still readable and understandable.

There are a few things that could improve in terms of mostly complying with the Solidity Style Guide, NatSpec usage, etc.

- Documentation stated that tests cover 75% of the code. This number should be as close to 100% as possible.
- Most of the functions are commented on and explained, and comments in the code were added where needed. It is recommended to add NatSpec, this could additionally increase the level of understanding of the code for reviewers.
- Documentation is very well written, and additional resources are linked where appropriate. Documentation is not too large, where it takes a lot of time to go through everything, but it's not too small either and explains all the concepts that the protocol uses.

# Systematic and Centralization risks

There are no centralization risks in this part of the protocol.

It should be noted that the protocol as a whole is made to be easily upgradeable. From this we can conclude 2 risks:
- Errors made during the upgrade. The protocol team needs to be very careful while upgrading the contracts.
- Protocol team turning malicious and upgrading contracts in a way that is not in favor of its users. However unlikely this scenario may be, it should be pointed out that it could happen.

Depending on 3rd party contracts, like Openzeppelin, opens the protocol to a risk where exploits found in 3rd party can have an affect on the protocol. We can already see that Canto is using package versions that have known vulnerabilities in them. The project team needs to closely monitor the dependencies that they are using, and if some vulnerability is found that could affect their code, they should take immediate action to remediate the risk.


# Gas optimizations

The protocol is designed in a way that saves a lot of gas for its users compared to other DEX protocols.
On the other hand, there are some changes in the code that could be made to save additional gas, like:
- using `unchecked` where there is no possibility for overflows
- `.length` not used in loops
- using custom errors
- and others that were mentioned in the bot report

# Conclusion

The overall audited code is well written, well structured, and follows best practices for contract development. I have not found any major vulnerabilities or risks that could be classified as medium or high.

# Time spent
10 hours

### Time spent:
10 hours