# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 2 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 3 |
| [GAS-3](#GAS-3) | Cache array length outside of loop | 3 |
| [GAS-4](#GAS-4) | State variables should be cached in stack variables rather than re-reading them from storage | 2 |
| [GAS-5](#GAS-5) | Use calldata instead of memory for function arguments that do not get mutated | 7 |
| [GAS-6](#GAS-6) | Use Custom Errors | 2 |
| [GAS-7](#GAS-7) | Don't initialize variables with default value | 3 |
| [GAS-8](#GAS-8) | Using `private` rather than `public` for constants, saves gas | 1 |
| [GAS-9](#GAS-9) | Use != 0 instead of > 0 for unsigned integer comparison | 16 |
| [GAS-10](#GAS-10) | `internal` functions not called by the contract should be removed | 12 |
### <a name="GAS-1"></a>[GAS-1] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (2)*:
```solidity
File: src/PaprController.sol

371:             if (collateralConfigs[i].collateral == address(0)) revert IPaprController.InvalidCollateral();

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)

```solidity
File: src/UniswapOracleFundingRateController.sol

112:         if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/UniswapOracleFundingRateController.sol)

### <a name="GAS-2"></a>[GAS-2] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (3)*:
```solidity
File: src/PaprController.sol

32:     bool public override liquidationsLocked;

35:     bool public immutable override token0IsUnderlying;

60:     mapping(address => bool) public override isAllowed;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)

### <a name="GAS-3"></a>[GAS-3] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (3)*:
```solidity
File: src/PaprController.sol

99:         for (uint256 i = 0; i < collateralArr.length;) {

118:         for (uint256 i = 0; i < collateralArr.length;) {

370:         for (uint256 i = 0; i < collateralConfigs.length;) {

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)

### <a name="GAS-4"></a>[GAS-4] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (2)*:
```solidity
File: src/UniswapOracleFundingRateController.sol

103:         _lastTwapTick = UniswapHelpers.poolCurrentTick(pool);

165:                 targetMarkRatio = targetMarkRatioMax;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/UniswapOracleFundingRateController.sol)

### <a name="GAS-5"></a>[GAS-5] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (7)*:
```solidity
File: src/NFTEDA/NFTEDA.sol

35:     function auctionID(INFTEDA.Auction memory auction) public pure virtual returns (uint256) {

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/NFTEDA.sol)

```solidity
File: src/NFTEDA/interfaces/INFTEDA.sol

61:     function auctionID(Auction memory auction) external pure returns (uint256);

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/interfaces/INFTEDA.sol)

```solidity
File: src/PaprController.sol

68:         string memory name,

69:         string memory symbol,

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)

```solidity
File: src/PaprToken.sol

18:     constructor(string memory name, string memory symbol)

18:     constructor(string memory name, string memory symbol)

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprToken.sol)

```solidity
File: src/ReservoirOracleUnderwriter.sol

64:     function underwritePriceForCollateral(ERC721 asset, PriceKind priceKind, OracleInfo memory oracleInfo)

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/ReservoirOracleUnderwriter.sol)

### <a name="GAS-6"></a>[GAS-6] Use Custom Errors
[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

*Instances (2)*:
```solidity
File: src/PaprController.sol

236:             revert("wrong caller");

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)

```solidity
File: src/libraries/OracleLibrary.sol

39:         require(twapDuration != 0, "BP");

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/OracleLibrary.sol)

### <a name="GAS-7"></a>[GAS-7] Don't initialize variables with default value

*Instances (3)*:
```solidity
File: src/PaprController.sol

99:         for (uint256 i = 0; i < collateralArr.length;) {

118:         for (uint256 i = 0; i < collateralArr.length;) {

370:         for (uint256 i = 0; i < collateralConfigs.length;) {

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)

### <a name="GAS-8"></a>[GAS-8] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (1)*:
```solidity
File: src/PaprController.sol

30:     uint256 public constant BIPS_ONE = 1e4;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)

### <a name="GAS-9"></a>[GAS-9] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (16)*:
```solidity
File: src/NFTEDA/NFTEDA.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/NFTEDA.sol)

```solidity
File: src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol)

```solidity
File: src/NFTEDA/interfaces/INFTEDA.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/interfaces/INFTEDA.sol)

```solidity
File: src/NFTEDA/libraries/EDAPrice.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/libraries/EDAPrice.sol)

```solidity
File: src/PaprController.sol

170:         if (request.swapParams.minOut > 0) {

172:         } else if (request.debt > 0) {

241:         if (amount0Delta > 0) {

245:             require(amount1Delta > 0); // swaps entirely within 0-liquidity regions are not supported

283:         if (excess > 0) {

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)

```solidity
File: src/interfaces/IFundingRateController.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/interfaces/IFundingRateController.sol)

```solidity
File: src/interfaces/IPaprController.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/interfaces/IPaprController.sol)

```solidity
File: src/interfaces/IUniswapOracleFundingRateController.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/interfaces/IUniswapOracleFundingRateController.sol)

```solidity
File: src/libraries/OracleLibrary.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/OracleLibrary.sol)

```solidity
File: src/libraries/PoolAddress.sol

4: pragma solidity >=0.8.0;

32:         require(key.token0 < key.token1);

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/PoolAddress.sol)

```solidity
File: src/libraries/UniswapHelpers.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/UniswapHelpers.sol)

### <a name="GAS-10"></a>[GAS-10] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (12)*:
```solidity
File: src/NFTEDA/libraries/EDAPrice.sol

10:     function currentPrice(

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/libraries/EDAPrice.sol)

```solidity
File: src/libraries/OracleLibrary.sol

10:     function getQuoteAtTick(int24 tick, uint128 baseAmount, address baseToken, address quoteToken)

34:     function timeWeightedAverageTick(int56 startTick, int56 endTick, int56 twapDuration)

56:     function latestCumulativeTick(address pool) internal view returns (int56) {

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/OracleLibrary.sol)

```solidity
File: src/libraries/PoolAddress.sol

22:     function getPoolKey(address tokenA, address tokenB, uint24 fee) internal pure returns (PoolKey memory) {

31:     function computeAddress(address factory, PoolKey memory key) internal pure returns (address pool) {

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/PoolAddress.sol)

```solidity
File: src/libraries/UniswapHelpers.sol

31:     function swap(

69:     function deployAndInitPool(address tokenA, address tokenB, uint24 feeTier, uint160 sqrtRatio)

82:     function poolCurrentTick(address pool) internal returns (int24) {

92:     function poolsHaveSameTokens(address pool1, address pool2) internal view returns (bool) {

100:     function isUniswapPool(address pool) internal view returns (bool) {

110:     function oneToOneSqrtRatio(uint256 token0ONE, uint256 token1ONE) internal pure returns (uint160) {

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/UniswapHelpers.sol)


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Missing checks for `address(0)` when assigning values to address state variables | 3 |
| [NC-2](#NC-2) |  `require()` / `revert()` statements should have descriptive reason strings | 11 |
| [NC-3](#NC-3) | Event is missing `indexed` fields | 7 |
| [NC-4](#NC-4) | Functions not used internally could be marked external | 4 |
### <a name="NC-1"></a>[NC-1] Missing checks for `address(0)` when assigning values to address state variables

*Instances (3)*:
```solidity
File: src/ReservoirOracleUnderwriter.sol

52:         oracleSigner = _oracleSigner;

53:         quoteCurrency = _quoteCurrency;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/ReservoirOracleUnderwriter.sol)

```solidity
File: src/UniswapOracleFundingRateController.sol

115:         pool = _pool;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/UniswapOracleFundingRateController.sol)

### <a name="NC-2"></a>[NC-2]  `require()` / `revert()` statements should have descriptive reason strings

*Instances (11)*:
```solidity
File: src/PaprController.sol

245:             require(amount1Delta > 0); // swaps entirely within 0-liquidity regions are not supported

312:             revert IPaprController.InvalidCollateralAccountPair();

318:             revert IPaprController.NotLiquidatable();

322:             revert IPaprController.MinAuctionSpacing();

371:             if (collateralConfigs[i].collateral == address(0)) revert IPaprController.InvalidCollateral();

415:             revert IPaprController.InvalidCollateral();

431:             revert IPaprController.OnlyCollateralOwner();

450:             revert IPaprController.ExceedsMaxDebt(debt, max);

471:         if (newDebt > max) revert IPaprController.ExceedsMaxDebt(newDebt, max);

473:         if (newDebt >= 1 << 200) revert IPaprController.DebtAmountExceedsUint200();

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)

