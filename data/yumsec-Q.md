# [QA-01] `EthenaMinting.sol` function `verifyOrder` emits wrong event

## Lines of code

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L343

## Proof of Concept

In the file `EthenaMinting.sol`, function `verifyOrder`, `if (order.beneficiary == address(0))`, contract emits the wrong event `InvalidAmount()`, instead it should emit `InvalidAddress()`.
```
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

## Tools Used

Manual analysis.

## Recommended Mitigation Steps
Emit right event `InvalidAddress()`.

```
function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    if (order.beneficiary == address(0)) revert InvalidAddress();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
    if (block.timestamp > order.expiry) revert SignatureExpired();
    return (true, taker_order_hash);
}
```

# [QA-02] `EthenaMinting.sol` function `belowMaxMintPerBlock` and `belowMaxRedeemPerBlock` have overflow risk

## Lines of code

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L98
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L105

## Proof of Concept

In the file `EthenaMinting.sol`, function `belowMaxMintPerBlock` and `belowMaxRedeemPerBlock` have overflow risk. If someone tries to mint large amount of token, with `mintedPerBlock[block.number] + mintAmount > max_uint256`, the function will revert with Reason: Arithmetic over/underflow, such vague error message will confuse users and make it hard to debug.

```
modifier belowMaxMintPerBlock(uint256 mintAmount) {
    if (mintedPerBlock[block.number] + mintAmount > maxMintPerBlock) revert MaxMintPerBlockExceeded();
    _;
}


modifier belowMaxRedeemPerBlock(uint256 redeemAmount) {
    if (redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();
    _;
}
```

## Tools Used

Manual analysis.

## Recommended Mitigation Steps

Avoid math overflow by leveraging subtraction.

```
modifier belowMaxMintPerBlock(uint256 mintAmount) {
    if (mintAmount > maxMintPerBlock - mintedPerBlock[block.number]) revert MaxMintPerBlockExceeded();
    _;
}


modifier belowMaxRedeemPerBlock(uint256 redeemAmount) {
    if (redeemAmount > maxRedeemPerBlock - redeemedPerBlock[block.number]) revert MaxRedeemPerBlockExceeded();
    _;
}
```

dddd