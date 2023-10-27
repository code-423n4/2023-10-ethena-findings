[L-1] silo can be made immutable.

Silo is deployed in constructor, and there is no setter in the StakedUSDeV2.

```solidity
  constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {
    silo = new USDeSilo(address(this), address(_asset));
    cooldownDuration = MAX_COOLDOWN_DURATION;
  }
```