
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
