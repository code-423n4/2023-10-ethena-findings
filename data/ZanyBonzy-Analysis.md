## **1. Codebase Reveiew**
### **1.1 General Review**
  - Ethena offers a permissionless stable coin USDe and yields to users in its ecosystem. Users can stake their `USDe` in exchange for stUSDe, which increases in value relative to USDe as the protocol earns yield.
  - Ethena allows users to earn a yield on their stETH by shorting ETH perps. This is a relatively low-risk strategy, as the long and short positions are offsetting each other. 
  - The generated revenue is distributed to users by sending it to an insurance fund and then to the staking contract every 8 hours. This means that Ethena users can earn a higher yield on their stETH than they could by simply staking it.
  - To maintain delta neutrality, The long stETH and short ETH perps creates a position with value that's fixed to the time of it's creation.
### **1.2 Scope and Architecture Overview**
#### **1.2.1 Contracts**
##### StableCoin contract 
  - [USDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol) - is a standard stablecoin ERC20 contract. Through this contract, ownership can be set for the owner, the minter role can be granted/removed to the minter(by the owner) and `USDe` tokens minted to the required account.
##### Minting & Redeeming contract
  - [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol) - is the contract mints and redeems USDe in a single transaction. Here, gatekeeper, minter and redeemer roles can be granted and revoked(not admin), tokens can be minted and redeemed, assets can be added into allow and deny lists, and users can add delegates for signing. To prevent excessive minting or redemptions, the max_mint_per_block and max_redeem_per_block features are implemented. 
##### Staking contracts
  - [StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol) - is the contract that allows users to stake their USDe tokens and earn stUSDe. The Ethena DAO decides how the rewards are distributed. Depending on if there's a cooldown period set or not, users can withdraw their tokens at anytime (following the ERC4626 standard), or if users have to wait for the cooldown period to expire before they can withdraw their tokens(breaking the ERC4626 standard). During the cooldown period, USDe is sent to the silo contract to hold, from which it is withdrawn to the user.
  - [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol) - is like the StakedUSDeV2 contract but without the cooldown period, it instead has the additional functionality of blacklisting certain user types, softly(prevents participation in staking) or fully (full account blacklisting). 
  - [USDeSilo.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol) - stores `USDe` during the stake cooldown process. USDe is withdrawn into the staking contracts to be passed on to the user.
##### Access-control contracts
  - [SingleAdminAccessControl.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol) -  is an abstract contract that provides a single admin role. Calls are made through it to grant, transfer, and revoke privileged roles.

#### **1.2.2 Roles**
##### StableCoin contract
  - **Owner/Admin** - is in charge of the contract. It adds and removes minters, cannot renounce ownership of the contract, can only transfer to a new owner.
  - **Minter** - mints the stablecoin tokens.
##### Minting & Redeeming contract
  - **Admin** - is a gnosis safe cold wallet. It owns and deploys the contract. It has grant/revoke privileged roles, set the maximum mint/redemption limits per block or disable mint/redemption all together, add/remove supported assets and custodian wallets etc.
  - **Gatekeeper** - In charge of disabling mint/redemption in cases of emergencies. It can also remove redeemer and minter roles.
  - **Minter** - calls the mint function with user’s signed orders. It doesn't hold any funds because user’s collateral is moved to the custodian address as defined in the route.
  - **Redeemer** - calls the redeem function with user’s signed orders. It holds the funds needed for redemption.
  - **Users and their delegates** - interact with the contract to mint/redeem tokens from their assets. Users can directly perform these interactions or add/remove delegate addresses to perform them. 
##### Staking contracts
  - **Admin** - owner of the contract, can adjust cooldown period, add/remove blacklist managers and redistribute balances of fully restricted addresses to others.
  - **Blacklist manager** - like discord/reddit power-mods, can blacklist/unblacklist users fully or softly for a *variety of reasons.
  - **Rewarder** - Transnfers rewards into the StakedUSDe to be vested for the next 8 hours.
  - **Soft_Restricted_Staker** - is prevented from taking part in the staking. Can perform other functions. Conditions for this is mostly for users in places not legally compliant with receiving yield from Ethena. Its important to note that he can still trade staking revwards token on the open market.
  - **Full_Restricted_Staker** - is prevented from interacting with the contract. His balance is redistributed to other accounts. These are accounts deemed suspicious, sanctioned or involded in malicious activities.

## **2. Risks to protocol**
  - **Centralization/Admin Risks** - is probably the biggest risk facing the protocol. There are a lot of trusted roles interacting in the protocol. All their actions can affect the protocol in a variety of ways.
  - **ThirdParty dependencies** - The contracts imports OZcontracts dependencies(version 4.9.0 is being used) . Any vulnerabilities in these imports can affect the contracts and pose a risk to the protocol's security.
  - **Non-standard ERC20 behaviours** - Tokens that do not fit the standard ERC20 profile also pose risks to the protocol.  
  - **Smart Contract vulnerabilities** - Flaws in the smart contracts also pose a risk to the protocol. Funds can be stolen, protocol can be DOSed by attackers.
  - **Other external factors** - Loss of private keys, scams, inside hacks, depegs, extreme volatility, etc can also in their own ways affect the protocols.
    
## **3. Audit approach**
We approached the audit in 3 general steps after which we generated our report.
- Documentation review -  We reviewed the new and old readMes, gitbook and general explanations provided by the devs on discord. While this was going on, we ran the contracts through slither and compared the generated reports to the bot report.
- Manual code review - Here, we manually reviewed the codebase, ran provided tests, tested out various attack vectors. We looked for ways to DOS the system, ways a user can withdraw more than they're entitled to and so on. We also tested out the functions' logic to make sure they work as intended.
- Codebase comparisons - After this was done, we took a look at the protocol's previous audits, compared the differences between the previous and new commits. We tried to find any similar protocol types, compared their implementations and tried to find any general vulnerabilities.

## **4. Conclusions**

  - The codebase looked really solid. Probably because it had been audited before, most of the low hanging fruits had been pointed out and a number of security measures had also been put in place, but the team has done a good job overall. For instance, for weird ERC-20 tokens, an asset allowlist has been implemented.
  - We do advice implementing a balance check before and after transfers, especially for fee on transfer tokens to protect from accounting issues.
  - Zero address should be checked for on constructors and set parameters should also be implemented to prevent unintended behaviours. Also more input validations should be implemented, and carefully.
  - Test coverage should be imporved from about 70%. It helps to catch basic bugs and improves code modularity.
  - Nonces should be safely casted to prevent oveflows.
  - OZ version should also be updated to the latest (5.0) as of time of auditing, to get the latest features and protections.
  - Also, we did notice an account can both be fully and softly restricted. Although this doesn't seem to affect the whole protocol, we recommend revoking one role, if it exists before granting the other.
  - Care should be taken when granting privileged roles so as not to grant an account more than one role (e.g granting a gatekeeper minter/redeemer role, owner granting himself minter role etc) to improve decentralization.
  - Above all, we recommend mitigting discovered issues, implementing constant upgrades and audits to keep the protocol always secure.
   





### Time spent:
30 hours