## Summary<a name="Summary">

### Non Critical Issues

| |Issue|number of Instances
|-|:-|:-|
| [N&#x2011;01](#N&#x2011;01) | remove useless variable and line in function `transferInRewards` | 1 |
| [N&#x2011;02](#N&#x2011;02) | remove useless check in function `_transferCollateral` | 1 |

## Non Critical Issues:

### <a href="#Summary">[N&#x2011;01]</a><a name="N&#x2011;01"> remove useless variable and line in function `transferInRewards`
    
the function `transferInRewards` is used by the `REWARDER_ROLE` to give rewards to usde stackers, it will only work if `getUnvestedAmount` is 0, revert otherwise. 
it assigns the value that the `REWARDER_ROLE` is vesting , by adding the `amount` of usde tokens vested plus `getUnvestedAmount` which is always 0.
that's why we can eliminate a memory varaiable and a line to simplify and optimize code to be like this 
`vestingAmount = amount;`
    
- *StakedUSDe.sol* ( [#L90](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L90-L93)):
    
```solidity=90
if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();

    vestingAmount = newVestingAmount;
```

### <a href="#Summary">[N&#x2011;02]</a><a name="N&#x2011;02"> remove useless check in function `_transferCollateral`
    
in the function `_transferCollateral` we transfer the collateral given by the user to the various custodians based on ratios, we already check that the sum of the ratios given to the function is 10_000, so there will be no remaining balance after transfering.
but in the function we check if there is some remaining amount, which will never be the case.
should delete the last two lines in the function as they are a branch that will never be visited.
    
- *EthenaMinting.sol* ( [#L430](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L430-L432)):
    
```solidity=430
if (remainingBalance > 0) {
      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
    }
```
    
- *EthenaMinting.sol* ( [#L370](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L370-L372)):
    
```solidity=370
if (totalRatio != 10_000) {
      return false;
    }
```
    
