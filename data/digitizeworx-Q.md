
## Impact

These contracts have logic to receive/hold funds:

- USDe - Holds USDe balances
- StakedUSDe - Holds staked USDe balances 
- EthenaMinting - Can receive assets like ETH

**Irrecoverable Funds**

If users accidentally send funds to these contracts, there is no way to recover them:

- No admin functions exist to withdraw funds
- Fallback/receive functions allow receiving assets
- Once in contracts, funds are permanently locked

## Proof of Concept

The USDe contract has a fallback receive function that allows receiving ETH payments:

```solidity
// USDe.sol

receive() external payable {
  // Allows receiving ETH
}
```

Once ETH is received here, there is no way to withdraw it from the contract.

Similarly, the contract inherits ERC20 token functionality which would allow it to receive and hold ERC20 tokens with no way to recover them.

To prevent accidental irrecoverable payments:

- The receive function could be removed
- A check could be added to reject ETH payments:

```solidity
receive() external payable {
  revert("Do not send ETH"); 
}
```

- An admin withdrawal function could be added

**USDe.sol**

The USDe token contract can receive and hold both ETH and ERC20 tokens with no way to recover them:

```solidity
// Allows receiving ETH
receive() external payable {}  

// ERC20 token logic allows receiving tokens
```

If ETH or tokens are sent here by mistake, they are irrecoverable.

This could occur during airdrops or if a user sends funds to the contract address accidentally. 

**Prevention**

- Remove the receive() function
- Explicitly reject ETH sends
- Implement an admin withdrawal function

**StakedUSDe.sol**

Same issues as USDe - it can receive and hold ETH and tokens with no recovery method.

Accidental sends could occur as users interact with the staking contract.

**EthenaMinting.sol**

The minting contract can receive ETH or whitelisted tokens. No withdrawal functionality.

Funds could be sent during the minting process by mistake.

**Mitigations** 

- Remove fallback/receive functions
- Reject accidental sends
- Implement admin withdrawals
- Warn users they are sending funds to a contract

## Recommended Mitigation Steps

- Use revert statements in fallbacks instead of accepting payments 

- Provide admin functions to recover assets like ETH

- Set approval limits before transferring tokens to contracts

- Pause token transfers on airdrops to prevent accidental sends

**User Warnings**

- Warn users they are sending funds to a contract address 

- Remind users to set approval limits for tokens

- Provide guidance if assets are sent to contracts by mistake
