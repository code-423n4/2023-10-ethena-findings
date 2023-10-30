QA-1. Incase of a compromised minter the minter can front-run  ```disableMintRedeem``` 
      function and still call redeem or mint the by monitoring the mempool.

QA-2. The [```verifyRoute```](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L353C5-L354C19) function will return true if the order type is redeem.
      A user trying to verify his route addresses and ratios will always get true if 
      the order type is set to redeem.
      This makes a false positive assumption to the user where the user might 
      actually sign a mint order with the order type as redeem but this will have no 
      effect since the real transaction will be submitted by the minter through the 
      ```mint``` function which checks ```if (order.order_type != OrderType.MINT) revert InvalidOrder();```. But this makes room for unnecessary reverts. 
      Consider returning false if the order type is redeem or remove 
```
 if (orderType == OrderType.REDEEM) {
      return true;
    }
``` 
from the function.
The best would be to return a custom error message if the order type is redeem.

QA-3. Wrong error passed in the [```verifyOrder```](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L343C5-L343C65) function in **EthenaMinting.sol** contract. Use ```InvalidAddress()``` instead.

QA-4. Will never likely be a problem but there is no restriction that the new owner(admin) is not a blacklisted address in the ```transferAdmin``` function in **SingleAdminAccessControl**.
This is reported because as we can see in the ```addToBlacklist``` and ```removeFromBlacklist``` functions there is a ```notOwner(target)``` modifier so the protocol clearly doesn't want the owner to be a blacklisted address. 
But if this is not to be fixed then consider removing the notOwner modifier in the ```removeBlacklist``` function so that the owner can be removed from blacklisted addresses.
 