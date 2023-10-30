### Overview of the protocol

- There are three types of tokens: Collateral tokens, USDe token and stUSDe token.
- Collateral tokens consist of Liquid Staking Derivatives (only stETH for now, more LSD or other tokens in the future)
- USDe token is a ERC20 stablecoin created by Ethena. It does not gain yield. Think of it like USDC.
- stUSDe (Staked USDe) is another ERC20 token that extends ERC4626 functionality. Holding stUSDe will gain yield like rETH. It is a non-rebasing yield bearing token, ie the amount of stUSDe will not increase over time but the value of stUSDe will increase over time. If 1 stUSDe == 1 USDe at the start, then 1 stUSDe may become 1.1 USDe in the future.
- USDe can be staked to get stUSDe. stUSDe can be burned for USDe.

### How the protocol works

- User deposits collateral tokens (eg stETH) for USDe tokens. The conversion rate is done by Ethena themselves. User has to sign before proceeding with the transactions.
- When user deposits collateral tokens, these collateral tokens are transferred to custodians. The custodians will then hedge against the collateral tokens to maintain a delta neutral position. All this is done off chain (or rather, off protocol). The extra yield from rebasing / hedging will be converted to stUSDe yield.
- The user has two choices with USDe. Firstly, they can hold onto USDe and use it like a normal stable coin. Secondly, they can stake their USDe tokens and get stUSDe tokens (staked USDe tokens). USDe tokens does not earn yield. stUSDe tokens earns yield.
- If a user chooses to stake USDe, he should be aware of the cooldown process when redeeming back his USDe. When staking USDe, the user will get back stUSDe. If the user wants to get back his USDe, he will have to burn stUSDe. If there is a cooldown process (the admin decides the duration of the cooldown, from 0 seconds -> 90 days), the user has to wait up to 90 days to get back his USDe token.

### Example process

- User has 10 stETH.
- User deposits 10stETH and gets 20,000 USDe (conversion is determined by Ethena)
- User deposits 20,000 USDe and gets 20,000 shares of stUSDe. (stUSDe uses ERC4626)
- One year later, user wants to get back his USDe. He burns 20,000 stUSDe for 21,000 USDe.
- Since cooldown period is active, user has to wait 90 days to get his 21,000 USDe.
- User wants to get back his stETH. he signs a transaction and burns 21,000 USDe to get back 10.5 stETH. (conversion is determined by Ethena)

### End to end process (code)

Depositing collateral (stETH) and getting USDe

- `mint()` in EthenaMinting.sol is called. Take note that the minter is calling the function, and not the user. The user probably has to approve some functions for the contract to use his assets
- collateral is transferred to the custodian address, according to the ratios provided (ratios must equate to 10,000)
- USDe is minted to the beneficiary
- I assume that there is another call for the user to claim their USDe from the beneficiary or that the beneficiary will transfer the USDe to the user off-code

Redeeming collateral (stETH) from USDe

- `redeem()` in EthenaMinting.sol is called. Once again, the minter is calling the function, and not the user.
- USDe is burned from the benefactor, which I assume is the user.
- Collateral is transferred to the beneficiary, which is Ethena. There should be another function for the user to get his collateral back.

Minting stUSDe

- Call `deposit()` directly from ERC4626.
- USDe will be deposited into StakedUSDe.sol and the appropriate stUSDe shares will be minted for the user

Withdrawing stUSDe

- `cooldownAssets()` is called in StakedUSDeV2. User inputs how much assets (USDe) he wants to get back and burns the appropriate amount of shares (stUSDe). For example, if user wants to get back 1000 USDe, he has to burn 100 stUSDe shares. `ensureCooldownOn` modifier is called to make sure that there is a cooldown set. Cooldowns mapping is updated, and `_withdraw()` in StakedUSDe.sol will be called.
- `_withdraw()` checks whether the assets and shares are non zero value, checks if the caller or receive has `FULL_RESTRICTED_STAKER_ROLE` before calling ERC4626 `_withdraw()`.
- ERC4626 `_withdraw()` will burn the shares from the owner and transfer the asset to the receiver. If cooldown is on, USDe will be transferred to the silo contract. If cooldown is off, USDe will be directly transferred to the receiver.
- `_withdraw()` in StakedUSDe.sol will check whether the total supply of stUSDe is above 1e18 to prevent inflation attack.
- If there is a cooldown, once cooldown is over, `unstake()` in StakedUSDeV2 is called. Function will check whether cooldown is over and transfer `userCooldown.underlyingAmount` of USDe to the user. Cooldown mapping is set to zero.

### Codebase quality analysis and review

