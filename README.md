# ✨ So you want to sponsor a contest

This `README.md` contains a set of checklists for our contest collaboration.

Your contest will use two repos: 
- **a _contest_ repo** (this one), which is used for scoping your contest and for providing information to contestants (wardens)
- **a _findings_ repo**, where issues are submitted (shared with you after the contest) 

Ultimately, when we launch the contest, this contest repo will be made public and will contain the smart contracts to be reviewed and all the information needed for contest participants. The findings repo will be made public after the contest report is published and your team has mitigated the identified issues.

Some of the checklists in this doc are for **C4 (🐺)** and some of them are for **you as the contest sponsor (⭐️)**.

---

# Repo setup

## ⭐️ Sponsor: Add code to this repo

- [ ] Create a PR to this repo with the below changes:
- [ ] Provide a self-contained repository with working commands that will build (at least) all in-scope contracts, and commands that will run tests producing gas reports for the relevant contracts.
- [ ] Make sure your code is thoroughly commented using the [NatSpec format](https://docs.soliditylang.org/en/v0.5.10/natspec-format.html#natspec-format).
- [ ] Please have final versions of contracts and documentation added/updated in this repo **no less than 24 hours prior to contest start time.**
- [ ] Be prepared for a 🚨code freeze🚨 for the duration of the contest — important because it establishes a level playing field. We want to ensure everyone's looking at the same code, no matter when they look during the contest. (Note: this includes your own repo, since a PR can leak alpha to our wardens!)


---

## ⭐️ Sponsor: Edit this README

Under "SPONSORS ADD INFO HERE" heading below, include the following:

- [ ] Modify the bottom of this `README.md` file to describe how your code is supposed to work with links to any relevent documentation and any other criteria/details that the C4 Wardens should keep in mind when reviewing. ([Here's a well-constructed example.](https://github.com/code-423n4/2022-08-foundation#readme))
  - [x ] When linking, please provide all links as full absolute links versus relative links
  - [x ] All information should be provided in markdown format (HTML does not render on Code4rena.com)
- [ ] Under the "Scope" heading, provide the name of each contract and:
  - [ ] source lines of code (excluding blank lines and comments) in each
  - [ ] external contracts called in each
  - [ ] libraries used in each
- [ ] Describe any novel or unique curve logic or mathematical models implemented in the contracts
- [ ] Does the token conform to the ERC-20 standard? In what specific ways does it differ?
- [ ] Describe anything else that adds any special logic that makes your approach unique
- [ ] Identify any areas of specific concern in reviewing the code
- [ ] Optional / nice to have: pre-record a high-level overview of your protocol (not just specific smart contract functions). This saves wardens a lot of time wading through documentation.
- [ ] Delete this checklist and all text above the line below when you're ready.

---

# Papr contest details
- Total Prize Pool: $60,500 USDC
  - HM awards: $42,500 USDC 
  - QA report awards: $5,000 USDC 
  - Gas report awards: $2,500 USDC 
  - Judge + presort awards: $10,000 USDC 
  - Scout awards: $500 USDC 
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-12-backed-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts December 16, 2022 20:00 UTC
- Ends December 21, 2022 20:00 UTC

## C4udit / Publicly Known Issues

The C4audit output for the contest can be found [here](add link to report) within an hour of contest opening.

*Note for C4 wardens: Anything included in the C4udit output is considered a publicly known issue and is ineligible for awards.*

[ ⭐️ SPONSORS ADD INFO HERE ]

## Overview

Papr facilitates NFT-backed loans. Borrowers deposit allowlisted NFT collateral and mint papr, which can then be exchanged on Uniswap for some other asset. Papr interest rates and the papr trading price are in a constant feedback loop. Interest rates are programmatically updated on chain as a function of papr’s trading price on Uniswap (the lower the trading price, the higher the interest to borrowers), and interest rates in turn affect the trading price, as borrowers open and close loans in response to rates.


![loop_diagram](https://user-images.githubusercontent.com/6678357/207438772-5bddff19-2b25-42d0-8eee-013fb8847fd2.png)

Interest accrues to the value of papr itself: over time, new borrowers are allowed less papr for the exact same collateral. When closing a loan, borrowers repay the exact same amount of papr that they minted. However, due to interest charges, it is expected that the market value of papr will have risen since they opened their loan.

To the extent that borrower incentives push the trading price of papr up over time, corresponding to these interest charges, papr holders are rewarded.

As an analogy, for those familiar with perpetuals, we can say that papr adapts the funding rate mechanism to set interest rates for loans and balance borrower and lender demand. In particular, papr tokens were heavily inspired by Squeeth, which pioneered perpetuals built on Uniswap V3 oracles and continuous, in-kind funding payments.

We *very strongly* encourage everyone to read our [whitepaper](https://backed.mirror.xyz/8SslPvU8of0h-fxoo6AybCpm51f30nd0qxPST8ep08c) to understand more!

## In Scope
Everything in `src/` is in scope. Below is an overview of the non-interface contracts. 
| Contract Name      | SLOC        | Descripton |
| -----------        | ----------- | ----------- |
| [PaprController](https://github.com/with-backed/papr/blob/master/src/PaprController.sol)     | 402       | Inherits `NFTEDAStarterIncentive`, `UniswapOracleFundingRateController`, and `ReservoirOracleUnderwriter`. Facilitates deposit and withdrawal of NFTs, minting and burning of papr, and liquidation auctions.      |
| [UniswapOracleFundingRateController](https://github.com/with-backed/papr/blob/master/src/UniswapOracleFundingRateController.sol)     | 110       | Source of `Target` and `Mark` values. Updates `Target` based on how the papr:underlying pool is trading on Uniswap.|
| [ReservoirOracleUnderwriter](https://github.com/with-backed/papr/blob/master/src/ReservoirOracleUnderwriter.sol)     | 79       | Validates and unpacks oracles messages from [Reservoir](https://github.com/reservoirprotocol/oracle).|
| [PaprToken](https://github.com/with-backed/papr/blob/master/src/PaprToken.sol)     | 23       | Simple ERC20 token that can be minted and burned by its deployer. |
| [NFTEDA](https://github.com/with-backed/papr/blob/master/src/NFTEDA/NFTEDA.sol)     | 73       | (NFT Exponential Decay Auction) Facilitates exponential price decay Dutch auctions for NFTs.|
| [NFTEDAStarterIncentive](https://github.com/with-backed/papr/blob/master/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol)     | 44       | Instance of NFTEDA that offers an auction discount to the starter of the auction. |
| [EDAPrice](https://github.com/with-backed/papr/blob/master/src/NFTEDA/libraries/EDAPrice.sol)     | 18       | A library for computing the current price of an exponential price decay auction.|
| [UniswapHelpers](https://github.com/with-backed/papr/blob/master/src/libraries/UniswapHelpers.sol)     | 65       | Library with various helpers for interacting with Uniswap v3.|
| [OracleLibrary](https://github.com/with-backed/papr/blob/master/src/libraries/OracleLibrary.sol)     | 47       | Library with various oracle methods, all adapted from [Uniswap/v3-periphery/OracleLibrary](https://github.com/Uniswap/v3-periphery/blob/main/contracts/libraries/OracleLibrary.sol).|
| [PoolAddress](https://github.com/with-backed/papr/blob/master/src/libraries/PoolAddress.sol)     | 30       | Library taken [from Uniswap/v3-periphery](https://github.com/Uniswap/v3-periphery/blob/main/contracts/libraries/PoolAddress.sol) with a single line change for solc >= 0.8.0 compatibility.|

## Out of Scope
There are a number of known limitations that are out of scope for the contest 
- It is possible for a malicious/faulty pool to be passed to UniswapOracleFundingRateController
- Many things can go wrong in PaprController and NFTEDA if a malicious/faulty NFT is used
- Many things can go wrong in NFTEDA if a malicious/faulty ERC20 is used for payment asset
- Many things can go wrong in PaprController if a malicious/faulty ERC20 is used for `underlying`
- Additionally, there are myriad possibilities for the state of the system: Target values, Mark values, oracle prices, Uniswap liquidity, and more. We are open to hearing about possible adverse scenarios, but be aware that we are aware of many and are OK with the possibility. 

## Additional Context
[Here](https://docs.google.com/spreadsheets/d/1rWu_lWWhB6WVNjMe9dwTxmnQ77wtUIGFPFhpju1e49I/edit?usp=sharing) Is a simple Google Sheet you can play with to understand how Target changes in response to Mark and time. 

## Running code 
`foundryup` + `forge install` + `forge test` will get you up and going. (More info on Foundry [here](https://github.com/foundry-rs/foundry)). Most of the PaprController tests are forking tests: relying on real chain state. To get these working, add an RPC url value (e.g. from Alchemy or Infura) for `MAINNET_RPC_URL` in a `.env` file. 

