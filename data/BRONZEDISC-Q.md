## QA
---

### natSpec missing [1]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol

```solidity
// @param/@notice missing
16:    function initTickTracking(bytes32 poolIdx, int24 tick) internal {

// @param missing
24:    function crossTicks(
39:    function accrueConcentratedGlobalTimeWeightedLiquidity(
69:    function accrueConcentratedPositionTimeWeightedLiquidity(

156:    function claimConcentratedRewards(
198:    function accrueAmbientGlobalTimeWeightedLiquidity(
224:    function accrueAmbientPositionTimeWeightedLiquidity(
256:    function claimAmbientRewards(
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol

```solidity
// @param missing
26:    function protocolCmd(bytes calldata cmd) public virtual {
41:    function userCmd(bytes calldata input) public payable {

54:    function claimConcentratedRewards(bytes32 poolIdx, int24 lowerTick, int24 upperTick, uint32[] memory weeksToClaim)
61:    function claimAmbientRewards(bytes32 poolIdx, uint32[] memory weeksToClaim) public payable {
65:    function setConcRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {
74:    function setAmbRewards(bytes32 poolIdx, uint32 weekFrom, uint32 weekTo, uint64 weeklyReward) public payable {

// @param and @return missing
85:    function acceptCrocProxyRole(address, uint16 slot) public pure returns (bool) {
```

---

### State variable and function names [2]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol

```solidity
// private and internal `functions` should preppend with `underline`
16:    function initTickTracking(bytes32 poolIdx, int24 tick) internal {
24:    function crossTicks(
39:    function accrueConcentratedGlobalTimeWeightedLiquidity(
69:    function accrueConcentratedPositionTimeWeightedLiquidity(
156:    function claimConcentratedRewards(
198:    function accrueAmbientGlobalTimeWeightedLiquidity(
224:    function accrueAmbientPositionTimeWeightedLiquidity(
256:    function claimAmbientRewards(
```

---

### Observations [3]

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol

```solidity
// remove useless comments
66:        // require(msg.sender == governance_, "Only callable by governance");
75:        // require(msg.sender == governance_, "Only callable by governance");
```