| Contract                 | Function                 | Explanation                                         | Comments                                                                     |
| ------------------------ | ------------------------ | --------------------------------------------------- | ---------------------------------------------------------------------------- |
| StakedUSDe               | addToBlacklist           | Add a user to the blacklist                         | Blacklistmanager, Either full or soft restricted                             |
| StakedUSDe               | removeFromBlacklist      | Remove a user from the blacklist                    | Blacklistmanager, Either full or soft restricted                             |
| StakedUSDe               | rescueTokens             | Rescue tokens from contract                         | Admin, rescue any tokens except USDe                                         |
| StakedUSDe               | redistributeLockedAmount | Burns stUSDe from full res and gives to another acc | Admin, huge centralization risk                                              |
| StakedUSDe               | totalAssets              | Returns the amount of USDe tokens vested            | subtracts total balance with getUnvestedAmount()                             |
| StakedUSDe               | getUnvestedAmount        | Returns the unvested USDe tokens                    | blacklistmanager Either full or soft restricted                              |
| StakedUSDe               | \_checkMinShares         | Check whether stUSDe minted or redeem is >1e18      | internal, to prevent inflation attack                                        |
| StakedUSDe               | \_deposit                | deposit USDe and get stUSDe                         | override ERC4626, checks role and call parent \_deposit                      |
| StakedUSDe               | \_withdraw               | withdraw stUSDe and get USDe                        | override ERC4626, checks role and call parent \_withdraw                     |
| StakedUSDe               | \_beforeTokenTransfer    | make sure full res cannot transfer stUSDe           | override stUSDe ERC20, take note of deprecation in OZ 5.0                    |
| StakedUSDeV2             | ensureCooldownOff        | ensure cooldownDuration = 0                         | If no cooldown, can withdraw / redeem stUSDe immediately                     |
| StakedUSDeV2             | ensureCooldownOn         | ensure cooldownDuration != 0                        | If cooldown, redeeming USDe have to wait for a fixed duration                |
| StakedUSDeV2             | unstake                  | Claims staking amount in silo after cooldown        | Is called after cooldownAssets / cooldownShares                              |
| StakedUSDeV2             | cooldownShares           | burns shares of stUSDe, get USDe assets back        | Input shares to burn for assets, cooldowns mapping is updated                |
| StakedUSDeV2             | cooldownAssets           | burns shares of stUSDe, get USDe assets back        | Input assets to get back, calculate shares. cooldowns mapping is updated     |
| StakedUSDeV2             | setCooldownDuration      | Sets the cooldown duration                          | Admin, set the cooldown duration. Cannot be > 90 days                        |
| USDeSilo.sol             | withdraw                 | Withdraws USDe                                      | onlyStakingVault, used in conjunction with StakedUSDeV2.unstake()            |
| USDe.sol                 | mint                     | mints USDe                                          | onlyMinter, which is Ethena                                                  |
| USDe.sol                 | setMinter                | sets minter                                         | onlyOwner, sets the minter of USDe                                           |
| USDe.sol                 | renounceOwnership        | Renounce Ownership of role                          | onlyOwner, override revert                                                   |
| SingleAdminAccessControl | transferAdmin            | Transfers admin role to new admin                   | onlyAdmin role, 2 step process, works with acceptAdmin()                     |
| SingleAdminAccessControl | acceptAdmin              | Accepts new admin role                              | onlyAdmin role, 2 step process, works with transferAdmin()                   |
| EthenaMinting            | mint                     | Mints stablecoins from assets                       | onlyMinter, verify route, order, transferCollateral and mint USDe            |
| EthenaMinting            | redeem                   | Redeem stablecoins for assets                       | onlyReedemer, verify route, order, transfer assets and burns USDe            |
| EthenaMinting            | setMaxMintPerBlock       | Sets the max mintPerBlock limit                     | Admin, only a certain amount of USDe can be minted per block                 |
| EthenaMinting            | setMaxRedeemPerBlock     | Sets the max redeemPerBlock limit                   | Admin, only a certain amount of USDe can be redeemed per block               |
| EthenaMinting            | disableMintRedeem        | Disables the mint and redeem                        | Gatekeeper, Sets max mint and redeem to 0                                    |
| EthenaMinting            | transferToCustody        | Transfers an asset to a custody wallet              | onlyMinter, transfers asset to custodian address                             |
| EthenaMinting            | verifyOrder              | Assert validity of signed order                     | checks signer, beneficiary, collateral_amt, usde_amt, expiry of order        |
| EthenaMinting            | verifyRoute              | Assert validity of route object per type            | OrderType Mint or Redeem. If mint, check total ratio == 10,000 and addresses |
| EthenaMinting            | verifyNonce              | Verify validity of nonce by checking its presence   | Not sure, used in conjunction with deduplicate order                         |
| EthenaMinting            | \_transferToBeneficiary  | Transfer supported asset to beneficiary address     | Called during redeem, after USDe is burned                                   |
| EthenaMinting            | \_transferCollateral     | Verify validity of nonce by checking its presence   | Called during mint, before USDe is minted                                    |

### Centralization Risks