```solidity
File: src/libraries/PoolAddress.sol

32:         require(key.token0 < key.token1);

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/PoolAddress.sol)

### <a name="NC-3"></a>[NC-3] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (7)*:
```solidity
File: src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

21:     event SetAuctionCreatorDiscount(uint256 discount);

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol)

```solidity
File: src/NFTEDA/interfaces/INFTEDA.sol

50:     event EndAuction(uint256 indexed auctionID, uint256 price);

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/interfaces/INFTEDA.sol)

```solidity
File: src/interfaces/IFundingRateController.sol

9:     event UpdateTarget(uint256 newTarget);

11:     event SetFundingPeriod(uint256 fundingPeriod);

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/interfaces/IFundingRateController.sol)

```solidity
File: src/interfaces/IPaprController.sol

67:     event IncreaseDebt(address indexed account, ERC721 indexed collateralAddress, uint256 amount);

85:     event ReduceDebt(address indexed account, ERC721 indexed collateralAddress, uint256 amount);

90:     event AllowCollateral(address indexed collateral, bool isAllowed);

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/interfaces/IPaprController.sol)

### <a name="NC-4"></a>[NC-4] Functions not used internally could be marked external

*Instances (4)*:
```solidity
File: src/ReservoirOracleUnderwriter.sol

