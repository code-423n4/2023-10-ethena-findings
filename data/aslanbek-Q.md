# [L-01] EthenaMinting#verify - unnecessary downcasting

[EthenaMinting.sol#L379](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L379)

Downcasting to uint64 is absolutely not needed. It limits the space of nonces from 2^256 to 2^64 (which is unlikely to matter) and uses more gas.

## Recommendation 
```diff
- uint256 invalidatorSlot = uint64(nonce) >> 8;
+ uint256 invalidatorSlot = uint64(nonce) >> 8;
```

