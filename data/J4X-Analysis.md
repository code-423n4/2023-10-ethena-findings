## Introduction
This report presents the results of the Ethena Labs audit hosted on Code4rena. The primary objectives of the audit were to identify potential vulnerabilities, propose mitigation strategies, and optimize gas usage within the codebase. The audit encompassed four days of rigorous analysis and testing of the Ethena Labs smart contract.

```
| Spec        | Value       |
| ----------- | ----------- |
| Name        | Ethena Labs |
| Duration    | 6 days      |
| nSLOC       | 588         |
```

## Methodology
The audit process was divided into three distinct phases:
### Recon
In the Recon phase, I conducted a preliminary review of the codebase to gain a broad understanding of its structure and functionality. This initial pass allowed me to familiarize myself with the code before diving deeper. Additionally, I examined the provided documentation and gathered supplementary information from online sources and the audit's Discord channel.
### Analysis
The Analysis phase constituted the primary focus of the audit, where I conducted an in-depth examination of the codebase. My approach involved meticulously documenting suspicious code sections, identifying minor bugs, and envisioning potential attack vectors. I explored how various attack scenarios could manifest within the code and delved into the relevant functions. Throughout this phase, I maintained communication with the protocol team to verify findings and clarify any ambiguities.
### Reporting
After compiling initial notes during the Analysis phase, I refined and expanded upon these findings. I also developed test cases to complement the identified issues and integrated them into the final report. Furthermore, I generated Quality Assurance (QA) and Gas reports, alongside this comprehensive analysis.

## System Overview

The Ethena Labs system comprises four key functionalities:

### Minting USDe
The minting functionality allows users to initiate a signed order to Ethena, which, after verifying the order parameters, calls the `mint()` function. The `mint()` function further validates the transaction, transferring funds from the user to a chosen set of custodians and minting the requested USDe to the user.

