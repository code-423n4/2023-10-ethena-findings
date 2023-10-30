# QA Report

## Low risk: **`orderType`** check in **`EthenaMinting.verifyRoute`** is either wrong or redundant
There is an if statement in the **`verifyRoute`** function that checks whether the **`orderType`** is **`REDEEM`** and if it is, it returns **`true`** (valid). There's two possible order types: **`REDEEM`** and **`MINT`**. This function is only supposed to be used in the **`mint`** function, hence only an order type of **`MINT`** should be valid. The check should either return **`false`** so the verification fails, or if it's intended, then it is completely redundant and could be removed altogether as the default return type is already **`true`**.

## Low risk: User can accidentally send ether to **`EthenaMinting`** contract
An user can accidentally send ether to the **`EthenaMinting`** contract and lock it within the contract. Currently, the only way to get the ether out is for the **`MINTER`** to transfer the ether to one of the custodian wallets and return it to the user from there. Alternatively, a withdrawal function can be implemented where the users can take the ether out themselves. This would also require changing the **`receive`** handler to persist the data to a storage mapping (currently it only emits an event).

