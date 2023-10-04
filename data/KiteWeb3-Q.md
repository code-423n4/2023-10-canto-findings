## Impact
Dangerous usage of block.timestamp for comparison. block.timestamp can be manipulated by miners. The value of block.timestamp can be influenced by miners to a certain degree, so this may have some risk if miners collude on time manipulation to influence the value returned by these functions. This can lead of mismatched values and a wrong function of the protocol.

## Vulnerability Details
contracts/mixins/LiquidityMining.sol#39-65: 
LiquidityMining.accrueConcentratedGlobalTimeWeightedLiquidity(bytes32,CurveMath.CurveState) 
Dangerous comparisons:
        - lastAccrued != 0 (contracts/mixins/LiquidityMining.sol#47)
        - time < block.timestamp (contracts/mixins/LiquidityMining.sol#50)
        - nextWeek < block.timestamp (contracts/mixins/LiquidityMining.sol#53-57)

contracts/mixins/LiquidityMining.sol#69-154:
LiquidityMining.accrueConcentratedPositionTimeWeightedLiquidity(address,bytes32,int24,int24) 
        Dangerous comparisons:
        - time < block.timestamp && tickTrackingIndex < numTickTracking (contracts/mixins/LiquidityMining.sol#94)
        - tickTracking.enterTimestamp < nextWeek (contracts/mixins/LiquidityMining.sol#105)
        - tickTracking.enterTimestamp < time (contracts/mixins/LiquidityMining.sol#107)
        - tickTracking.exitTimestamp < nextWeek (contracts/mixins/LiquidityMining.sol#119)
        - nextWeek < block.timestamp (contracts/mixins/LiquidityMining.sol#98-102)
        - nextWeek < block.timestamp (contracts/mixins/LiquidityMining.sol#116)

contracts/mixins/LiquidityMining.sol#176:
LiquidityMining.claimConcentratedRewards(address,bytes32,int24,int24,uint32[]) 
        Dangerous comparisons:
        - require(bool,string)(week + WEEK < block.timestamp,Week not over yet) 

contracts/mixins/LiquidityMining.sol#198-222:
LiquidityMining.accrueAmbientGlobalTimeWeightedLiquidity(bytes32,CurveMath.CurveState) 
        Dangerous comparisons:
        - lastAccrued != 0 (contracts/mixins/LiquidityMining.sol#204)
        - time < block.timestamp (contracts/mixins/LiquidityMining.sol#207)
        - nextWeek < block.timestamp (contracts/mixins/LiquidityMining.sol#210-214)

contracts/mixins/LiquidityMining.sol#224-254: LiquidityMining.accrueAmbientPositionTimeWeightedLiquidity(address,bytes32):
        - time < block.timestamp (contracts/mixins/LiquidityMining.sol#237)
        - nextWeek < block.timestamp (contracts/mixins/LiquidityMining.sol#240-244)

contracts/mixins/LiquidityMining.sol#268:
LiquidityMining.claimAmbientRewards(address,bytes32,uint32[]) 
        Dangerous comparisons:
        - require(bool,string)(week + WEEK < block.timestamp,Week not over yet) 

## Recommendations
It is recommended to follow the 15-second rule, i.e., if the time-dependent event can vary by 15 seconds and maintain integrity, it is safe to use a block.timestamp. If possible, it is recommended to use Oracles.