![EthenaLabs-Minting USDe.drawio.png](https://user-images.githubusercontent.com/58374099/279087304-f4a1f55a-6151-4d08-badb-32e9e1928a13.png)

**Recommendation:** 
The current implementation allows users who are blacklisted from staking to transfer their `USDe` funds to a new address and stake from there. It is suggested to extend the blocklist to cover the USDe token, not just stUSDe, to prevent this evasion.
### Staking `USDe`

The staking functionality relies on the standard `ERC4626` deposit and mint functions. However, it enforces restrictions on addresses with the `FULL_RESTRICTED_STAKER_ROLE` or `SOFT_RESTRICTED_STAKER_ROLE`. Currently, users with the `SOFT_RESTRICTED_STAKER_ROLE` role can circumvent these restrictions by transferring funds to a new address and staking from there, rendering the `SOFT_RESTRICTED_STAKER_ROLE` ineffective.

**Recommendations:**
In the current implementation, any user that is blacklisted from staking can just transfer his `USDe` funds to a new address and stake from that address. Unless the blocklist is also implemented for the `USDe` token, this will not be preventable, effectively making the `SOFT_RESTRICTED_STAKER_ROLE` needless.
### Unstaking `USDe`

The unstaking functionality offers two methods depending on whether an admin has set a cooldown period. Users can either use the standardized `withdraw()` and `redeem()` functions if no cooldown period is in effect. If a cooldown is imposed, users must use the `cooldownAssets()` or `cooldownShares()` functions, which first transfer the funds into the USDe silo. 

![EthenaLabs-Unstake Tokens.drawio.png](https://user-images.githubusercontent.com/58374099/279087329-5fb5e5ad-5ba5-4aed-a188-142580763ea4.png)

This functionally is again access restricted using the role `FULL_RESTRICTED_STAKER_ROLE`. If a user has the role `SOFT_RESTRICTED_STAKER_ROLE` he will still be able to unstake his tokens. A fully restricted user nevertheless should not be able to withdraw any tokens.

**Recommendations:**
The current functionality of unstaking is developed in a way in which users are still able to withdraw tokens even if they are fully restricted. This vulnerabilities need to be fixed to be able to correctly adhere to governmental regulations and sanctions.

Additionally a soft restricted user that wants to keep on accruing rewards can just never unstake his tokens, receiving yield. This might lead to issues with governments (US) that do not allow this, as in that case the protocol would be informed, then would set the role for the user, but the user would just leave his funds inside the vault and keep on accruing yield. 

### Redeeming `USDe`
Redeeming USDe mirrors the minting process, where users send a signed order to Ethena. After verifying the order parameters, the `redeem()` function is called, which further validates the transaction, burns the user's USDe tokens, and transfers the funds to the benefactor.

![EthenaLabs-Redeeming USDe.drawio.png](https://user-images.githubusercontent.com/58374099/279087324-c0fb8494-8443-4016-ac62-7ef6ac0b1aa9.png)
**Recommendations**

The current implementation does not allow a user to withdraw his collateral in form of the same asset as he has deposited it, if the asset gets removed from the supported assets meanwhile. I would recommend to give users the functionality to still be able to withdraw in the same asset they have deposited, but just remove new mints based on it.

## Potential attack vectors
Various assets within the Ethena Labs system may be targeted by attackers. These include:
- user's `USDe` tokens
- `USDe`  tokens stored in the `stUSDe` contract
- `USDe`  tokens in the `USDeSilo`
- `stUSDe` tokens of users
- transferability of blocklisted `stUSDe`
- assets backing up the `USDe` stablecoin (e.g., `WETH`, `stETH`)
- the overall functionality of the protocol

In addition to common ERC20 attack paths, the protocol's structure introduces new attack vectors:
- The ability for an attacker to mint arbitrary `USDe`.
- The potential to overwrite or provide incorrect custodian addresses that receive user funds.
- The risk of an attacker removing all supported assets, leading to a denial-of-service situation for redeeming and minting.
- The possibility of a malicious user bypassing the blocklist.

## Testing

The testsuite, written in Foundry, includes two groups of test cases: minting and staking. It demonstrates a well-structured approach and features dedicated fuzzing tests for specific functions.

**Recommendations**
- Incorporate test cases for duplicate Orders (nonce reuse).
- Expand fuzz testing coverage for various functionalities.
- Utilize invariants to enhance fuzz testing.

## Systemic Risk
The Ethena Labs system exhibits a notable degree of centralization. Trust in the security of the multisig, which serves as the default admin, underlies several security assumptions. Multisigs, while theoretically secure, can be vulnerable when misused. An example is the [Ronin Bridge Hack](https://rekt.news/ronin-rekt/) in which a 9-signature multisig was compromised. The protocol's security heavily relies on the protocol team's proper management of this multisig. Users must trust that the protocol team securely maintains the multisig, as they cannot directly verify it.

Another concern is the compromise of the protocol keys, particularly the MINTER and GATEKEEPER roles. The holder of the MINTER role has the ability to mint arbitrary tokens, albeit protected by the GATEKEEPER role, managed by the same entity. If both roles were to be compromised simultaneously, the risk of unauthorized minting and potential rug-pulling arises.

A notable issue that can undermine user trust in the protocol is the protocol's ability to seize users' `stUSDe` tokens by assigning the `FULL_RESTRICTED_STAKER_ROLE` and then transferring the tokens using `redistributeLockedAmount()`.

Moreover, all collateral is stored outside the minting contract on custody addresses. However, the security of these addresses could not be assessed due to the absence of code for these addresses.

Recommendations:
- Implement trusted external entities (security companies) to manage the GATEKEEPER roles, reducing the risk of both roles being compromised simultaneously.
- Consider removing the `redistributeLockedAmount()` function to assure users they can retain their tokens even if blacklisted. Alternatively, modify the function to allow burning the funds rather than transferring them to an arbitrary address.
- Introduce a time lock for the `setMaxMintPerBlock()` and `setMaxRedeemPerBlock()` functions to allow users time to protect their funds in the event of a compromised multisig.
- Add constants for the mints/redeems per block that the values can not be set above. This effectively reduces the attack surface in case of a compromised multisig
- Conduct regular audits of custodian code and balances by external entities to provide users with a way to verify the security of their collateral.

## Code Quality
The codebase maintains a high level of quality. However, some functions lack proper NatSpec documentation and inline comments, which would aid in comprehending their functionality. Additionally, certain functions return unnecessary values that are never utilized, potentially causing confusion. Opportunities for minor gas optimizations are also present.
## Conclusions

I was able to work on the codebase for 4 days. My work can be comprised into three phases:

- Day 1: Documentation review, initial code inspection, and gaining an understanding of the system's functionalities.
- Days 2-3: In-depth code analysis, vulnerability identification, and bug hunting.
- Day 4: Reporting findings, including bug reports and this comprehensive analysis.

Total time spent: 20 hours.

### Time spent:
20 hours