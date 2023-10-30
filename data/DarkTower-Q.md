## Summary
### Low Risk Issues
| Tag |Issue|Contexts|
|-|:-|:-:|
L-1 | Wrong error reverted in the `verifyOrder` function | 1
L-2 | Potential for USDe to end up in custodian wallet | 1
L-3 | Use `safeTransfer` instead of `transfer` in `USDeSillo::withdraw` | 1 |

Total: 3 contexts over 3 issues

### Non-critical Issues
| Tag |Issue|Contexts|
|-|:-|:-:|
| NC-1| Unnecessary addition of `getUnvestedAmount()` in the `transferInRewards` function | 1 |

Total: 1 contexts over 1 issues


## [L-1] Wrong revert error in the `verifyOrder` function
The revert message for when the `order.beneficiary` address is a zero address is wrong as it logs `InvalidAmount` which is literally non-related to a zero address check error name you'd expect.

Context: EthenaMinting::verifyOrder
```solidity
  // @> Line 343:
  if (order.beneficiary == address(0)) revert InvalidAmount();
```


## [L-2] Potential for USDe to end up in custodian wallet
As confirmed by the sponsor, the custodian wallet is expected to never be sent/hold USDe as the custodian wallet literally holds the assets transferred after minting of USDe e.g stETH, ETH, etc before it gets delegated to the perps exchange. With this in mind, it makes sense to revert transactions that attempt to transfer USDe to the custodian wallet when the `transferToCustody` function is called.

Context: EthenaMinting::transferToCustody
```solidity
  function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {
    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();
    // @> let's have a check here to revert txns that attempt to transfer USDe to the custodian wallet
    if (asset == NATIVE_TOKEN) {
      (bool success,) = wallet.call{value: amount}("");
      if (!success) revert TransferFailed();
    } else {
      IERC20(asset).safeTransfer(wallet, amount);
    }
    emit CustodyTransfer(wallet, asset, amount);
  }
```

Usually, when this occurs right now funds will just be retreived from the custodian wallet and another transfer of the intended `asset` will be initiated but it's a great thing to consider only allowing transfers of assets that are supposed to be sent to the custodian address. A whitelist could suffice in this scenario and render a check similar to this below:

```solidity
  if (!_custodianAssets.contains(asset)) revert InvalidAsset(); 
```

## [L-3] Use `safeTransfer` instead of `transfer` in `USDeSillo::withdraw`
It is good to add a require() statement that checks the return value of token transfers or to use something like OpenZeppelin’s safeTransfer/safeTransferFrom unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

Context: USDeSillo::withdraw.
```solidity 
function withdraw(address to, uint256 amount) external onlyStakingVault {
@>    USDE.transfer(to, amount);
  }
```

## [NC-1]Unnecessary addition of `getUnvestedAmount()` in the `transferInRewards` function.
Function `transferInRewards` in StakedUSDe.sol has an unnecessary operation which always results in adding zero.

Context: StakedUSDe::transferInRewards
```solidity
uint256 newVestingAmount = amount + getUnvestedAmount();
```

As the function first checks `if (getUnvestedAmount() > 0) revert StillVesting();`, the value of `getUnvestedAmount()` will always be zero at the next line. 

Mitigation steps: Simply delete this part `+ getUnvestedAmount()` at line 91 in the code of `StakedUSDe.sol`