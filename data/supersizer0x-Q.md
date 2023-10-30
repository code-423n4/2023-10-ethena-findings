## Low findings 
###### If an attacker can figure out a user's signature then he can sign the same data causing dos for the users since the order dosnt match 
an attacker might be able to frontrun the user  order and cause the signer to be a diffrent address and it will revert in [`verifyOrder`](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L341)
The protocol should use flashbots
###### a user can take an  approved order then switch the order details and get free funds
when the user's order is approved he can switch the signature details and if the backend dosnt look for the tx nonce then an attacker will be able to use new order details with unfair advantages
example:
1. attacker makes an order for 100 usde for 1 stETH
2. The server approves of the trade 
3. attacker frontrund for 1000 usde for 1 stETH 
```solidity
   address signer = ECDSA.recover( 
            taker_order_hash,
            signature.signature_bytes
```
[`verifyOrder`](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L341)
make sure the server checks the tx nonce because its not checked in the code 
###### validator might be able to get out of `MintPerBlock` by having a double block
since the validator has control over 2 consective blocks he  can steal more funds.
ex:
Alice the validator steals 100k maxMint limit 
On the second block Alice steals another 100k maxMint limit and exludes the gateKeeper transaction
Alice might even be able to beat out the gateKeeper transaction by having providing a high gas fee  in the third block
Make sure to know this risk,their is not such a good fix but its unlikey for this to happen and to be executed!
###### wallet can be malicious and cause a lot of gas 
The same attack as the one highlighted  in the bot report but in a diffrent location and less severe

```solidity
(bool success, ) = wallet.call{value: amount}(""); 
```
Make sure to use a lower level call with out returndata
[`wallet.call`](
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L250)