# Ethena Labs Audit Contest Analysis Report | 24 Oct 2023 - 30 Oct 2023

---

| #   | Topic                                                                                                            |
| --- | ---------------------------------------------------------------------------------------------------------------- |
| 1   | Architecture Overview (Protocol Explanation, Codebase Explanation, Examples Scenarios of Intended Protocol Flow) |
| 2   | Codebase Quality Analysis                                                                                        |
| 3   | Centralization Risks                                                                                             |
| 4   | Systemic Risks                                                                                                   |
| 5   | Attack Vectors Discussed During the Audit                                                                        |
| 6   | Example of Report that turned out to be invalid after I wrote a `really good Explanation and PoC` (The report explain the `unstaking` really well, so you can learn for it) |

*Pdf link -> https://www.docdroid.net/pwoydhp/analysis-report-pdf*

---

# 1. Architecture Overview (Protocol Explanation, Codebase Explanation, Examples Scenarios of Intended Protocol Flow)

## 1.1. Protocol Explanation
- ###### Overview:
  Ethena is developing a DeFi ecosystem with a primary goal of offering a permissionless stablecoin, USDe, that allows users to earn yield within the system. This process is a contrast to traditional stablecoins like USDC, where the central authority (e.g., Circle) benefits from the yield. In Ethena's ecosystem, users can stake their USDe to earn stUSDe, which appreciates over time as the protocol generates yield.

- ###### Smart Contract Infrastructure:
  - `USDe.sol`: The contract for the USDe stablecoin, limited in functionality with controls for minting privileges.
  - `EthenaMinting.sol`: This contract mints and redeems USDe in a single, atomic, trustless transaction. Central to user interactions, handling minting and redemption of USDe. It employs EIP712 signatures for transactions, routing collateral through predefined safe channels, and includes security measures against potential compromises by limiting minting and providing emergency roles (GATEKEEPERS) to intervene in suspicious activities.
  - `StakedUSDeV2.sol`: Allows USDe holders to stake their tokens for stUSDe, earning yield from the protocol's profits. It incorporates mechanisms to prevent exploitation of yield payouts and has a cooldown period for unstaking. For legal compliance, it can restrict certain users (based on jurisdiction or law enforcement directives) from staking or freeze their assets, with the provision of fund recovery in extreme cases.

- ###### Roles in Ethena Ecosystem:
  - `USDe` minter - can mint any amount of USDe tokens to any address. Expected to be the EthenaMinting contract.
  - `USDe` owner - can set token minter and transfer ownership to another address
  - `USDe` token holder - can not just transfer tokens but burn them and sign permits for others to spend their balance
  - `StakedUSDe` admin - can rescue tokens from the contract and also to redistribute a fully restricted staker's stUSDe balance, as well as give roles to other addresses (for example the FULL_RESTRICTED_STAKER_ROLE role)
  - `StakedUSDeV2` admin - has all power of "StakedUSDe admin" and can also call the setCooldownDuration method
  - `REWARDER_ROLE` - can transfer rewards into the StakedUSDe contract that will be vested over the next 8 hours
  - `BLACKLIST_MANAGER_ROLE` - can do/undo full or soft restriction on a holder of stUSDe
  - `SOFT_RESTRICTED_STAKER_ROLE` - address with this role can't stake his USDe tokens or get stUSDe tokens minted to him
  - `FULL_RESTRICTED_STAKER_ROLE` - address with this role can't burn his stUSDe tokens to unstake his USDe tokens, neither to transfer stUSDe tokens. His balance can be manipulated by the admin of StakedUSDe
  - `MINTER_ROLE` - can actually mint USDe tokens and also transfer EthenaMinting's token or ETH balance to a custodian address
  - `REDEEMER_ROLE` - can redeem collateral assets for burning USDe
  - `EthenaMinting admin` - can set the maxMint/maxRedeem amounts per block and add or remove supported collateral assets and custodian addresses, grant/revoke roles
  - `GATEKEEPER_ROLE` - can disable minting/redeeming of USDe and remove MINTER_ROLE and REDEEMER_ROLE roles from authorized accounts

- ###### What custodian is? (Chat GPT says)
  - In the realm of cryptocurrencies and blockchain technology, a "custodian" refers to an entity (company or smart contract) that holds and safeguards an individual's or institution's digital assets. The role of a custodian is critical in scenarios where security and proper asset management are paramount. This concept isn't exclusive to digital assets; traditional financial institutions have custodians as well.
  - Within a blockchain environment, an address usually refers to a specific destination where cryptocurrencies are sent. Think of it like an account number in the traditional banking sense.
    In the context of smart contracts or decentralized applications, `custodianAddresses` likely refer to the collection of blockchain addresses that are authorized to hold assets on behalf of others.
    These addresses are controlled by the custodian, which could be a smart contract or a third-party service that maintains the security of the private keys associated with these addresses.

##

## 1.2. Codebase Explanation & Examples Scenarios of Intended Protocol Flow

###

#### All possible Actions and Flows in Ethena Protocol:

##### 1. Minting USDe:
Users provide stETH as collateral to mint USDe. The system gives an RFQ (Request for Quote) detailing how much USDe they can create. Upon agreement, the user signs a transaction, and Ethena mints USDe against the stETH, which is then employed in various yield-generating activities, primarily shorting ETH perpetual futures to maintain a delta-neutral position.

###### Additional Explanations Related to the `Minting`:
- So, `USDe` will be equivalent to using DAI, USDC, USDT (`USDe` contract is just the ERC20 token for the stablecoin and this token will be the token used in `StakedUSDeV2.sol` staking contract) where it `doesn't have any yield` and `only the holders of stUSDe` will earn the generated yield.
- The `EthenaMinting.sol` contract is the place where `minting` is done.

###

