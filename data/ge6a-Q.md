## User cannot create a second cooldown if the first one hasn't finished.

### POC


```
  function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {
    if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();

    uint256 shares = previewWithdraw(assets);

    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    cooldowns[owner].underlyingAmount += assets;

    _withdraw(_msgSender(), address(silo), owner, assets, shares);

    return shares;
  }
```

Scenario

1. User calls cooldownAssets() for X assets. After this operation,
cooldowns[owner].cooldownEnd = timestamp + 14 days
cooldowns[owner].cooldownEnd = X;

2. After 10 days, the user decides they want to redeem more assets and calls cooldownAssets() with assets Y. After this operation:
cooldowns[owner].cooldownEnd = now + 14 days;
cooldowns[owner].cooldownEnd = X+Y;

So, the user will have to wait 14 days from that moment until they are able to unstake X+Y assets, regardless of the 10 days that have already passed. During periods of high volatility, the inability to withdraw funds in portions may lead to losses for the user. It's also important to note that the description of the functions doesn't clarify that calling it again resets the timer.

### Mitigation steps

I think that it is a good idea to add the ability to create a new cooldown for each cooldownAssets() / cooldownShares() call.