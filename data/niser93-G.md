## Gas Optimizations


|       |Issue  |Instances|Total Gas Saved|
|:-----:|:------|:-------:|:-------------:|
|[[GO-01](#go-01-remove-ternary-operator-in-order-to-use-unchecked-and--instead-of-)]|Remove ternary operator in order to use unchecked and >= instead of <|1|4526|
|[[GO-02](#go-02-state-variable-read-in-a-loop)]|State variable read in a loop|5|1908|
|[[GO-03](#go-03-using-ternary-operator-instead-of-ifelse)]|Using ternary operator instead of if/else|1|1896|
|[[GO-04](#go-04-using-a-positive-conditional-flow-to-save-a-not-opcode)]|Using a positive conditional flow to save a NOT opcode|1|2750|

Total: 8 instances over 4 issues with **11080 gas** saved


Gas totals are estimates using `npx hardhat test` and `hardhat-gas-reporter`.



## Gas Optimizations

### [GO-01] Remove ternary operator in order to use unchecked and >= instead of <


*Estimated saving:*
| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `LiquidityMiningPath` | 1540432 | 1535906 | -4526 | -0.29% |



*There are 1 instances of this issue:*              

```diff
File: LiquidityMining.sol

-   uint32 dt = uint32(
-       nextWeek < block.timestamp
-           ? nextWeek - time
-           : block.timestamp - time
-   );
+   uint32 dt;
+   if(nextWeek >= block.timestamp) {
+       unchecked{dt = uint32(block.timestamp - time);}
+   }
+   else{
+       unchecked{dt = uint32(nextWeek - time);}
+   }

```
[[53-57](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L53-L57)]


### [GO-02] State variable read in a loop
The state variable should be cached in a local variable rather than reading it on every iteration of the for-loop, which will replace each Gwarmaccess (**100 gas**) with a much cheaper stack read.


*Estimated saving:* 
| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `LiquidityMiningPath` | 1540432 | 1538524 | -1908 | -0.12% |



*There are 5 instances of this issue:*              


```diff
File: LiquidityMining.sol

    uint256 liquidity = pos.liquidity_;
+   uint256 wweek = WEEK;
    for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
        uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i];
        uint32 origIndex = tickTrackingIndex;
        uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
        uint32 time = lastAccrued;
        // Loop through all in-range time spans for the tick or up to the current time (if it is still in range)
        while (time < block.timestamp && tickTrackingIndex < numTickTracking) {
            TickTracking memory tickTracking = tickTracking_[poolIdx][i][tickTrackingIndex];
-              uint32 currWeek = uint32((time / WEEK) * WEEK);
-              uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
+               uint32 currWeek = uint32((time / WEEK) * WEEK);
+               uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);

```
[[87-97](https://github.com/code-423n4/2023-10-canto/blob/29c92a926453a49c8935025a4d3de449150fc2ff/canto_ambient/contracts/mixins/LiquidityMining.sol#L87-L97)]


### [GO-03] Using ternary operator instead of if/else


*Estimated saving:*
| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `LiquidityMiningPath` | 1540432 | 1538536 | -1896 | -0.12% |


*There are 1 instances of this issue:*              

```diff
File: LiquidityMining.sol

-   if (tickTracking.enterTimestamp < time) {
-       // Tick was already active when last claim happened, only accrue from last claim timestamp
-       tickActiveStart = time;
-   } else {
-       // Tick has become active this week
-       tickActiveStart = tickTracking.enterTimestamp;
-   }
+   tickActiveStart = tickTracking.enterTimestamp >= time ? time : tickTracking.enterTimestamp;

```
[[53-57](https://github.com/code-423n4/2023-10-canto/blob/40edbe0c9558b478c84336aaad9b9626e5d99f34/canto_ambient/contracts/mixins/LiquidityMining.sol#L53-L57)]

### [GO-04] Using a positive conditional flow to save a NOT opcode
In order to save some gas (NOT opcode costing 3 gas), switch to a positive statement



```diff
- if(!condition){
-     action1();
- }else{
-     action2();
- }
+ if(condition){
+     action2();
+ }else{
+     action1();
+ }
```            

*Estimated saving:*
| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `LiquidityMiningPath` | 1540432 | 1537682 | -2750 | -0.18% |



*There is 1 instance of this issue:*               

```
File: LiquidityMining.sol


  86           if (lastAccrued != 0) {
  87               uint256 liquidity = pos.liquidity_;
  88               for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
  89                   uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i];
  90                   uint32 origIndex = tickTrackingIndex;
  91                   uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
  92                   uint32 time = lastAccrued;
  93                   // Loop through all in-range time spans for the tick or up to the current time (if it is still in range)
  94                   while (time < block.timestamp && tickTrackingIndex < numTickTracking) {
  95                       TickTracking memory tickTracking = tickTracking_[poolIdx][i][tickTrackingIndex];
  96                       uint32 currWeek = uint32((time / WEEK) * WEEK);
  97                       uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
  98                       uint32 dt = uint32(
  99                           nextWeek < block.timestamp
  100                              ? nextWeek - time
  101                              : block.timestamp - time
  102                      );
  103                      uint32 tickActiveStart; // Timestamp to use for the liquidity addition
  104                      uint32 tickActiveEnd;
  105                      if (tickTracking.enterTimestamp < nextWeek) {
  106                          // Tick was active before next week, need to add the liquidity
  107                          if (tickTracking.enterTimestamp < time) {
  108                              // Tick was already active when last claim happened, only accrue from last claim timestamp
  109                              tickActiveStart = time;
  110                          } else {
  111                              // Tick has become active this week
  112                              tickActiveStart = tickTracking.enterTimestamp;
  113                          }
  114                          if (tickTracking.exitTimestamp == 0) {
  115                              // Tick still active, do not increase index because we need to continue from here
  116                              tickActiveEnd = uint32(nextWeek < block.timestamp ? nextWeek : block.timestamp);
  117                          } else {
  118                              // Tick is no longer active
  119                              if (tickTracking.exitTimestamp < nextWeek) {
  120                                  // Exit was in this week, continue with next tick
  121                                  tickActiveEnd = tickTracking.exitTimestamp;
  122                                  tickTrackingIndex++;
  123                                  dt = tickActiveEnd - tickActiveStart;
  124                              } else {
  125                                  // Exit was in next week, we need to consider the current tick there (i.e. not increase the index)
  126                                  tickActiveEnd = nextWeek;
  127                              }
  128                          }
  129                          timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +=
  130                              (tickActiveEnd - tickActiveStart) * liquidity;
  131                      }
  132                      time += dt;
  133                  }
  134                  if (tickTrackingIndex != origIndex) {
  135                      tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = tickTrackingIndex;
  136                  }
  137              }
  138          } else {
  139              for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
  140                  uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
  141                  if (numTickTracking > 0) {
  142                      if (tickTracking_[poolIdx][i][numTickTracking - 1].exitTimestamp == 0) {
  143                          // Tick currently active
  144                          tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking - 1;
  145                      } else {
  146                          tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking;
  147                      }
  148                  }
  149              }
  150          }

```
[[86-150](https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L86-L150)]

```diff
File: LiquidityMining.sol

-   86           if (lastAccrued != 0) {
-   87               uint256 liquidity = pos.liquidity_;
-   88               for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
-   89                   uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i];
-   90                   uint32 origIndex = tickTrackingIndex;
-   91                   uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
-   92                   uint32 time = lastAccrued;
-   93                   // Loop through all in-range time spans for the tick or up to the current time (if it is still in range)
-   94                   while (time < block.timestamp && tickTrackingIndex < numTickTracking) {
-   95                       TickTracking memory tickTracking = tickTracking_[poolIdx][i][tickTrackingIndex];
-   96                       uint32 currWeek = uint32((time / WEEK) * WEEK);
-   97                       uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
-   98                       uint32 dt = uint32(
-   99                           nextWeek < block.timestamp
-   100                              ? nextWeek - time
-   101                              : block.timestamp - time
-   102                      );
-   103                      uint32 tickActiveStart; // Timestamp to use for the liquidity addition
-   104                      uint32 tickActiveEnd;
-   105                      if (tickTracking.enterTimestamp < nextWeek) {
-   106                          // Tick was active before next week, need to add the liquidity
-   107                          if (tickTracking.enterTimestamp < time) {
-   108                              // Tick was already active when last claim happened, only accrue from last claim timestamp
-   109                              tickActiveStart = time;
-   110                          } else {
-   111                              // Tick has become active this week
-   112                              tickActiveStart = tickTracking.enterTimestamp;
-   113                          }
-   114                          if (tickTracking.exitTimestamp == 0) {
-   115                              // Tick still active, do not increase index because we need to continue from here
-   116                              tickActiveEnd = uint32(nextWeek < block.timestamp ? nextWeek : block.timestamp);
-   117                          } else {
-   118                              // Tick is no longer active
-   119                              if (tickTracking.exitTimestamp < nextWeek) {
-   120                                  // Exit was in this week, continue with next tick
-   121                                  tickActiveEnd = tickTracking.exitTimestamp;
-   122                                  tickTrackingIndex++;
-   123                                  dt = tickActiveEnd - tickActiveStart;
-   124                              } else {
-   125                                  // Exit was in next week, we need to consider the current tick there (i.e. not increase the index)
-   126                                  tickActiveEnd = nextWeek;
-   127                              }
-   128                          }
-   129                          timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +=
-   130                              (tickActiveEnd - tickActiveStart) * liquidity;
-   131                      }
-   132                      time += dt;
-   133                  }
-   134                  if (tickTrackingIndex != origIndex) {
-   135                      tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = tickTrackingIndex;
-   136                  }
-   137              }
-   138          } else {
-   139              for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
-   140                  uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
-   141                  if (numTickTracking > 0) {
-   142                      if (tickTracking_[poolIdx][i][numTickTracking - 1].exitTimestamp == 0) {
-   143                          // Tick currently active
-   144                          tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking - 1;
-   145                      } else {
-   146                          tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking;
-   147                      }
-   148                  }
-   149              }
-   150          }


+   86             if (lastAccrued == 0) {
+   87                 for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
+   88                     uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
+   89                     if (numTickTracking > 0) {
+   90                         if (tickTracking_[poolIdx][i][numTickTracking - 1].exitTimestamp == 0) {
+   91                             // Tick currently active
+   92                             tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking - 1;
+   93                         } else {
+   94                             tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking;
+   95                         }
+   96                     }
+   97                 }
+   98             } else {
+   99                 uint256 liquidity = pos.liquidity_;
+   100                 for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
+   101                     uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i];
+   102                     uint32 origIndex = tickTrackingIndex;
+   103                     uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);
+   104                     uint32 time = lastAccrued;
+   105                     // Loop through all in-range time spans for the tick or up to the current time (if it is still in range)
+   106                     while (time < block.timestamp && tickTrackingIndex < numTickTracking) {
+   107                         TickTracking memory tickTracking = tickTracking_[poolIdx][i][tickTrackingIndex];
+   108                         uint32 currWeek = uint32((time / WEEK) * WEEK);
+   109                         uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
+   110                         uint32 dt = uint32(
+   111                             nextWeek < block.timestamp
+   112                                 ? nextWeek - time
+   113                                 : block.timestamp - time
+   114                         );
+   115                         uint32 tickActiveStart; // Timestamp to use for the liquidity addition
+   116                         uint32 tickActiveEnd;
+   117                         if (tickTracking.enterTimestamp < nextWeek) {
+   118                             // Tick was active before next week, need to add the liquidity
+   119                             if (tickTracking.enterTimestamp < time) {
+   120                                 // Tick was already active when last claim happened, only accrue from last claim timestamp
+   121                                 tickActiveStart = time;
+   122                             } else {
+   123                                 // Tick has become active this week
+   124                                 tickActiveStart = tickTracking.enterTimestamp;
+   125                             }
+   126                             if (tickTracking.exitTimestamp == 0) {
+   127                                 // Tick still active, do not increase index because we need to continue from here
+   128                                 tickActiveEnd = uint32(nextWeek < block.timestamp ? nextWeek : block.timestamp);
+   129                             } else {
+   130                                 // Tick is no longer active
+   131                                 if (tickTracking.exitTimestamp < nextWeek) {
+   132                                     // Exit was in this week, continue with next tick
+   133                                     tickActiveEnd = tickTracking.exitTimestamp;
+   134                                     tickTrackingIndex++;
+   135                                     dt = tickActiveEnd - tickActiveStart;
+   136                                 } else {
+   137                                     // Exit was in next week, we need to consider the current tick there (i.e. not increase the index)
+   138                                     tickActiveEnd = nextWeek;
+   139                                 }
+   140                             }
+   141                             timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][i] +=
+   142                                 (tickActiveEnd - tickActiveStart) * liquidity;
+   143                         }
+   144                         time += dt;
+   145                     }
+   146                     if (tickTrackingIndex != origIndex) {
+   147                         tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = tickTrackingIndex;
+   148                     }
+   149                 }
+   150             }

```


