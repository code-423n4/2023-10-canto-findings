In functions like accrueConcentratedGlobalTimeWeightedLiquidity and accrueAmbientGlobalTimeWeightedLiquidity, you can avoid repetitive calculations by storing intermediate results in variables.

Proof of concept for optimizing the accrueConcentratedGlobalTimeWeightedLiquidity function:
function accrueConcentratedGlobalTimeWeightedLiquidity(bytes32 poolIdx, CurveMath.CurveState memory curve) internal {
    uint32 lastAccrued = timeWeightedWeeklyGlobalConcLiquidityLastSet_[poolIdx];
    
    // Only set time on first call
    if (lastAccrued != 0) {
        uint256 liquidity = curve.concLiq_;
        uint32 currentTime = uint32(block.timestamp);
        uint32 time = lastAccrued;
        
        while (time < currentTime) {
            uint32 currWeek = uint32((time / WEEK) * WEEK);
            uint32 nextWeek = uint32(((time + WEEK) / WEEK) * WEEK);
            uint32 dt = uint32(
                nextWeek < currentTime
                    ? nextWeek - time
                    : currentTime - time
            );
            
            uint256 liquidityToAdd = dt * liquidity;
            timeWeightedWeeklyGlobalConcLiquidity_[poolIdx][currWeek] += liquidityToAdd;
            time += dt;
        }
    }
    
    timeWeightedWeeklyGlobalConcLiquidityLastSet_[poolIdx] = uint32(block.timestamp);
}

In this optimized version:
We calculate currentTime outside the loop to avoid recalculating it repeatedly.
We calculate liquidityToAdd based on dt and liquidity once, reducing redundant calculations.
The loop continues until time reaches currentTime, ensuring that we accrue liquidity accurately up to the current time.
You can apply a similar optimization approach to the accrueAmbientGlobalTimeWeightedLiquidity function. This will make the code more efficient and easier to read while avoiding redundant calculations.





