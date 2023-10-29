# Concerns regarding protocol design

### Minting and Redemption

The ratio between collateral and USDe amount either issued or burned is directly dependent on the price, which is acquired through completely off-chain methods. What is left implicit in this design choice is the fact that the redeemer will not only operate as the peg controller but also as the price oracle. This added centralization risk is one that needs to be taken into account by the sponsors.

Gatekeepers are in charge of keeping watch on off-putting mint prices; however, under heavy market turmoil, it is likely that prices are going to range a lot between exchanges. For example, during the March 2020 crash, BitMEX froze its BTC/USD exchange price at around 4k by shutting down the exchange. This is a double-edged knife, since if gatekeepers are too strict, the protocol will stop operating at high volume, but if it's too soft, a more careful compromised minter will be able to extract much larger amounts of value from the protocol by steadily minting lower amounts every so often.

### Role restriction

Since the project is to be implemented on the Ethereum main net, both `SOFT_RESTRICTED_STAKER_ROLE` and `FULL_RESTRICTED_STAKER_ROLE` targets can use MEV manipulation in order to withdraw their staked USDe and avoid losing funds. It is recommended that whenever the protocol intends to assign these roles, you submit it via a private mempool such as flashbots in order to prevent front running. Alternatively, the protocol could consider expanding the blacklist to tokens in the silo as well, though that would imply freezing USDe instead.

### Delta neutral position management

There is actually another way for the protocol to leak value that wasn't mentioned before. When opening a perpetual contract position, like with any other trade, it is susceptible to slippage losses due to a large position entering the market with relatively low liquidity. It's important to be aware of the fact stETH is much less liquid than ETH as well. The protocol can simply choose to eat the slippage, but depending on when or what exchange this is done, it is possible the protocol assumes some percentage losses. 

This is critical to the yield the protocol offers, since if a 7 or 8 figures position is opened, it could get away with several months of earned yield. This also applies for closing positions, so if there is a very large withdrawal demand and the protocol needs that extra collateral to cover it, these losses will occur. Since it's a concern so external to the contracts' scope, I decided to leave this in the analysis.

### Redistribution of compromised funds

If the administrator chooses to burn a blocked users' stUSDe, they can choose not to mint the token amount for a desired owner, as shown below. The protocol should be very wary of this design choice, as choosing to simply redistribute busted funds can have serious legal implications.

```solidity
      // to address of address(0) enables burning
      if (to != address(0)) _mint(to, amountToDistribute);
```

# Overall quality

The code is very stubborn due to the fact that the protocol has gone through a few audits already. It's worth noting as well that stablecoin projects that rely on centralized actors for key operations are usually one of the most difficult protocols to find vulnerabilities in. One of the two contests that had no high or medium findings was an audit for a stablecoin project. This protocol is not different in this regard. The test coverage is also very high in detail, it's not very often that you see this level of care.

### Time spent:
7.5 hours