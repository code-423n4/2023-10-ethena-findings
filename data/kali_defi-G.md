[G-01] |missing unsupported asset transfer check
          in EthenaMinting::transferToCustody function didn't check if  assest is a supported asset.
          Recommended to add a check before calling safetransfer function.
          This will save gas for the case when asset is not supported.EthenaMinting.sol#L247-L256
      function transferToCustody(
          address wallet,
          address asset,
          uint256 amount
         ) external nonReentrant onlyRole(MINTER_ROLE) {
          if (wallet == address(0) || !_custodianAddresses.contains(wallet))
            revert InvalidAddress();
            ...
          else {
    +        if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();
             IERC20(asset).safeTransfer(wallet, amount);
          }

[G-02] | In ethenaMinting.sol::verifyRoute to cach totalRatio we defined new variable uint256 totalRatio and use 
         it after two if condetion but instead of this we can define this variable after two if condition and use 
         it to save gas.
         This will save gas for the case when if condition return false. EthenaMinting.sol#L351-L374

  -       uint256 totalRatio;
        if (route.addresses.length != route.ratios.length) {
            return false;
        }
        if (route.addresses.length == 0) {
            return false;
        }
 +       uint256 totalRatio;

        
[G-03] | In ethenaMinting.sol::_deduplicateOrder we can get invalidatorStorage mapping from verifyNonce() 
         function and use it to save gas. EthenaMinting.sol#L391-L396
          So verifyNonce() return (true, invalidatorSlot, invalidator, invalidatorBit, invalidatorStorage) 
          instead of returning (true, invalidatorSlot, invalidator, invalidatorBit) 

[G-04] | In ethenaMinting.sol::_transferCollateral function didn't check if (benefactor.balance < amount). 
           Recommended to add a check before calling safetransfer function. EthenaMinting.sol#L413-L433
        function _transferCollateral(
         uint256 amount,
         address asset,
         address benefactor,
         address[] calldata addresses,
         uint256[] calldata ratios
         ) internal {
        ...
 +       if (benefactor.balance < amount) revert InvalidAmount();
        ...
