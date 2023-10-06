At https://github.com/code-423n4/2023-10-canto/blob/main/canto_ambient/contracts/mixins/MarketSequencer.sol#L298, 

```solidity
        unchecked { // Only arithmetic in block is ++i which will never overflow
        for (uint i = 0; i < dirs.length; ++i) {
            (int128 nextBase, int128 nextQuote) = applyConcentrated
                (curve, flow, cntx, dirs[i]);
            flow.accumFlow(nextBase, nextQuote);
        }
        }
```

This is fragile because it is easy to insert more arithmetic within this block and not realize that it is unchecked.

Much better to use the common pattern (which also has the advantage of not needing a comment):

```solidity
        for (uint i = 0; i < dirs.length;) {
            (int128 nextBase, int128 nextQuote) = applyConcentrated
                (curve, flow, cntx, dirs[i]);
            flow.accumFlow(nextBase, nextQuote);

            unchecked { ++i; }
        }
```