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


