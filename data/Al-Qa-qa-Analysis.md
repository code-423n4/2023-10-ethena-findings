# Summary

|List|Title|Description|
| ---- | ----- | ----------- |
|1|Overview of the EthenaLabs protocol|Services and Features provided by the protocol|
|2|System Overview|Detailing the parts of the system and explaining each part separately|
|3|System Architecture|Explaining the Architecture Used by EthenaLabs to Achieve their Goal|
|4|Documentation|Overview of the documentation and its quality|
|5|Centralization Risks|Risks that should be taken into consideration|
|6|Learnings from this Audit|New concepts and information I learned from auditing EthenaLabs protocol|
|7|My view on the project|My opinion about the protocol's success when it launches|
|8|Time spent on analysis|The total time I spent analyzing and auditing the codebase| 


# Overview of the EthenaLabs protocol

EthenaLabs is a stable value crypto token tied to dollar value. Ethenalab team thinks about the problems related to the stable crypto tokens tied to the dollar and managed to solve this problem using different solutions and approaches.

What makes EthenaLabs dollar different from other stable dollar tokens, is their mechanism to achieve stability. The stability of the dollar is not achieved by saving dollars as collateral equal to the tokens minted like **USDC** or **USDT**, nor it is achieved by cryptocurrencies collaterals that exceed their price like **DAI**. The dollar achieves its stability value using Delta-Neutral Stability.

In brief, the protocol uses Staked Ethereum tokens (stETH) to mint new dollars, and with these tokens, he makes short tradings for these ETHUSD perpetual in Off-Exchange Settlement.

So if the ETH price goes down, which means the collateral value decreases, short Ethereum perpetual will make profits, which will compensate for the loss of Ethereum collateral value.

If the other thing happens, and the Ethereum price goes up, the collateral value increases, but this will not increase the value of the Protocol dollar token (USDe), since the Short Ethereum perpetual is losing in this situation, so it will be neutral.

- **ETH** value increased -> Short Trades value decreased -> collateral value remains constant.
- **ETH** value decreased -> Short Trades value increased -> collateral value remains constant.

Another feature the protocol offers is Yeild Generation, Let's see how the protocol is making money.

The protocol generates yields in two ways:
- Staking Ethereum (use Ethereum in consense of the Ethereum blockchain).
- Funding fees provided by perpetual exchanges from shorting ether position.

The protocol provides a staking solution for **USDe** holders, they can put their dollars in the contract, participate in the yield generation, and earn money.

EthenaLabs will not make users cover the loss if the yields are negative. knowing that the yields calculated by the protocol in the previous months and years are kind of hard to become negative, because of the delta-neutral strategy, in addition to fees, and yields generated when staking Ethereum. 


# System overview
After we take a look at the protocol and its functionality and features, let's dive into the system itself, and know how it works.

The System consists of 3 core functionality:
- Minting (Creating) new dollars and Redeeming (Burning) dollars.
- Staking **USDe** tokens and unStake tokens.
- Authorization contracts (restricts protocol functionalities to specific trusted People).

## Protocol Smart Contract Files and Privilate Roles

- `SingleAdminAccessControl.sol`
  - The Authorization layer to be used in the protocol contracts, to restrict some actions to specific people.
  - Only the `ADMIN_ROLE` can give people other roles like `MINTER_ROLE`.
  - `ADMIN_ROLE` can be changed from one address to another.
  - There should be only one address that has `ADMIN_ROLE`.

- `USDe.sol`
  - The token contract that is tied to the USD dollar value.
  - Only one address can mint new tokens which are `minter`.
  - Has an `ADMIN_ROLE` which has the ability to change the `minter` address.
  - `USDe holder`: can transfer tokens in addition to burning them.
 
- `EthenaMinting.sol`
  - People can't mint or redeem tokens directly, they need to make a signature request for minting or redeeming first, and after approval from EthenaLab team, it can be used to mint or redeem tokens.
  - Only `MINTER_ROLE` can mint new tokens, and only `REDEEMER_ROLE` can redeem tokens.
  - The number of tokens to mint should not exceed `maxMintPerBlock`, and the number of tokens to redeem should not exceed `maxRedeemPerBlock`.
  - `ADMIN_ROLE` can change `maxMintPerBlock` value, and `maxRedeemPerBlock` value, add/remove Supported assets (tokens) to be used as collaterals, add/remove Custodian Addresses (The addresses that will take the collaterals `Off-Exchange`).
  - `GATE_KEEPER` can add/remove `MINTER_ROLE` or `REDEEM_ROLE`.
  - `ADMIN_ROLE` can add/remove any role.
 
- `StakedUSDe.sol`
  - The contract that will be used as a vault for **USDe** tokens by users to earn yields (more money).
  - Rewards can be added to the contract every 8 hours.
  - `SOFT_RESTRICTED_STAKER_ROLE` address can't stake his **USDe** tokens or get **stUSDe** tokens minted to him
  - `FULL_RESTRICTED_STAKER_ROLE` address can't burn his **stUSDe** tokens to unstake his **USDe** tokens, neither to transfer **stUSDe** tokens. His balance can be manipulated by the `ADMIN_ROLE` address.
  - `BLACKLIST_MANAGER_ROLE` can restrict users either make them `SOFT_RESTRICTED_STAKER_ROLE` or `FULL_RESTRICTED_STAKER_ROLE`.
  - `ADMIN_ROLE` can transfer any ERC20 tokens locked in the contract except **USDe**.
 
