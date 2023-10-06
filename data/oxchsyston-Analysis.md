          ABOUT THE PROTOCOL
     The audit is a part of the Canto protocol which is use to incentivize liquidity for Ambient pools deployed on Canto. The two contracts in the audit, the `ILiquidityMining.sol` is the proxy contract which is only callable by itself and it's child contracts through `delegate` calls. The LiquidityMining.sol is used to incentivize a certain width of liquidity. This makes up the liquidity mining sidecar.
          APPROACH TAKEN IN AUDITING THE CODEBASE
In auditing the codebase, I evaluated any possibility of recieving more reward than the Liquidity provider should. I also look at the possibility of users recieving any undue  reward or recieving reward when they are not eligible for any incentive.
           CODEBASE QUALITY ANALYSIS
The codebase was well written but it lacked a well detailed in-code comment which should aid in understanding the mindset of the developer. This made made the codebase a little harder to audit. Overall the codebase was of a high standard.
     There are no centralization risk in the codebase. The reward mechanism was a little vague as in the example, if the reward for the week is 10 Canto, and there are two liquidity providers the reward will be distributed to both of them 5 Canto each. But this does not explain a situation where both LPs are not providing equal liquidity to the protocol. This may lead to possible unfair distribution of the rewards. There are no systemic risks in the protocol codebase.







### Time spent:
13 hours