##### 2. Yield Generating:
Ethena generates yield by taking advantage of the differences in staking returns (3-4% for stETH) and shorting ETH perpetuals (6-8%). Profits are funneled into an insurance fund and later distributed to stakers, enhancing the value of stUSDe relative to USDe.

###

##### 3. Maintaining Delta Neutrality: 
Ethena employs a strategy involving stETH and short positions in ETH perpetuals, ensuring the value stability of users' holdings against market volatility.

###### *Example № 1:*
  1. Initial Setup:
    - The user initiates the process by sending 10 stETH to Ethena. The stETH is a token representing staked Ethereum in the Ethereum 2.0 network, allowing holders to earn rewards while keeping liquidity. At the time of the transaction, Ethereum's price is $2,000; thus, 10 stETH equals $20,000.
    - Ethena uses these 10 stETH to mint 20,000 USDe stablecoins for the user, reflecting the stETH's dollar value.
    - Simultaneously, Ethena opens a short position on Ethereum perpetual futures (ETH perps) equivalent to 10 ETH. Given the current Ethereum price of $2,000, this also represents a $20,000 position. This short position means that Ethena is betting on the Ethereum price going down.
  2. Market Movement and Its Impact:
    - Now, the market faces significant volatility, and the price of Ethereum drops by 90%. As a result, the value of the user's 10 stETH decreases to $2,000 (reflecting the 90% drop from the original $20,000 value).
    - However, because Ethena shorted 10 ETH worth of perps, the decrease in Ethereum's price is advantageous for this position. The short ETH perps position now has an unrealized profit of $18,000. This profit occurs because Ethena 'borrowed' the ETH at a higher price to open the position and can now 'buy' it back at a much lower price, pocketing the difference.
  3. Redemption Process:
    - The user decides to redeem their 20,000 USDe. For Ethena to honor this request, they need to provide the user with the equivalent value in stETH that the USDe represents.
    - Ethena closes the short position on the ETH perps, which means they 'buy' back the ETH at the current market price, realizing the $18,000 profit due to the price difference from when they opened the short position.
    - With the $18,000, Ethena purchases 90 stETH at the current market price ($200 per stETH, as the price has dropped by 90%).
    - Ethena then returns the original 10 stETH along with the 90 stETH purchased from the profits of the short position. So, the user receives 100 stETH, which, at the current market price, is worth $20,000.

###### *Example № 2:*
  1. Initial Condition:
    - The price of ETH is $2,000.
    - The user sends in 10 stETH (equivalent to 10 ETH) to Ethena to mint 20,000 USDe (since 10 ETH at $2,000 per ETH is worth $20,000).
    - Ethena takes these 10 stETH and opens a short position on 10 ETH's worth of perpetual futures (perps) to hedge against the price movement of ETH.
  2. Market Movement:
    - The market goes up by 50%. Therefore, the price of ETH (and stETH, as it’s pegged to the ETH value) increases to $3,000.
  3. Position Analysis:
    - The user's 10 stETH is now worth $30,000 due to the market increase.
    - However, Ethena's short position is now at a notional loss because it was betting on the price of ETH going down, not up. The loss on the short position is $10,000 (the increase in value per ETH is $1,000, and Ethena shorted 10 ETH).
  4. Redemption Process:
    - If the user decides to redeem their USDe, they will present their 20,000 USDe.
    - Considering the market movement, the short position's loss needs to be covered. Ethena has to close the short position and realize the loss of $10,000.
    - After covering the $10,000 loss, there's $20,000 worth of stETH left (approximately 6.67 stETH at the new rate of $3,000 per stETH) to return to the user.
  5. End Result:
    - The user initially had assets worth $20,000 (10 stETH). If they hadn't engaged with Ethena and simply held onto their 10 stETH, their assets would now be worth $30,000 due to the positive market movement.
    - By choosing to use Ethena's hedging mechanism, they've forfeited potential gains to safeguard against potential losses. They receive approximately 6.67 stETH (worth $20,000) back after the redemption process, missing out on the additional $10,000 value increase.
    - Essentially, the user's assets remained stable in USDe value, but they did not benefit from ETH's bullish market. Their asset value didn’t decrease, but they also lost potential profit

###

---

##

# 2. Codebase Quality Analysis

1. **`USDe.sol`**:

    ![usde-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/usde-contract.png?raw=true)

   - Code Organization:
     - The contract is well-organized and follows the best practices for code layout and structure.
     - It uses OpenZeppelin contracts, which are widely recognized and audited.
     - The constructor initializes the contract and sets the owner/admin.
     - It provides functions to set the minter and mint USDe tokens.

   - Modifiers:
     - The contract uses the `Ownable2Step` modifier, which enforces two-step ownership transfer.
     - The `onlyRole` modifier is used to restrict certain functions to specific roles, enhancing security.

   - Minting:
     - The contract allows only the minter to mint new tokens, which is a good security measure.
     - It checks if the provided minter is valid.

   - Gas Efficiency:
     - The contract uses the SafeERC20 library for safe token transfers, ensuring protection against reentrancy attacks.
     - Gas-efficient practices are followed throughout the contract.

   - Overall, `USDe.sol` appears to be well-structured and follows best practices for security and code organization.

####

