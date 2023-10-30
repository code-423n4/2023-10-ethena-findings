## 1. REDUNDANT `openzeppelin ERC20.sol` CONTRACT IMPORT CAN BE OMITTED IN THE `USDe.sol` CONTRACT

The `USDe.sol` contract imports the `openzeppelin ERC20` contract to the `USDe.sol` contract as shown below:

```solidity
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

But the `USDe` contract does not directly inherit from the `ERC20.sol` contract but it is implicitly imported from the `ERC20Burnable and ERC20Permit` contracts From which the `USDe` contract inherits from.

Hence it is recommended to remove the redundant `ERC20.sol` import.

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L4
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L15

## 2. THE RETURN VALUE OF THE `_grantRole` DURING `DEFAULT_ADMIN_ROLE` ASSIGNMENT IS NOT CHECKED FOR SUCCESS, IN THE `EthenaMinting.constructor` FUNCTION

The `EthenaMinting.constructor` function sets the `DEFAULT_ADMIN_ROLE` role to the deployer of the `EthenaMinting` contract as shown below:

    _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);

The `_grantRole` is inherited from the `openzeppelin AccessControl.sol` contract and the function returns a `bool` value to indicate the success of the transaction. But this return value is not checked in the `EthenaMinting.constructor` to ensure that the `msg.sender` is succesfulyy assigned the `DEFAULT_ADMIN_ROLE`.

Hence it is recommended to modify the above `_grantRole` function call as shown below to check for the `return` value.

    bool success = _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
    require(success, "Role assignment failed");

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L124
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L139
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L79-L80

## 3. THE RETURN BOOLEAN VALUE OF THE `_revokeRole` FUNCTION CALL IS NOT CHECKED FOR SUCCESS IN THE `EthenaMinting.removeMinterRole` AND `EthenaMinting.removeRedeemerRole` FUNCTIONS

The `EthenaMinting.removeMinterRole` function and the `EthenaMinting.removeRedeemerRole` functions are used to revoke the `MINTER_ROLE` and `REDEEMER_ROLE` respectively from the addresses passed in by the `GATEKEEPER_ROLE`. 

The revoking of the roles are executed as shown below:

    _revokeRole(MINTER_ROLE, minter);

    _revokeRole(REDEEMER_ROLE, redeemer);

The issue here is that the `openzeppelin AccessControl._revokeRole` function returns a bool value which is not checked for success inside the `removeMinterRole` function or `removeRedeemerRole` function. 

Hence even if the `_revokeRole` function call is not executed succesfully the `removeMinterRole` function and `removeRedeemerRole` function will proceed exeuction thinking that the respective roles have been revoked for the passed in addresses.

Hence it is recommended to check the return boolean values of the `removeMinterRole` and `removeRedeemerRole` functions and revert the transaction if the return boolean value is `false`.

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L277-L279
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L283-L285

## 4. UNRELATED ERROR MESSAGE IS GENERATED FOR THE `order.beneficiary == address(0)` CONDITION IN THE `EthenaMinting.verifyOrder` FUNCTION

The `EthenaMinting.verifyOrder` function is used to verify the order validity for a respective order. In this function there is a conditinal check to ensure that the `order.beneficiary != address(0)` as shown below:

    if (order.beneficiary == address(0)) revert InvalidAmount();

Here if the `beneficiary` address is equal to the `address(0)` the transaction will revert with the `InvalidAmount()` error message. But this error message does not describe the actual root cause for the revert. 

Hence it is recommendedt to create a new error as `error InvalidBeneficiary()` and generate this error when the `order.beneficiary == address(0)` condition occurs. 

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L343

## 5. THE RETURN VALUE OF THE `transfer` FUNCTION IS NOT CHECKED IN THE `USDeSilo.withdraw` FUNCTION

The `USDeSilo.withdraw` function is used to withdraw the `USDe` tokens once the cool down period is over. The `USDeSilo.withdraw` function implementation is shown below:

```solidity
  function withdraw(address to, uint256 amount) external onlyStakingVault {
    USDE.transfer(to, amount);
  }
```

As it is shown above the `return` value of the `transfer` function is not checked for `success` during the `withdraw` function execution. And even though the openzeppeling `SafeERC20 library` is imported into the `USDe` the `safeTransfer` is not used in the `USDeSilo.withdraw` function.

As a result the `USDeSilo.withdraw` transaction will be considered successful even if the claimed underlying assets are not correctly received by the receiving party.

Hence it is recommended to either check the return value of the `transfer` function for `success == true` condition or to use the `safeTransfer` function in the `USDeSilo.withdraw` function. 

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L28-L30

## 6. CRITICAL FUNCTIONS IN THE `EthenaMinting.sol` DO NOT EMIT EVENTS AFTER THIER SUCCESFUL TRNASACTION EXECUTION

The critical functions in the `EthenaMinting.sol` contract such as the `setMaxMintPerBlock`, `setMaxRedeemPerBlock` and `disableMintRedeem` do not emit events after the succesful execution of thier transactinos. 

It is recommended to emit events in these critical functions after thier succesful execution for off-chain analysis.

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L219-L221
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L224-L226
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L229-L232

## 7. DISCREPENCY IN THE THE NATSPEC COMMENTS AND THE FUNCTION IMPLEMENTATION OF THE `StakedUSDe.addToBlacklist` AND `StakedUSDe.removeFromBlacklist` FUNCTIONS

The `Natspec` comments of the `StakedUSDe.addToBlacklist` and `StakedUSDe.removeFromBlacklist` functions state that the owner of the `StakedUSDe` contract can invoke the above functions as shown below:

```solidity
   * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to blacklist addresses.
```

But the actual function implementation has the `onlyRole(BLACKLIST_MANAGER_ROLE)` which enables only the `BLACKLIST_MANAGER_ROLE` to execute the above functions. Hence the `DEFAULT_ADMIN_ROLE` can not execute the `StakedUSDe.addToBlacklist` and `StakedUSDe.removeFromBlacklist` functions unless the `owner` is given the `BLACKLIST_MANAGER_ROLE` too. 

Hence it is recommended to update the `Natspec` comments of the above two functions accordingly to indicate that the addresses with the `BLACKLIST_MANAGER_ROLE` can only execute the `StakedUSDe.addToBlacklist` and `StakedUSDe.removeFromBlacklist` functions. The `Natspec` comments should be modified as follows:

```solidity
   * @notice Allows the blacklist managers to blacklist addresses.
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L102
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L108
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L116
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L122

## 8. THERE IS NO INPUT VALIDATION ON THE `to` ADDRESS FOR `address(0)` IN THE `StakedUSDe.rescueTokens` FUNCTION AND HENCE RESCUED FUNDS COULD BE LOST IF SENT TO `address(0)`

The `StakedUSDe.rescueTokens` function is used to rescue the tokens accidentally sent to the contract. This function is only callable by the `DEFAULT_ADMIN_ROLE`. But there is no input validation on the `to` address passed in. Hence if the address(0) is passed in as the `to` address the rescued funds could be lost.

Hence it is recommended to perform the input validation on the `to` address in the `rescueTokens` function as shown below:

    require(to != address(0), "to address can not be address(0)");

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L138-L141