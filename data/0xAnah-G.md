# CANTO GAS OPTIMIZATIONS



## INTRODUCTION
Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."

Emphasizing the significance of our recommendations, we strive to improve the code's efficiency while maintaining its readability. We recognize the importance of code that is easily maintainable and understandable for both developers and auditors.



## TABLE OF FINDINGS
| Number |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| G-01| Declaring Unnecessary variables | 2 | 6 |
| G-02| Add unchecked blocks for subtractions where the operands cannot underflow | 3 | 120 |
| G-03| Calculations should be memoized rather than re-calculating them | 3 | 6 per iteration |





## [G-01] Declaring Unnecessary variables
some varibles were defined even though they are used once. Not defining variables can reduce gas cost and contract size.

### 2 Instances
1. #### Declaration of `curve` variable in the `claimConcentratedRewards()` function is unnecessary
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L169
```solidity
file: contracts/mixins/LiquidityMining.sol

156:    function claimConcentratedRewards(
157:        address payable owner,
158:        bytes32 poolIdx,
159:        int24 lowerTick,
160:        int24 upperTick,
161:        uint32[] memory weeksToClaim
162:    ) internal {
163:        accrueConcentratedPositionTimeWeightedLiquidity(
164:            owner,
165:            poolIdx,
166:            lowerTick,
167:            upperTick
168:        );
169:        CurveMath.CurveState memory curve = curves_[poolIdx];   //@audit unnecessary variable declaration
170:        // Need to do a global accrual in case the current tick was already in range for a long time without any modifications that triggered an accrual
171:        accrueConcentratedGlobalTimeWeightedLiquidity(poolIdx, curve);
172:        bytes32 posKey = encodePosKey(owner, poolIdx, lowerTick, upperTick);
.
.
.        
196:    }
```
In the `claimConcentratedRewards()` function above a variable `curve` was declared but was only used once in the function. We could use `curves_[poolIdx]` directly in place of the `curve` variable. The diff below shows how the code should be refactored: 
```diff
diff --git a/canto_ambient/contracts/mixins/LiquidityMining.sol b/canto_ambient/contracts/mixins/LiquidityMining.sol
index a83c273..154d1f8 100644
--- a/canto_ambient/contracts/mixins/LiquidityMining.sol
+++ b/canto_ambient/contracts/mixins/LiquidityMining.sol
@@ -166,9 +166,8 @@ contract LiquidityMining is PositionRegistrar {
             lowerTick,
             upperTick
         );
-        CurveMath.CurveState memory curve = curves_[poolIdx];
         // Need to do a global accrual in case the current tick was already in range for a long time without any modifications that triggered an accrual
-        accrueConcentratedGlobalTimeWeightedLiquidity(poolIdx, curve);
+        accrueConcentratedGlobalTimeWeightedLiquidity(poolIdx, curves_[poolIdx]);
         bytes32 posKey = encodePosKey(owner, poolIdx, lowerTick, upperTick);
         uint256 rewardsToSend;
         for (uint256 i; i < weeksToClaim.length; ++i) {
```
```
Estimated gas saved: 3 gas units
```