2. **`EthenaMinting.sol`**:

    ![ethena-minting-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/ethena-minting-contract.png?raw=true)

   - Code Organization:
     - It imports external libraries and contracts, including OpenZeppelin contracts.
     - The constructor initializes contract parameters and roles.
     - It contains functions for minting and redeeming USDe tokens.

   - Security Measures:
     - The contract enforces access control using role-based access control (RBAC) with different roles for minters, redeemers, and gatekeepers.
     - Gas limits for minting and redeeming are enforced to prevent abuse.

   - Signature Verification:
     - It verifies the signature of orders, ensuring that the orders are signed by authorized parties.
     - It uses EIP-712 for signature verification.

   - Gas Efficiency:
     - Gas-efficient practices are followed, and SafeERC20 is used for token transfers.

   - Domain Separator:
     - The contract computes the domain separator for EIP-712, enhancing security.

   - Deduplication:
     - The contract implements deduplication of taker orders to prevent replay attacks.

   - Supported Assets:
     - The contract maintains a list of supported assets.

   - Custodian Addresses:
     - It keeps track of custodian addresses and allows transfers to custodian wallets.

   - Overall, `EthenaMinting.sol` is well-structured, secure, and follows best practices for code organization and security.

####

3. **`StakedUSDe.sol:`**

  The contract inherits **Implementation of the ERC4626 "Tokenized Vault Standard" from OZ**

    ![erc4626-oz-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/erc4626-oz-contract.png?raw=true)

    ####

    ![staked-usde-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/staked-usde-contract.png?raw=true)

- Code Organization:
  - It imports external libraries and contracts, including OpenZeppelin contracts.
  - The constructor initializes contract parameters and roles.
  - It contains functions for transferring rewards, managing the blacklist, rescuing tokens, and redistributing locked amounts.
  - Public functions are provided to query the total assets and unvested amounts.
  - The `decimals` function is overridden to return the number of decimal places.
  - Custom modifiers ensure input validation and role-based access control.
  - Hooks and functions are in place to enforce restrictions on specific roles and token transfers.

####

4.  **`StakedUSDeV2.sol:`**

  ![staked-usde-v2-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/staked-usde-v2-contract.png?raw=true)

- Code Organization:
  - The contract extends `StakedUSDe` and inherits its code organization structure.
  - It introduces additional state variables, including `cooldowns`, `silo`, `MAX_COOLDOWN_DURATION`, and `cooldownDuration`.
  - Custom modifiers, `ensureCooldownOff` and `ensureCooldownOn`, are defined to control the execution of functions based on cooldown status.
  - The constructor initializes the contract and sets the cooldown duration.
  - External functions are provided for withdrawing, redeeming, and performing cooldown actions for assets and shares.
  - The contract enforces different behavior based on the cooldown duration, adhering to or breaking the ERC4626 standard.
  - The `setCooldownDuration` function allows the admin to update the cooldown duration.

####

5. **`USDeSilo.sol:`**

  ![usde-silo-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/usde-silo-contract.png?raw=true)

  - The contract is primary goal of `USDeSilo.sol` is to to `hold the funds` for the `cooldown period` whn user initiate `unstaking`. 

  **General Observations:**
   - Role-based access control is implemented for various functions, enhancing security.
   - Gas-efficient practices, such as using SafeERC20, are followed throughout the code.
   - The codebase of protocol includes comprehensive comments and region divisions for clarity.
   - Note that, The use of EIP-712 for signature verification adds an extra layer of security.

6. **`SingleAdminAccessControl.sol:`**
    
  ![single-admin-access-control-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/single-admin-access-control-contract.png?raw=true)

  - EthenaMinting uses SingleAdminAccessControl rather than the standard AccessControl.
###

In summary, the Ethena Protocol's codebase appears to be of high quality, with a strong focus on security and code organization.

###

---

##

# 3. Centralization Risks
Actually, the `Ethena` Protocol contains many roles, each with quite a few abilities. This is necessary for the Protocol's logic and purpose.

The protocol assigns important roles like "MINTER," "REWARDER," and "ADMIN" to specific entities, potentially exposing the system to undue influence or risks if these roles are compromised.

So, these roles introduce several **centralization risks**. The most significant one is the scenario in which the **`MINTER` role becomes compromised**. An attacker/minter could then `mint a billion USDe` without collateral and dump them into pools, causing a black swan event that our insurance fund cannot cover.

However, `Ethena` **addresses this problem** by enforcing on-chain **mint and redeem limitations of 100k USDe** per block."

From the documentation:
> Our solution is to enforce an on chain mint and redeem limitation of 100k USDe per block. In addition, we have `GATEKEEPER` roles with the ability to disable mint/redeems and remove `MINTERS`,`REDEEMERS`. `GATEKEEPERS` acts as a safety layer in case of compromised `MINTER`/`REDEEMER`. They will be run in seperate AWS accounts not tied to our organisation, constantly checking each transaction on chain and disable mint/redeems on detecting transactions at prices not in line with the market. In case compromised `MINTERS` or `REDEEMERS` after this security implementation, a hacker can at most mint 100k USDe for no collateral, and redeem all the collateral within the contract (we will hold ~$200k max), for a max loss of $300k in a single block, before `GATEKEEPER` disable mint and redeem. The $300k loss will not materialy affect our operations.

###

In summary, `Ethena` actually introduces several centralization risks due to the presence of many different roles in the Protocol. **However, at the same time, the team has done its best to enforce measures that reduce the largest potential attack scenario to a maximum loss of $300k, which will not materially affect the `Ethena` operations/ecosystem.**
###

---

##

# 4. Systemic Risks
Here’s an analysis of potential systemic

1. Smart Contract Vulnerability Risk:
Smart contracts can contain `vulnerabilities` that can be exploited by attackers. If a smart contract has `critical security flaws`, such as logic problems, this could lead to asset loss or `system manipulation`. I strongly recommend that, once the protocol is audited, necessary actions be taken to `mitigate any issues` identified by `C4 Wardens`

###

2. Third-Party Dependency Risk:
Contracts rely on external data sources, such as `@openzeppelin/contracts-upgradeable`, and there is a risk that if any `issues` are found with these `dependencies` in your contracts, the `Ethena` protocol could also be affected.

I observed that `old versions` of `OpenZeppelin` are used in the project, and these should be updated to the latest version:

