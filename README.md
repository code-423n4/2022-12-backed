# Papr contest details
- Total Prize Pool: $60,500 USDC
  - HM awards: $42,500 USDC 
  - QA report awards: $5,000 USDC 
  - Gas report awards: $2,500 USDC 
  - Judge + presort awards: $10,000 USDC 
  - Scout awards: $500 USDC 
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-12-papr-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts December 16, 2022 20:00 UTC
- Ends December 21, 2022 20:00 UTC

## C4udit / Publicly Known Issues

The C4audit output for the contest can be found [here](add link to report) within an hour of contest opening.

There are a number of known limitations that are out of scope for the contest 
- It is possible for a malicious/faulty pool to be passed to UniswapOracleFundingRateController
- Many things can go wrong in PaprController and NFTEDA if a malicious/faulty NFT is used
- Many things can go wrong in NFTEDA if a malicious/faulty ERC20 is used for payment asset
- Many things can go wrong in PaprController if a malicious/faulty ERC20 is used for `underlying`
- Additionally, there are myriad possibilities for the state of the system: Target values, Mark values, oracle prices, Uniswap liquidity, and more. We are open to hearing about possible adverse scenarios, but be aware that we are aware of many and are OK with the possibility. 

## Overview

Papr facilitates NFT-backed loans. Borrowers deposit allowlisted NFT collateral and mint papr, which can then be exchanged on Uniswap for some other asset. Papr interest rates and the papr trading price are in a constant feedback loop. Interest rates are programmatically updated on chain as a function of papr’s trading price on Uniswap (the lower the trading price, the higher the interest to borrowers), and interest rates in turn affect the trading price, as borrowers open and close loans in response to rates.


