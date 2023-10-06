## [L-01] Claimable week elements are not forced to conform to the week standard used throughout

The `uint32[] memory weeksToClaim` input variable in both the `claimConcentratedRewards()` and `claimAmbientRewards()` functions do not enforce the week normalisation standard used throughout the contract. This exacerbates the unbounded input array issue highlighted already.

Github [link](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L175)

### Recommended Mitigation

Consider requiring that every element in `uint32[] memory weeksToClaim` satisfies an additional require check in the respective for-loops as follows.

```solidity
File: contracts/mixins/LiquidityMining.sol
175:             uint32 week = weeksToClaim[i];
176:             require(week % WEEK == 0, "Invalid week");

268:             uint32 week = weeksToClaim[i];
269:             require(week % WEEK == 0, "Invalid week");
```

## [L-02] Governance can reset or remove liquidity rewards after or during the reward period

It is possible for governance to reset or remove the liquidity rewards for any given pool after, or during, the liquidity mining period. This could be used to exploit liquidity providers.

Github [link](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L65)

To see this, run the following patch, which demonstrates how governance can reset the liqudity rewards after the liquidity period has expired, but before users have claimed.

```diff
diff --git a/test_canto/TestLiquidityMiningOrig.js b/test_canto/TestLiquidityMining.js
index bd21a32..9656cfa 100644
--- a/test_canto/TestLiquidityMiningOrig.js
+++ b/test_canto/TestLiquidityMining.js
@@ -272,6 +272,24 @@ describe("Liquidity Mining Tests", function () {
 
                await time.increase(604800 * 5); // fast forward 1000 seconds so that rewards accrue
 
+               //////////////////////////////////////////////////
+               // OWNER RESETS LIQUIDITY REWARDS
+               //////////////////////////////////////////////////
+               
+               let setRewards0 = abi.encode(
+                       // [code, poolHash, startWeek, endWeek, rewardPerWeek]
+                       ["uint8", "bytes32", "uint32", "uint32", "uint64"],
+                       [
+                               117,
+                               ethers.utils.keccak256(poolHash),
+                               Math.floor(timestampBefore / 604800) * 604800,
+                               Math.floor(timestampBefore / 604800) * 604800 + 604800 * 2,
+                               BigNumber.from("0"), // YOU GET NOTHING! ZERO!
+                       ]
+               );
+               tx = await dex.protocolCmd(8, setRewards0, true);
+               await tx.wait();
+
                //////////////////////////////////////////////////
                // CLAIM REWARDS ACCRUED FROM CONCENTRATED REWARDS
                //////////////////////////////////////////////////
@@ -302,7 +320,7 @@ describe("Liquidity Mining Tests", function () {
 
                // expect dex to have 2 less CANTO since we claimed for 2 weeks worth of rewards
                expect(dexBalBefore.sub(dexBalAfter)).to.equal(
-                       BigNumber.from("2000000000000000000")
+                       BigNumber.from("0")
                );
        });
 });
```

### Recommended Mitigation

Consider dis-allowing the rewards for a pool/week combination to be changed after they have been set once.

## [N-01] Add clarifying comment to storage variable

The `tickTrackingIndexAccruedUpTo_` storage variable lacks a clarifying comment, such as those found with all other equally complex storage variables.

Github [link](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/StorageLayout.sol#L192)

### Recommended Mitigation

Consider adding a clarifying comment to the `tickTrackingIndexAccruedUpTo_` storage variable.

```solidity
File: contracts/mixins/StorageLayout.sol
191:     // Pool -> Position -> Tick -> Index
192:     mapping(bytes32 => mapping(bytes32 => mapping(int32 => uint32))) tickTrackingIndexAccruedUpTo_;
```

## [N-02] Remove commented out logic

No longer used logic remains as commented out code.

```solidity
File: contracts/callpaths/LiquidityMiningPath.sol
66:         // require(msg.sender == governance_, "Only callable by governance");

75:         // require(msg.sender == governance_, "Only callable by governance");
```

Github [link](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/callpaths/LiquidityMiningPath.sol#L66)

### Recommended Mitigation

Remove lines 66 and 75 from `LiquidityMiningPath.sol`.