```json
  "name": "@openzeppelin/contracts-upgradeable",
  "description": "Secure Smart Contract library for Solidity",
  "version": "4.9.2",
```

The latest version is `4.9.3` (as of July 28, 2023), while the project uses version `4.9.2`.

###

---

##

# 5. Attack Vectors Discussed During the Audit
1. Issues related to Roles (Centralization Risks). Problems with roles changing.
2. Breaking of Main Protocol Invariants
    - EthenaMinting.sol - User's signed EIP712 order, if executed, must always execute as signed. ie for mint orders, USDe is minted to user and collateral asset is removed from user based on the signed values.
    - Max mint per block should never be exceeded.
    - USDe.sol - Only the defined minter address can have the ability to mint USDe.
3. DoS for important protocol functions/flows such as `EtehnaMinting.sol#mint()` (Minting Flow), `EtehnaMinting.sol#redeem()` (Redemption Flow), `StakedUSDe#deposit()/StakedUSDe#_deposit()` (Depositing Flow), `StakedUSDeV2#unstake()` (Unstaking Flow). 
4. Token transfer fails.
5. Minting more than 100k USDe per block.
6. Users cannot unstake/withdraw/redeem.
7. Can users withdraw more than they actually are supposed to?
8. Minting without providing collateral amount?

###

---

##

# 6. Example of report that turned out to be invalid after I wrote a `really good Explanation and PoC` (The report explain the `unstaking` really well, so you can learn for it)

## REPORT:

---

## Title: Users will not be able to `withdraw/redeem` their assets/shares from `StackedUSDeV2` contract. (Inefficient logic)

## Explanation
[`StakedUSDeV2.sol`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol) is where holders of `USDe` stablecoin can stake their stablecoin, get `stUSDe` in return and `earn yield`. The Etehna Protocol yield is paid out by having a `REWARDER` role of the staking contract send yield in `USDe`, increasing the `stUSDe` value with respect to `USDe`. This contract is a modification of the `ERC4626 standard`, with a change to vest in rewards linearly over 8 hours

When the `unstake` process is initiated, from the user's perspective, `stUSDe` is burnt immediately, and they will be able to invoke the [`withdraw()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L52-L60) function after `cooldown` is up to get their `USDe` in return. Behind the scenes, on `burning of stUSDe`, `USDe` is sent to a seperate [USDeSilo](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol) contract to hold the funds for the `cooldown period`. And on `withdrawal`, the staking contract moves user funds from `USDeSilo` contract out to the `user's address`.

##### 

1. Functions respond for `redemption`, `starting of cooldown` and `transferring of converted underlying asset to USSDeSilo contract` are [cooldownAssets() and cooldownShares()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L95-L122)

```jsx
  function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {
    if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();

    uint256 shares = previewWithdraw(assets);

    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    cooldowns[owner].underlyingAmount += assets;

    _withdraw(_msgSender(), address(silo), owner, assets, shares);

    return shares;
  }

  /// @notice redeem shares into assets and starts a cooldown to claim the converted underlying asset
  /// @param shares shares to redeem
  /// @param owner address to redeem and start cooldown, owner must allowed caller to perform this action
  function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
    if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();

    uint256 assets = previewRedeem(shares);

    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    cooldowns[owner].underlyingAmount += assets;

    _withdraw(_msgSender(), address(silo), owner, assets, shares);

    return assets;
  }
```

2. After `cooldown` is finished, user can call [`unstake()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L78-L90) to `claim the staking amount after`.
```jsx
  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }
```
*The function check if the cooldown is finished by `block.timestamp >= userCooldown.cooldownEnd` check.*
*REMEMBER: The assets are transferred from `USDeSilo` contract -> https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L28-L30*

#### 

Described logic above work only if the cooldown is set to number greater than zero. (aka the cooldown is active)

