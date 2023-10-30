
## 1. Make the code more readable and improved :-

In before section we can see the condition in if statement revert when `totalSupply` is greater than zero and less than `MIN_SHARES`(1 ether). However 1 ether is equal to `1e18` so there is no need to check zero ness of `totalSupply`. Have a look into **After** section which is more improved version where we combined conditions that total supply must be greater than Zero and greater than `MIN_SHARES`(1 ether).

**Before**
```solidity
  /// @notice ensures a small non-zero amount of shares does not remain, exposing to donation attack
  function _checkMinShares() internal view {
    uint256 _totalSupply = totalSupply();
    if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();
  }
```

**After**
```solidity
function _checkMinShares() internal view {
    // Checks if the total supply is greater than or equal to the minimum number of shares.
    uint256 _totalSupply = totalSupply();
    require(_totalSupply >= MIN_SHARES, "MinSharesViolation");
  }
```
Code snippet:-
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L191C1-L195C1


## 2. Refactor the function transferInRewards() :-
Here we can see `getUnvestedAmount()` was validated in if statement that greater than ZERO revert, below this operation again added to `newVestingAmount` variable. However `getUnvestedAmount()` is zero there is no need to add zero to another `uint`. And `newVestingAmount` is used to assign the state varaible `vestingAmount` and in Emit.

**Before**
```solidity
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```


**After**
```solidity
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    //@audit changed here
    vestingAmount = amount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);//@audit changed here
    emit RewardsReceived(amount, amount);//@audit changed here
  }
```

code snippet:-
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89C1-L99C4

## 3 . Add a dust amount check in mint() function to avoid unfair mint:-
First look into the [mint()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L162) function having multi-validation mechanism but not dust amount check whether the input amount is equal to output amount. After validation it blindly call the [ _transferCollateral()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L175C4-L175C25) internal fucntion to transfer the collateral amount to custodian address with specified ratio without checking the dust amount of the collateral.

Recommedation :-
Add mechanism to check whether the input amount is equal to output amount .

code snippet:-
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L162C1-L187C4
