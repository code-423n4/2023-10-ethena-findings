1.should use EIP-1271 Standard Signature Validation Method for Contracts Standard way to verify a signature when the account is a smart contract
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L342
Recommend to use  EIP-1271 Standard  to verify  a signature when the account is a smart contract

2.unnecessary bitwise or opration
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L394
If `invalidator` not it zero transaction must be revert in `verifyNonce` , thus  `invalidator` == 0 , The bitwise OR operation with 0 on any number always results in the same number, so the OR operation is unnecessary

3.unnecessary addition operation
If the result of `getUnvestedAmount` is bigger than zero the transaction must be revert. So blow addition operation is unnecessary result in waste of gas
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91