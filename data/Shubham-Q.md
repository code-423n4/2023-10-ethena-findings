## Low Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-01](#L-01) | Users can transfer in small amounts before VESTING_PERIOD duration has passed | 1 |

## Non-Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-01](#NC-01) | Unnecessary check in `verifyRoute()` | 1 |
| [NC-02](#NC-02) | Unnecessary computation in `transferInRewards()` | 1 |


## [L-01] Users can transfer in small amounts before VESTING_PERIOD duration has passed

`getUnvestedAmount()` returns 0 when timeSinceLastDistribution >= VESTING_PERIOD that is greater or equal to 8 hours. 
But even if the `timeSinceLastDistribution` is close to 8 hours & vesting amount is high, the function would still return 0 or if the amount to send is small & the not much time has passed since the last transfer. 

```solidity
File: StakedUSDe.sol

  function getUnvestedAmount() public view returns (uint256) {
    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

    if (timeSinceLastDistribution >= VESTING_PERIOD) {
      return 0;
    }
    /// @note - will return 0 in some cases
    return ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) / VESTING_PERIOD;       
  }
```
Suppose
- timeSinceLastDistribution is 7.5 hours (27,000 sec)
- vestingAmount is 15
- ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) = (28800 - 27000) * 15 = 27000
- 27000 / VESTING_PERIOD = 27000 / 28800 = 0

Similar situation would be if vestingAmount is 1, then Rewarder can call `transferInRewards()` with a huge amount the very next second & rewards will be sent.
The Rewarder will try calling `transferInRewards()` & it will pass even though VESTING_PERIOD duration hasn't been achieved.
This enables rewards to be transferred before the set time period. Worst case is that users can transfer in small amounts before the 8 hours criteria. 

## [NC-01] Unnecessary check in `verifyRoute()`

`verifyRoute()` is only called inside `mint()` in EthenaMinting contract.
We can see that the 1st if condition already checks for the Order Type which should be `MINT`. So inside `verifyRoute()` the 1st check will always return false.

```solidity
File: EthenaMinting.sol
  
    function mint(Order calldata order, Route calldata route, Signature calldata signature)
      external
      override
      nonReentrant
      onlyRole(MINTER_ROLE)
      belowMaxMintPerBlock(order.usde_amount)
    {
      if (order.order_type != OrderType.MINT) revert InvalidOrder();
      verifyOrder(order, signature);
      if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
      ......


    function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
      // routes only used to mint                           /// @note - will never get executed
      if (orderType == OrderType.REDEEM) {
        return true;
      }
      .....
```
Consider removing this check.

## [NC-02] Unnecessary computation in `transferInRewards()`

`getUnvestedAmount()` is called inside `transferInRewards()` to check if amount is still vesting.
It adds its value to the `newVestingAmount` variable & set to `vestingAmount`. But it makes no sense because getUnvestedAmount() will only pass the 1st check if its 0 & then later adding 0 to amount is a waste.

```solidity
File: StakedUSDe.sol

  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();    /// @note - getUnvestedAmount() is always 0

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```
The amount passed could be directly added to vestingAmount without the need of newVestingAmount.