- `StakedUSDeV2.sol`
  - Has all the functionality of `StakedUSDe.sol`, but with the ability to add a period that users should wait in order to unstake their **USDe** tokens.
  - The user's balance will be transferred to `USDeSile.sol` contract, and can only be claimed by the user if the period (cooldown) passes.
  - `ADMIN_ROLE` can change the cooldown period, but its maximum is 90 days.
 
- `USDeSilo.sol`
  - The contract that will hold users **USDe** tokens, when they request unstake.
  - Only the Staking contract can withdraw tokens, which is `StakedUSDeV2.sol`.


# System Architecture
The system relies on a combination of blockchain technology, Web development technology, and Trading Companies.

- The token **USDe** is a decentralized token built on top of the Ethereum network, in addition to Staking and earning yields functionality.
- The system relies on trading platforms in order to open short prepeturals to make delta-neutral which will make the token (**USDe**) price relatively unchanged.
- The system relies heavily on trusted roles to make the system work properly, which should be saved securely to not break the system in the future.
- The system depends on Web2 technology to make some of the functionalities that are responsible for making the system work properly like:
  - Storing Authorization account information (wallet addresses) in a secure cloud-providing solution (AWS).
  - Calculated the earned yields.
  - Distributing the earned yields to the stakers.
 
EthenaLabs should be sure of the safety of the servers they are relying on, and the trading platforms they are using, since they are part of the all system of the protocol. If one of these things gets compromised it may lead to the breaking of the token system.


# Documentation
EthenaLabs provides good and detailed documentation, which makes it easy to understand how the system works. They also provide a good explanation for the common processes that will be made by users like minting and staking, they provide a video guide, in addition to written documentation.

The documentation is great for understanding the workflow of the system, and how to use it. However the documentation for the smart contract code itself is not detailed, there is no reference for smart contracts explaining each smart contract (.sol) file usage, functions, and variables. This makes the auditing process harder, and updating the system (protocol) in the future may be harder too, because of a lack of detailed code documentation.

Unfortunately, I found that code comments are not detailed, like the technical documentation too, this is not a good thing, as it will be hard to understand what functions do. Some functions have comments that explain their functionality, and how they work, but others don't.


# Centralization risks

### Systematic Risks
As we said in the Architecture section, the system relies on three different technologies: Blockchain, Web2, and trading platforms. This makes the system easier to compromise. The hackers now have 3 different options if they succeed in breaking one of them, the system will be broken.

- Any error in the trading platforms that EthenaLabs relies on to make ETH/USD shorts can affect token stability and price.
- If the servers used to store Authorized address information get hacked, the hacker can control the system.
- Any problem or issue that happens to the server, trading platforms, or the Authorized account, will lead to the break of the system.

### Centralization Risks
The system relies heavily on trusted roles, it uses a lot of roles to manage the system in a bureaucratic way, so if one Authorized role gets compromised, it can compromise the roles below it.

The system has a very important role which is `ADMIN_ROLE`, any anyone reaches the information of this address, and succeeds in controlling it, the system is over.

- `ADMIN_ROLE`: Controls all the system.
- `MINTER_ROLE`: can mint any number of tokens that not exceeding `maxMintPerBlock` each block.
- `REDEEMER_ROLE`: can redeem any number of tokens that not exceeding maxRedeemPerBlock each block.
- `GATE_KEEPER`: can add/remove `MINTER_ROLE` or `REDEEMER_ROLE`.
- `BLACKLIST_MANAGER_ROLE`: can block users from staking or withdrawing their tokens in the staking contract.
  

# Learnings from this Audit
Our journey in auditing and analyzing EthenaLabs protocol code gave us a lot of valuable information.

- We learned more about stable crypto tokens.
- Some trading concepts like Delta-Neutral, Futures vs Perpetuals.
- Vaults standard `ERC4626`, and OpenZeppelin implementation for it.
- Increase our information for permit tokens, and approve spending by signature mechanism.
- OpenZeppelin Access control contracts, and Access control mechanism in smart contracts.

# My view on the project
- The project offers a stable ERC20 token tied to dollar values with the same value of collaterals to pay in order to get some tokens, this is great as it can attract people to use it as it is decentralized, you don't need to pay an excess amount in order to get dollars, so it providers a great user experience for users.

- The protocol allows users to stake their **USDe** to earn more money, this can encourage people to mint tokens, and enjoy the staking process. Since the protocol team promised to bear the loss in negative yield, this can encourage people to stake their tokens as they will either win or in the worst case no win and no lose (neutral).

_Although the benefits of the system are great there are some points I think can make people less addicted to using this protocol._

- The users should rely on the EthenaLab team in order to mint or redeem, and this is not the best user experience for users, especially Web3 users.
- Allowing the `minter` to mint any amount of tokens he wants to can make people worry about minting more tokens which can cause inflation.
- Relying on a lot of centralized solutions like AWS, servers, and trading platforms can decrease the adoption for people to use the protocol since Web3 users prefer decentralization.
- The staking process is attractive but letting `BLACKLIST_MANAGER_ROLE` can freeze staker tokens, will make people afraid of participating in the staking process.
- The protocol can't afford losses if the yields are negative or the Delta-neutral strategy that controls the stability of the price of **USDe** forever, The company will have to make people share the loss if the company can't afford it only. And if this happens the level of trust in the company will decrease significantly.


# Time spent on analysis
I spent 4 days auditing and analyzing the codebase, with a total hours ~20 Hours.


### Time spent:
20 hours