| Contract                 | Role                        | Risk   | Explanation                                                  |
| ------------------------ | --------------------------- | ------ | ------------------------------------------------------------ |
| StakedUSDe               | Admin                       | High   | Able to redistribute any blacklisted user's stUSDE           |
| StakedUSDe               | Blacklist Manager           | High   | Blacklists a user, cannot transfer stUSDe                    |
| StakedUSDe               | FULL_RESTRICTED_STAKER_ROLE | None   | Cannot transfer stUSDe, only can burn                        |
| StakedUSDe               | SOFT_RESTRICTED_STAKER_ROLE | None   | Cannot deposit USDe                                          |
| StakedUSDe               | Rewarder                    | None   | Transfers USDe into the contract as rewards                  |
| StakedUSDeV2             | Admin                       | Medium | Controls the cooldown duration, max 90 days                  |
| USDeSilo                 | Staking Vault               | None   | Withdraws USDe in the silo                                   |
| USDe                     | Owner                       | High   | Controls the setting of minter role, can mint unlimited USDe |
| USDe                     | Minter                      | High   | Can mint unlimited USDe                                      |
| SingleAdminAccessControl | Admin                       | High   | Able to grant and revoke any other roles but not its own     |
| EthenaMinting            | Minter                      | High   | Controls mint                                                |
| EthenaMinting            | Redeemer                    | High   | Controls redeem                                              |
| EthenaMinting            | Admin                       | High   | Controls max mint and redeem limit                           |
| EthenaMinting            | Gatekeeper                  | Medium | Blocks all mint and redeem, admin can override               |

### Mechanism Review

##### StakedUSDeV2.sol

- withdraw() and redeem() is overriden from ERC4626, which is correct. All sUSDe withdrawals / redemption for USDe must wait for the cooldown duration. The USDe will be held in the silo contract.
- Admin can set the cooldown duration. If cooldown is off, user can immediately withdraw stUSDe for USDe. If cooldown is on, user will burn stUSDe for USDe which will be sent to the silo contract for safekeeping until the cooldown is over. User must wait for the cooldown to get their USDe back.

##### StakedUSDe.sol

- \_beforeTokenTransfer() is overriden from ERC20. Take note that this function is deprecated in the newest OZ 5.0 version. (\_update is used instead). If the protocol has a new StakedUSDe version, make sure the that OZ version used is <5.0 or update the hook.
- Role holders cannot renounce role. Prevents restricted role from renouncing their own role. Does not matter if other important roles get compromised, because admin can directly call grantRole() again.
- redistributeLockedAmount(). Burns the stUSDe amount from a fully restricted user and mints the same amount to an address of the admin's choice. Extremely centralized, but probably needed for legal reasons?

### Preliminary Questions

1. Does USDe get burned for stUSDe or is stUSDe an extra token?
2. What is asset() in StakedUSDe.sol? USDe or stUSDe?
3. Quite confused with the getUnvestedAmount() function
4. What can FULL_RESTRICTED user do?
5. What can SOFT_RESTRICTED user do?
6. Who is the Benefactor?
7. Who is the Beneficiary?
8. Where is the approval to transfer collateral / burn USDe since Ethena is calling burn / redeem?

### Preliminary Answers

1. USDe does not get burned for stUSDe. Users deposit USDe into the StakedUSDe.sol contract and get back stUSDe. The deposit function to deposit USDe is found in ERC4626 abstract contract. Yes, stUSDe is another token.
2. USDe is the asset(). stUSDe is the share. stUSDe is another ERC20 token.
3. This is to prevent users from staking USDe and immediately withdrawing to game the reward system. Say there is no cooldown to withdraw. Malicious User has 10,000 USDe to stake. When the rewarder is going to deposit the rewards via `transferInRewards()`, the malicious user can frontrun this deposit by staking 10,000 USDe. Afterwards, he can immediately withdraw his staked USDe to get a portion of the rewards. To prevent this from happening, `getUnvestedAmount()` is used so that malicious user cannot take advantage of the rewards.
4. Only applicable for stUSDe tokens. FULL_RESTRICTED users cannot transfer their stUSDe tokens. This means they cannot partake in the staking/unstaking of USDe because they cannot receive or get USDe tokens (due to the override \_beforeTokenTransfer hook). The only thing they can do is to burn their stUSDe tokens.
5. Only applicable for stUSDe tokens. SOFT_RESTRICTED user is not supposed to be able to deposit their USDe tokens for stUSDe or withdraw their stUSDe tokens for USDe. They can still get stUSDe tokens from the open market.
6. Judging by collateral transfer, benefactor transfers collateral to different custody addresses and the signer is the benefactor, so the benefactor should be the user
7. Transferring assets to the beneficiary when redeeming, after USDe is burned from the benefactor, so the beneficiary should be Ethena.
8. Approval is probably called publicly, from the front end and through the public approve function in ERC20. Approves the EthenaMinting contract to spend collateral / burn USDe.

### Time spent:
30 hours