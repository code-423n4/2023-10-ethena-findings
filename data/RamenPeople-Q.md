# The setMinter function does not include time-lock (LOW)

## Lines of code
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L23-L26 

## Affected contract
USDe.sol

## Impact
In the case of insider attack or key compromise of the USDe owner (and it could be an ordinary EOA, as the `USDe.sol` contract allows for transferring ownership), they can immediately select a new address as `minter` and start minting tokens.

*There are limits imposed on the `mint` function (`maxMintPerBlock`) in the `EthenaMinting`, but they are not present in `USDe`. This risk can be reduced further.

## Proof of Concept
Currently, the `setMinter` function has no delay. The address can be selected as `minter` immediately and start minting `USDe`.

```solidity
function setMinter(address newMinter) external onlyOwner {
    emit MinterUpdated(newMinter, minter);
    minter = newMinter;
  }
```

## Tools Used
Manual review

## Recommended Mitigation Steps

Use a timelock on `setMinter` (e.g. the one from openzeppelin) so that the change of the minter occurs with a delay. This will allow for significant risk reduction at a relatively low cost. 

In the case of insider attack or key compromise of the USDe owner, the attacker changing the `minter` address will have to wait a selected amount of time before they can start minting tokens. This will allow both the team and users to react.

What's more, the proposed recommendation works great with the existing security feature `GATEKEEPER_ROLE` as it will have enough time to reduce the protocol losses from ~100k to 0 by calling the `disableMintRedeem` function.
