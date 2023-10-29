# The setMinter function does not include timelock

## Lines of code
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L23-L26 

## Affected contract
USDe

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

# Incorrect enforcement of maximum mint and redeem per block

## Lines of code
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L435-L448 

## Affected contract
EthenaMinting

## Impact
The functions `_setMaxMintPerBlock` and `_setMaxRedeemPerBlock` lack enforcement of the stated in the documentation per-block limit of 100k USDe, as there's no code to restrict the input values to this limit. This discrepancy allows for potential misuse where more than 100k USDe can be minted or redeemed per block, contrary to the specified system design, leading to financial risks.

## Proof of Concept
The documentation for the contest contains information that clearly indicates the project's predetermined limits that users recognize as enforced promises:

“Our solution is to enforce an on chain mint and redeem limitation of 100k USDe per block.”
“In case compromised MINTERS or REDEEMERS after this security implementation, a hacker can at most mint 100k USDe for no collateral, and redeem all the collateral within the contract (we will hold ~200k max),for a max loss of 300k in a single block…”

Even though the public functions can be called only by the trusted `DEFAULT_ADMIN_ROLE`, the stated limits are not enforced on chain.

```solidity
/// @notice Sets the max mintPerBlock limit
  function setMaxMintPerBlock(uint256 _maxMintPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _setMaxMintPerBlock(_maxMintPerBlock);
  }

  /// @notice Sets the max redeemPerBlock limit
  function setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _setMaxRedeemPerBlock(_maxRedeemPerBlock);
  }
```

The internal functions also contain no restrictions on the submitted values of `_maxMintPerBlock`, `_maxRedeemPerBlock`:

```solidity
  /// @notice Sets the max mintPerBlock limit
  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
    uint256 oldMaxMintPerBlock = maxMintPerBlock;
    maxMintPerBlock = _maxMintPerBlock;
    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
  }

  /// @notice Sets the max redeemPerBlock limit
  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
    maxRedeemPerBlock = _maxRedeemPerBlock;
    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
  }

```

## Tools Used
Manual review

## Recommended Mitigation Steps
Implement necessary checks within the `_setMaxMintPerBlock` and `_setMaxRedeemPerBlock` functions to enforce the 100k USDe per-block limitation for both minting and redeeming operations to adhere to the system's designed safety measures.

If the team wants to keep the ability to change the limits they should implement a separate function with a time-lock mechanism to manage adjustments to the `maxMintPerBlock` and `maxRedeemPerBlock` parameters. This function should have a predefined delay period, ensuring that any changes to the limits are deliberate and well-communicated, allowing users to anticipate and prepare for the adjustments.

# Inconsistent approach to emitting errors

## Lines of code
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L249-L251 
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L402-L405

## Affected contract
EthenaMinting

## Impact
In the `EthenaMinting` contract, the functions `transferToCustody` and `_transferToBeneficiary` handle `NATIVE_TOKEN` in two different ways, which may cause incorrect data readings. In identical use cases, sometimes the `amount` is verified and sometimes not.

## Proof of Concept
The `transferToCustody` function in the case of L249-L251 does not check whether the `amount` is acceptable. If the amount is incorrect, a `TransferFailed` error will be returned.

```solidity
    if (asset == NATIVE_TOKEN) {
      (bool success,) = wallet.call{value: amount}("");
      if (!success) revert TransferFailed();
```

In a similar case, the `_transferToBeneficiary` function verifies the `amount` passed as the parameter on line L403. If the `amount` is invalid, an `InvalidAmount` error will be returned.

```solidity
      if (address(this).balance < amount) revert InvalidAmount();
      (bool success,) = (beneficiary).call{value: amount}("");
      if (!success) revert TransferFailed();
```

## Tools Used
Manual review

## Recommended Mitigation Steps
It is important that the approach is consistent and present throughout the entire contract. The team can either:

a) add a similar check for `amount` to the `transferToCustody` function - this will allow for clearer error logging.
b) remove the verification for `amount` in the `_transferToBeneficiary` function - this will allow for slightly lower gas consumption when calling the function.

