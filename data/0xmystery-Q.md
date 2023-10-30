## Use an efficient logic in setter functions
When intending to emit both the old and new values, there isn't a need to cache the old value that will only be used once. Simply emit both values before assigning a new value to the state variable. For example, the following setter function may be refactored as follows:

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L436-L440

```diff
  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
+    emit MaxMintPerBlockChanged(maxMintPerBlock, _maxMintPerBlock);
-    uint256 oldMaxMintPerBlock = maxMintPerBlock;
    maxMintPerBlock = _maxMintPerBlock;
-    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
  }
```
All other instances entailed:

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L442-L447

```solidity
  /// @notice Sets the max redeemPerBlock limit
  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
    maxRedeemPerBlock = _maxRedeemPerBlock;
    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L126-L134

```solidity
  function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (duration > MAX_COOLDOWN_DURATION) {
      revert InvalidCooldown();
    }

    uint24 previousDuration = cooldownDuration;
    cooldownDuration = duration;
    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
  }
```
## Be consistent in the code logic when emitting events in setter functions
By convention, it's recommended emitting the old value followed by the new one in the same event instead of the other way round.

Here's one instance found:

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L23-L26

```diff  
  function setMinter(address newMinter) external onlyOwner {
-    emit MinterUpdated(newMinter, minter);
+    emit MinterUpdated(minter, newMinter);
    minter = newMinter;
  }
```
## Delta neutrality caution
Users should be cautioned about the impermanent losses entailed arising from the delta-neutral stability strategy adopted by the protocol, specifically if the short positions were to encounter hefty losses. Apparently, the users could have held on to their collateral, e.g. `stETH or WETH`, and ended up a lot richer with the equivalent amount of `USDe`. I suggest all minting entries to begin with stable coins like `USDC, DAI etc` that could be converted to `stETH` to generate yield if need be instead of having users depositing `stETH` from their wallet reserves. Psychologically, this will make the users feel better as the mentality has been fostered more on preserving the 1:1 peg of `USDe` at all times. 

## Easy DoS on big players when minting and redeeming in EthenaMinting.sol
As indicated on the contest description, users intending to mint/redeem a large amount will need to mint/redeem over several blocks due to `maxMintPerBlock` or `maxRedeemPerBlock`. However, these RFQ's are prone to DoS because [`mintedPerBlock[block.number] + mintAmount > maxMintPerBlock`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L98) or [`redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L105) could revert by only 1 wei in excess.

While these issues could be sorted by the backend to make a full use of `maxMintPerBlock` or `maxRedeemPerBlock` per block, it will make the intended logic a lot more efficient by auto reducing the RFQ amount to perfectly fill up the remaining quota for the current block. Better yet, set up a queue system where request amount running in hundreds of thousands or millions may be auto split up with multiple orders via only one signature for batching.

## Inexpedient code lines
In the function below, the if block already dictates that `getUnvestedAmount() == 0` manages to avoid a revert. Hence, consider refactoring the following code lines as it makes no sense adding 0 value `getUnvestedAmount()` of to the addend, `amount`:   
      
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89-L99

```diff
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();

-    vestingAmount = newVestingAmount;
+    vestingAmount = amount;

    // The rest of the codes
  }
```
## Emission of identical values
Under the context of the above/preceding recommendation, `transferInRewards()` should also have its `emit` refactored below. Otherwise, you are practically emitting two identical values that defeat the purpose of contrasting the old and the values.

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89-L99

```diff
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();

-    vestingAmount = newVestingAmount;
+    emit RewardsReceived(vestingAmount, amount);
+    vestingAmount = amount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

-    emit RewardsReceived(amount, newVestingAmount);
  }
```
## Typo mistakes
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L94
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L110

```diff
-  /// @param owner address to redeem and start cooldown, owner must allowed caller to perform this action
+  /// @param owner address to redeem and start cooldown, owner must allow caller to perform this action
```