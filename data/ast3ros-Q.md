# [NC-1] Custodian length can be zero when EthenaMinting is created.

The _custodians array is not checked for zero length in constructor. If there is no custodian set, functionality of the EthenalMinting such as `mint` can be blocked. 

        if (address(_usde) == address(0)) revert InvalidUSDeAddress();
        if (_assets.length == 0) revert NoAssetsProvided();
        if (_admin == address(0)) revert InvalidZeroAddress();
        usde = _usde;

        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);

        for (uint256 i = 0; i < _assets.length; i++) {
            addSupportedAsset(_assets[i]);
        }

        for (uint256 j = 0; j < _custodians.length; j++) {
            addCustodianAddress(_custodians[j]);
        }

https://github.com/code-423n4/2023-10-ethena/blob/de36fe47c5d2c47e739b8e07fc69bd24f28e5762/contracts/EthenaMinting.sol#L119-L132

To address this issue, add checking length for _custodians array.

        if (_custodians.length == 0) revert NoAssetsProvided();

# [NC-2] Unnecessary calculating getUnvestedAmount() in newVestingAmount

The getUnvestedAmount is unnecessary in calculating `newVestingAmount` since getUnvestedAmount is equal 0.

        if (getUnvestedAmount() > 0) revert StillVesting();
        uint256 newVestingAmount = amount + getUnvestedAmount();

https://github.com/code-423n4/2023-10-ethena/blob/de36fe47c5d2c47e739b8e07fc69bd24f28e5762/contracts/StakedUSDe.sol#L91