64:     function underwritePriceForCollateral(ERC721 asset, PriceKind priceKind, OracleInfo memory oracleInfo)

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/ReservoirOracleUnderwriter.sol)

```solidity
File: src/UniswapOracleFundingRateController.sol

45:     function updateTarget() public override returns (uint256 nTarget) {

63:     function newTarget() public view override returns (uint256) {

72:     function mark() public view returns (uint256) {

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/UniswapOracleFundingRateController.sol)


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Unsafe ERC20 operation(s) | 7 |
| [L-2](#L-2) | Unspecific compiler version pragma | 10 |
### <a name="L-1"></a>[L-1] Unsafe ERC20 operation(s)

*Instances (7)*:
```solidity
File: src/PaprController.sol

101:             collateralArr[i].addr.transferFrom(msg.sender, address(this), collateralArr[i].id);

202:             underlying.transfer(params.swapFeeTo, fee);

203:             underlying.transfer(proceedsTo, amountOut - fee);

226:             underlying.transfer(params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);

514:             underlying.transfer(params.swapFeeTo, fee);

515:             underlying.transfer(proceedsTo, amountOut - fee);

546:             papr.transfer(auction.nftOwner, totalOwed - debtCached);

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)

### <a name="L-2"></a>[L-2] Unspecific compiler version pragma

*Instances (10)*:
```solidity
File: src/NFTEDA/NFTEDA.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/NFTEDA.sol)

```solidity
File: src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol)

```solidity
File: src/NFTEDA/interfaces/INFTEDA.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/interfaces/INFTEDA.sol)

```solidity
File: src/NFTEDA/libraries/EDAPrice.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/NFTEDA/libraries/EDAPrice.sol)

```solidity
File: src/interfaces/IFundingRateController.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/interfaces/IFundingRateController.sol)

```solidity
File: src/interfaces/IPaprController.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/interfaces/IPaprController.sol)

```solidity
File: src/interfaces/IUniswapOracleFundingRateController.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/interfaces/IUniswapOracleFundingRateController.sol)

```solidity
File: src/libraries/OracleLibrary.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/OracleLibrary.sol)

```solidity
File: src/libraries/PoolAddress.sol

4: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/PoolAddress.sol)

```solidity
File: src/libraries/UniswapHelpers.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/with-backed/papr/blob/master/src/libraries/UniswapHelpers.sol)

