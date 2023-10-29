## Imprecise management of allowance in the cooldown functions could lead to lose of funds.

```jsx
function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
  if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();

  uint256 assets = previewRedeem(shares);

  cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
  cooldowns[owner].underlyingAmount += assets;

  _withdraw(_msgSender(), address(silo), owner, assets, shares);

  return assets;
}
```
The standard allowance / approval flow in all the ERC standards works in the way that the approver allows the spender to spend the given amount of tokens as the spender wants to. The cooldown functions inside StakedUSDeV2 do allow a spender to cooldown assets for a given owner. But the owner is always set in the cooldowns array. If the spender wanted to cool down the assets for unstacking it into the own wallet, it would be necessary to first transfer it from the approver to the spender and then calling cooldown with owner == msg.sender. This is not only a waste of gas, it is also an imprecise management of allowance that could potentially lead to loss of funds. Therefore, I recommend adding a receiver parameter to specify the address which is added to the cooldowns mapping.
Links to the cooldown functions:
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L95-L106
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L111-L122