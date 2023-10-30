## Risk associated with extended order expiry duration
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L346

In the context of EIP712-based operations for minting or redeeming, a potential risk arises when users generate a signed order using an off-chain price. Subsequently, they submit this order to the Ethena backend, which undergoes a series of validations. The concern is that if an address with the 'Minter' role submits a signed order more than 5 minutes after its creation, the off-chain price might have changed. It's important to note that this scenario assumes that the 'Minter' role has been compromised.

One potential solution to mitigate this risk is to implement a `MAX_EXPIRY_ORDER_TIME` constant, which would restrict the execution of orders that exceed a predefined maximum expiry time.

## Risk associated with extended order expiry duration
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L235-L244
Within EthenaMinting, there exists a function that allows smart contracts to delegate or undelegate an address for signing. However, it lacks a comprehensive validation process.

```diff
/// @notice Enables smart contracts to delegate an address for signing
    function setDelegatedSigner(address _delegateTo) external {
+       if (delegatedSigner[_delegateTo][msg.sender]) revert SignerExists;
        delegatedSigner[_delegateTo][msg.sender] = true;
        emit DelegatedSignerAdded(_delegateTo, msg.sender);
    }

    /// @notice Enables smart contracts to undelegate an address for signing
    function removeDelegatedSigner(address _removedSigner) external {
+       if (!delegatedSigner[_delegateTo][msg.sender]) revert SignerNotExists;
        delegatedSigner[_removedSigner][msg.sender] = false;
        emit DelegatedSignerRemoved(_removedSigner, msg.sender);
    }
```