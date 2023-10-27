# No Zero address check for the custodian which holds the stETH token

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