2. #### Declaration of `curve` variable in the `claimAmbientRewards()` function is unnecessary
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L261
```solidity
file: contracts/mixins/LiquidityMining.sol

256:    function claimAmbientRewards(
257:        address owner,
258:        bytes32 poolIdx,
259:        uint32[] memory weeksToClaim
260:    ) internal {
261:        CurveMath.CurveState memory curve = curves_[poolIdx];   //@audit unnecessary variable declaration
262:        accrueAmbientPositionTimeWeightedLiquidity(payable(owner), poolIdx);
263:        accrueAmbientGlobalTimeWeightedLiquidity(poolIdx, curve);
264:        bytes32 posKey = encodePosKey(owner, poolIdx);
.
.
.
289:    }
```
In the `claimAmbientRewards()` function above a variable `curve` was declared but was only used once in the function. We could use `curves_[poolIdx]` directly in place of the `curve` variable. The diff below shows how the code should be refactored: 
```diff
diff --git a/canto_ambient/contracts/mixins/LiquidityMining.sol b/canto_ambient/contracts/mixins/LiquidityMining.sol
index a83c273..8caac95 100644
--- a/canto_ambient/contracts/mixins/LiquidityMining.sol
+++ b/canto_ambient/contracts/mixins/LiquidityMining.sol
@@ -258,8 +258,7 @@ contract LiquidityMining is PositionRegistrar {
         bytes32 poolIdx,
         uint32[] memory weeksToClaim
     ) internal {
-        CurveMath.CurveState memory curve = curves_[poolIdx];
-        accrueAmbientPositionTimeWeightedLiquidity(payable(owner), poolIdx);
+        accrueAmbientPositionTimeWeightedLiquidity(payable(owner), curves_[poolIdx]);
         accrueAmbientGlobalTimeWeightedLiquidity(poolIdx, curve);
         bytes32 posKey = encodePosKey(owner, poolIdx);
         uint256 rewardsToSend;
```
```
Estimated gas saved: 3 gas units
```



## [G-02] Add unchecked blocks for subtractions where the operands cannot underflow
There are some checks to avoid an underflow, so it's safe to use unchecked to have some gas savings.

### 3 Instances
1. #### Use `unchecked` block in `accrueConcentratedGlobalTimeWeightedLiquidity` function to cut on gas cost
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L56
```solidity
file: contracts/mixins/LiquidityMining.sol

39:    function accrueConcentratedGlobalTimeWeightedLiquidity(
40:        bytes32 poolIdx,
41:        CurveMath.CurveState memory curve
42:    ) internal {
43:        uint32 lastAccrued = timeWeightedWeeklyGlobalConcLiquidityLastSet_[
44:            poolIdx
45:        ];
46:        // Only set time on first call
47:        if (lastAccrued != 0) {
48:            uint256 liquidity = curve.concLiq_;
49:            uint32 time = lastAccrued;
50:            while (time < block.timestamp) {
51:                uint32 currWeek = uint32((time / WEEK) * WEEK);
52:                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
53:                uint32 dt = uint32(
54:                    nextWeek < block.timestamp
55:                        ? nextWeek - time  //@audit calculation can be done in an unchecked block
56:                        : block.timestamp - time //@audit calculation can be done in an unchecked block
57:                );
.
.
.
65;    }
```
We can reduce the gas cost of the `accrueConcentratedGlobalTimeWeightedLiquidity()` function by putting the calculation `uint32(nextWeek < block.timestamp ? nextWeek - time : block.timestamp - time)` in an `unchecked` block so as save on gas cost of the compiler underflow checks since `uint32(((time + WEEK) / WEEK) * WEEK)` statement ensures that the value of `nextWeek` variable would always be greater than the value of the `time` variable and also the `block.timestamp` would always be greater than the value of `time` variable. Implementing this changes we could save up to `40` gas units. The code should be refactored as shown in the diff below:
```diff
diff --git a/canto_ambient/contracts/mixins/LiquidityMining.sol b/canto_ambient/contracts/mixins/LiquidityMining.sol
index a83c273..a13acaa 100644
--- a/canto_ambient/contracts/mixins/LiquidityMining.sol
+++ b/canto_ambient/contracts/mixins/LiquidityMining.sol
@@ -50,11 +50,11 @@ contract LiquidityMining is PositionRegistrar {
             while (time < block.timestamp) {
                 uint32 currWeek = uint32((time / WEEK) * WEEK);
                 uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
-                uint32 dt = uint32(
-                    nextWeek < block.timestamp
-                        ? nextWeek - time
-                        : block.timestamp - time
-                );
+                uint32 dt;
+                unchecked {
+                    dt = uint32(nextWeek < block.timestamp ? nextWeek - time : block.timestamp - time);
+                }
+
                 timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += dt * liquidity;
                 time += dt;
             }
```
```
Estimated gas saved: 40 gas units
```

