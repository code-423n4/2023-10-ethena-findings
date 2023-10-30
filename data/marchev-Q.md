
# [QA-1] `StakedUSDeV2#addToBlacklist()` User can frontrun their account blacklisting and withdraw all of their funds 

A user whose account is about to be blacklisted via the `StakedUSDeV2#addToBlacklist()` can monitor the Mempool and frontrun the transaction. That way they can withdraw all of their funds **before** being blacklisted. This results in a leak of funds by criminal activity/terrorism which would otherwise remain captured within the protocol and could later be distributed to another account.

This attack could be achieved regardless of whether cooldown is turned on or off since account restricted status is only checked in the `_withdraw()` function which with a frontrunning attack would be performed before the user is blacklisted. The `unstake()` function contains no checks for the restriction status which allows funds to be withdrawn once the cooldown period has expired.

**Recommendation:**
- Add check for restricted accounts to the `StakedUSDeV2#unstake()` function. If a user has the `FULL_RESTRICTED_STAKER_ROLE`, `mint()` the burned tokens against the blacklisted account which would enable protocol owners to distribute them.
- Use a protected private mempool such as merkle.io to avoid the `addToBlacklist()` function being frontrun.

# [QA-2] `StakedUSDeV2#cooldownAssets()` Cooldown period is reset if user tries to withdraw more funds before the expiry of their previous withdrawal

The cooldown mechanism has a flaw: it resets if a user attempts to withdraw more funds before their previous withdrawal has expired.

Example scenario:

1. Alice deposits funds in a `StakedUSDeV2` pool with a cooldown period of $P$.
2. Alice initiates a withdrawal $W_1$ using `cooldownAssets()` or `cooldownShares()`, setting the cooldown expiry at $C_1 = T_1 + P$, where $T_1$ is the current `block.timestamp`.
3. 1. Alice queues another withdrawal $W_2$ using the same methods, but this action resets her account's cooldown period to $C_2 = T_2 + P$, where $T_2$ is the current `block.timestamp` when she initiated her second withdrawal.

The issue here is that because there is only one cooldown entry allowed per account, in the given scenario, Alice can only unstake her $W_1$ withdrawal after the $C_2$ cooldown period has passed, even though she should be able to do so after $C_1$ expires.

**Recommendation:**
To prevent this problem and allow rightful fund unstaking, implement a mechanism that assigns a separate cooldown period to each withdrawal.

# [QA-3] `vestingAmount` in `transferInRewards()` will always be equal to `amount` being transferred in

Currently, the implementation of `transferInRewards()` calculates the new `vestingAmount` like this:

```sol
if (getUnvestedAmount() > 0) revert StillVesting();
uint256 newVestingAmount = amount + getUnvestedAmount();

vestingAmount = newVestingAmount;
```

But based on the `if` statement just before the assignment, `getUnvestedAmount()` will always be equal to `0`, thus essentially `uint256 newVestingAmount = amount`. This is superfluous and costs unnecessary gas upon each invocation of `transferInRewards()`. It also makes the code unnecessarily complex and leads to poor readability.

**Recommendation:**
Simplify the implementation by assigning `vestingAmount` to `amount` directly:
```diff
--- a/contracts/StakedUSDe.sol
+++ b/contracts/StakedUSDe.sol
@@ -88,9 +88,8 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
    */
   function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
     if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();

-    vestingAmount = newVestingAmount;
+    vestingAmount = amount;
     lastDistributionTimestamp = block.timestamp;
     // transfer assets from rewarder to this contract
     IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
```

[QA-4] Wrong error thrown in case of invalid order beneficiary

The `EthenaMinting#verifyOrder()` function throws a wrong error in case of invalid beneficiary address. It throws `InvalidAmount()` but it should be `InvalidAddress()` instead.

```sol
if (order.beneficiary == address(0)) revert InvalidAmount(); //@audit Incorrect error is thrown here
```

**Recommendation:**
Throw `InvalidZeroAddress()` error instead of `InvalidAmount()`:
```diff
@@ -340,7 +340,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     bytes32 taker_order_hash = hashOrder(order);
     address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
     if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
-    if (order.beneficiary == address(0)) revert InvalidAmount();
+    if (order.beneficiary == address(0)) revert InvalidZeroAddress();
     if (order.collateral_amount == 0) revert InvalidAmount();
     if (order.usde_amount == 0) revert InvalidAmount();
     if (block.timestamp > order.expiry) revert SignatureExpired();
```