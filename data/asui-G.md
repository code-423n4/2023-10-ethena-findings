G-01. In the ```transferInRewards``` function remove 
```uint256 newVestingAmount = amount + getUnvestedAmount();``` 
and directly set ```vestingAmount = amount``` .
Because it makes no sense to add ```uint256 newVestingAmount = amount + getUnvestedAmount();```
when just the above line ```if (getUnvestedAmount() > 0) revert StillVesting();``` checks that if the ```getUnvestedAmount()``` is greater than 0 
it will revert meaning we will always add amount + 0. 