### Low 01 - Unnecessary math operation and local variable declaration
In StakedUSDe.sol - `transferInRewards()`, adding zero math operation is performed and the result is assigned to a new local variable. 

```solidity
// contracts/StakedUSDe.sol
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
|>  uint256 newVestingAmount = amount + getUnvestedAmount();
    vestingAmount = newVestingAmount;
...
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L91)

As seen above, `getUnvesteAmount()` would have to return zero to pass the `if` statement, but when `getUnvestedAmount()` is zero, there is no need to add zero to `amount` and delacring `newVestingAmount` which is the same value as `amount`.

Recommendation:
Since `if` statement would ensure `getUnvestedAmount()` return zero, directly assign `vestingAmount = amount`.

### Low 02 - Emitting duplicated values is unnecessary.
In StakedUSDe.sol - `transferInRewards()`, `emit RewardsReceived(amount, newVestingAmount)` emits two variables `amount` and `newVestingAmount`. However, `amount` and `newVestingAmount` will always be the same value.

```solidity
// contracts/StakedUSDe.sol
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();
...
|>  emit RewardsReceived(amount, newVestingAmount);
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L98C1-L98C52)

As seen above, when `if` statement passes, `getUnvestedAmount()` returns zero, which means `newVestingAmount` will equal `amount`. And `RewardsReceived` will always emit the same value twice, which is unnecessary.

Recommendation:
(1) Change to `emit RewardsReceived(amount)`;
(2) Or if the intention is to allow adding previously unvested amount to amount, then remove `if` statement.

### Low 03 - Balance redistribution may cause loss of funds
In StakedUSDe.sol - `redistributeLockedAmount()`, when the input `to` argument is `address(0)`, the function will not correctly handle it. The function will not revert but will continue to burn tokens causing permanent loss of funds. And base on the function doc, the intended behavior should be funds transfer, not permanent burning of funds on any occasion.

```solidity
// contracts/StakedUSDe.sol
  /**
   * @dev Burns the full restricted user amount and mints to the desired owner address.
   * @param from The address to burn the entire balance, with the FULL_RESTRICTED_STAKER_ROLE
   * @param to The address to mint the entire balance of "from" parameter.
   */
  function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
      uint256 amountToDistribute = balanceOf(from);
      _burn(from, amountToDistribute);
|>    if (to != address(0)) _mint(to, amountToDistribute);
      emit LockedAmountRedistributed(from, to, amountToDistribute);
    } else {
      revert OperationNotAllowed();
    }
  }
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L153)

As seen above, when the admin redistributes funds from a fully restricted staker to another address, when `to` is incorrectly set as zero, the function will not revert, instead it will simply burn the entire balance. 

Recommendations:
Revert when `to` is `address(0)`.

### Low 04 - The current cool-down mechanism is problematic - might cause user's assets to be locked longer than expected.

In StakedUSDeV2.sol - `cooldownAssets()`, users can withdraw their deposited assets to USDeSilo.sol where assets will be locked for a set cool-down period. However, currently implementation might allow unexpected longer or shorter cool-down period under certain conditions.

Suppose cool-down period is 90 days: When a user initiates `cooldownAssets()` with 100 ether first. Then 60 days later, the user initiates another `cooldownAssets()` for another 50 ether. However, user's total 150 ether redeemed assets are now locked for another 90 days. Their deposited 100 ether is locked for 150 days, and this is not the promised 90-day cool-down;

```solidity
// contracts/StakedUSDeV2.sol
  function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {
    if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();
    uint256 shares = previewWithdraw(assets);
|>  cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
|>  cooldowns[owner].underlyingAmount += assets;
    _withdraw(_msgSender(), address(silo), owner, assets, shares);
    return shares;
  }
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L100-L101)

```solidity
// contracts/StakedUSDeV2.sol
  function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
    if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();
    uint256 assets = previewRedeem(shares);
|>  cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
|>  cooldowns[owner].underlyingAmount += assets;
    _withdraw(_msgSender(), address(silo), owner, assets, shares);
    return assets;
  }
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L116-L117)

As seen above, whenever a user calls `cooldownAssets()` or `cooldownShares()`, their assets will be directly added to the existing `underlyingAmount` and their `cooldownEnd` time will be updated to a new timestamp without checking whether the user has an existing `cooldownEnd`.

Recommendations:
(1)Either only allow one time withdraw per cooldown period and revert when user's `cooldownEnd` is non-zero;
(2)Or if asset is allowed to be addded before an existing cooldown, consider use a mapping to track `cooldownEnd` key with the corresponding `underlyingAmount`.

### Low 05 - No on-chain check to ensure mint, redeem will always submit a benefactor's nonce in an incrementing manner

In EthenaMinting.sol - `mint()` and `redeem()`, only `MINTER_ROLE` and `REDEEMER_ROLE` can submit `mint()` and `redeem()` order. Trust has to be placed on `MINTER_ROLE` and `REDEEMER_ROLE` to always submit the user order nonce correctly. However, if `MINTER_ROLE` and `REDEEMER_ROLE` submit an incorrect nonce (e.g. skipping or jumping numbers) there is no way on-chain process will detect it. 

This could happen either by mistake or when `MINTER_ROLE` or `REDEEMER_ROLE` key is compromised. Malicious `MINTER_ROLE` or `REDEEMER_ROLE` could submit random future nonces which might cause a user unable to `mint()` or `redeem()` in the future due to nonce collision.

```solidity
// contracts/EthenaMinting.sol
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
|>  if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
...
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L172)
```solidity
// contracts/EthenaMinting.sol
  function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
|>  (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
    return valid;
  }
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L391-L395)
```solidity
// contracts/EthenaMinting.sol
  function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
...
    uint256 invalidatorSlot = uint64(nonce) >> 8;
    uint256 invalidatorBit = 1 << uint8(nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    uint256 invalidator = invalidatorStorage[invalidatorSlot];
|>  if (invalidator & invalidatorBit != 0) revert InvalidNonce();
...
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L383)

As seen above, in `mint()` or `redeem()`, `verifyNonce()` will run every time to make sure the `nonce` submitted by `MINTER_ROLE` or `REDEEMER_ROLE` is not clashing with an existing nonce. However, it didn't check whether the `nonce` is incrementing by 1 from the previous `nonce`. The lack of this on-chain checks, allows room for jumping or skipping nonce to go through whenever the situation allows, which will cause future `mint()` or `redeem()` orders to revert causing DOS of a certain user's `mint()` or `redeem()` and user collaterals can be locked permanently.

Recommendations:
Add additional checks in `verifyNonce()` to ensure a user's nonce always increments by 1.


