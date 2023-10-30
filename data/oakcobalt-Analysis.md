### Description:

Ethena is an Ethereum-based protocol, aiming to create a crypto-native solution for money independent of traditional banking systems. It introduces a stable synthetic dollar called USDe, which is both censorship-resistant and scalable. 

USDe achieves stability through delta-hedging Ethereum collateral and transparent on-chain backing. The 'Internet Bond' complements this by offering a dollar-denominated savings instrument, utilizing yield from staked Ethereum and revenue from perpetual and futures markets. 

**Existing Standards:**
- The protocol adheres to conventional Solidity and Ethereum practices, primarily utilizing the ERC20 standards, along with the openzepplin's access control patterns for assigning various roles. Although, access control is customized and extended to enforce a single admin control over all other roles.

- Additionally, it incorporates features like blacklisting, maximum per-block minting/redeeming, token rescue mechanism.

- The protocol also primarily uses ERC4626 standards for staking and unstaking accounting, with a customized cool-down withdraw mechanism design.

It's worth noting that the protocol relies on off-chain accounting and pricing conversion mechanisms, and also involves counter party's during funds transfer.

## 1- Approach:
- Scope: I initiated the process by evaluating the extent of the code, which then directed our method for scrutinizing and dissecting the project.

- Roles: Then I focus on role setup for the minting, redeeming, staking and unstacking process. 

Considering the extensive authority held by these roles within the system, users must have complete confidence that the entities responsible for supervising these roles will consistently act in a correct and beneficial manner for the system and its users.

- Top-down approach: 
```
1.EthenaMinting.sol, SingleAdminAccessControl.sol
2.StakedUSDe.sol, SingleAdminAccessControl.sol
3.StakedUSDeV2.sol, SingleAdminAccessControl.sol
4.USDe.sol
5.USDeSilo.sol
```
SingleAdminAccessControl and EthenaMinting are reviewed together due to the fact that the role setup are integrated in SingleAdminAccessControl.


## 2-  Codebase analysis:
In my assessment, the quality of the codebase is currently satisfactory, with measures in position to manage diverse roles and ensure adherence to established standards. However, there remains room for enhancement in the following aspects.

| Codebase Quality Categories  | Comments |
| --- | --- |
| **Mechanism code**  | Current mechanism code on nonce storage is lacking sufficient checking to ensure nonce is strictly incrementing by 1. |
| **Code Comments**  | Insufficient overall code comments, particularly for detailed bit operations. Additional comments and NatSpec documentation would have aided the auditor and reduced sponsor inquiries, saving time.  |
| **Documentation** | Overall protocol documentation is robust in explaining the protocol's innovation in token economics. However, there is insufficient documentation on specific off-chain mechanism and accounting in the process, with respect to slippage prevention, fee accounting, order submitting, etc.|



## 3-  Centralization Risks:

Here's an analysis of potential centralization risks in the provided contracts:

### EthernaMinting.sol
- **Centralization Risks:**
    - Off-chain user background check process.
    - Single admin-role can remove and add all other roles including the minter role and redeemer role. 
      This means if admin-role key is leaked, all other access controled functions will be eroded 
      instantly.
    - Admin removal or adding OES operator address (custodian address) before user redeems. Scenario of 
      admin key loss.
    - Admin removal of supported Asset. Scenario of admin key loss.

### StakedUSDe.sol
- **Centralization Risks:**
    - Address black-listing.
    - Single admin-role can remove and add all other roles and act as other role admins. 
      This means if admin-role key is leaked, all other access controled functions will be eroded 
      instantly.
    - `rescueTokens()`, centralized admin can rescue tokens to a designated address of his choosing

### SingleAdminAccessControl.sol
- **Centralization Risks:**
    - Single admin instead of multiple role admin is implemented: This increases the centralization risks. For example, If the admin key becomes compromised, gatekeeper role becomes non-existing. Since admin can change gateKeeper role and directly perform sensitive minting or redeeming functions, such as disableMintRedeem.

## 4- Systemic Risks:

### Price slippage and lack of on-chain checks:

In EthernaMinting.sol, collateral price slippage can happen at the time of mint or redeem order settlement on-chain. This means the actual amount of collateral transferred might not reflect the values of USDe minted.

In addition, the minter and redeemer role become the on-chain single source of truth for fair price conversion between USDe and collateral tokens.

### Reliability of off-chain accounting:

Reliability of off-chain accounting of user mint amount, collateral amount, ratios of custodian distribution or actual custodian transfer, custodian address, etc.


### Counterparty Risks:

The reliability of transparent and trustworthy custodian addresses increases the counterparty risks. 

It's recommended that OES providers offer the most transparent and verifiable accounting of funds transfer and storage to counter such risks.

### Price manipulation:

Ethena protocol allows peg arbitrage on defi markets. But since current defi pool that includes USDe has low liquidity and still in the test stage (see example below). It can be assumed that at the beginning stage defi pool price of USDe will be prone to price manipulation, which might cause USDe's value extremely volatile to user arbitrage. 

Currently, there is a test Crv/USDe pool on Curve which has relatively low reserve.

![Curve Test Crv/USDe Pool Reserves](https://github.com/code-423n4/2023-10-ethena/assets/138168196/ced1549b-aa13-4ec1-ac5c-877085ff7fce)


## 5- Mechanism Review:

### Slippage windows:

**EthenaMinting.sol:**

There are multiple windows of collateral price slippage from the beginning of the user query on dApp UI, to the user signing a mint or redeem order signature based on quote amounts, to Ethena minter or redeemer role pick up user's signature and generating order and submitting an order on behalf of users, and to the submitted transaction finally settled on-chain. 

So far, the protocol's approach to dealing with slippage is only preemptive, including a slippage into the user's quote on the dApp UI before the user signs a signature to approve the quoted amount. This is insufficient to prevent collateral slippage after user signing signatures, and also insufficient to prevent price slipping further from an acceptable range. In `EthenaMinting.sol`, It's recommended to add on-chain verification confirming the settled amount is within acceptable range to prevent USDe being under-collateralized at the time of order settlement.

### Nonce on-chain verification:

**EthenaMinting.sol:**

In `EthenaMinting.sol`, when mint or redeem order is submitted, it's minter or redeemer role's job to submit a nonce of user's order. Current on-chain check only checks whether the submitted nonce is clashing with an existing nonce in storage. 

However, this is insufficient in ensuring nonce is submitted strictly in an incrementing manner and incrementing exactly `1` from the previous one. In the case of a minter or redeemer role mistakes or minter or redeemer key leakage, random nonce number will pass, which will create clashing of user nonce's in the future, potentially causing a user unable to mint or redeem.


### Cool-down mechanism:

**StakedUSDeV2.sol:**

Owner’s asset might be forced to be locked longer than set cool-down duration.

Current implementation of cool-down will renew the cooldown end timestamp whenever a user requested more assets to be withdrawn, regardless of whether a user has current on-going cooldown or not. This will cause a user's existing locked asset's cool-down to be extended to an additional cool-down cycle. It's recommended to refine the cool-down accounting logic to use a mapping to correspond each `coolDownEnd` timestamp to their respective asset value to avoid extended and unfair lock of user assets.

### Conclusion:

In summary, while Ethena demonstrates promise as a crypto-native financial protocol, it requires further enhancements in addressing several issues related to systemic risks, centralization risks and mechanism review as detailed above.

### Time spent:

30 hours




### Time spent:
30 hours