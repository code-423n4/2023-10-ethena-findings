Emit events before external calls

In the project, several functions emit an event as their final statement. Whenever feasible, prioritize emitting events before external calls. While funds are not at risk for external call + event ordering in cases of reentrancy, emitting events after external calls can potentially disrupt frontends and monitoring tools during reentrancy attacks.

function setDelegatedSigner(address _delegateTo) external {
        delegatedSigner[_delegateTo][msg.sender] = true;
        emit DelegatedSignerAdded(_delegateTo, msg.sender);
}

function removeDelegatedSigner(address _removedSigner) external {
        delegatedSigner[_removedSigner][msg.sender] = false;
        emit DelegatedSignerRemoved(_removedSigner, msg.sender);
}

 function transferToCustody(
        address wallet,
        address asset,
        uint256 amount
    ) external nonReentrant onlyRole(MINTER_ROLE) {
        if (wallet == address(0) || !_custodianAddresses.contains(wallet))
            revert InvalidAddress();
        if (asset == NATIVE_TOKEN) {
            (bool success, ) = wallet.call{value: amount}("");
            if (!success) revert TransferFailed();
        } else {
            IERC20(asset).safeTransfer(wallet, amount);
        }
        // CustodyTransfer 的事件，它通知区块链网络有资产转移到了托管钱包。
        emit CustodyTransfer(wallet, asset, amount);
}

function removeSupportedAsset(
        address asset
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();
        emit AssetRemoved(asset);
    }

function removeCustodianAddress(
        address custodian
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        if (!_custodianAddresses.remove(custodian))
            revert InvalidCustodianAddress();
        emit CustodianAddressRemoved(custodian);
    }

