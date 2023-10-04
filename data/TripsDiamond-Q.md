# [01] Unrestricted Access to Setting Rewards

**Severity:** CRITICAL

**Impact:** Anyone can set rewards and potentially disrupt the intended protocol behavior.

**Description:**  
The `setConcRewards` and `setAmbRewards` public functions lack a modifier to restrict access. As a result, any account can call these functions to modify rewards. This can lead to unintended changes and potential exploitation. Code here:

https://github.com/code-423n4/2023-10-canto/blame/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L66

https://github.com/code-423n4/2023-10-canto/blame/37a1d64cf3a10bf37cbc287a22e8991f04298fa0/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L75

**Recommendations/Mitigation:**  
Reinstate the governance check (which seems to be commented out) to ensure only the designated governance can call these functions. This can be done using the `require(msg.sender == governance_, "Only callable by governance");` statement.
