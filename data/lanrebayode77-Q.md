
### 1. Redundant Line of Code
In ```StakedUSDe.transferInRewards()```, a check was done to ensure that ```getUnvestedAmount()``` returns zero, the next line then adds the return value of ```getUnvestedAmount()``` to amount to get ```newVestingAmount```, a computation that seems unnecessary. ```vestingAmount``` could just have been set to equal ```amount``` directly.
``` solidity
function transferInRewards(
        uint256 amount
    ) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
        if (getUnvestedAmount() > 0) revert StillVesting();
        uint256 newVestingAmount = amount + getUnvestedAmount(); //@audit redundant

        vestingAmount = newVestingAmount;
        lastDistributionTimestamp = block.timestamp;
        // transfer assets from rewarder to this contract
        IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

        emit RewardsReceived(amount, newVestingAmount);
    }
```
remove the redundant line of code.
``` solidity
function transferInRewards(
        uint256 amount
    ) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
        if (getUnvestedAmount() > 0) revert StillVesting();
   
        vestingAmount = amount; //set vestingAMount directly
        lastDistributionTimestamp = block.timestamp;
        // transfer assets from rewarder to this contract
        IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

        emit RewardsReceived(amount, newVestingAmount);
    }
```

### 2. Some missing zero checks in the constructor.
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L126-L131
Include zero checks for every address being passed in to maintain consistency in security measures. 
``` solidity
  for (uint256 i = 0; i < _assets.length; i++) {
      addSupportedAsset(_assets[i]);
      if (address(_assets[i]) == address(0)) revert InvalidUSDeAddress();
    }

    for (uint256 j = 0; j < _custodians.length; j++) {
      addCustodianAddress(_custodians[j]);
     if (address(_custodians[i]) == address(0)) revert InvalidUSDeAddress();
    }
```