2. #### Use `unchecked` block in `accrueAmbientGlobalTimeWeightedLiquidity` function to cut on gas cost
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L213
```solidity
file: contracts/mixins/LiquidityMining.sol

198:    function accrueAmbientGlobalTimeWeightedLiquidity(
199:        bytes32 poolIdx,
200:        CurveMath.CurveState memory curve
201:    ) internal {
202:        uint32 lastAccrued = timeWeightedWeeklyGlobalAmbLiquidityLastSet_[poolIdx];
203:        // Only set time on first call
204:        if (lastAccrued != 0) {
205:            uint256 liquidity = curve.ambientSeeds_;
206:            uint32 time = lastAccrued;
207:            while (time < block.timestamp) {
208:                uint32 currWeek = uint32((time / WEEK) * WEEK);
209:                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
210:                uint32 dt = uint32(
211:                    nextWeek < block.timestamp
212:                        ? nextWeek - time  //@audit calculation can be done in an unchecked block
213:                        : block.timestamp - time  //@audit calculation can be done in an unchecked block
214:                );
.
.
.
222:    }
```
We can reduce the gas cost of the `accrueAmbientGlobalTimeWeightedLiquidity()` function by putting the calculation `uint32(nextWeek < block.timestamp ? nextWeek - time : block.timestamp - time)` in an `unchecked` block so as save on gas cost of the compiler underflow checks since `uint32(((time + WEEK) / WEEK) * WEEK)` statement ensures that the value of `nextWeek` variable would always be greater than the value of the `time` variable and also the `block.timestamp` would always be greater than the value of `time` variable. Implementing this changes we could save up to `40` gas units. The code should be refactored as shown in the diff below:
```diff
diff --git a/canto_ambient/contracts/mixins/LiquidityMining.sol b/canto_ambient/contracts/mixins/LiquidityMining.sol
index a83c273..bd7e3b6 100644
--- a/canto_ambient/contracts/mixins/LiquidityMining.sol
+++ b/canto_ambient/contracts/mixins/LiquidityMining.sol
@@ -207,11 +207,12 @@ contract LiquidityMining is PositionRegistrar {
             while (time < block.timestamp) {
                 uint32 currWeek = uint32((time / WEEK) * WEEK);
                 uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
-                uint32 dt = uint32(
-                    nextWeek < block.timestamp
-                        ? nextWeek - time
-                        : block.timestamp - time
-                );
+                uint32 dt;
+
+                unchecked {
+                    dt = uint32(nextWeek < block.timestamp ? nextWeek - time : block.timestamp - time);
+                }
+
                 timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;
                 time += dt;
             }
```
```
Estimated gas saved: 40 gas units
```


