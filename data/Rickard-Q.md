# [L-01] Inconsistent behaviour in `renounceOwnership` 
[https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L33-L35](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L33-L35)

The function `renounceOwnership` currently only contains a `revert` message in the body, which results in a waste of gas and improper use of the function. Furthermore, the function's name does not properly reflect the intention and purpose of the error message. Also, the `onlyOwner` modifier seems pointless in this particular case. Due to time constraints and the contest deadline, I was unable to reach one of the sponsors for further information.
````solidity
33:    function renounceOwnership() public view override onlyOwner {
34:      revert CantRenounceOwnership();
35:    }
````
## Recommended Mitigation Steps
It would be helpful to consider renaming the function to something like `haltRenounceOwnership`, which would make it easier for both users and admins to understand the code. Additionally, it may be worth considering changing it to an error message and adding a simple check, such as `if (msg.sender == owner) revert`, to improve the functionality.
# [L-02] Restrict security measure `disableMintRedeem`
[https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L228-L232](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L228-L232)
````solidity
228:  /// @notice Disables the mint and redeem
229:  function disableMintRedeem() external onlyRole(GATEKEEPER_ROLE) {
230:    _setMaxMintPerBlock(0);
231:    _setMaxRedeemPerBlock(0);
232:  }
````
It's important to note that the `disableMintRedeem` function in the contract grants a lot of power to entities with the `GATEKEEPER_ROLE`. They can use it to disable minting and redeeming, which can be necessary in emergencies or certain operational situations. However, if misused, it can pose significant risks.
## Recommended Mitigation Steps
To make it more secure, it's recommended to consider implementing a timelock mechanism.
# [L-03] Missing Validation for `_setMaxMintPerBlock`/`_setMaxRedeemPerBlock`
[https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L435-L440](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L435-L440)      
[https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L442-L447](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L442-L447)

The functions `_setMaxMintPerBlock` and `_setMaxRedeemPerBlock` are responsible for setting the maximum limit for minting and redeeming tokens per block, respectively. However, these functions lack a validation step to ensure that the new value being set is greater than the old value. This means that it's possible to accidentally set a new maximum limit that is equal to or less than the previous value, which could result in unexpected outcomes.
## Recommended Mitigation Steps
````diff
/// @notice Sets the max mintPerBlock limit
  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
    uint256 oldMaxMintPerBlock = maxMintPerBlock;
+   require(_maxMintPerBlock > oldMaxMintPerBlock, "New value must be greater than old value");
    maxMintPerBlock = _maxMintPerBlock;
    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
  }

  /// @notice Sets the max redeemPerBlock limit
  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
+   require(_maxRedeemPerBlock > oldMaxRedeemPerBlock, "New value must be greater than old value");
    maxRedeemPerBlock = _maxRedeemPerBlock;
    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
  }
````
# [L-04] Cross-Chain Replay Attack
When the constructor of the `USDE` token runs, it determines the `_chainId` that it should assign and stores it in a `uint256 private immutable`. In case Ethereum undergoes a fork in the future, the `_chainId` will be altered, but the permits will still be associated with the old chainid. This will make it impossible for users to use any new permits on the forked chain, thus rendering the token unusable on the new chain permanently.
Please consult EIP1344 for more details: [https://eips.ethereum.org/EIPS/eip-1344#rationale](https://eips.ethereum.org/EIPS/eip-1344#rationale)
## Recommended Mitigation Steps
To mitigate potential issues, it is recommended to calculate the `_chainId` dynamically for each invocation. As a gas optimization, the deployment pre-calculated hash for the permits can be stored in an immutable variable. The permit function can then be validated to ensure the current `_chainId` matches the cached hash. If it does not match, recalculate it immediately.
# [I-01] Typos
Fix the typos in `contracts/StakedUSDeV2.sol`:
- `allowed` -> `allow`      
[https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L94](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L94)
- `allowed` -> `allow`      
[https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L110](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L110)