The `Ethena` can decide to `disable the cooldown period`, so the users to be able `unstake without cooldown period`. If this is done, the user will be able to call directly [`withdraw()/redeem()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L52-L73) to unstake:
```jsx
  function withdraw(uint256 assets, address receiver, address owner)
    public
    virtual
    override
    ensureCooldownOff
    returns (uint256)
  {
    return super.withdraw(assets, receiver, owner);
  }

  /**
   * @dev See {IERC4626-redeem}.
   */
  function redeem(uint256 shares, address receiver, address owner)
    public
    virtual
    override
    ensureCooldownOff
    returns (uint256)
  {
    return super.redeem(shares, receiver, owner);
  }
```
*REMEMBER: The assets are transferred from `StakedUSDeV2` contract -> https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L28-L30*


## Proof of Concept
So, the problem in Ethena protocol logic aries exactly when `Ethena` decide to `disable the cooldown period`.
Lets illustrate the following example:

### Scenario

- Cooldown Period: 50 days

1. Alice deposit her 10000 `USDe` tokens in `StakedUSDeV2` contract.
2. Bob also deposit his 20000 `USDe` tokens in `StakedUSDeV2` contract.
3. After some time Bob decide to unstake and call `cooldownAssets()` to start of cooldown period and converted underlying asset are transferred to `USSDeSilo` contract and he redeem his 20000 `USDe` tokens.
4. After 30 days `Ethena` disable cooldown period.
5. Now users are supposed to unstake directly from `withdraw()/redeem()`.
6. But when the Bob tries to call `withdraw()/redeem()` he will not be able to get his converted underlying asset, because they are in `USDeSilo` contract.
7. Alice call `withdraw()/redeem()` and get his converted underlying asset.

## Tools Used

- Manual Inspection

---

After all I saw that users that already `cooldownAssets()/cooldownShares()` can again call `unstake()` function to get their converted underlying asset.

# Ethena Labs Audit Contest Analysis Report | 24 Oct 2023 - 30 Oct 2023

---

| #   | Topic                                                                                                            |
| --- | ---------------------------------------------------------------------------------------------------------------- |
| 1   | Architecture Overview (Protocol Explanation, Codebase Explanation, Examples Scenarios of Intended Protocol Flow) |
| 2   | Codebase Quality Analysis                                                                                        |
| 3   | Centralization Risks                                                                                             |
| 4   | Systemic Risks                                                                                                   |
| 5   | Attack Vectors Discussed During the Audit                                                                        |
| 6   | Example of Report that turned out to be invalid after I wrote a `really good Explanation and PoC` (The report explain the `unstaking` really well, so you can learn for it) |

*Pdf link -> https://www.docdroid.net/pwoydhp/analysis-report-pdf*

---

# 1. Architecture Overview (Protocol Explanation, Codebase Explanation, Examples Scenarios of Intended Protocol Flow)

## 1.1. Protocol Explanation
- ###### Overview:
  Ethena is developing a DeFi ecosystem with a primary goal of offering a permissionless stablecoin, USDe, that allows users to earn yield within the system. This process is a contrast to traditional stablecoins like USDC, where the central authority (e.g., Circle) benefits from the yield. In Ethena's ecosystem, users can stake their USDe to earn stUSDe, which appreciates over time as the protocol generates yield.

- ###### Smart Contract Infrastructure:
  - `USDe.sol`: The contract for the USDe stablecoin, limited in functionality with controls for minting privileges.
  - `EthenaMinting.sol`: This contract mints and redeems USDe in a single, atomic, trustless transaction. Central to user interactions, handling minting and redemption of USDe. It employs EIP712 signatures for transactions, routing collateral through predefined safe channels, and includes security measures against potential compromises by limiting minting and providing emergency roles (GATEKEEPERS) to intervene in suspicious activities.
  - `StakedUSDeV2.sol`: Allows USDe holders to stake their tokens for stUSDe, earning yield from the protocol's profits. It incorporates mechanisms to prevent exploitation of yield payouts and has a cooldown period for unstaking. For legal compliance, it can restrict certain users (based on jurisdiction or law enforcement directives) from staking or freeze their assets, with the provision of fund recovery in extreme cases.

- ###### Roles in Ethena Ecosystem:
  - `USDe` minter - can mint any amount of USDe tokens to any address. Expected to be the EthenaMinting contract.
  - `USDe` owner - can set token minter and transfer ownership to another address
  - `USDe` token holder - can not just transfer tokens but burn them and sign permits for others to spend their balance
  - `StakedUSDe` admin - can rescue tokens from the contract and also to redistribute a fully restricted staker's stUSDe balance, as well as give roles to other addresses (for example the FULL_RESTRICTED_STAKER_ROLE role)
  - `StakedUSDeV2` admin - has all power of "StakedUSDe admin" and can also call the setCooldownDuration method
  - `REWARDER_ROLE` - can transfer rewards into the StakedUSDe contract that will be vested over the next 8 hours
  - `BLACKLIST_MANAGER_ROLE` - can do/undo full or soft restriction on a holder of stUSDe
  - `SOFT_RESTRICTED_STAKER_ROLE` - address with this role can't stake his USDe tokens or get stUSDe tokens minted to him
  - `FULL_RESTRICTED_STAKER_ROLE` - address with this role can't burn his stUSDe tokens to unstake his USDe tokens, neither to transfer stUSDe tokens. His balance can be manipulated by the admin of StakedUSDe
  - `MINTER_ROLE` - can actually mint USDe tokens and also transfer EthenaMinting's token or ETH balance to a custodian address
  - `REDEEMER_ROLE` - can redeem collateral assets for burning USDe
  - `EthenaMinting admin` - can set the maxMint/maxRedeem amounts per block and add or remove supported collateral assets and custodian addresses, grant/revoke roles
  - `GATEKEEPER_ROLE` - can disable minting/redeeming of USDe and remove MINTER_ROLE and REDEEMER_ROLE roles from authorized accounts

- ###### What custodian is? (Chat GPT says)
  - In the realm of cryptocurrencies and blockchain technology, a "custodian" refers to an entity (company or smart contract) that holds and safeguards an individual's or institution's digital assets. The role of a custodian is critical in scenarios where security and proper asset management are paramount. This concept isn't exclusive to digital assets; traditional financial institutions have custodians as well.
  - Within a blockchain environment, an address usually refers to a specific destination where cryptocurrencies are sent. Think of it like an account number in the traditional banking sense.
    In the context of smart contracts or decentralized applications, `custodianAddresses` likely refer to the collection of blockchain addresses that are authorized to hold assets on behalf of others.
    These addresses are controlled by the custodian, which could be a smart contract or a third-party service that maintains the security of the private keys associated with these addresses.

##

## 1.2. Codebase Explanation & Examples Scenarios of Intended Protocol Flow

###

#### All possible Actions and Flows in Ethena Protocol:

##### 1. Minting USDe:
Users provide stETH as collateral to mint USDe. The system gives an RFQ (Request for Quote) detailing how much USDe they can create. Upon agreement, the user signs a transaction, and Ethena mints USDe against the stETH, which is then employed in various yield-generating activities, primarily shorting ETH perpetual futures to maintain a delta-neutral position.

###### Additional Explanations Related to the `Minting`:
- So, `USDe` will be equivalent to using DAI, USDC, USDT (`USDe` contract is just the ERC20 token for the stablecoin and this token will be the token used in `StakedUSDeV2.sol` staking contract) where it `doesn't have any yield` and `only the holders of stUSDe` will earn the generated yield.
- The `EthenaMinting.sol` contract is the place where `minting` is done.

###

##### 2. Yield Generating:
Ethena generates yield by taking advantage of the differences in staking returns (3-4% for stETH) and shorting ETH perpetuals (6-8%). Profits are funneled into an insurance fund and later distributed to stakers, enhancing the value of stUSDe relative to USDe.

###

##### 3. Maintaining Delta Neutrality: 
Ethena employs a strategy involving stETH and short positions in ETH perpetuals, ensuring the value stability of users' holdings against market volatility.

###### *Example № 1:*
  1. Initial Setup:
    - The user initiates the process by sending 10 stETH to Ethena. The stETH is a token representing staked Ethereum in the Ethereum 2.0 network, allowing holders to earn rewards while keeping liquidity. At the time of the transaction, Ethereum's price is $2,000; thus, 10 stETH equals $20,000.
    - Ethena uses these 10 stETH to mint 20,000 USDe stablecoins for the user, reflecting the stETH's dollar value.
    - Simultaneously, Ethena opens a short position on Ethereum perpetual futures (ETH perps) equivalent to 10 ETH. Given the current Ethereum price of $2,000, this also represents a $20,000 position. This short position means that Ethena is betting on the Ethereum price going down.
  2. Market Movement and Its Impact:
    - Now, the market faces significant volatility, and the price of Ethereum drops by 90%. As a result, the value of the user's 10 stETH decreases to $2,000 (reflecting the 90% drop from the original $20,000 value).
    - However, because Ethena shorted 10 ETH worth of perps, the decrease in Ethereum's price is advantageous for this position. The short ETH perps position now has an unrealized profit of $18,000. This profit occurs because Ethena 'borrowed' the ETH at a higher price to open the position and can now 'buy' it back at a much lower price, pocketing the difference.
  3. Redemption Process:
    - The user decides to redeem their 20,000 USDe. For Ethena to honor this request, they need to provide the user with the equivalent value in stETH that the USDe represents.
    - Ethena closes the short position on the ETH perps, which means they 'buy' back the ETH at the current market price, realizing the $18,000 profit due to the price difference from when they opened the short position.
    - With the $18,000, Ethena purchases 90 stETH at the current market price ($200 per stETH, as the price has dropped by 90%).
    - Ethena then returns the original 10 stETH along with the 90 stETH purchased from the profits of the short position. So, the user receives 100 stETH, which, at the current market price, is worth $20,000.

###### *Example № 2:*
  1. Initial Condition:
    - The price of ETH is $2,000.
    - The user sends in 10 stETH (equivalent to 10 ETH) to Ethena to mint 20,000 USDe (since 10 ETH at $2,000 per ETH is worth $20,000).
    - Ethena takes these 10 stETH and opens a short position on 10 ETH's worth of perpetual futures (perps) to hedge against the price movement of ETH.
  2. Market Movement:
    - The market goes up by 50%. Therefore, the price of ETH (and stETH, as it’s pegged to the ETH value) increases to $3,000.
  3. Position Analysis:
    - The user's 10 stETH is now worth $30,000 due to the market increase.
    - However, Ethena's short position is now at a notional loss because it was betting on the price of ETH going down, not up. The loss on the short position is $10,000 (the increase in value per ETH is $1,000, and Ethena shorted 10 ETH).
  4. Redemption Process:
    - If the user decides to redeem their USDe, they will present their 20,000 USDe.
    - Considering the market movement, the short position's loss needs to be covered. Ethena has to close the short position and realize the loss of $10,000.
    - After covering the $10,000 loss, there's $20,000 worth of stETH left (approximately 6.67 stETH at the new rate of $3,000 per stETH) to return to the user.
  5. End Result:
    - The user initially had assets worth $20,000 (10 stETH). If they hadn't engaged with Ethena and simply held onto their 10 stETH, their assets would now be worth $30,000 due to the positive market movement.
    - By choosing to use Ethena's hedging mechanism, they've forfeited potential gains to safeguard against potential losses. They receive approximately 6.67 stETH (worth $20,000) back after the redemption process, missing out on the additional $10,000 value increase.
    - Essentially, the user's assets remained stable in USDe value, but they did not benefit from ETH's bullish market. Their asset value didn’t decrease, but they also lost potential profit

###

---

##

# 2. Codebase Quality Analysis

1. **`USDe.sol`**:

    ![usde-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/usde-contract.png?raw=true)

   - Code Organization:
     - The contract is well-organized and follows the best practices for code layout and structure.
     - It uses OpenZeppelin contracts, which are widely recognized and audited.
     - The constructor initializes the contract and sets the owner/admin.
     - It provides functions to set the minter and mint USDe tokens.

   - Modifiers:
     - The contract uses the `Ownable2Step` modifier, which enforces two-step ownership transfer.
     - The `onlyRole` modifier is used to restrict certain functions to specific roles, enhancing security.

   - Minting:
     - The contract allows only the minter to mint new tokens, which is a good security measure.
     - It checks if the provided minter is valid.

   - Gas Efficiency:
     - The contract uses the SafeERC20 library for safe token transfers, ensuring protection against reentrancy attacks.
     - Gas-efficient practices are followed throughout the contract.

   - Overall, `USDe.sol` appears to be well-structured and follows best practices for security and code organization.

####

2. **`EthenaMinting.sol`**:

    ![ethena-minting-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/ethena-minting-contract.png?raw=true)

   - Code Organization:
     - It imports external libraries and contracts, including OpenZeppelin contracts.
     - The constructor initializes contract parameters and roles.
     - It contains functions for minting and redeeming USDe tokens.

   - Security Measures:
     - The contract enforces access control using role-based access control (RBAC) with different roles for minters, redeemers, and gatekeepers.
     - Gas limits for minting and redeeming are enforced to prevent abuse.

   - Signature Verification:
     - It verifies the signature of orders, ensuring that the orders are signed by authorized parties.
     - It uses EIP-712 for signature verification.

   - Gas Efficiency:
     - Gas-efficient practices are followed, and SafeERC20 is used for token transfers.

   - Domain Separator:
     - The contract computes the domain separator for EIP-712, enhancing security.

   - Deduplication:
     - The contract implements deduplication of taker orders to prevent replay attacks.

   - Supported Assets:
     - The contract maintains a list of supported assets.

   - Custodian Addresses:
     - It keeps track of custodian addresses and allows transfers to custodian wallets.

   - Overall, `EthenaMinting.sol` is well-structured, secure, and follows best practices for code organization and security.

####

3. **`StakedUSDe.sol:`**

  The contract inherits **Implementation of the ERC4626 "Tokenized Vault Standard" from OZ**

    ![erc4626-oz-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/erc4626-oz-contract.png?raw=true)

    ####

    ![staked-usde-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/staked-usde-contract.png?raw=true)

- Code Organization:
  - It imports external libraries and contracts, including OpenZeppelin contracts.
  - The constructor initializes contract parameters and roles.
  - It contains functions for transferring rewards, managing the blacklist, rescuing tokens, and redistributing locked amounts.
  - Public functions are provided to query the total assets and unvested amounts.
  - The `decimals` function is overridden to return the number of decimal places.
  - Custom modifiers ensure input validation and role-based access control.
  - Hooks and functions are in place to enforce restrictions on specific roles and token transfers.

####

4.  **`StakedUSDeV2.sol:`**

  ![staked-usde-v2-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/staked-usde-v2-contract.png?raw=true)

- Code Organization:
  - The contract extends `StakedUSDe` and inherits its code organization structure.
  - It introduces additional state variables, including `cooldowns`, `silo`, `MAX_COOLDOWN_DURATION`, and `cooldownDuration`.
  - Custom modifiers, `ensureCooldownOff` and `ensureCooldownOn`, are defined to control the execution of functions based on cooldown status.
  - The constructor initializes the contract and sets the cooldown duration.
  - External functions are provided for withdrawing, redeeming, and performing cooldown actions for assets and shares.
  - The contract enforces different behavior based on the cooldown duration, adhering to or breaking the ERC4626 standard.
  - The `setCooldownDuration` function allows the admin to update the cooldown duration.

####

5. **`USDeSilo.sol:`**

  ![usde-silo-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/usde-silo-contract.png?raw=true)

  - The contract is primary goal of `USDeSilo.sol` is to to `hold the funds` for the `cooldown period` whn user initiate `unstaking`. 

  **General Observations:**
   - Role-based access control is implemented for various functions, enhancing security.
   - Gas-efficient practices, such as using SafeERC20, are followed throughout the code.
   - The codebase of protocol includes comprehensive comments and region divisions for clarity.
   - Note that, The use of EIP-712 for signature verification adds an extra layer of security.

6. **`SingleAdminAccessControl.sol:`**
    
  ![single-admin-access-control-contract](https://github.com/radeveth/Ethena-c4-Contest/blob/main/single-admin-access-control-contract.png?raw=true)

  - EthenaMinting uses SingleAdminAccessControl rather than the standard AccessControl.
###

In summary, the Ethena Protocol's codebase appears to be of high quality, with a strong focus on security and code organization.

###

---

##

# 3. Centralization Risks
Actually, the `Ethena` Protocol contains many roles, each with quite a few abilities. This is necessary for the Protocol's logic and purpose.

The protocol assigns important roles like "MINTER," "REWARDER," and "ADMIN" to specific entities, potentially exposing the system to undue influence or risks if these roles are compromised.

So, these roles introduce several **centralization risks**. The most significant one is the scenario in which the **`MINTER` role becomes compromised**. An attacker/minter could then `mint a billion USDe` without collateral and dump them into pools, causing a black swan event that our insurance fund cannot cover.

However, `Ethena` **addresses this problem** by enforcing on-chain **mint and redeem limitations of 100k USDe** per block."

From the documentation:
> Our solution is to enforce an on chain mint and redeem limitation of 100k USDe per block. In addition, we have `GATEKEEPER` roles with the ability to disable mint/redeems and remove `MINTERS`,`REDEEMERS`. `GATEKEEPERS` acts as a safety layer in case of compromised `MINTER`/`REDEEMER`. They will be run in seperate AWS accounts not tied to our organisation, constantly checking each transaction on chain and disable mint/redeems on detecting transactions at prices not in line with the market. In case compromised `MINTERS` or `REDEEMERS` after this security implementation, a hacker can at most mint 100k USDe for no collateral, and redeem all the collateral within the contract (we will hold ~$200k max), for a max loss of $300k in a single block, before `GATEKEEPER` disable mint and redeem. The $300k loss will not materialy affect our operations.

###

In summary, `Ethena` actually introduces several centralization risks due to the presence of many different roles in the Protocol. **However, at the same time, the team has done its best to enforce measures that reduce the largest potential attack scenario to a maximum loss of $300k, which will not materially affect the `Ethena` operations/ecosystem.**
###

---

##

# 4. Systemic Risks
Here’s an analysis of potential systemic

1. Smart Contract Vulnerability Risk:
Smart contracts can contain `vulnerabilities` that can be exploited by attackers. If a smart contract has `critical security flaws`, such as logic problems, this could lead to asset loss or `system manipulation`. I strongly recommend that, once the protocol is audited, necessary actions be taken to `mitigate any issues` identified by `C4 Wardens`

###

2. Third-Party Dependency Risk:
Contracts rely on external data sources, such as `@openzeppelin/contracts-upgradeable`, and there is a risk that if any `issues` are found with these `dependencies` in your contracts, the `Ethena` protocol could also be affected.

I observed that `old versions` of `OpenZeppelin` are used in the project, and these should be updated to the latest version:

```json
  "name": "@openzeppelin/contracts-upgradeable",
  "description": "Secure Smart Contract library for Solidity",
  "version": "4.9.2",
```

The latest version is `4.9.3` (as of July 28, 2023), while the project uses version `4.9.2`.

###

---

##

# 5. Attack Vectors Discussed During the Audit
1. Issues related to Roles (Centralization Risks). Problems with roles changing.
2. Breaking of Main Protocol Invariants
    - EthenaMinting.sol - User's signed EIP712 order, if executed, must always execute as signed. ie for mint orders, USDe is minted to user and collateral asset is removed from user based on the signed values.
    - Max mint per block should never be exceeded.
    - USDe.sol - Only the defined minter address can have the ability to mint USDe.
3. DoS for important protocol functions/flows such as `EtehnaMinting.sol#mint()` (Minting Flow), `EtehnaMinting.sol#redeem()` (Redemption Flow), `StakedUSDe#deposit()/StakedUSDe#_deposit()` (Depositing Flow), `StakedUSDeV2#unstake()` (Unstaking Flow). 
4. Token transfer fails.
5. Minting more than 100k USDe per block.
6. Users cannot unstake/withdraw/redeem.
7. Can users withdraw more than they actually are supposed to?
8. Minting without providing collateral amount?

###

---

##

# 6. Example of report that turned out to be invalid after I wrote a `really good Explanation and PoC` (The report explain the `unstaking` really well, so you can learn for it)

## REPORT:

---

## Title: Users will not be able to `withdraw/redeem` their assets/shares from `StackedUSDeV2` contract. (Inefficient logic)

## Explanation
[`StakedUSDeV2.sol`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol) is where holders of `USDe` stablecoin can stake their stablecoin, get `stUSDe` in return and `earn yield`. The Etehna Protocol yield is paid out by having a `REWARDER` role of the staking contract send yield in `USDe`, increasing the `stUSDe` value with respect to `USDe`. This contract is a modification of the `ERC4626 standard`, with a change to vest in rewards linearly over 8 hours

When the `unstake` process is initiated, from the user's perspective, `stUSDe` is burnt immediately, and they will be able to invoke the [`withdraw()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L52-L60) function after `cooldown` is up to get their `USDe` in return. Behind the scenes, on `burning of stUSDe`, `USDe` is sent to a seperate [USDeSilo](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol) contract to hold the funds for the `cooldown period`. And on `withdrawal`, the staking contract moves user funds from `USDeSilo` contract out to the `user's address`.

##### 

1. Functions respond for `redemption`, `starting of cooldown` and `transferring of converted underlying asset to USSDeSilo contract` are [cooldownAssets() and cooldownShares()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L95-L122)

```jsx
  function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {
    if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();

    uint256 shares = previewWithdraw(assets);

    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    cooldowns[owner].underlyingAmount += assets;

    _withdraw(_msgSender(), address(silo), owner, assets, shares);

    return shares;
  }

  /// @notice redeem shares into assets and starts a cooldown to claim the converted underlying asset
  /// @param shares shares to redeem
  /// @param owner address to redeem and start cooldown, owner must allowed caller to perform this action
  function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
    if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();

    uint256 assets = previewRedeem(shares);

    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    cooldowns[owner].underlyingAmount += assets;

    _withdraw(_msgSender(), address(silo), owner, assets, shares);

    return assets;
  }
```

2. After `cooldown` is finished, user can call [`unstake()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L78-L90) to `claim the staking amount after`.
```jsx
  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }
```
*The function check if the cooldown is finished by `block.timestamp >= userCooldown.cooldownEnd` check.*
*REMEMBER: The assets are transferred from `USDeSilo` contract -> https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L28-L30*

