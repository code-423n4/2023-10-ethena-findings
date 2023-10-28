
# [QA-1] `StakedUSDeV2#addToBlacklist()` User can frontrun their account blacklisting and withdraw all of their funds 

A user whose account is about to be blacklisted via the `StakedUSDeV2#addToBlacklist()` can monitor the Mempool and frontrun the transaction. That way they can withdraw all of their funds **before** being blacklisted. This results in a leak of funds by criminal activity/terrorism which would otherwise remain captured within the protocol and could later be distributed to another account.

This attack could be achieved regardless of whether cooldown is turned on or off since account restricted status is only checked in the `_withdraw()` function which with a frontrunning attack would be performed before the user is blacklisted. The `unstake()` function contains no checks for the restriction status which allows funds to be withdrawn once the cooldown period has expired.

**Recommendation:**
- Add check for restricted accounts to the `StakedUSDeV2#unstake()` function. If a user has the `FULL_RESTRICTED_STAKER_ROLE`, `mint()` the burned tokens against the blacklisted account which would enable protocol owners to distribute them.
- Use a protected private mempool such as merkle.io to avoid the `addToBlacklist()` function being frontrun.
