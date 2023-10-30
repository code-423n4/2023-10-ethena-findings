### 1. Cache array elements to save gas.

"route.addresses[i]" and "route.ratios[i]" are used multitimes, cache them to save gas.
```solidity
File: EthenaMinting.sol

363:    for (uint256 i = 0; i < route.addresses.length; ++i) {
364:      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address36:(0) || route.ratios[i] == 0)
365:      {
366:        return false;
367:      }
368:      totalRatio += route.ratios[i];
369:    }
```
| [Line #363-369](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L363-L369) | 

### 2. Save transfer times in loop

If *remainingBalance* not 0, the last address (addresses[addresses.length - 1]) will be transferred twice.
```solidity
File: EthenaMinting.sol

original:
423:    uint256 totalTransferred = 0;
424:    for (uint256 i = 0; i < addresses.length; ++i) {
425:      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
426:      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
427:      totalTransferred += amountToTransfer;
428:    }
429:    uint256 remainingBalance = amount - totalTransferred;
430:    if (remainingBalance > 0) {
431:      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
432:    }


optimized:
423:    uint256 remainingBalance = amount;
424:    uint256 lastAddressIndex = addresses.length - 1;
425:    for (uint256 i = 0; i < lastAddressIndex; ++i) {
426:      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
427:      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
428:      remainingBalance -= amountToTransfer;
429:    }
430:    token.safeTransferFrom(benefactor, addresses[lastAddressIndex], remainingBalance);
```
| [Line #423-432](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L423-L432) | 

### 3. Cache function result to save gas
Function "getUnvestedAmount()" get called twice (L90 and L91), cache its result to save gas.
```solidity
File: StakedUSDe.sol

89:  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
90:    if (getUnvestedAmount() > 0) revert StillVesting();
91:    uint256 newVestingAmount = amount + getUnvestedAmount();
``````
| [Line #89-91](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89-L91) | 