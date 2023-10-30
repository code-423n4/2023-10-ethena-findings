## 1. THE REUNDANT `getUnvestedAmount()` FUNCTION CALL CAN BE OMITTED WHILE CALCULATING THE `newVestingAmount` VALUE IN THE `StakedUSDe.transferInRewards` FUNCTION TO SAVE GAS

The `StakedUSDe.transferInRewards` function is used by the owner to transfer rewards from the controller contract into this contract.

Within the `transferInRewards` function execution the `newVestingAmount` variable is updated as follows:

    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();

But as evident from the above code snippet the issue here is that the `getUnvestedAmount()` calculation is redundant when calculating the `newVestingAmount` since the transaction will revert if the `getUnvestedAmount() > 0`. 

Hence the only possiblity for transaction to pass the first `if` statement is when the `getUnvestedAmount() == 0`. Which means the `amount + getUnvestedAmount()` is effectively equal to the `amount + 0 = amount`.

Hence it is recommneded to omit the `getUnvestedAmount()` function call while calculating the `newVestingAmount` in the `StakedUSDe.transferInRewards` function to save gas. The modified code to calculate the `newVestingAmount` is shown below:

    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount;

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L90-L91

## 2. `CALLDATA` VARIABLE CAN BE USED INPLACE OF `STATE` VARIABLES WHEN EMITTING EVENTS TO SAVE GAS

The `EthenaMinting._setMaxMintPerBlock` function and the `EthenaMinting._setMaxRedeemPerBlock` functions are used to set the `maxMintPerBlock` and `maxRedeemPerBlock` state variables respectively.

In each of the above functions once the state variables are set, events are emitted as shown below:

    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock); 

    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);

Here the `state` variables are used for the event emits where as the `calldata` variables can be used in their place. This would save an extra `SLOAD` operation. The modified lines of code are shown below:

    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, _maxMintPerBlock); 

    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, _maxRedeemPerBlock);

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L439
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L446
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L133

## 3. REDUNDANT INITIALIZATION OF THE `totalRatio` LOCAL VARIABLE TO `0` IN THE `EthenaMinting.verifyRoute` FUNCTION

In the `EthenaMinting.verifyRoute` function the local variable `totalRatio` is declared as follows:

    uint256 totalRatio = 0;

It is recommended not to initialize the local variable to `0` since by default the value will be `0` for any local memory variable. Hence the `totalRatio` variable declaration can be modified as follows to save gas.

    uint256 totalRatio;

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L356
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L423

## 4. RECOMMENEDED TO MODIFY THE `invalidator` VALUE RETRIEVAL BY CODE SNIPPET IN THE `EthenaMinting.verifyNonce` FUNCTION TO SAVE GAS

The `EthenaMinting.verifyNonce` function is used to verify the order nonce during the execution of the `EthenaMinting._deduplicateOrder` function. 

During the verification of the `nonce` the `invalidator` is calculated as follows:

    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    uint256 invalidator = invalidatorStorage[invalidatorSlot]; 

But there is no need to declare the `storage invalidatorStorage` mapping variable since the `invalidator` can directly retrieved from the `_orderBitmaps` as follows:

    uint256 invalidator = _orderBitmaps[sender][invalidatorSlot];

Above modification to the `invalidator` value retrieval will save gas on one `SSTORE` and one `SLOAD` opcode operation during the execution of the `EthenaMinting.verifyNonce` function execution.

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L381-L382

## 5. RECOMMENDED TO USE `delete` TO SET THE STORAGE VARIABLES TO DEFAULT VALUE OF `0` TO GAIN A GAS REFUND

The `StakedUSDeV2.unstake` function allows an user to claim the staking amount after the cooldown has finished. During the execution of the `StakedUSDeV2.unstake` function the respective `UserCooldown` struct parameters are set to `0` before the `remaining underlying assets` are withdrawn to the user as shown below:

      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

But it is recommended to use the `delete` to get a gas refunds instead of setting the parameters to `0`. Hence the above code should be modified as follows:

      delete userCooldown.cooldownEnd;
      delete userCooldown.underlyingAmount;

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L83-L84