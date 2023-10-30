# Non-critical
1. Parameters and struct members names not mixedCase

```solidity
File: EthenaMinting.sol
36:    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address
```
| [Line #36](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L36) | 
```solidity
File: IEthenaMinting.sol

20:    SignatureType signature_type;
21:    bytes signature_bytes;
30:    OrderType order_type;
35:    address collateral_asset;
36:    uint256 collateral_amount;
37:    uint256 usde_amount;
63:  function verifyRoute(Route calldata route, OrderType order_type) external view returns (bool);
```
| [Line #20](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L20) | [Line #21](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L21) | [Line #30](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L30) | [Line #35](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L35) | [Line #36](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L36) | [Line #37](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L37) | [Line #63](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L63) | 