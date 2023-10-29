## EthenaMinting.sol
1) in verifyNonce could be used yul,resulting in 2862gas usage from the original 2894gas used,saving in this way 32gas per call of this function,updated code:
```
function verifyNonce(address sender, uint256 nonce) public view returns (bool, uint256, uint256, uint256) {
    if (nonce == 0) revert InvalidNonce();
    uint256 invalidatorSlot;
    uint256 invalidatorBit;
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    uint256 invalidator;
    assembly{
      invalidatorSlot := div(nonce, 256)
      invalidatorBit := shl(mod(nonce, 256), 1)
      invalidator := sload(invalidatorSlot)
    }
    if (invalidator & invalidatorBit != 0) revert InvalidNonce();
    return (true, invalidatorSlot, invalidator, invalidatorBit);

  }
```
updateNonce: https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L377
## StakedUSDe.sol
1) could be assigned ``amount+getUnvestedAmount`` directly to vestingAmount without creating a new temp variable in transferInRewards function,updated function:
```
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    vestingAmount = amount + getUnvestedAmount();
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, vestingAmount);
}
```