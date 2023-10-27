[L-1] silo can be made immutable.

Silo is deployed in constructor, and there is no setter in the StakedUSDeV2.

```solidity
  constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {
    silo = new USDeSilo(address(this), address(_asset));
    cooldownDuration = MAX_COOLDOWN_DURATION;
  }
```


[L-2] _deduplicateOrder returning a true and checks is redundant since it never returns a false `valid` from verifyNonce, which simply revert.

```solidity
if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();

  function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
    (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
    return valid;
  }

  function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
    if (nonce == 0) revert InvalidNonce();
    uint256 invalidatorSlot = uint64(nonce) >> 8;
    uint256 invalidatorBit = 1 << uint8(nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    uint256 invalidator = invalidatorStorage[invalidatorSlot];
    if (invalidator & invalidatorBit != 0) revert InvalidNonce();

    return (true, invalidatorSlot, invalidator, invalidatorBit);
  }
```