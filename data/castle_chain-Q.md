### [L-01] the returned value of the function `verifyOrder()` should be checked  

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L339-L348

```solidity 
  function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    if (order.beneficiary == address(0)) revert InvalidAmount();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
    if (block.timestamp > order.expiry) revert SignatureExpired();
    return (true, taker_order_hash);
  }
```  
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L202C4-L202C4
```solidity 
    verifyOrder(order, signature);
```

### [L-02] a check can be removed to enhance the simplicity of the code 
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L353
the function `verifyRoute()` only get called in the function `mint()` which reverts if the orderType is REDEEM so this check can be removed . 
```solidity 
    if (orderType == OrderType.REDEEM) {
      return true;
    }
``` 
```solidity 
  function mint(Order calldata order, Route calldata route, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(MINTER_ROLE)
    belowMaxMintPerBlock(order.usde_amount)
  {
    if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
```

### in the function `transferInRewards()` , adding the value of the function `getUnvestedAmount()` can be removed to enhance the code , since the value of this function will always be zero 
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L90-L91

```solidity 
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();
```

### the modifier `notOwner()` should be removed from the function `removeFromBlacklist()` since adding the owner to the blacklist is not possible . 

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L123 

the function `removeFromBlacklist()` revoke the blacklist role from the `target` address , so due to preventing the owner from being blacklisted , the check of that the target is not the owner can be removed 
```solidity
  function removeFromBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
    bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
    _revokeRole(role, target);
  }
```

