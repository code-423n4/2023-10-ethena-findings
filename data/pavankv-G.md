
## 1 .Refactor the function to save gas :-
Here we can see `getUnvestedAmount()` was validated in if statement that greater than ZERO revert, below this operation again added to `newVestingAmount` variable. However `getUnvestedAmount()` is zero there is no need to add zero to another `uint`. And `newVestingAmount` is used to assign the state varaible `vestingAmount` and in Emit. If we want optimised version please look into **After** section and our findings doesn't cause any test fails.

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

**HardHat Gas BenchMark**

Before
| transferInRewards                                | 4382            | 43041 | 44614  | 66934 | 24      |

After
| transferInRewards                                | 4382            | 42663 | 44080  | 66400 | 24   	|

Exact runtime gas saved = 66934 - 66400 = 543 per call