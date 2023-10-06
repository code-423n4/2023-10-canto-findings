
## Cache array’s length in for loops

This will save ~3 gas per iteration

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L174

``` 
for (uint256 i; i < weeksToClaim.length; ++i) {
```

## Use custom errors instead of require with a string to save gas

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L194

```
require(sent, "Sending rewards failed");
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L176

```
require(week + WEEK < block.timestamp, "Week not over yet");
            require(
                !concLiquidityRewardsClaimed_[poolIdx][posKey][week],
                "Already claimed"
            );
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L268

```
require(week + WEEK < block.timestamp, "Week not over yet");
            require(
                !ambLiquidityRewardsClaimed_[poolIdx][posKey][week],
                "Already claimed"
            );
```

## Cache block.timestamp to save gas

Each call to block.timestamp costs 2 gas in the while loops

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L50

```
while (time < block.timestamp) {
                uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
                uint32 dt = uint32(
                    nextWeek < block.timestamp
                        ? nextWeek - time
                        : block.timestamp - time
                );
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L94

```
while (time < block.timestamp && tickTrackingIndex < numTickTracking) {
                    TickTracking memory tickTracking = tickTracking_[poolIdx][i][tickTrackingIndex];
                    uint32 currWeek = uint32((time / WEEK) * WEEK);
                    uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
                    uint32 dt = uint32(
                        nextWeek < block.timestamp
                            ? nextWeek - time
                            : block.timestamp - time
                    );
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L207

```
while (time < block.timestamp) {
                uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
                uint32 dt = uint32(
                    nextWeek < block.timestamp
                        ? nextWeek - time
                        : block.timestamp - time
                );
                timeWeightedWeeklyGlobalAmbLiquidity_[poolIdx][currWeek] += dt * liquidity;
                time += dt;
```

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L237

```
            while (time < block.timestamp) {
                uint32 currWeek = uint32((time / WEEK) * WEEK);
                uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
                uint32 dt = uint32(
                    nextWeek < block.timestamp
                        ? nextWeek - time
                        : block.timestamp - time
                );
```

## .call (bool success) transfer can be done gas-optimized

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L193

```
 (bool sent, ) = owner.call{value: rewardsToSend}("");
```

Can be changed to 

```
bool success;
 assembly {
 success := call(gas(), dest, amount, 0, 0)
 }

```

So that return value doesn’t have to be stored, saving gas.

## >= costs less gas than >

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L50
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L54
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L94
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L99
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L105
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L107
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L119
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L141
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L174
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L176
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L192
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L207
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L211
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L237
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L241
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L266
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L268
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L276
https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L285

## ++i is more gas efficient than i++

https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/LiquidityMining.sol#L122

```
tickTrackingIndex++;
```