#### 

Described logic above work only if the cooldown is set to number greater than zero. (aka the cooldown is active)

The `Ethena` can decide to `disable the cooldown period`, so the users to be able `unstake without cooldown period`. If this is done, the user will be able to call directly [`withdraw()/redeem()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L52-L73) to unstake:
```jsx
  function withdraw(uint256 assets, address receiver, address owner)
    public
    virtual
    override
    ensureCooldownOff
    returns (uint256)
  {
    return super.withdraw(assets, receiver, owner);
  }

  /**
   * @dev See {IERC4626-redeem}.
   */
  function redeem(uint256 shares, address receiver, address owner)
    public
    virtual
    override
    ensureCooldownOff
    returns (uint256)
  {
    return super.redeem(shares, receiver, owner);
  }
```
*REMEMBER: The assets are transferred from `StakedUSDeV2` contract -> https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L28-L30*


## Proof of Concept
So, the problem in Ethena protocol logic aries exactly when `Ethena` decide to `disable the cooldown period`.
Lets illustrate the following example:

### Scenario

- Cooldown Period: 50 days

1. Alice deposit her 10000 `USDe` tokens in `StakedUSDeV2` contract.
2. Bob also deposit his 20000 `USDe` tokens in `StakedUSDeV2` contract.
3. After some time Bob decide to unstake and call `cooldownAssets()` to start of cooldown period and converted underlying asset are transferred to `USSDeSilo` contract and he redeem his 20000 `USDe` tokens.
4. After 30 days `Ethena` disable cooldown period.
5. Now users are supposed to unstake directly from `withdraw()/redeem()`.
6. But when the Bob tries to call `withdraw()/redeem()` he will not be able to get his converted underlying asset, because they are in `USDeSilo` contract.
7. Alice call `withdraw()/redeem()` and get his converted underlying asset.

## Tools Used

- Manual Inspection

---

After all, I observed that users who have already called `cooldownAssets()/cooldownShares()` can call the `unstake()` function again to retrieve their converted underlying asset.

### Time spent:
35 hours