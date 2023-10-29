# Executive summary of the analysis:

1. **Web2 components that were not part of the scope of the audit should be audited.**
When accepting minting orders, especially in terms of whether the user actually sends collateral (especially in the case of a wormhole), and what address it was sent from. Another thing should be limited confidence in the read price as the provider possibly can fake it.
2. **The system is fragile and susceptible to inappropriate actions, consider adding additional security mechanisms like time-locks, and limits enforcement**
The system is designed in such a way that it works when appropriate assumptions are met and trusted roles behave as expected or predicted. However, everything breaks down when someone decides to act differently. An example of this is a statement from the documentation that says "It will always be pointed to the EthenaMinting.sol contract", and the `setMinter` function allows changes, so it doesn't have to be this way and it is only a promise. The situation is very similar in the case of the mint and redeem per block limits. Even though it is set at the start as promised in the documentation, there are no code enforcement requirements that would prevent a drastic change. There are no mechanisms to slow down and limit the power that would work over time and protect users.
3. **High centralization of power, consider minimizing it**
Trusted roles contain very large permissions. Even though the team declares the use of multi-sig and has blocked renouncing the owner role (thus preventing the project from being left without an owner) the system is still under the complete control of individuals who can make changes immediately

## Actions to take:
1. Conduct an audit of web2 components.
2. Add the limits promised in the documentation to your functions, and if you want to be able to change them, add new functions that will implement these changes with a delay.
3. Extend tests to ensure 100% coverage.
4. Add attack vectors covered during the audit to your tests so that when changes are made to the code, they are still verified.
5. Review the diagram, identified threats, and threat scenarios internally with the team. You know your code best and they can help you identify something that was not detected by the research as we had limited time.

## Diagram (expires in 1 week):
https://we.tl/t-TG5h1p9i4p

## Identified threats:
* Unauthorized mint
* Tokens theft
* DoS
* Loss of control over the contract
* Unlimited mint
* Inconsistency with the documentation
* Lack of collateral
* Privilege escalation
* Price manipulation

# Identified threat scenarios:
* Access control to the functions is defined in a different way than it is in the code.
* Only minting is enabled and redeeming isn't
* The address in route could be outside the _custodianAddresses
* Someone other than DEFAULT_ADMIN_ROLE can add a custodian address.
* Someone other than MINTER can mint USDe.
* MINTER address is not always EthenaMinting contract address. Can be pointed to another address.
* Mint more than 100k USDe in one block.
* Redeem more than 200k USDe in one block.
* Someone other than DEFAULT_ADMIN_ROLE can re-enable minting.
* Someone other than DEFAULT_ADMIN_ROLE can re-enable redeeming.
* setCooldownDuration introduces implications and unfair advantage for someone
* REWARDER_ROLE can transfer rewards only to stakedUSDe, not also StakedUSDeV2
* REWARDER_ROLE can transfer rewards where they want
* BLACKLIST_MANAGER_ROLE can block higher roles
* BLACKLIST_MANAGER_ROLE can lock user funds and transfers by using the address of the contract
* totalAssets at stakedcontracts does not return a valid amount
* transferInRewards is not usable for EOA's
* Lack of access control for setMinter
* Lack of access control for mint
* Possibility of calling the renounceOwnership function
* Instant change of minter by owner
* The outside user does not send collateral while the minter executes the order (Wormhole hack case)
* Unprotected granting role
* Setting low limits for setMaxMintPerBlock and setMaxRedeemPerBlock
* Disabling the minting by the gatekeeper
* Insufficient tokens to be burnt on redeem
* Unprotected functions that should be called only by specific roles (ADMIN, MINTER, REDEEMER, GATEKEEPER).
* Losing the only admin role
* Insufficient validation of redeem attributes: redeeming someone else's USDe tokens.
* Reentrying the redeem function.
* Unprotected accepting the admin
* Unprotected gatekeepers roles
* Gatekeeper removes admin role
* Submitting the same order multiple times.
* Lack of check for collateral transfer
* Insufficient validation of order attributes: wrong check of signature.
* Insufficient validation of order attributes: use of the malicious route.
* Insufficient validation of order attributes: use of incorrect order type.
* Insufficient validation of order attributes: use of fake collateral.
* Insufficient validation of order attributes: too high USDe amount.
* Insufficient validation of nonce.
* Restricted role blocks staking contract.
* Admin sets high cooldown duration
* StakedUSDe admin redistributes other tokens than stUSDe
* StakedUSDe admin instead of rescuing the tokens, steals the tokens from the contract.
* Withdrawing tokens belonging to others.
* Bypassing assumption about SOFT_RESTRICTED_STAKER_ROLE: "They cannot deposit USDe to get stUSDe or withdraw stUSDe for USDe. However they can participate in earning yield by buying and selling stUSDe on the open market."
* Bypassing assumption about FULL_RESTRCITED_STAKER_ROLE: "cannot move their funds, and only Ethena can unfreeze the address"
* Bypassing the cooldown period on assets and shares.
* Lack of slippage protection
* Manipulation of the amounts returned by the ERC4626 preview* functions.
* Rounding issues in ERC4626 converting and previewing functions
* Early depositors can manipulate the exchange rate
* Non-compliance with ERC4626
* Lack of check for maximum amounts in ERC4626


### Time spent:
48 hours