#### [NC-01] Maximum Line Length over 120 characters
This breaks the style guide (https://docs.soliditylang.org/en/latest/style-guide.html#maximum-line-length)

Instances (2):

```
File: contracts/EthenaMinting.sol

247:   function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {

339:   function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {

```