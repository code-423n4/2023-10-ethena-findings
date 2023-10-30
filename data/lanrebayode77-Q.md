
### Redundant Line of Code
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