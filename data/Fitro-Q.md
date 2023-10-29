# EthenaMinting.sol incorrect error in verifyOrder()

In the **`verifyOrder()`** there is a condition that checks whether the **order.beneficiary** address is a zero address. If this condition evaluates to true, the code currently reverts with a custom error **`InvalidAmount()`**. This is incorrect, the correct error would be **`InvalidZeroAddress()`**.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L343

```Solidity
function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
 >  if (order.beneficiary == address(0)) revert InvalidAmount();  
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
    if (block.timestamp > order.expiry) revert SignatureExpired();
    return (true, taker_order_hash);
  }
```

Change the cosutom error **`InvalidAmount()`** for **`InvalidZeroAddress()`**.