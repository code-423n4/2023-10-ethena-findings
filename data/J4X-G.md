## [GAS-01] Unnecessary call to `getUnvestedAmount()`
[StakedUSDe Line 91](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91)

**Issue Description:**
The call to `getUnvestedAmount()` is redundant because the if statement above it already checks that `getUnvestedAmount()` returns 0.

**Recommended Mitigation:**
Remove the call to `getUnvestedAmount()`.

```diff
uint256 newVestingAmount = amount;
```

---
## [GAS-02] Deduplication can be greatly simplified
[EthenaMinting Line 391](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L391)

**Issue Description:**
The `verifyNonce()` function always returns a true boolean value if it does not revert, which results in unnecessary gas consumption. This value is also returned by `_deduplicateOrder()`, which likewise always returns true if it does not revert. The return value from these functions does not impact the code.

**Recommended Mitigation:**

Adjust the `verifyNonce()` function to remove the unnecessary return value as shown below:

```solidity
function verifyNonce(address sender, uint256 nonce) public view override returns (uint256, uint256, uint256) {
	if (nonce == 0) revert InvalidNonce();
	uint256 invalidatorSlot = uint64(nonce) >> 8;
	uint256 invalidatorBit = 1 << uint8(nonce);
	mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
	uint256 invalidator = invalidatorStorage[invalidatorSlot];
	if (invalidator & invalidatorBit != 0) revert InvalidNonce();
	  
	return (invalidatorSlot, invalidator, invalidatorBit);
}
```

Additionally, remove the return value from `verifyNonce()` and the return value from the `_deduplicateOrder()` function, as illustrated below:

```solidity
function _deduplicateOrder(address sender, uint256 nonce) private {
	(uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
	mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
	invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
}
```

Finally, replace the `if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();` statements in `redeem()` and `mint()` with a simple call to `_deduplicateOrder()`. The custom error `Duplicate()` is no longer necessary and can be removed. If the nonce has already been used, the call will always revert with `InvalidNonce`.

---
## [GAS-03] `verifyOrder()` can be simplified
[EthenaMinting Line 339](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L339)

**Issue Description:**
The function `verifyOrder()` is called in `mint()` and `redeem`, but the return values are not utilized at all. If the function is intended only for internal use, the return values can be removed. If the function is meant for external users to verify their own orders, a boolean return value is sufficient.

**Recommended Mitigation:**
Either remove the entire return functionality if it's intended only for internal use, or if it's intended for external users to verify their own orders, simplify the function to return only a boolean value, thus saving gas.

---
## [GAS-04] Full functionality of `getUnvestedAmount()` is not always needed
[StakedUSDe Line 91](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91)

**Issue Description:**
In the call to `getUnvestedAmount()` in `transferInRewards()`, it is only necessary to check if the vesting period has already passed since the last transfer, not to retrieve the full functionality of `getUnvestedAmount()`. This can be inlined to reduce gas costs.

**Recommended Mitigation:**
Modify the if clause as follows: `if (block.timestamp - lastDistributionTimestamp < VESTING_PERIOD) revert StillVesting();`.

---
## [GAS-05] Modifier `onlyStakingVault` can be inlined
[USDeSilo Line 23](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L23)

**Issue Description:**
The `USDeSilo.sol` contract employs the modifier `onlyStakingVault` only once, which can be inlined to simplify the code.

**Recommended Mitigation:**
To address this issue, directly integrate the require statement into the `withdraw()` function and remove the modifier. The updated function should appear as follows:

```solidity
function withdraw(address to, uint256 amount) external {
	if (msg.sender != STAKING_VAULT) revert OnlyStakingVault();
	USDE.transfer(to, amount);
}
```

---
## [GAS-06] Order of ifs can be changed to optimize `verifyRoute()`
[EthenaMinting Line 360](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L360)

**Issue Description:**
In the `verifyRoute()` function, the if clauses first check for the length of both arrays being equal before checking if there are any addresses. The order of these checks can be altered to check the length equality only if there are any addresses, potentially saving gas.

**Recommended Mitigation:**
Change the order of the function calls as follows:

```solidity
function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
	// routes only used to mint
	if (orderType == OrderType.REDEEM) {
		return true;
	}
	
	uint256 totalRatio = 0;
	
	if (route.addresses.length == 0) {
		return false;
	}
	if (route.addresses.length != route.ratios.length) {
		return false;
	}
	...
}
```

---
## [GAS-07] Remove Unnecessary Check in `verifyRoute()`
[EthenaMinting Line 364](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L364)

**Issue Description:**
In the `verifyRoute()` function, the check for `route.addresses[i] == address(0)` is not required because it is already verified by checking `!_custodianAddresses.contains(route.addresses[i])`. This is because `address(0)` can never be added as a custodian, as confirmed by the check `if (custodian == address(0) || ...) revert InvalidCustodianAddress();` in `addCustodianAddress()`.

**Recommended Mitigation:**
Remove the check, adapting the requirements to:

```solidity
if (!_custodianAddresses.contains(route.addresses[i]) || route.ratios[i] == 0)
```
