1. Other non-standard tokens that might affect system protocols. 
- Upgradable Tokens - Tokens like USDC, USDT have logics can be changed over time. Consider introducing a logic that temporarily freezes interactions with these tokens if an upgrade is detected.
- Tokens that implement a blocklist - important roles and other contracts can get blocklisted from using these tokens and might trap funds in the contract.
- Tokens with multiple addresses -  if a only one of the addresses is added to the supported assets list, users will only be able to interact with the token using that address. If they try to interact with the token using a different address, they will not be able to do so. Also, when these tokens are being delisted, all their addresses must be removed, or users will be able to bypass the delisting using the not-delisted address.

2.  Unused code should be removed and comment updated; 

No need to add `getUnvestedAmount` to `amount` since it's 0;  
```
  /// @notice The amount of the last asset distribution from the controller contract into this
  /// contract + any unvested remainder at that time
  uint256 public vestingAmount;
 ```
```
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount(); //@note, getUnvestedAmount == 0

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```

or it can be refactored, e.g
```
  /// @notice The amount of the last asset distribution from the controller contract into this contract
  uint256 public vestingAmount;
```
```
 function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    vestingAmount = amount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
    emit RewardsReceived(amount);
  }
```
```
  /// @notice Event emitted when the rewards are received
  event RewardsReceived(uint256 indexed amount);
```

3.  Consider Introducing a check in `removeSupportedAsset` function to ensure that stEth cannot be removed from supported assets list, It's the base asset for the protocol, and its removal affects protocol yield generation.
e.g
```
function removeSupportedAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();
     if (asset == address(stETH) revert Error();
    emit AssetRemoved(asset);
  }
```      
4. Consider introducing a custodian length check in EthenaMinting constructor. As it stands, 0 custodian address can also be passed into the contract. 
e.g
```
constructor(
   ...
    {
    if (address(_usde) == address(0)) revert InvalidUSDeAddress();
    if (_custodians.length == 0) revert NoCustodianProvided();
    ...
    }
```
5. In `redistributeLockedAmount` function, consider checking that the `address to` is not a zero address before burning. As the function stands, if `address to` is 0, the entire balance will be burned without any token being redistributed. There's no revert or `else`, so upon 0 address, the condition silently fails, the interaction continues and nothing is reallly distributed,
```  
function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
      uint256 amountToDistribute = balanceOf(from);
      _burn(from, amountToDistribute);
      // to address of address(0) enables burning
      if (to != address(0)) _mint(to, amountToDistribute); //@note, if 0, interaction continues

      emit LockedAmountRedistributed(from, to, amountToDistribute);
    } else {
      revert OperationNotAllowed();
    }
  }
```
6. Avoid low level calls to custom addresses for best practices

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L250