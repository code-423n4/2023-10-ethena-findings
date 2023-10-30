### [L-01] Line 91 in StakedUSDe.sol is redundant as getUnvestedAmount will always be zero.

In StakedUSDe.transferInRewards(), the first line checks if the return value of `getUnvestedAmount()` is greater than 0. If the return value is greater, revert. Else, continue. The second addition line is redundant as `getUnvestedAmount()` will always be 0.

```
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    //@audit getUnvestedAmount must return 0
    if (getUnvestedAmount() > 0) revert StillVesting();
    //@audit If getUnvestedAmount can only return 0 for function to proceed, this next addition is redundant since getUnvestedAmount always returns 0
    uint256 newVestingAmount = amount + getUnvestedAmount();
```

Remove the second addition line.

```
    uint256 newVestingAmount = amount + getUnvestedAmount();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L173

### [L-02] OpenZeppelin does not use hooks in the latest 5.0 version release

> In 5.0, token hooks were removed from ERC20, ERC721, and ERC1155, favoring a single `_update` function. This can also be overridden for custom behaviors in any `mint`, `transfer`, or `burn` operation.

Take note when compiling the contracts or upgrading to a new contract. The override of `_beforeTokenTransfer()` will not work and the full restricted blacklisted users can still trade their tokens. Best practice is to override the \_update function in the new ERC20 contract and use the latest openzeppelin version.

https://blog.openzeppelin.com/introducing-openzeppelin-contracts-5.0


### [L-03] Benefactor and beneficiary makes is unnecessarily confusing and creates additional steps for token transfer

In EthenaMinting, there is a benefactor and beneficiary, which I assume one is the user and one is a trusted address. For the user to get his token, he must wait for the minter to mint his tokens, and then wait for the beneficiary to give him his USDe. There will be a lot of waiting time, which increases the centralization risks of the protocol. It may be hard for the user to call `mint()` directly since the protocol has to specify how much USDe they can mint for their given collateral, but the USDe should be directly minted to the user themselves.

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L431

### [N-01] Best practice is to always call super.\_beforeTokenTransfer when overriding the beforeTokenTransfer hook

Although the parent hook is empty now, it is good to call the parent function in case OpenZeppelin implements new functionalities in the parent contract.

> Always call the parent’s hook in your override using super. This will make sure all hooks in the inheritance tree are called: contracts like ERC20Pausable rely on this behavior.

https://docs.openzeppelin.com/contracts/4.x/extending-contracts#rules_of_hooks
