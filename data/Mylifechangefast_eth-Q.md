# No Zero address check for the custodian who holds the stETH token

### Impact
***The issue I mentioned regarding the custodian contract not checking for an address of 0 (zero) in its constructor could potentially affect the behavior of the system. In Ethereum, an address of 0 represents an invalid or non-existent address, often used as a placeholder or to indicate the absence of a valid address.***

##### Recommendation
>Check if the custodian is `zero address`.

# Add Timelock to critical functions

### Summary 
***It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).***

consider adding timelock to:
```
function setMaxMintPerBlock(uint256 _maxMintPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {
function setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {
function setDelegatedSigner(address _delegateTo) external {
function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
function setMinter(address newMinter) external onlyOwner {
```

# `Constant` should be defined rather than using magic numbers
## Summary
***A magic number is a numeric literal that is used in the code without any explanation of its meaning. The use of magic numbers makes programs less readable and hence more difficult to maintain and update.
Even assembly can benefit from using readable constants instead of hex/numeric literals.***

```
    for (uint256 i = 0; i < addresses.length; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
 ```

```
    if (totalRatio != 10_000) {
      return false;
```

# No Zero address check for the beneficiary
***The issue I mentioned regarding the custodian contract not checking for an address of 0 (zero) in its constructor could potentially affect the behavior of the system. In Ethereum, an address of 0 represents an invalid or non-existent address, often used as a placeholder or to indicate the absence of a valid address***
```
  function _transferToBeneficiary(address beneficiary, address asset, uint256 amount) internal {
    if (asset == NATIVE_TOKEN) {
      if (address(this).balance < amount) revert InvalidAmount();
```

# No zero check for critical function
***The ` isSupportedAsset` function doesn't check if the asset that anyone can input is an address(0) the function then checks if the address(0) is supported, what if an attacker creates an address that would be supported?*** *Prevention is better than cure*

# No input validation for `transferInRewards` which could be set to zero and still waste gas when the function is called.

***`transferInRewards` function with an added check for the amount argument to ensure it's not zero:***
```
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) {
    require(amount > 0, "Amount must be greater than zero");
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
}
```
***With this modification, the function will only proceed if the amount provided is greater than zero, and it will revert with the message "Amount must be greater than zero" if the amount is zero or negative. This helps ensure that no zero or negative values are accepted for the amount argument.***

 

