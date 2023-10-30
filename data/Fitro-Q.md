# [01] EthenaMinting.sol incorrect error in verifyOrder()

In the **`verifyOrder()`** there is a condition that checks whether the **order.beneficiary** address is a zero address. If this condition evaluates to true, the code currently reverts with a custom error **`InvalidAmount()`**. This is incorrect, the correct error would be **`InvalidZeroAddress()`**.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L343

```Solidity
function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    if (order.beneficiary == address(0)) revert InvalidAmount();  
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
    if (block.timestamp > order.expiry) revert SignatureExpired();
    return (true, taker_order_hash);
  }
```

Change the cosutom error **`InvalidAmount()`** for **`InvalidZeroAddress()`**.

# [02] EthenaMinting.sol encodeRoute() is not used

**`encodeRoute()`** is never used in the protocol and can be removed.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L334-L336

```Soldity
function encodeRoute(Route calldata route) public pure returns (bytes memory) {
    return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);
  }
```

# [03] StakedUSDe.sol transferInRewards() use getUnvestedAmount() to calculate the newVestingAmount but always returns 0.

**`transferInRewards()`** is used to transfer rewards from the controller contract to StakeUSDe.sol. Within this function, there is a calculation for **newVestingAmount = amount + getUnvestedAmount()**.
**`getUnvestedAmount()`** always will returns 0 because if not the condition before can never be passed (**if (getUnvestedAmount() > 0) revert StillVesting()**). This is because **`getUnvestedAmount()`** only returns 0 when the vesting period is completed.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L173-L181

```Solidity
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
```Solidity
 function getUnvestedAmount() public view returns (uint256) {
    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

    if (timeSinceLastDistribution >= VESTING_PERIOD) {
      return 0;
    }

    return ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) / VESTING_PERIOD;
  }
```
Remove **`getUnvestedAmount()`** in the calcualtion of **newVestingAmount**  because always returns 0.


