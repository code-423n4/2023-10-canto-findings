0. QA: LiquidityMining:: & LiquidityMiningPath:: - Strongly recommend to use named import paths instead whenever possible.

https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/mixins/LiquidityMining.sol#L5-L8
https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L5-L8

Suggested changes:
```solidity
-- import "../libraries/SafeCast.sol";
-- import "./PositionRegistrar.sol";
-- import "./StorageLayout.sol";
-- import "./PoolRegistry.sol";'
++ import { SafeCast } from "../libraries/SafeCast.sol";
++ import { PositionRegistrar } from "./PositionRegistrar.sol";
++ import { StorageLayout } from "./StorageLayout.sol";
++ import { PoolRegistry } from "./PoolRegistry.sol";
```

and

Suggested changes:
```solidity
-- import "../libraries/SafeCast.sol";
-- import "../mixins/StorageLayout.sol";
-- import "../mixins/LiquidityMining.sol";
-- import "../libraries/ProtocolCmd.sol";
++ import { SafeCast } from "../libraries/SafeCast.sol";
++ import { StorageLayout } from "../mixins/StorageLayout.sol";
++ import { LiquidityMining } from "../mixins/LiquidityMining.sol";
++ import { ProtocolCmd } from "../libraries/ProtocolCmd.sol";
```

1. QA: LiquidityMining:: - Explicitly mark visibility of state variables, in this case `constant WEEK` has no explicit visibility set.

https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/mixins/LiquidityMining.sol#L13

Standard practice to explicitly declare visibility of functions and state variables.

Suggested change:
```solidity
-- uint256 constant WEEK = 604800; // Week in seconds 604800
++ uint256 constant public WEEK = 604800; // Week in seconds 604800
```

2. QA: LiquidityMining::claimConcentratedRewards - L187: `Ratio` would have been a better choice of words compared to `Percentage`, more accurate, as it is an actual ratio and not actual percentage representation in the arithmetic computation in the line below this comment.

https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/mixins/LiquidityMining.sol#L187

Dev comment:
`// Percentage of this weeks overall in range liquidity that was provided by the user times the overall weekly rewards`

Suggested:
`// Ratio of this weeks overall in range liquidity that was provided by the user times the overall weekly rewards`

3. LOW: LiquidityMining::claimConcentratedRewards - Potential reentrancy vulnerability via low level call() function.

OK if sent to EOA, otherwise if this call is to a user/attacker/controlled contract that contains a fallback() function with callback function inside it to call a vulnerable function in DEX protocol... 

https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/mixins/LiquidityMining.sol#L193

```solidity
(bool sent, ) = owner.call{value: rewardsToSend}("");
```

Recommendation:

Recommend that they do a check for code length > 0...IF they dont allow user controlled contracts here...

4. LOW: claimAmbientRewards() - no payable modifier for address/receiver parameter.

In the context of the `claimAmbientRewards` function, if you do not include the `payable` modifier when declaring the `owner` parameter, and you attempt to use the `owner.call{value: rewardsToSend}("")` to send rewards to the user, it may result in a compilation error or runtime failure, preventing the transfer of rewards to the user. 

However, the calling function from LiquidityMiningPath.sol does indeed pass `payable(msg.sender)` for this address parameter, which would avoid any native token sending issues, but this contract may/will still revert during compiling. LOW because of the low impact, can just modify the contract before compiling again and deploying.

https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/mixins/LiquidityMining.sol#L257

```solidity
-- address owner,
++ address payable owner,
```

5. QA: LiquidityMiningPath:: protocolCmd() &userCmd - change to `external` instead of `public` visibility modifier since this function isnt going to be called internally at all and only called externally via delegatecall via main DEX contract; Reentrancy risk & access control?

https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L26
https://github.com/code-423n4/2023-10-canto/blob/e9183df85f40a3f588f369c8f3a8fdd8f7503f38/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L41

