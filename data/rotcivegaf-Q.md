# Ethenea QA report

## `verifyNonce` first param returns always `true`

The boolean value returned by the function `verifyNonce` is returned by (`_deduplicateOrder`)[https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L392] function and finally used by [`mint`](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L172) and [`redeem`](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L203) functions

The value is always `true` so the call it's never revert with `Duplicate()` error

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L385

## `verifyOrder` first param returns always `true`

The boolean value returned by the function `verifyOrder` is always `true` and is not used by the contract, I recommend remove this returned value and returning only `taker_order_hash`

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L347

## Shadow variable `owner` inherit from `SingleAdminAccessControl`

`owner` param variable is shadowing `owner()` inherit from `SingleAdminAccessControl`

- [StakedUSDeV2.sol#L42](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L42)
- [StakedUSDeV2.sol#L52](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L52)
- [StakedUSDeV2.sol#L62](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L62)
- [StakedUSDeV2.sol#L95](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L95)
- [StakedUSDeV2.sol#L111](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L111)

Recommendation: use `_owner` instead of `owner` to avoid collissions as you do on for example [StakedUSDe.sol#L225](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L225)

## Unused errors

- `error InvalidAffirmedAmount` in [IEthenaMinting.sol#L47](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IEthenaMinting.sol#L47)
- `error SlippageExceeded();` on [IStakedUSDe.sol#L18](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IStakedUSDe.sol#L18)

Recommendation: remove or use this errors

## Unused import
`SafeERC20.sol` is imported but not used:
- [USDeSilo.sol#L5](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L5)

Recommendation: remove unused import

## Unused events

- `CustodyWalletAdded` on [IEthenaMintingEvents.sol#L29](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IEthenaMintingEvents.sol#L29)
- `CustodyWalletRemoved` on [IEthenaMintingEvents.sol#L29](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IEthenaMintingEvents.sol#L32)
- `SiloUpdated` on [IStakedUSDeCooldown.sol#L17](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IStakedUSDeCooldown.sol#L17)


## Critical functions dont emit events
Critical functions should emit events

- `setMaxMintPerBlock` on [EthenaMinting.sol#L219](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L219)
- `setMaxRedeemPerBlock` on [EthenaMinting.sol#L224](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L224)
- `disableMintRedeem` on [EthenaMinting.sol#L229](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L229)

Recommendation: emit events on critical functions

## EIP712 is not correctly implemented for the Route struct

As stated in the EIP712 standard - "The array values are encoded as the keccak256 hash of the concatenated encodeData of their contents". This is not correctly followed in EthenaMinting::encodeRoute as it does not do this for the Route struct array fields. While this is usually a more serious problem, the method is not called in the protocol and can just be removed.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L334

## Nonce could overflow

Nonce can overflow and even if do a `safeCast` only con be used 256

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L379-L380