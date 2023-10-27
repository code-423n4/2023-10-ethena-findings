# Summary

## Disclaimer

Please note, that although all gas optimizations provided in this report were tested against the provided smart contract tests for this audit, it is important to note that the developers who are going to apply them should still do so with caution.

To help minimize the chance of introducing any new vulnerabilities even further, it is also highly recommended to conduct thorough code reviews to all of the changes in a granular manner. 

## Gas optimizations context

All gas optimizations were benchmarked using the tests that are present in the protocol repo for this contest. The configuration for them was used as is.


# [G-01] Unnecessary call to `StakedUSDe::getUnvestedAmount` inside of `StakedUSDe::transferInRewards` 

**Average savings per function call: 369 GAS**

## Description

There is a redundant call to `StakedUSDe::getUnvestedAmount` inside of `StakedUSDe::transferInRewards` function. As it can be seen on line 90,  `StakedUSDe::getUnvestedAmount` is called once and its return value is checked to be greater than zero - if not, then the function reverts. What we can also see though, is that on the next line (91) we `StakedUSDe::getUnvestedAmount` is being called once again and its value is being added up with another one. This operation is completely redundant, and can be removed in order to save gas.

## Lines of code

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L91

## Recommended optimizaiton

```diff
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
-   uint256 newVestingAmount = amount + getUnvestedAmount();
+   uint256 newVestingAmount = amount;

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
}
```