3. #### Use `unchecked` block in `accrueAmbientPositionTimeWeightedLiquidity` function to cut on gas cost
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L243
```solidity
file: contracts/mixins/LiquidityMining.sol

224:    function accrueAmbientPositionTimeWeightedLiquidity(
225:        address payable owner,
226:        bytes32 poolIdx
227:    ) internal {
228:        bytes32 posKey = encodePosKey(owner, poolIdx);
229:        uint32 lastAccrued = timeWeightedWeeklyPositionAmbLiquidityLastSet_[
230:            poolIdx
231:        ][posKey];
232:        // Only init time on first call
233:        if (lastAccrued != 0) {
234:            AmbientPosition storage pos = lookupPosition(owner, poolIdx);
235:            uint256 liquidity = pos.seeds_;
236:            uint32 time = lastAccrued;
237:            while (time < block.timestamp) {
238:                uint32 currWeek = uint32((time / WEEK) * WEEK);
239:                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
240:                uint32 dt = uint32(
241:                    nextWeek < block.timestamp
242:                        ? nextWeek - time  //@audit calculation can be done in an unchecked block
243:                        : block.timestamp - time  //@audit calculation can be done in an unchecked block
244:                );
.
.
.
    }
```
We can reduce the gas cost of the `accrueAmbientPositionTimeWeightedLiquidity()` function by putting the calculation `uint32(nextWeek < block.timestamp ? nextWeek - time : block.timestamp - time)` in an `unchecked` block so as save on gas cost of the compiler underflow checks since `uint32(((time + WEEK) / WEEK) * WEEK)` statement ensures that the value of `nextWeek` variable would always be greater than the value of the `time` variable and also the `block.timestamp` would always be greater than the value of `time` variable. Implementing this changes we could save up to `40` gas units. The code should be refactored as shown in the diff below:
```diff
diff --git a/canto_ambient/contracts/mixins/LiquidityMining.sol b/canto_ambient/contracts/mixins/LiquidityMining.sol
index a83c273..9ddc25f 100644
--- a/canto_ambient/contracts/mixins/LiquidityMining.sol
+++ b/canto_ambient/contracts/mixins/LiquidityMining.sol
@@ -237,11 +237,11 @@ contract LiquidityMining is PositionRegistrar {
             while (time < block.timestamp) {
                 uint32 currWeek = uint32((time / WEEK) * WEEK);
                 uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
-                uint32 dt = uint32(
-                    nextWeek < block.timestamp
-                        ? nextWeek - time
-                        : block.timestamp - time
-                );
+                uint32 dt;
+
+                unchecked {
+                    dt = uint32(nextWeek < block.timestamp ? nextWeek - time : block.timestamp - time);
+                }
                 timeWeightedWeeklyPositionAmbLiquidity_[poolIdx][posKey][
                     currWeek
                 ] += dt * liquidity;
```
```
Estimated gas saved: 40 gas units
```



## [G-03] Calculations should be memoized rather than re-calculating them
In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls to pure functions and returning the cached result when the same inputs occur again.

### 3 Instances
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L88
- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L139
```solidity
file: contracts/mixins/LiquidityMining.sol

69:     function accrueConcentratedPositionTimeWeightedLiquidity(
70:         address payable owner,
71:         bytes32 poolIdx,
72:         int24 lowerTick,
73:         int24 upperTick
74:     ) internal {
75:         RangePosition72 storage pos = lookupPosition(
76:             owner,
77:             poolIdx,
78:             lowerTick,
79:             upperTick
80:         );
81:         bytes32 posKey = encodePosKey(owner, poolIdx, lowerTick, upperTick);
82:         uint32 lastAccrued = timeWeightedWeeklyPositionConcLiquidityLastSet_[
83:             poolIdx
84:         ][posKey];
85:         // Only set time on first call
86:         if (lastAccrued != 0) {
87:             uint256 liquidity = pos.liquidity_;
88:            for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {  //@audit memoize `upperTick - 10` calculation
.
.
.
138:        } else {
139:            for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {  //@audit memoize `upperTick - 10` calculation
140:                uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
141:                if (numTickTracking > 0) {
142:                    if (tickTracking_[poolIdx][i][numTickTracking - 1].exitTimestamp == 0) {
143:                        // Tick currently active
144:                        tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking - 1;
145:                    } else {
146:                        tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking;
147:                    }
148:                }
149:            }
150:        }
151:        timeWeightedWeeklyPositionConcLiquidityLastSet_[poolIdx][
152:            posKey
153:        ] = uint32(block.timestamp);
154:    }
```
Rather than having to perform `upperTick - 10` calculation on every iteration of the loop we could memoize (i.e cache result of the calculation) the calculation there by saving the upto `6` gas unit per iteration of the loop. The code should be refactored as shown in the diff below:
```diff
diff --git a/canto_ambient/contracts/mixins/LiquidityMining.sol b/canto_ambient/contracts/mixins/LiquidityMining.sol
index a83c273..73bc136 100644
--- a/canto_ambient/contracts/mixins/LiquidityMining.sol
+++ b/canto_ambient/contracts/mixins/LiquidityMining.sol
@@ -85,7 +85,8 @@ contract LiquidityMining is PositionRegistrar {
         // Only set time on first call
         if (lastAccrued != 0) {
             uint256 liquidity = pos.liquidity_;
-            for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
+            int24 _upperTick = upperTick - 10;
+            for (int24 i = lowerTick + 10; i <= _upperTick; ++i) {
                 uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i];
                 uint32 origIndex = tickTrackingIndex;
                 uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
@@ -136,7 +137,8 @@ contract LiquidityMining is PositionRegistrar {
                 }
             }
         } else {
-            for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
+            int24 _upperTick = upperTick - 10;
+            for (int24 i = lowerTick + 10; i <= _upperTick; ++i) {
                 uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
                 if (numTickTracking > 0) {
                     if (tickTracking_[poolIdx][i][numTickTracking - 1].exitTimestamp == 0) {```
