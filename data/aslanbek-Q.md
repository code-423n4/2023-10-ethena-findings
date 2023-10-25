[L-01] verifyNonces may revert for valid nonces
Due to downcasting to uint64, nonces with the same % 2^64 will use the same slot in the mapping
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L379