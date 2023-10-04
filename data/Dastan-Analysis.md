The code contains nested loops and multiple if and else statements, which can make it hard to read and maintain. To improve the readability and maintainability of your code, you can refactor it by breaking it down into smaller functions and using more structured control flow. 

##refactored code:


function accrueConcentratedPositionTimeWeightedLiquidity(
    address payable owner,
    bytes32 poolIdx,
    int24 lowerTick,
    int24 upperTick
) internal {
    RangePosition72 storage pos = lookupPosition(
        owner,
        poolIdx,
        lowerTick,
        upperTick
    );
    bytes32 posKey = encodePosKey(owner, poolIdx, lowerTick, upperTick);
    uint32 lastAccrued = timeWeightedWeeklyPositionConcLiquidityLastSet_[poolIdx][posKey];

    if (lastAccrued == 0) {
        initializePositionAccrual(poolIdx, posKey, lowerTick, upperTick);
    }

    for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
        accruePositionForTick(poolIdx, posKey, i, lastAccrued, pos.liquidity_);
    }

    timeWeightedWeeklyPositionConcLiquidityLastSet_[poolIdx][posKey] = uint32(block.timestamp);
}

function initializePositionAccrual(
    bytes32 poolIdx,
    bytes32 posKey,
    int24 lowerTick,
    int24 upperTick
) internal {
    for (int24 i = lowerTick + 10; i <= upperTick - 10; ++i) {
        uint32 numTickTracking = uint32(tickTracking_[poolIdx][i].length);

        if (numTickTracking > 0) {
            if (tickTracking_[poolIdx][i][numTickTracking - 1].exitTimestamp == 0) {
                tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking - 1;
            } else {
                tickTrackingIndexAccruedUpTo_[poolIdx][posKey][i] = numTickTracking;
            }
        }
    }
}

function accruePositionForTick(
    bytes32 poolIdx,
    bytes32 posKey,
    int24 tick,
    uint32 lastAccrued,
    uint256 liquidity
) internal {
    uint32 tickTrackingIndex = tickTrackingIndexAccruedUpTo_[poolIdx][posKey][tick];
    uint32 origIndex = tickTrackingIndex;
    uint32 numTickTracking = uint32(tickTracking_[poolIdx][tick].length);
    uint32 time = lastAccrued;

    while (time < block.timestamp && tickTrackingIndex < numTickTracking) {
        TickTracking memory tickTracking = tickTracking_[poolIdx][tick][tickTrackingIndex];
        uint32 currWeek = uint32((time / WEEK) * WEEK);
        uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
        uint32 dt = uint32(
            nextWeek < block.timestamp
                ? nextWeek - time
                : block.timestamp - time
        );

        uint32 tickActiveStart;
        uint32 tickActiveEnd;

        if (tickTracking.enterTimestamp < nextWeek) {
            if (tickTracking.enterTimestamp < time) {
                tickActiveStart = time;
            } else {
                tickActiveStart = tickTracking.enterTimestamp;
            }

            if (tickTracking.exitTimestamp == 0) {
                tickActiveEnd = uint32(nextWeek < block.timestamp ? nextWeek : block.timestamp);
            } else {
                if (tickTracking.exitTimestamp < nextWeek) {
                    tickActiveEnd = tickTracking.exitTimestamp;
                    tickTrackingIndex++;
                    dt = tickActiveEnd - tickActiveStart;
                } else {
                    tickActiveEnd = nextWeek;
                }
            }

            timeWeightedWeeklyPositionInRangeConcLiquidity_[poolIdx][posKey][currWeek][tick] +=
                (tickActiveEnd - tickActiveStart) * liquidity;
        }
        time += dt;
    }

    if (tickTrackingIndex != origIndex) {
        tickTrackingIndexAccruedUpTo_[poolIdx][posKey][tick] = tickTrackingIndex;
    }
}


### Time spent:
2 hours