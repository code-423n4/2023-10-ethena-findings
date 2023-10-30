# QA Report - Ethena Labs Audit Contest | 24 Oct 2023 - 30 Oct 2023

---

# Executive summary

### Overview

|               |                                                                          |
| :-----------  | :----------------------------------------------------------------------- |
| Project Name  | Ethena Labs                                                              |
| Repository    | https://github.com/code-423n4/2023-10-ethena                             |
| Website       | [Link](https://www.ethena.fi/)                                           |
| Twitter       | [Link](https://twitter.com/ethena_labs)                                  |
| Documentation | [Link](https://ethena-labs.gitbook.io/ethena-labs/10CaMBZwnrLWSUWzLS2a/) |
| Methods       | Manual Review                                                            |
| Total nSLOC   | 588 over 6 contracts                                                     |



### Scope

`USDe.sol`
`EthenaMinting.sol` and the contract it extends, `SingleAdminAccessControl.sol`
`StakedUSDeV2.sol`, the contract it extends, `StakedUSDe.sol` and the additional contract it creates `USDeSilo.sol`

| Contract                                                                                                                      | SLOC | Purpose                                                                                                                  | Libraries used                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ----------------------------------------------------------------------------------------------------------------------------- | ---- | ------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [USDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol)                                         | 24   | USDe token stablecoin contract that grants another address the ability to mint USDe                                      | [`@openzeppelin/ERC20Burnable.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Burnable.sol) [`@openzeppelin/ERC20Permit.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Permit.sol) [`@openzeppelin/Ownable2Step.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) |
| [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol)                       | 295  | The contract where minting and redemption occurs. USDe.sol grants this contract the ability to mint USDe                 | [`@openzeppelin/ReentrancyGuard.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/ReentrancyGuard.sol)                                                                                                                                                                                                                                                                                                              |
| [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol)                             | 130  | Extension of ERC4626. Users stake USDe to receive stUSDe which increases in value as Ethena deposits protocol yield here | [`@openzeppelin/ReentrancyGuard.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/ReentrancyGuard.sol) [`@openzeppelin/ERC20Permit.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Permit.sol) [`@openzeppelin/ERC4626.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC4626.sol)        |
| [StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol)                         | 76   | Extends StakedUSDe, adds a redemption cooldown.                                                                          |                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| [USDeSilo.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol)                                 | 20   | Contract to temporarily hold USDe during redemption cooldown                                                             |                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| [SingleAdminAccessControl.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol) | 43   | EthenaMinting uses SingleAdminAccessControl rather than the standard AccessControl                                       |                                                                                                                                                                                                                                                                                                                                                                                                                                                            |



### Findings Summary

| ID                | Title |  Severity                  |
| ----------------- | ----- |  ------------------------- |
| [L-01](#L-01)     | The protocol does not ensure that the `EthenaMinting.sol` contract is the `minter` in the `USDe` contract, which can lead to impossible minting of `USDe` tokens for an indefinite period of time | _Low_ |
| [L-02](#L-02)     | The `_computeDomainSeparator()` function doesn't fully comply with the rules for `EIP-712: DOMAIN_SEPARATOR` | _Low_ |
| [L-03](#L-03)     | EIP712 has not been accurately applied to the `Route` struct | _Low_ |
| [L-04](#L-04)     | Malicious actor can evade the `FULL_RESTRICTED_STAKER_ROLE` role | _Low_ |
| [L-05](#L-05)     | `getUnvestedAmount()` is not needed to be read in `StackedUSDe.sol#transferInRewards()` function for the second time | _Low_ |
| [NC-01](#NC-01)   | Don't grant DEFAULT_ADMIN_ROLE twice in the `EthenaMinting` contract | _Non Critical_ |
| [NC-02](#NC-02)   | Write modifier for `if (msg.sender != minter) revert OnlyMinter();` check in the `USDe#mint()` function, so to be consistent with `USDe#setMinter()` function which performs the access control check in modifier | _Non Critical_ |
| [NC-03](#NC-03)   | Change the names of `removeFromBlacklist()` and `addToBlacklist()` functions | _Non Critical_ |
| [NC-04](#NC-04)   | Failure to validate functions return values can result in errors | _Non Critical_ |
| [NC-05](#NC-05)   | Add check in `rescueTokens()` if the `to` parameter is `FULL_RESTRICTED_STAKER_ROLE` and if revert | _Non Critical_ |
| [NC-06](#NC-06)   | The System Exclusively Accepts Nonces | _Non Critical_ |
| [NC-07](#NC-07)   | When adding an asset to `supportedAssets`, perform a check to ensure that the supported asset is not the `NATIVE_TOKEN`. If it is, then revert | _Non Critical_ |
| [NC-08](#NC-08)   | Don't assign role on `msg.sender` | _Non Critical_ |
| [NC-09](#NC-09)   | It is [recommended](https://github.com/ethereum/eips/issues/155) to use deadline for [meta-transactions](https://medium.com/coinmonks/complete-guide-to-meta-transactions-c46ca51dbd21) | _Non Critical_ |
| [NC-10](#NC-10)   | Lack of Event Emitting on Important for the Protocol functions | _Non Critical_ |

##

----

##

## [L-01] The protocol does not ensure that the `EthenaMinting.sol` contract is the `minter` in the `USDe` contract, which can lead to a impossible minting of `USDe` tokens for an indefinite period

### Summary:
The Ethena protocol consists of two contracts, "USDe" and "EthenaMinting." The "USDe" contract allows tokens to be minted only by the designated "minter" address, and this address can be set via the "setMinter" function. The "EthenaMinting" contract attempts to mint USDe tokens via the "mint" function of the "USDe" contract. However, the vulnerability is that the "EthenaMinting" contract has not been properly set as the minter in the "USDe" contract. In such a scenario, USDe tokens cannot be minted until the "EthenaMinting" contract is granted the minter role in the "USDe" contract.

- [`EthenaMinting#mint()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L162-L187)
```jsx
  function mint(Order calldata order, Route calldata route, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(MINTER_ROLE)
    belowMaxMintPerBlock(order.usde_amount)
  {
    if (order.order_type != OrderType.MINT) revert InvalidOrder();
    verifyOrder(order, signature);
    if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
    // Add to the minted amount in this block
    mintedPerBlock[block.number] += order.usde_amount;
    _transferCollateral(
      order.collateral_amount, order.collateral_asset, order.benefactor, route.addresses, route.ratios
    );
    usde.mint(order.beneficiary, order.usde_amount);
    emit Mint(
      msg.sender,
      order.benefactor,
      order.beneficiary,
      order.collateral_asset,
      order.collateral_amount,
      order.usde_amount
    );
  }
```
#####
- [`USDe#mint()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L28-L31)
```jsx
  function mint(address to, uint256 amount) external {
    if (msg.sender != minter) revert OnlyMinter();
    _mint(to, amount);
  }
```

In the [contest page](https://code4rena.com/contests/2023-10-ethena-labs#top:~:text=USDe%20minter%20%2D%20can%20mint%20any%20amount%20of%20USDe%20tokens%20to%20any%20address.%20Expected%20to%20be%20the%20EthenaMinting%20contract) write:
> USDe minter - can mint any amount of USDe tokens to any address. Expected to be the EthenaMinting contract

So it is essential to set the `minter` role in `USDe.sol` contract still in `USDe constructor` to `EthenaMinting` contract, otherwise **after deploying the Ethena Protocol minting from `EthenaMinting` contract will be impossible.**

### Impact:
- **After deploying the Ethena Protocol minting from `EthenaMinting` contract will be impossible.**
- Disrupt the minting of USDe tokens for an indefinite period, causing a temporary halt in token minting. If the "EthenaMinting" contract is not set as the minter or if it loses the minter role in the "USDe" contract, users will be unable to mint USDe tokens, affecting the protocol's functionality and potentially causing inconvenience to users.

### Recommendation Migration Steps:
- In the constructor of `USDe.sol` contract set the `minter` to be the `EthenaContract.sol` contract.

So, implement the constructor of `USDe.sol` contract as follow:

```jsx
    constructor(address admin, address ethenaMinting) ERC20("USDe", "USDe") ERC20Permit("USDe") {
        if (admin == address(0)) revert ZeroAddressException();
        _transferOwnership(admin);
        minter = ethenaMinting;
    }
```
---

## [L-02] The `_computeDomainSeparator()` function doesn't fully comply with the rules for `EIP-712: DOMAIN_SEPARATOR`

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L451-L453

### Summary:
The `_computeDomainSeparator()` function doesn't fully comply with the rules for `EIP-712: DOMAIN_SEPARATOR`. As indicated in [this article](https://officercia.mirror.xyz/nlIR1RkT5xIv4sZFYiOXCPhF2BJyAaJtOeVr6zsULsA#:~:text=EIP%2D712%3A%C2%A0DOMAIN_SEPARATOR), in the `EIP-712: DOMAIN_SEPARATOR` section, it is mentioned that `uint256` should be used instead of `uint` (same for `int`).

However, in the `EthenaMinting.sol` contract, the `DOMAIN_SEPARATOR` is generated as follows:

```jsx
function _computeDomainSeparator() internal view returns (bytes32) {
    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
}
```

As we can see from [solidity documentation](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#:~:text=block.chainid%20(uint)%3A%20current%20chain%20id), the global/special variable `block.chainid` is of type `uint` by default, not `uint256`.
Consequently, the `_computeDomainSeparator` function doesn't explicitly cast `block.chainid` to `uint256`. This deviation means that the `EthenaMinting` contract doesn't fully adhere to the rules for `EIP-712: DOMAIN_SEPARATOR`.

### Recommended Migration Steps:

Implement the `_computeDomainSeparator` as follows:

```jsx
function _computeDomainSeparator() internal view returns (bytes32) {
    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, uint256(block.chainid), address(this)));
}
```

---

## [L-03] EIP712 has not been accurately applied to the `Route` struct

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L334-L336

### Summary:

As defined by the [EIP712 standard](https://eips.ethereum.org/EIPS/eip-712#:~:text=The%20array%20values%20are%20encoded%20as%20the%20keccak256%20hash%20of%20the%20concatenated%20encodeData%20of%20their%20contents), array values should be encoded as the `keccak256` hash of the concatenated `encodeData` of their contents. The current implementation in `EthenaMinting.sol#encodeRoute()` does not adhere to this rule for the `Route` Struct array fields. 

### Recommended Migration Steps:
Only, because the `encodeRoute()` function is not invoked in the protocol, can be safely removed.

---

## [L-04] Malicious actor can evade the `FULL_RESTRICTED_STAKER_ROLE` role

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L106-L113

### Summary:
The protocol has introduced the `FULL_RESTRICTED_STAKER_ROLE` to empower the `StakedUSDe` owner with the authority to `blacklist` certain addresses and potentially manipulate their balances. However, this mechanism exhibits a vulnerability that could be bypassed with front funning attack.

1. A user, either through malicious actions or being identified as a bad actor, comes under scrutiny.
2. The `StakedUSDe` owner decides to blacklist the user's address and initiates a transaction by calling `addToBlacklist()` for that specific address.
3. The user is vigilantly monitoring Ethereum's mempool and swiftly front-runs this transaction. As a result, the user rapidly transfers their entire stUSDe balance to another address that they control.
4. While the user's original address is indeed blacklisted, they retain control over all their tokens, which are now held in the new address, and these tokens remain fully functional.

---

## [L-05] `getUnvestedAmount()` is not needed to be read in `StackedUSDe.sol#transferInRewards()` function for the second time

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89-L99

### Summary:
The `StackedUSDe.sol#transferInRewards()` function allows the owner to `transfer rewards` from the `controller contract` into this contract. However the function can be executed only if `getUnvestedAmount()` is greater than zero:
```jsx
    if (getUnvestedAmount() > 0) revert StillVesting();
```

Later in the function `newVestingAmount` used "add specified `amount` from owner and `getUnvestedAmount()`". However `getUnvestedAmount()` will always be zero.

```jsx
    /**
     * @notice Allows the owner to transfer rewards from the controller contract into this contract.
     * @param amount The amount of rewards to transfer.
     */
    function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
        if (getUnvestedAmount() > 0) revert StillVesting();
        uint256 newVestingAmount = amount + getUnvestedAmount();

        vestingAmount = newVestingAmount;
        lastDistributionTimestamp = block.timestamp;
        // transfer assets from rewarder to this contract
        IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

        emit RewardsReceived(amount, newVestingAmount);
    }
```

### Recommended Migration Steps:
Rewrite the `StackedUSDe.sol#transferInRewards()` function as follow:

```diff
    /**
     * @notice Allows the owner to transfer rewards from the controller contract into this contract.
     * @param amount The amount of rewards to transfer.
     */
    function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
        if (getUnvestedAmount() > 0) revert StillVesting();
    -   uint256 newVestingAmount = amount + getUnvestedAmount();

    -   vestingAmount = newVestingAmount;
        lastDistributionTimestamp = block.timestamp;
        // transfer assets from rewarder to this contract
        IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    -   emit RewardsReceived(amount, newVestingAmount);
    +   emit RewardsReceived(amount, amount);
    }
```

---


## [NC-01] Don't grant DEFAULT_ADMIN_ROLE twice in the `EthenaMinting` contract

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L111-L146

### Summary:
One of the arguments in the constructor of the `EthenaMinting` contract is `admin`. In the constructor, it initially grants the `DEFAULT_ADMIN_ROLE` to `msg.sender`. Then, the code checks if `msg.sender` is different from the `admin` parameter, and if it is, the `DEFAULT_ADMIN_ROLE` is granted to the `admin` address. This logic is not efficient, and it is also inefficient for one contract to have two `DEFAULT_ADMIN_ROLE` addresses. 

### Mitigation:
```diff
  constructor(
    IUSDe _usde,
    address[] memory _assets,
    address[] memory _custodians,
    address _admin,
    uint256 _maxMintPerBlock,
    uint256 _maxRedeemPerBlock
  ) {
    if (address(_usde) == address(0)) revert InvalidUSDeAddress();
    if (_assets.length == 0) revert NoAssetsProvided();
-   if (_admin == address(0)) revert InvalidZeroAddress();
+   if (_admin == address(0) && msg.sender != admin) revert CUSTOM_ERROR();
    usde = _usde;

    _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);

    for (uint256 i = 0; i < _assets.length; i++) {
      addSupportedAsset(_assets[i]);
    }

    for (uint256 j = 0; j < _custodians.length; j++) {
      addCustodianAddress(_custodians[j]);
    }

    // Set the max mint/redeem limits per block
    _setMaxMintPerBlock(_maxMintPerBlock);
    _setMaxRedeemPerBlock(_maxRedeemPerBlock);

    if (msg.sender != _admin) {
      _grantRole(DEFAULT_ADMIN_ROLE, _admin);
    }

    _chainId = block.chainid;
    _domainSeparator = _computeDomainSeparator();

    emit USDeSet(address(_usde));
  }
```

---

## [NC-02] Write modifier for `if (msg.sender != minter) revert OnlyMinter();` check in the `USDe#mint()` function, so to be consistent with `USDe#setMinter()` function which performs the access control check in modifier

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L28-L31
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L23-L26

### Code:
```jsx
  function setMinter(address newMinter) external onlyOwner {
    emit MinterUpdated(newMinter, minter);
    minter = newMinter;
  }

  function mint(address to, uint256 amount) external {
    if (msg.sender != minter) revert OnlyMinter();
    _mint(to, amount);
  }
```

---

## [NC-03] Change the names of `removeFromBlacklist()` and `addToBlacklist()` functions

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L106-L127

### Summary:
The functions is not properly named, because the roles that are `granted/revoked` are `SOFT_RESTRICTED_STAKER_ROLE` and `FULL_RESTRICTED_STAKER_ROLE`.

---

## [NC-04] Failure to validate functions return values can result in errors

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L112
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L126

### Summary:
Within the `StakedUSDe` contract, there are functions named `addToBlacklist()` and `removeFromBlacklist()` that utilize the `_grantRole` and `_revokeRole` methods respectively. It is essential to validate the return values of these functions in both scenarios. This ensures that when a role is granted, it confirms the target's absence of the role previously, and when a role is revoked, it confirms that the target indeed held this role.

---

## [NC-05] Add check in `rescueTokens()` if the `to` parameter is `FULL_RESTRICTED_STAKER_ROLE` and if revert

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L138-L141

### Summary
> FULL_RESTRICTED_STAKER_ROLE - address with this role can't burn his stUSDe tokens to unstake his USDe tokens, neither to transfer stUSDe tokens. His balance can be manipulated by the admin of StakedUSDe.

Every transaction that contains some sort of token transferring, revert if participant is someone with `FULL_RESTRICTED_STAKER_ROLE`, expect `rescueTokens()` function that transfer tokens accidentally sent to the contract. As the result owner can mistakenly transfer tokens to address with `FULL_RESTRICTED_STAKER_ROLE` thinking that transfer tokens accidentally sent to the contract.

### Mitigation
Add check in `rescueTokens()` if the `to` parameter is `FULL_RESTRICTED_STAKER_ROLE` and if revert

---

## [NC-06] The System Exclusively Accepts Nonces

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol

### Summary:
Description: Presently, the nonce system implemented in `_deduplicateOrder()` restricts us to having distinct slots for a user. Each slot accommodates bits. Nevertheless, in the `_orderBitmaps` mapping, both key and value types are uint256, which implies that distinct nonces could unintentionally intersect, even when they shouldn't (e.g., and).

### Recommendation Migration Steps:
While it is improbable for nonces to reach such large values, we advise the team to address this concern, document the anticipated behavior in this situation, and implement any necessary corrections.

---

## [NC-07] When adding an asset to `supportedAssets`, perform a check to ensure that the supported asset is not the `NATIVE_TOKEN`. If it is, then revert

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L413-L433

### Summary:
This will improve the code readability, because of the checks related to native tokens (ethers) performed in `_transferCollateral()` will be no needed.

```jsx
  function _transferCollateral(
    uint256 amount,
    address asset,
    address benefactor,
    address[] calldata addresses,
    uint256[] calldata ratios
  ) internal {
    // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
    IERC20 token = IERC20(asset);
    uint256 totalTransferred = 0;
    for (uint256 i = 0; i < addresses.length; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }
    uint256 remainingBalance = amount - totalTransferred;
    if (remainingBalance > 0) {
      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
    }
  }
```

---

## [NC-08] Don't assign role on `msg.sender`

### GitHub Links:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L124C36-L124C46

### Summary
This a common smart contract security concern. In the specific context of Ethereum smart contracts, roles are used for access control: they designate which addresses have permission to perform certain actions, such as minting or burning tokens. When don't assign roles on `msg.sender` leads to a more secure system of role management to prevent the abuse of critical permissions in the smart contract environment. It's a call for stronger security measures and more thoughtful control distribution, especially for functions that can significantly impact the tokenomics or the participants' trust in the platform.

Overall - This is bad solution. Better use approach like in UniswapV3.

---

## [NC-09] It is [recommended](https://github.com/ethereum/eips/issues/155) to use deadline for [meta-transactions](https://medium.com/coinmonks/complete-guide-to-meta-transactions-c46ca51dbd21)
### GitHub Links
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol

---

## [NC-10] Lack of Event Emitting on Important for the Protocol functions

### Summary
The protocol currently lacks event emission in several crucial functions, which can hinder transparency and make tracking important actions challenging. Here are the instances where events are missing:
1. `USDe.sol#mint()`: The `mint()` function in the USDe contract does not emit an event when new tokens are minted, making it difficult to track and verify minting activities.
2. `EthenaMinting#disableMintRedeem()`: While the internal functions called within `disableMintRedeem()`, such as `_setMaxMintPerBlock()` and `_setMaxRedeemPerBlock()`, emit events for the new set values, the main goal of `disableMintRedeem()` differs. Adding an event specifically for `disableMintRedeem` would provide clarity and enhance visibility.
3. `StakedUSDe#_deposit()`: The `_deposit()` function in the StakedUSDe contract lacks event emission, making it challenging to monitor deposit activities.
4. `StakedUSDe#rescueTokens()`: The `rescueTokens()` function does not emit an event, potentially causing issues in tracking rescued tokens.
5. `removeFromBlacklist()`, `addToBlacklist()`: These functions in the protocol lack event emission, making it difficult to track changes to the blacklist.
6. For StakedUSDeV2: `withdraw()`, `redeem()`, `unstake()`: These functions do not emit events, which can hinder monitoring and auditing processes.

Adding appropriate events to these functions would greatly improve the protocol's transparency and auditability.

---
