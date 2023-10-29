# QA Report

## **Table of Contents**

|                                                                                            | Issue                                                                          |
| ------------------------------------------------------------------------------------------ | :----------------------------------------------------------------------------- |
| [QA‑01](#qa-01-verifyorder-currently-accepts-an-expired-signature-and-emits-a-wrong-event) | `verifyOrder()` currently accepts an expired signature and emits a wrong event |
| [QA‑02](#qa-02-the-soft_restricted_staker_role-is-a-legal-case-in-the-making)              | The `SOFT_RESTRICTED_STAKER_ROLE` is a legal case in the making                |
| [QA‑03](#qa-03-max-mint-per-block-could-actually-be-exceeded)                              | Max mint per block could actually be exceeded                                  |
| [QA‑04](#qa-04-_transfercollateral-should-be-made-more-efficient)                          | `_transferCollateral()` should be made more efficient                          |
| [QA‑05](#qa-05-setters-should-always-have-equality-checkers)                               | Setters should always have equality checkers                                   |
| [QA‑06](#qa-06-potential-role-oversight-while-depositing)                                  | Potential role oversight while depositing                                      |

## QA-01 `verifyOrder()` currently accepts an expired signature and emits a wrong event

### Impact

No inclusive checks means the order verification process currently accepts an already expired signature.

### Proof of Concept

Take a look at `verifyOrder()`

```solidity
  /// @notice assert validity of signed order
  function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    if (order.beneficiary == address(0)) revert InvalidAmount();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
  //@audit-issue
    if (block.timestamp > order.expiry) revert SignatureExpired();
    return (true, taker_order_hash);
  }
```

The current condition only checks if the block timestamp has surpassed the order's expiry. This means even if the order expires it would be validated the check would not consider the signature expired, when in fact it should be.

### Recommended Mitigation Steps

Make these changes to `verifyOrder()`

```diff
  /// @notice assert validity of signed order
  function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
-    if (order.beneficiary == address(0)) revert InvalidAmount();
+    if (order.beneficiary == address(0)) revert InvalidAddresst();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
-    if (block.timestamp > order.expiry) revert SignatureExpired();
+    if (block.timestamp >= order.expiry) revert SignatureExpired();
    return (true, taker_order_hash);
  }
```

## QA-02 The `SOFT_RESTRICTED_STAKER_ROLE` is a legal case in the making

### Impact

The current configuration of the `SOFT_RESTRICTED_STAKER_ROLE` allows addresses with this role to circumvent deposit restrictions by transferring their `USDe` tokens to another address they control. This could easily lead to a legal case since one can link these transfers to each other and if restrictions were to be placed on a user from a specific country and they are not fixed could easily land the team a lawsuit.

### Proof Of Concept

The below has been stated under `Known Issues`

> - `SOFT_RESTRICTED_STAKER_ROLE` can be bypassed by user buying/selling stUSDe on the open market
>   But that's not the only case as there are other methods one could by pass this

1. Assign an address the `SOFT_RESTRICTED_STAKER_ROLE`.
2. Attempt a direct deposit with this address, which should fail due to restrictions.
3. Transfer `USDe` tokens from this address (with `SOFT_RESTRICTED_STAKER_ROLE`) to another address owned by the same entity.
4. Use the secondary address to deposit the transferred `USDe` tokens. This operation will succeed, demonstrating the loophole.

### Recommended Mitigation Steps

Review and possibly refine the permissions associated with the `SOFT_RESTRICTED_STAKER_ROLE` to ensure that they align with the intended functionality, since this could lead to legal issues, on one hand I would suggest to just scrap it and instead have only the `FULL_RESTRICTED_STAKER_ROLE`

## QA-03 Max mint per block could actually be exceeded

### Impact

> NB: First this is only submittted as a QA since it's been listed to be a main invariant, but since it requires admin mistake it sits on the fence of med/QA

### Proof Of Concept

As stated under the `Main Invariants` section of the contest's readMe

> Max mint per block should never be exceeded.
> But current code implementation does not follow this based on two reasons:

- If the admin calls `setMaxMintPerBlock()` it sets the new max mint for that specific block and any block that comes after it, this can be seen below:

```solidity
  function setMaxMintPerBlock(uint256 _maxMintPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _setMaxMintPerBlock(_maxMintPerBlock);
  }

  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
    uint256 oldMaxMintPerBlock = maxMintPerBlock;
    maxMintPerBlock = _maxMintPerBlock;
    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
  }
```

The above easily means that for the current block the max mint could always be altered

### Recommended Mitigation Steps

To ensure that more than max is not minted then there is a need to introduce timing to the process and ensure that the current max mint can't be changed for the current timestamp.
Got it. Here's the revised report:

## QA-04 `_transferCollateral()` should be made more efficient

### Impact

Low, since at most this could only be `1 wei` but might still lead to issues regarding `ratio` if this ever amounts to a large number.

### Proof of Concept

Take a look at `_transferCollateral()`:

```solidity
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
    //@audit
    if (remainingBalance > 0) {
      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
    }
  }
```

As seen, the `_transferCollateral` function, designed to transfer supported assets to a range of custody addresses based on defined ratios, but it has an inefficiency in how it handles the remaining balance after all the transfers.

As seen, if there is a remaining balance, it is sent to the last address in the `addresses` array:

```solidity
if (remainingBalance > 0) {
  token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
}
```

This approach assumes that the last address is always the intended recipient of any remaining balance. It doesn't provide flexibility and can cause unexpected results if, for example, the `addresses` array order changes, or this amounts to a considerable sum for that specific `address` leading to a break in the ratio logic.

### Recommended Mitigation Steps

Allow specification of which address among the `addresses` array should receive any remaining balance.

## QA-04 Setters should always have equality checkers

### Impact

The `transferAdmin` function in its current state does not validate if the `newAdmin` being set is different from the `_currentDefaultAdmin`. This oversight can lead to unnecessary operations where the admin role is "transferred" to the same address that already possesses it, wasting gas and potentially causing confusion among the stakeholders.

### Proof Of Concept

Consider the function:

```solidity
  //@audit setters equality check
  function transferAdmin(address newAdmin) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (newAdmin == msg.sender) revert InvalidAdminChange();
    _pendingDefaultAdmin = newAdmin;
    emit AdminTransferRequested(_currentDefaultAdmin, newAdmin);
  }
```

Here's the sequence that demonstrates the gap:

1. The current admin (`_currentDefaultAdmin`) calls the `transferAdmin` function with their own address as the `newAdmin`.
2. The function only checks if the `newAdmin` address is the same as the `msg.sender`.
3. No checks are in place to determine if `newAdmin` is the same as `_currentDefaultAdmin`.
4. As a result, the operation proceeds even if the `newAdmin` is the same as `_currentDefaultAdmin`.

### Recommended Mitigation Steps

Modify the function to include a check comparing `newAdmin` to `_currentDefaultAdmin`. If they are the same, the function should revert the transaction.
`solidity
    if (newAdmin == _currentDefaultAdmin) revert RedundantAdminChange();
    `

## QA-05 Potential role oversight while depositing

### Impact

While depositing there seem to be a check for users who have been assigned the `SOFT_RESTRICTED_STAKER_ROLE` but no consideration is applied for users with the `FULL_RESTRICTED_STAKER_ROLE`.

### Proof of Concept

The following check is performed to prevent users with the `SOFT_RESTRICTED_STAKER_ROLE` from using the function:

```solidity
if (hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, receiver)) {
  revert OperationNotAllowed();
}
```

However, no check is done for users with the `FULL_RESTRICTED_STAKER_ROLE`

### Recommended Mitigation Steps

Introduce a check for the `FULL_RESTRICTED_STAKER_ROLE`
