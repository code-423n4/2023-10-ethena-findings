# initializing an unused library.

In `USDeSilo.sol` file, `SafeERC20` lib is imported and is declared to be used for `IERC20` variables ([#L13](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L13)), but it is not used at any part of the code.

### Risks
This can cause misunderstanding for people when seeing the code, as they may think that `SafeERC20` lib functions will be used in the contract.

### Mitigation
You should remove the import statement as well as the line declaring its usage as it is not important.

**Lines to remove in `USDeSilo.sol`**: ([#L13](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L13), [#L5](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L5))

# Constant Values used without a declaration

In `EthenaMinting.sol` the totalRatio, which is used by the Ethena to validate routes in `EthenaMinting::verifyRoute`, is `10_000`. This value used twice by the devs in two different positions in the `EthenaMinting.sol`: ([#L370](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L370), [#L425](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L425))

### Risks
Inconsistency and potential confusion during code reviews and maintenance

### Mitigation
It is better to save these kinds of variables as constant and use them when needed. This will help improve code syntax and readability, and avoid syntax errors.

For example, we can name a variable `TOTAL_RATIO` and set its value to `10_000`, and use the variable when needed.

```solidity
    uint256 private constant TOTAL_RATIO = 10_000;
```

# Non-used function declarations

In `EthenaMinting.sol`, `encodeRoute(Route)` function is declared but it is not used in the protocol.

This function was used when there was a wrong EIP712 signature implementation for `Route`, But the protocol removed it when figuring out that it was implemented wrongly by the previous audit done of the protocol by Pashov.

The reference for removing using this function from pashov audits: [the discussion for this issue](https://github.com/pashov/audits/blob/master/solo/Ethena-security-review.md#discussion-3).

### Risks
Protocol devs forgot to remove the function body, they removed the calling only. So it is unused now, which can lead to misleading, and additional gas costs when deploying the contract.


### Mitigation
Remove the declaration of this function in `EthenaMinting.sol`: [#L334-L336](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L334-L336)

# Refactor the reused code in a separate function

In `StakedUSDe.sol`, these two functions `cooldownAssets()` and `cooldownShares()` have a common syntax for making a withdrawal request. It is better to refactor the code syntax to improve functions readability and improve code syntax.

### Mitigation

1- Make a new function called `_withdrawAssets`  for example, and add reused code in it.

```solidity
// StakedUSDe.sol
function _withdrawAssets(address owner, uint256 assets, uint256 shares) private {
    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    cooldowns[owner].underlyingAmount += assets;
        
    _withdraw(_msgSender(), address(silo), owner, assets, shares);
}
```

2- Remove the reused code from `cooldownAssets()` and `cooldownShares()` functions, then add the new function we created instead.

```diff
// StakedUSDe.sol
function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {
    if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();

    uint256 shares = previewWithdraw(assets);

-   cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
-   cooldowns[owner].underlyingAmount += assets;

-   _withdraw(_msgSender(), address(silo), owner, assets, shares);
+   _withdrawAssets(owner,assets, shares);

    return shares;
}
```
```diff
// StakedUSDe.sol
function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
    if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();

    uint256 assets = previewRedeem(shares);

-   cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
-   cooldowns[owner].underlyingAmount += assets;

-   _withdraw(_msgSender(), address(silo), owner, assets, shares);
+   _withdrawAssets(owner,assets, shares);

    return shares;
}
```

# Incorrect Commenting Syntax

NatScep syntax is preceded by triple slashes, in order for the code editor to know it and format the comment.

In `EthenaMinting.sol`, there is wrong commenting syntax [#L68](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L68).

You should follow the commenting syntax pattern to improve readability.

### Mitigation
Add an additional slash to follow the commenting sytax.