```
Estimated gas saved: 6 gas units per loop iteration
```

- https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L184
```solidity
file: contracts/mixins/LiquidityMining.sol

156:    function claimConcentratedRewards(
157:        address payable owner,
158:        bytes32 poolIdx,
159:        int24 lowerTick,
160:        int24 upperTick,
161:        uint32[] memory weeksToClaim
162:    ) internal {
163:        accrueConcentratedPositionTimeWeightedLiquidity(
164:            owner,
165:            poolIdx,
166:            lowerTick,
167:            upperTick
168:        );
169:        CurveMath.CurveState memory curve = curves_[poolIdx];
170:        // Need to do a global accrual in case the current tick was already in range for a long time without any
.
.
. 
182:            if (overallInRangeLiquidity > 0) {
183:                uint256 inRangeLiquidityOfPosition;
184:                for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j) {  //@audit memoize `upperTick - 10` calculation
185:                    inRangeLiquidityOfPosition += timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];
186:                }
187:                // Percentage of this weeks overall in range liquidity that was provided by the user times the overall weekly rewards
188:                rewardsToSend += inRangeLiquidityOfPosition * concRewardPerWeek_[poolIdx][week] / overallInRangeLiquidity;
189:            }
190:            concLiquidityRewardsClaimed_[poolIdx][posKey][week] = true;
191:        }
192:        if (rewardsToSend > 0) {
193:            (bool sent, ) = owner.call{value: rewardsToSend}("");
194:            require(sent, "Sending rewards failed");
195:        }
196:    }
```
Rather than having to perform `upperTick - 10` calculation on every iteration of the loop we could memoize (i.e cache result of the calculation) the calculation there by saving the upto `6` gas unit per iteration of the loop. The code should be refactored as shown in the diff below:
```diff
diff --git a/canto_ambient/contracts/mixins/LiquidityMining.sol b/canto_ambient/contracts/mixins/LiquidityMining.sol
index a83c273..97f6391 100644
--- a/canto_ambient/contracts/mixins/LiquidityMining.sol
+++ b/canto_ambient/contracts/mixins/LiquidityMining.sol
@@ -181,7 +181,8 @@ contract LiquidityMining is PositionRegistrar {
             uint256 overallInRangeLiquidity = timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][week];
             if (overallInRangeLiquidity > 0) {
                 uint256 inRangeLiquidityOfPosition;
-                for (int24 j = lowerTick + 10; j <= upperTick - 10; ++j) {
+                int24 _upperTick = upperTick - 10;
+                for (int24 j = lowerTick + 10; j <= _upperTick; ++j) {
                     inRangeLiquidityOfPosition += timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][week][j];
                 }
                 // Percentage of this weeks overall in range liquidity that was provided by the user times the overall weekly rewards
```
```
Estimated gas saved: 6 gas units per loop iteration
```


## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.