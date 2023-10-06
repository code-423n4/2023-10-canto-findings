Basic analysis

| Severity | Occurrences |
|----------|-------------|
| High     | 1           |
| Medium   | 1           |
| Low      | 0           |
| Gas      | 0           |
| Analysis | 20 min.     |


# Time allocations

|            |            |
|------------|------------|
| Start date | 06.10.2023 |
| End date   | 06.10.2023 |
| **Total**  | **1 day**  |

| Members       | Positions            | Time spent |
|---------------|----------------------|------------|
| **0x3b**      | full time researcher | ~3H        |


# Short description
In **LiquidityMining**, there exist two distinct accrual functions:

1. `accrueConcentratedPositionTimeWeightedLiquidity`: This function is designed to accrue rewards for the Concentrated Automated Market Maker (CAMM), which leverages the UNI v3 mechanism for concentrated liquidity provision.

2. `accrueAmbientPositionTimeWeightedLiquidity`: On the other hand, this function is responsible for accruing rewards for the Ambient Automated Market Maker (AMM), which closely resembles the UNI v2-style market maker.

Both of these accrual mechanisms operate on a weekly basis. Every week, users' rewards are determined based on their staked positions and whether those positions were within a favorable tick range. At the end of each week, users have the opportunity to claim the rewards associated with their positions.






### Time spent:
3 hours