![loop_diagram](https://user-images.githubusercontent.com/6678357/207438772-5bddff19-2b25-42d0-8eee-013fb8847fd2.png)

Interest accrues to the value of papr itself: over time, new borrowers are allowed less papr for the exact same collateral. When closing a loan, borrowers repay the exact same amount of papr that they minted. However, due to interest charges, it is expected that the market value of papr will have risen since they opened their loan.

To the extent that borrower incentives push the trading price of papr up over time, corresponding to these interest charges, papr holders are rewarded.

As an analogy, for those familiar with perpetuals, we can say that papr adapts the funding rate mechanism to set interest rates for loans and balance borrower and lender demand. In particular, papr tokens were heavily inspired by Squeeth, which pioneered perpetuals built on Uniswap V3 oracles and continuous, in-kind funding payments.

We *very strongly* encourage everyone to read our [whitepaper](https://backed.mirror.xyz/8SslPvU8of0h-fxoo6AybCpm51f30nd0qxPST8ep08c) to understand more!

## Commit 
Contest code is hosted on Backed's Github, this is the relevant commit https://github.com/with-backed/papr/tree/7b28c4362c88f35728f139107d3e7b0a3345fed7

### Files in scope
|File|[SLOC](#nowhere "(nSLOC, SLOC, Lines)")|Description and [Coverage](#nowhere "(Lines hit / Total)")|Libraries|
|:-|:-:|:-|:-|
|_Contracts (5)_|
|[src/PaprToken.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/PaprToken.sol)|[23](#nowhere "(nSLOC:23, SLOC:23, Lines:31)")|Simple ERC20 token that can be minted and burned by its deployer., &nbsp;&nbsp;[100.00%](#nowhere "(Hit:2 / Total:2)")| `solmate/*`|
|[src/NFTEDA/extensions/NFTEDAStarterIncentive.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol)|[44](#nowhere "(nSLOC:38, SLOC:44, Lines:78)")|Instance of NFTEDA that offers an auction discount to the starter of the auction., &nbsp;&nbsp;[70.00%](#nowhere "(Hit:7 / Total:10)")| `solmate/*`|
|[src/ReservoirOracleUnderwriter.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/ReservoirOracleUnderwriter.sol) [🧮](#nowhere "Uses Hash-Functions") [🔖](#nowhere "Handles Signatures: ecrecover")|[79](#nowhere "(nSLOC:76, SLOC:79, Lines:118)")|Validates and unpacks oracles messages from Reservoir., &nbsp;&nbsp;[75.00%](#nowhere "(Hit:9 / Total:12)")| `solmate/*` `@reservoir/*`|
|[src/UniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/UniswapOracleFundingRateController.sol)|[110](#nowhere "(nSLOC:110, SLOC:110, Lines:174)")|Source of Target and Mark values. Updates Target based on how the papr:underlying pool is trading on Uniswap., &nbsp;&nbsp;[100.00%](#nowhere "(Hit:49 / Total:49)")| `solmate/*`|
|[src/PaprController.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/PaprController.sol) [📤](#nowhere "Initiates ETH Value Transfer") [🌀](#nowhere "create/create2") [Σ](#nowhere "Unchecked Blocks")|[402](#nowhere "(nSLOC:339, SLOC:402, Lines:560)")|Inherits NFTEDAStarterIncentive, UniswapOracleFundingRateController, and ReservoirOracleUnderwriter. Facilitates deposit and withdrawal of NFTs, minting and burning of papr, and liquidation auctions., &nbsp;&nbsp;[95.33%](#nowhere "(Hit:143 / Total:150)")| `solmate/*` `openzeppelin-contracts/*` `solady/*`|
|_Abstracts (1)_|
|[src/NFTEDA/NFTEDA.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/NFTEDA/NFTEDA.sol) [🧮](#nowhere "Uses Hash-Functions")|[73](#nowhere "(nSLOC:64, SLOC:73, Lines:124)")|(NFT Exponential Decay Auction) Facilitates exponential price decay Dutch auctions for NFTs., &nbsp;&nbsp;[95.65%](#nowhere "(Hit:22 / Total:23)")| `solmate/*`|
|_Libraries (4)_|
|[src/NFTEDA/libraries/EDAPrice.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/NFTEDA/libraries/EDAPrice.sol)|[18](#nowhere "(nSLOC:13, SLOC:18, Lines:23)")|A library for computing the current price of an exponential price decay auction., &nbsp;&nbsp;[0.00%](#nowhere "(Hit:0 / Total:5)")| `solmate/*` `v3-core/*`|
|[src/libraries/PoolAddress.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/libraries/PoolAddress.sol) [🧮](#nowhere "Uses Hash-Functions")|[30](#nowhere "(nSLOC:30, SLOC:30, Lines:48)")|Library taken from Uniswap/v3-periphery with a single line change for solc >= 0.8.0 compatibility., &nbsp;&nbsp;[0.00%](#nowhere "(Hit:0 / Total:4)")||
|[src/libraries/OracleLibrary.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/libraries/OracleLibrary.sol) [Σ](#nowhere "Unchecked Blocks")|[47](#nowhere "(nSLOC:39, SLOC:47, Lines:62)")|Library with various oracle methods, all adapted from Uniswap/v3-periphery/OracleLibrary., &nbsp;&nbsp;[0.00%](#nowhere "(Hit:0 / Total:15)")| `v3-core/*` `fullrange/*`|
|[src/libraries/UniswapHelpers.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/libraries/UniswapHelpers.sol)|[65](#nowhere "(nSLOC:54, SLOC:65, Lines:113)")|Library with various helpers for interacting with Uniswap v3., &nbsp;&nbsp;[44.44%](#nowhere "(Hit:8 / Total:18)")| `v3-core/*` `fullrange/*`|
|_Interfaces (4)_|
|[src/interfaces/IUniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/interfaces/IUniswapOracleFundingRateController.sol)|[8](#nowhere "(nSLOC:8, SLOC:8, Lines:19)")|-||
|[src/interfaces/IFundingRateController.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/interfaces/IFundingRateController.sol)|[17](#nowhere "(nSLOC:17, SLOC:17, Lines:58)")|-| `solmate/*`|
|[src/NFTEDA/interfaces/INFTEDA.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/NFTEDA/interfaces/INFTEDA.sol)|[28](#nowhere "(nSLOC:28, SLOC:28, Lines:66)")|-| `solmate/*`|
|[src/interfaces/IPaprController.sol](https://github.com/with-backed/papr/blob/7b28c4362c88f35728f139107d3e7b0a3345fed7/src/interfaces/IPaprController.sol)|[99](#nowhere "(nSLOC:74, SLOC:99, Lines:273)")|-| `solmate/*`|
|Total (over 14 files):| [1043](#nowhere "(nSLOC:913, SLOC:1043, Lines:1747)") |[83.33%](#nowhere "Hit:240 / Total:288")|

## External imports
* **@reservoir/ReservoirOracle.sol**
  * [src/ReservoirOracleUnderwriter.sol](https://github.com/with-backed/papr/blob/master/src/ReservoirOracleUnderwriter.sol)
* **fullrange/libraries/FullMath.sol**
  * [src/libraries/OracleLibrary.sol](https://github.com/with-backed/papr/blob/master/src/libraries/OracleLibrary.sol)
  * [src/libraries/UniswapHelpers.sol](https://github.com/with-backed/papr/blob/master/src/libraries/UniswapHelpers.sol)
* **fullrange/libraries/TickMath.sol**
  * [src/libraries/OracleLibrary.sol](https://github.com/with-backed/papr/blob/master/src/libraries/OracleLibrary.sol)
  * [src/libraries/UniswapHelpers.sol](https://github.com/with-backed/papr/blob/master/src/libraries/UniswapHelpers.sol)
* **openzeppelin-contracts/access/Ownable2Step.sol**
  * [src/PaprController.sol](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)
* **solady/utils/Multicallable.sol**
  * [src/PaprController.sol](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)
* **solmate/tokens/ERC20.sol**
  * [src/NFTEDA/NFTEDA.sol](https://github.com/with-backed/papr/blob/master/src/NFTEDA/NFTEDA.sol)
  * [src/NFTEDA/interfaces/INFTEDA.sol](https://github.com/with-backed/papr/blob/master/src/NFTEDA/interfaces/INFTEDA.sol)
  * [src/PaprController.sol](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)
  * [src/PaprToken.sol](https://github.com/with-backed/papr/blob/master/src/PaprToken.sol)
  * [src/UniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/master/src/UniswapOracleFundingRateController.sol)
  * [src/interfaces/IFundingRateController.sol](https://github.com/with-backed/papr/blob/master/src/interfaces/IFundingRateController.sol)
  * [src/interfaces/IPaprController.sol](https://github.com/with-backed/papr/blob/master/src/interfaces/IPaprController.sol)
* **solmate/tokens/ERC721.sol**
  * [src/NFTEDA/NFTEDA.sol](https://github.com/with-backed/papr/blob/master/src/NFTEDA/NFTEDA.sol)
  * [src/NFTEDA/interfaces/INFTEDA.sol](https://github.com/with-backed/papr/blob/master/src/NFTEDA/interfaces/INFTEDA.sol)
  * [src/PaprController.sol](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)
  * [src/ReservoirOracleUnderwriter.sol](https://github.com/with-backed/papr/blob/master/src/ReservoirOracleUnderwriter.sol)
  * [src/interfaces/IPaprController.sol](https://github.com/with-backed/papr/blob/master/src/interfaces/IPaprController.sol)
* **solmate/utils/FixedPointMathLib.sol**
  * [src/NFTEDA/extensions/NFTEDAStarterIncentive.sol](https://github.com/with-backed/papr/blob/master/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol)
  * [src/NFTEDA/libraries/EDAPrice.sol](https://github.com/with-backed/papr/blob/master/src/NFTEDA/libraries/EDAPrice.sol)
  * [src/PaprController.sol](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)
  * [src/UniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/master/src/UniswapOracleFundingRateController.sol)
* **solmate/utils/SafeCastLib.sol**
  * [src/UniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/master/src/UniswapOracleFundingRateController.sol)
* **solmate/utils/SafeTransferLib.sol**
  * [src/NFTEDA/NFTEDA.sol](https://github.com/with-backed/papr/blob/master/src/NFTEDA/NFTEDA.sol)
  * [src/PaprController.sol](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)
* **v3-core/contracts/interfaces/IUniswapV3Factory.sol**
  * [src/libraries/UniswapHelpers.sol](https://github.com/with-backed/papr/blob/master/src/libraries/UniswapHelpers.sol)
* **v3-core/contracts/interfaces/IUniswapV3Pool.sol**
  * [src/libraries/OracleLibrary.sol](https://github.com/with-backed/papr/blob/master/src/libraries/OracleLibrary.sol)
  * [src/libraries/UniswapHelpers.sol](https://github.com/with-backed/papr/blob/master/src/libraries/UniswapHelpers.sol)
* **v3-core/contracts/libraries/SafeCast.sol**
  * [src/NFTEDA/libraries/EDAPrice.sol](https://github.com/with-backed/papr/blob/master/src/NFTEDA/libraries/EDAPrice.sol)
  * [src/libraries/UniswapHelpers.sol](https://github.com/with-backed/papr/blob/master/src/libraries/UniswapHelpers.sol)
  
## Scoping details answers
```
- If you have a public code repo, please share it here:  No, private right now
- How many contracts are in scope?: 14  
- Total SLoC for these contracts?:  1043
- How many external imports are there?: 11 
- How many separate interfaces and struct definitions are there for the contracts within scope?:  12
- Does most of your code generally use composition or inheritance?:   about 50/50
- How many external calls?:   5
- What is the overall line coverage percentage provided by your tests?:  80%
- Is there a need to understand a separate part of the codebase / get context in order to audit this part of the protocol?:   True
- Please describe required context:   Protocol is built on top of Uniswap v3, and so some uniswap understanding required. 
- Does it use an oracle?:  true; We use Uniswap v3 on chain and also use Reservoir oracle for NFT prices, in the TrustUs model
- Does the token conform to the ERC20 standard?:  Yes
- Are there any novel or unique curve logic or mathematical models?: We use a funding rate mechanism similar to squeeth, see formula in whitepaper https://backed.mirror.xyz/8SslPvU8of0h-fxoo6AybCpm51f30nd0qxPST8ep08c
- Does it use a timelock function?:  No
- Is it an NFT?: No
- Does it have an AMM?: We use Uniswap v3  
- Is it a fork of a popular project?:   false
- Does it use rollups?:   false
- Is it multi-chain?:  false
- Does it use a side-chain?: false
```

## Additional Context
[Here](https://docs.google.com/spreadsheets/d/1rWu_lWWhB6WVNjMe9dwTxmnQ77wtUIGFPFhpju1e49I/edit?usp=sharing) Is a simple Google Sheet you can play with to understand how Target changes in response to Mark and time. 

If you're using Slither, ensure you're using the latest version to avoid this sourceMap issue: crytic/crytic-compile#281. 

## Running code 
`foundryup` + `forge install` + `forge test` will get you up and going. (More info on Foundry [here](https://github.com/foundry-rs/foundry)). Most of the PaprController tests are forking tests: relying on real chain state. To get these working, add an RPC url value (e.g. from Alchemy or Infura) for `MAINNET_RPC_URL` in a `.env` file or otherwise run `export MAINNET_RPC_URL=<your-mainnet-rpc-url-goes-here>`

