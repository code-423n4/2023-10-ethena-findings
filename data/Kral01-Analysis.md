# Ethena Labs Smart Contract Analysis

## Table of Contents

- [1. Summary](#1-summary)
- [2. Audit Approach](#2-audit-approach)
- [3. Architecture Overview](#3-architecture-overview)
- [4. Architecture recommendations](#4-architecture-recommendations)
- [5. Centralization risks](#5-centralization-risks)
- [6. Monitoring Recommendations](#6-monitoring-recommendations)
- [7. Financial activity](#7-financial-activity)

## <a name="1-summary"></a>1. Summary

For this analysis, I will center my attention on the scope of the ongoing audit contest `2023-10-Ethena-Labs`. I begin by outlining the code audit approach employed for the in-scope contracts, followed by sharing my perspective on the architecture, and concluding with observations about the implementation's code.

Please note that unless explicitly stated otherwise, any architectural risks or implementation issues mentioned in this document are not to be considered vulnerabilities or recommendations for altering the architecture or code solely based on this analysis. As an auditor, I acknowledge the importance of thorough evaluation for design decisions in a complex project, taking associated risks into consideration as one single part of an overarching process. It is also important to recognize that the project team may have already assessed these risks and determined the most suitable approach to address or coexist with them. 

## <a name="2-audit-approach"></a>2. Audit Approach

Time spent: 55 hours

### 2.1 Documentation and scope

The initial step involved examining audit documentation and scope to grasp the audit's concepts and boundaries, and prioritize my efforts. It is to be mentioned that because the [documentation](https://ethena-labs.gitbook.io/ethena-labs/10CaMBZwnrLWSUWzLS2a/) provided by the team was top notch and with a relatively low SLoC, it was very easy to grasp a general outline of the codebase quickly.

### 2.2 Setup and Test

The setup was straightforward and I faced no issues. The already provided tests served as a documentation which provided insight into how the codebase is supposed to be used and will be used when it launches. The tests were top notch and had very impressive code coverage for the contracts in scope.

### 2.3 Code review

After getting an initial understanding and setting up the tests, I began to thoroughly examine the codebase by entering from the User's perspective and began to explore each process flows to go 'deeper' into the codebase. I commented heavily on the code and taking down notes as I was going through the codebase.

### 2.4 Threat modelling

I began formulating specific assumptions that, if compromised, could pose security risks to the system. This process aids me in determining the most effective exploitation strategies. While not a comprehensive threat modeling exercise, it shares numerous similarities with one.

### 2.5 Exploitation and Proof of Concept

After my code analysis, I began to explore each potential vulnerability that came to my mind while writing tests. My primary objective in this phase is to challenge critical assumptions, generate new ones in the process, and enhance this process by leveraging coded proofs of concept to expedite the development of successful exploits. I was going through an iterative process of explore, find leads, write tests to test my assumptions and uncover new leads.

### 2.6 Report Issues

The most effective approach to bring more value to sponsors (and, hopefully, to auditors) is to document what can be gained by exploiting each vulnerability. This assessment helps in evaluating whether these exploits can be strategically combined to create a more significant impact on the system's security. In some cases, seemingly minor and moderate issues can compound to form a critical vulnerability when leveraged wisely. This has to be balanced with any risks that users may face. 

## <a name="3-architecture-overview"></a>3. Architecture Overview

### 3.1 Architecture summary

Ethena labs introduces USDe which is a stablecoin that solves the [stablecoin trilemma](https://multicoin.capital/2021/09/02/solving-the-stablecoin-trilemma/) by solving the following problems:

1. **Scalability**:  It is achieved by utilizing derivatives which allows for USDe to scale with capital efficiency. Since the staked ETH collateral can be perfectly hedged with a short position of equivalent notional, the synthetic dollar only requires 1:1 "collateralization."
2. **Stability**: It is provided through hedges executed against the transferred assets immediately on issuance that ensures the synthetic USD value of USDe backing in all market conditions.
3. **Censorship Resistance**: It is achieved by separating backing from the banking system and storing trustless backing assets outside of centralized liquidity venues in onchain, transparent, 24/7 auditable, programmatic custody account solutions.

Here's how the USDe token works:

1. Users request a price from the Ethena Pricing API and sign and send a request for order to Ethena if they are satisfied with the price.
2. Once the signed order is received, Ethena's server checks that the user has the required balance and approvals and that the dynamic hedging server can currently handle the order.
3. The mint function is called when these checks are passed by the `MINTER_ROLE` which is an insider role in Ethena. The transfer of the user's assets and the minted synthetic dollars happen in a single transaction.
4. Upon success the of the transaction, the hedging system actions the mint events to ensure the delta neutrality of the protocol's portfolio.

![EthenaLabsUSDe](https://github.com/Abelaby/Public_Audits/blob/main/Audits/Code4rena/Ethena/EthenaLabsUSDe.png)

There is also a staking system which allows users to deposit USDe and get stUSDe, which represents the interest of the Staked USDe,  stUSDe is a token that increases in value by leveraging on the yields deposited into the protocol. The staking rewards and staked USDe withdrawls are controlled by a cooldown period.

![EthenaLabsStake](https://github.com/Abelaby/Public_Audits/blob/main/Audits/Code4rena/Ethena/EthenaLabsStake.png)

### 3.2 Analysis of codebase in Scope

1. [USDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol) : This contract defines the USDe stablecoin that allows another contract to mint USDe coins. The only allowed contract is the one with the `MINTER` role which is the EthenaMinting contract as per the docs. We can say that only the `EthenaMinting` contract is allowed to mint the USDe token, which is a nice safety feature. This contract inherits from `Ownable2Step`, `ERC20Burnable` , `ERC20Permit` from the OZ library.

2. [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol) : This is the main contract which  allows the `mint` and `redeem` functionality of the protocol. USDe.sol grants this contract the ability to mit USDe ie; with the `MINTER` role. The main entry points in this contract are  `mint` and `redeem` which can only be called by the `MINTER_ROLE` and `REDEEMER_ROLE` respectively. These roles are internal Ethena roles whom receive the orders and signatures from the Users and call the appropriate function after running checks. There is a `maxMintPerBlock` and `maxRedeemPerBlock` which are safeguards against when the MINTER/REDEEMER role is compromised, to decrease the impact of lost funds. The protocol mentioned that the 'highest loss that could occur through mint and redeem was atmost ~300K which was negligible given the scope of USDe token' . This is a nice security feature in case the authorized roles are ever compromised. There is another role `GATEKEEPER_ROLE` which acts as an admin in this contract, this role has the ability to pause the mint/redeem by setting the max values as zero, this role can also remove and add new MINTER_ROLE and REDEEMER_ROLE. This contract uses the `ReentrancyGuard`, `SafeERC20`, `ECDSA`, `EnumerableSet` from the OZ library.

3. [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol) : This is the base contract that implements the core staking functionality. This is inherited by the `StakedUSDeV2` contract. This contract implements the `_withdraw` and `_deposit` functions with proper permission checks. The four major roles in this contract are `REWARDER_ROLE`, `SOFT_RESTRICTED_STAKER_ROLE`, `FULL_RESTRICTED_STAKER_ROLE` and `BLACKLIST_MANAGER_ROLE`. The two blacklist roles are for controlling addresses by limiting them of functionalities such as depositing and withdrawing. The manager can add and remove addresses from the blacklist. The `REWARDER_ROLE` transfers the rewards to the contract that should be available for distribution. There is a `minShares` check which is a nice addition to avoid donation attacks. This contract inherits from `SingleAdminAccessControl`, `ReentrancyGuard` and `ERC20Permit`.

4. [StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol) : This is the entry point for the end user. There is a cooldown mechanism that can be configured upto 90 days which serves as a cooldown for each user to withdraw their assets by unstaking. When want to withdraw their assets, the corresponding asset of the user is send to the `USDeSilo` contract which holds the asset USDe till the cooldown period of the user is over. Then after cooldown, the user can call `unstake` and withdraw their USDe from the silo contract. This contract inherits the `StakedUSDe` contract.

5. [USDeSilo.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol) : This contract is to temporarily hold USDe during redemption cooldown of Users. There is modifier to check if the `withdraw` function is called by the Staking contract. This contract can be seen as a small temporary vault that holds the USDe tokens before redeeming.

6. [SingleAdminAccessControl.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol) : This contract is a simplified version of the the OZ access control libary implemented by Ethena to be used by their contracts. As the name suggests, it grants the top admin role to one single entity. This contract has the necessary functions such as `transferAdmin`, `grantRole`, `revokeRole` and a modifier `notAdmin`. This contract inherits from `IERC531` and `AccessControl` from OZ library.


### 3.3 Privileged roles.

- `USDe` minter - can mint any amount of `USDe` tokens to any address. Expected to be the `EthenaMinting` contract
- `USDe` owner - can set token `minter` and transfer ownership to another address
- `StakedUSDe` admin - can rescue tokens from the contract and also to redistribute a fully restricted staker's `stUSDe` balance, as well as give roles to other addresses (for example the `FULL_RESTRICTED_STAKER_ROLE` role)
- `StakedUSDeV2` admin - has all power of "`StakedUSDe` admin" and can also call the `setCooldownDuration` method
- `REWARDER_ROLE` - can transfer rewards into the `StakedUSDe` contract that will be vested over the next 8 hours
- `BLACKLIST_MANAGER_ROLE` - can do/undo full or soft restriction on a holder of `stUSDe`
- `SOFT_RESTRICTED_STAKER_ROLE` - address with this role can't stake his `USDe` tokens or get `stUSDe` tokens minted to him
- `FULL_RESTRICTED_STAKER_ROLE` - address with this role can't burn his `stUSDe` tokens to unstake his `USDe` tokens, neither to transfer `stUSDe` tokens. His balance can be manipulated by the admin of `StakedUSDe`
- `MINTER_ROLE` - can actually mint `USDe` tokens and also transfer `EthenaMinting`'s token or ETH balance to a custodian address
- `REDEEMER_ROLE` - can redeem collateral assets for burning `USDe`
- `EthenaMinting` admin - can set the maxMint/maxRedeem amounts per block and add or remove supported collateral assets and custodian addresses, grant/revoke roles
- `GATEKEEPER_ROLE` - can disable minting/redeeming of `USDe` and remove `MINTER_ROLE` and `REDEEMER_ROLE` roles from authorized accounts

## <a name="4-architecture-recommendations"></a>4. Architecture recommendations

Here are some improvements that I noticed that can be added to the codebase, note that these are recommendations and sgould be implemented after proper validation that it doesn't create any new issues.

### 4.1 Users can't delegate to other Users for only redeem/mint

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L235C3-L238C4

The `setDelegatedSigner` delegates a delegatee to have power to mint/redeem. This can cause issues when the delegate wants to only delegate the user to perform one operation ie; mint/redeem. Suppose Alice delegates Bob to mint tokens. She only wants Bob to mint tokens for her. Ideal flow will be Bob mints tokens for Alice. But at the same time if Alice has USDe tokens in her account, then Bob can redeem her tokens as long as he is the delegatee.

This is better addressed by having a ENUM like:

```solidity
ENUM ACCESS{
  INVALID, //-0
  MINT, //-1
  REDEEM //2
}
```
And assign it to the `delegatedSigner` mapping instead of true/false
```diff
function setDelegatedSigner(address _delegateTo) external {
-    delegatedSigner[_delegateTo][msg.sender] = true; //Replace with enum, default will be invalid
+    delegatedSigner[_delegateTo][msg.sender] = ACCESS.MINT;
   emit DelegatedSignerAdded(_delegateTo, msg.sender);
  }
```

### 4.2 Grant role to privileged actors in the constructor itself

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L111C2-L147C1
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L70C2-L81C4

In [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol) and [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol) the roles, `REDEEMER_ROLE`, `MINTER_ROLE`, `GATEKEEPER_ROLE` and  `BLACKLIST_MANAGER_ROLE` are not assigned when the contract is initilalized but by calling `granRole()` externally(as per the tests). It will be better to have the constructor set the roles when the contracts are initilaized as they will be set right away and in any case, wont remain unassigned and cause issues.

In EthenaMinting.sol:

```diff
 constructor(
    IUSDe _usde,
    address[] memory _assets,
    address[] memory _custodians,
    address _admin,
    uint256 _maxMintPerBlock,
    uint256 _maxRedeemPerBlock,
    address minter,
    address redeemer,
    address gateKeeper
  ) {
    ---------------------------
    -------------------------------
    -----------------------------
+ _grantRole(REDEEMER_ROLE, minter );
+ _grantRole(MINTER_ROLE, redeemer);
+ _grantRole(GATEKEEPER_ROLE, gateKeeper);
}
```

In StakesUSDe.sol:

```diff
 constructor(IERC20 _asset, address _initialRewarder, address _owner, address blacklistManager)
    ERC20("Staked USDe", "stUSDe")
    ERC4626(_asset)
    ERC20Permit("stUSDe")
  {
    if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {
      revert InvalidZeroAddress();
    }

    _grantRole(REWARDER_ROLE, _initialRewarder);
    _grantRole(DEFAULT_ADMIN_ROLE, _owner);
+   _grantRole(BLACKLIST_MANAGER_ROLE, blacklistManager);
  }
```


### 4.3 Zero value is added, can be avoided

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89C1-L99C4

The `transferInRewards` in `StakedUSDe.sol` has a redundant '0' added into it. The function first checks `if (getUnvestedAmount() > 0) revert StillVesting()` then in the nect line it adds the `getUnvestedAmount()` into `newUnvestedAmount()`, the uint256 value returned by `getUnvestedAmount()` will be '0' and there is no need to add it and can be removed. 

```diff
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();
+    uint256 newVestingAmount = amount;

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```

### 4.4 Unnecessary calculation can be avoided.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L173C1-L181C4

The `getUnvestedAmount()` in `StakedUSD.sol` returns the unvestedAmount by running a calculation based on the `lastDistributionTimestamp` and `vestingAmount`, it will return zero after the calculation if vestingAmount is zero. So the code can be refactored instead of performing additional calculation.

```diff
function getUnvestedAmount() public view returns (uint256) {
    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

-    if (timeSinceLastDistribution >= VESTING_PERIOD) {
+    if(timtimeSinceLastDistribution >= VESTING_PERIOD || vestingAmount == 0 ){
      return 0;
    }

    return ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) / VESTING_PERIOD;
  }
```

### 4.5 Add Minter/Redeemer function is missing

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L277C2-L285C4

We can see that in [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol) there are `removeMinterRole` and `removeRedeemerRole` but never functions defined to add these roles(yeah, they can be assigned by calling granRole() externally but so does revokeRole() ) , either remove these two or add functions for `adding` those roles for convinience.

```diff
+  function addMinterRole(address minter) external onlyRole(GATEKEEPER_ROLE) { //@audit addminter function is missing
+    _grantRole(MINTER_ROLE, minter);
+  }

+  function addRedeemerRole(address redeemer) external onlyRole(GATEKEEPER_ROLE) {
+    _grantRole(REDEEMER_ROLE, redeemer);
+  }
```

## <a name="5-centralization-risks"></a>5. Centralization risks

This protocol contains a large amount of centralization, the single `DEFAULT_ADMIN_ROLE` controls almost everything in this protocol and can prove very fatal if there is a risk of the address being compromised. In case of a compromisation;

1. The admin can transfer ownership to another address
```solidity
function transferAdmin(address newAdmin) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (newAdmin == msg.sender) revert InvalidAdminChange();
    _pendingDefaultAdmin = newAdmin;
    emit AdminTransferRequested(_currentDefaultAdmin, newAdmin);
  }
```
2. Admin can grant roles to previleged roles such as `REWARDER_ROLE`, `MINTER_ROLE`, `REDEEMER_ROLE`, `GATEKEEPER_ROLE`
```solidity
function grantRole(bytes32 role, address account) public override onlyRole(DEFAULT_ADMIN_ROLE) notAdmin(role) {
    _grantRole(role, account);
  }
```

Given the importance and scope of this project, I strongly believe that this much access given to a single entity can prove very fatal in case of any compromise. I strongly advice the Ethena team to look toward this.

## <a name="6-monitoring-recommendations"></a>6. Monitoring Recommendations

While ``audits`` help in ``identifying`` code-level ``issues`` in the current implementation and potentially the code ``deployed`` in production, the ``Ethena`` team is encouraged to consider incorporating monitoring activities in the production environment. Ongoing monitoring of deployed contracts helps identify potential threats and issues affecting production environments. With the goal of providing a complete ``security assessment``, the monitoring ``recommendations`` section raises several actions addressing trust assumptions and out-of-scope components that can benefit from ``on-chain monitoring``. 

## <a name="7-financial-activity"></a>7. Financial activity

  - Consider monitoring the token transfers and user deposits and withdraws  to identify:
    * Transfers during normal operations to establish a baseline of healthy properties. Any large deviation, such as an ``unexpectedly`` large withdrawal, may indicate unusual ``behavior`` of the contracts or an ongoing attack.
    * Transactions that revert

These may indicate a ``user interface`` bug, an ongoing attack or other ``unexpected`` edge cases.

### Time spent:
55 hours