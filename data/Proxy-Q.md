# Low and Non-Critical

### [Low Risk](#low-risk-1)

| Total Low Risk Issues | 1 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [L-01](#l-01-wrong-accounting-in-stakedusde-in-variable-vestingamount) | Wrong accounting in `StakedUSDe` in variable `vestingAmount` | 1 |

### [Non-Critical](#non-critical-1)

| Total Non-Critical Issues | 1 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [NC-01](#nc-01-unnecessary-if-revert-statement) | Unnecessary if-revert statement | 4 |

## Low Risk

### [L-01] Wrong accounting in `StakedUSDe` contract in variable `vestingAmount`

According to the description of `vestingAmount` in the comments `vestingAmount` is:

[comment](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L40-L42)

```solidity
/// @notice The amount of the last asset distribution from the controller contract into this
/// contract + any unvested remainder at that time
uint256 public vestingAmount;
```

`vestingAmount` is updated in `transferInRewards()` but it will never be the input amount + any unvested remainder at that time, only the input amount.
This is because the line `if (getUnvestedAmount() > 0) revert StillVesting();` reverts if `getUnvestedAmount()` returns anything greater than 0.

[transferInRewards()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89-L99)

```solidity
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount(); // @audit `getUnvestedAmount()` will always return 0

    vestingAmount = newVestingAmount; // @audit `vestingAmount` will not be updated correctly
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
}
```

## Non-critical

### [NC-01] Unnecessary if-revert statement

**First two instances**

In `EthenaMinting` contract, functions `mint()` and `redeem()` use an unneccessary if-revert statement while calling `_deduplicateOrder()`

[mint()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L172)

[redeem()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L203)

```solidity
if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
```

The `_deduplicateOrder()` function only returns true or reverts before returning anything. Thus the `revert Duplicate()` will never be executed.

[_deduplicateOrder()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L391)

```solidity
function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
    (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
    return valid; // @audit returns true, otherwise reverts before returning anything
}
```

`verifyNonce()` function used in `_deduplicateOrder()`

[verifyNonce()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L377)

```solidity
function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
    if (nonce == 0) revert InvalidNonce();
    uint256 invalidatorSlot = uint64(nonce) >> 8;
    uint256 invalidatorBit = 1 << uint8(nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    uint256 invalidator = invalidatorStorage[invalidatorSlot];
    if (invalidator & invalidatorBit != 0) revert InvalidNonce();

    return (true, invalidatorSlot, invalidator, invalidatorBit);
}
```

**Second two instances**

In `EthenaMinting` contract, functions `transferToCustody()` and `verifyRoute()` don't need to check for `address(0)`.

[transferToCustody()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L248)

```solidity
if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();
```

[verifyRoute()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L364)

```solidity
if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
```

Because these functions only allow usage of `_custodianAddresses` there is no need to check for `address(0)` because the function `addCustodianAddress()` that is used to add addresses to the variable already checks for `address(0)`

[addCustodianAddress()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L297-L303)

```solidity
/// @notice Adds an custodian to the supported custodians list.
function addCustodianAddress(address custodian) public onlyRole(DEFAULT_ADMIN_ROLE) {
    if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
      revert InvalidCustodianAddress();
    }
    emit CustodianAddressAdded(custodian);
}
```