### Report 1:
Unnecessary Call of getUnvestedAmount() function for addition operation in a situation it must always return zero 
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91
```solidity
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
>>>    if (getUnvestedAmount() > 0) revert StillVesting();
---  uint256 newVestingAmount = amount + getUnvestedAmount();
+++ uint256 newVestingAmount = amount;
    ...
```
As noted in the code above if getUnvestedAmount() is greater than zero the code will revert meaning the transferInRewards() function can only run if getUnvestedAmount() is zero, the problem is using it in addition operation to calculate "newVestingAmount" since it would always be zero, the code should be reaccess for possible mistake in function implementation or it should be simply removed.
###  Report 2:
## Title
Risk of Token loss due to Missing Address Validity Check in Mint Function
## Link to Affected Code
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L28-L31
## Impact
The absence of an address zero validity check in the mint function of the USDe.sol contract would allow accidental transfers to zero address (0x0). This would effectively burn tokens as reverse to minting, as they would become irretrievable. This could have a significant impact on the total supply of tokens and their value.

## Proof of Concept
The mint function in the USDe.sol contract does not contain a check to prevent minting tokens to the zero address. Here is the function for reference:
```solidity
function mint(address to, uint256 amount) external {
    if (msg.sender != minter) revert OnlyMinter();
    _mint(to, amount);
}
```
## Tools Used
The issue was identified through manual code review

## Recommended Mitigation Steps
To mitigate this issue, it is recommended to include a validity check for the "to" address in the mint function. This can be done by adding a require statement that ensures the "to" address is not the zero address, i.e
```solidity
function mint(address to, uint256 amount) external {
+++ require(to != address(0), "Mint to the zero address");
    if (msg.sender != minter) revert OnlyMinter();
    _mint(to, amount);
}
```
This will throw an exception and revert the transaction if someone tries to mint tokens to the zero address. This simple check can prevent potential token loss and maintain the integrity of the token’s total supply.