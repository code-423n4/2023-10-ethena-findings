## [G-01] Set the constructor as payable
One can save circa 10 opcodes and a bit of gas if the constructors are set to payable.
But, there are risks due to payable constructors ability to take ETH when deploying.
**Locations**
```sol
// contracts/USDe.sol#L18-L21
  constructor(address admin) ERC20("USDe", "USDe") ERC20Permit("USDe") {
    if (admin == address(0)) revert ZeroAddressException();
    _transferOwnership(admin);
  }
// contracts/USDeSilo.sol#L18-L21
  constructor(address stakingVault, address usde) {
    STAKING_VAULT = stakingVault;
    USDE = IERC20(usde);
  }

// contracts/EthenaMinting.sol#L111-L146
  constructor(
    IUSDe _usde,
    address[] memory _assets,
    address[] memory _custodians,
    address _admin,
    uint256 _maxMintPerBlock,
    uint256 _maxRedeemPerBlock
  ) {
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


    // Set the max mint/redeem limits per block
    _setMaxMintPerBlock(_maxMintPerBlock);
    _setMaxRedeemPerBlock(_maxRedeemPerBlock);


    if (msg.sender != _admin) {
      _grantRole(DEFAULT_ADMIN_ROLE, _admin);
    }


    _chainId = block.chainid;
    _domainSeparator = _computeDomainSeparator();


    emit USDeSet(address(_usde));
  }

// contracts/StakedUSDeV2.sol#L42-L45
  constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {
    silo = new USDeSilo(address(this), address(_asset));
    cooldownDuration = MAX_COOLDOWN_DURATION;
  }

// contracts/StakedUSDe.sol#L70-L81
  constructor(IERC20 _asset, address _initialRewarder, address _owner)
    ERC20("Staked USDe", "stUSDe")
    ERC4626(_asset)
    ERC20Permit("stUSDe")
  {
    if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {
      revert InvalidZeroAddress();
    }


    _grantRole(REWARDER_ROLE, _initialRewarder);
    _grantRole(DEFAULT_ADMIN_ROLE, _owner);
  }
```
**Remediation**
Set the constructors to payable which will save gas. 
Check it will not lead to bad effects within the instance of an upgrade pattern.
## [G-02] ABI Encode costs more gas than ABI Encode Packed
This contract utilises abi.encode() within the method called encodeOrder. 
Within abi.encode(), the main types are padded to 32 bytes with dynamic arrays including their length, but abi.encodePacked() just utilises the minimum needed memory to encode this data.
**Location**
```sol
// contracts/EthenaMinting.sol#L321-L321
    return abi.encode(

// contracts/EthenaMinting.sol#L335-L335
    return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);

// contracts/EthenaMinting.sol#L452-L452
    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```
**Remediation**
If not required, then utilise abi.encodePacked() rather than abi.encode().
## [G-03] Functions that are internal are not being called
There are internal functions that are not being called anywhere in the contract.
Internal functions are able to be called just within that contracts, and they are not being called within the contract. 
When internal functions are built and not called then they cost unnecessary gas and are more complicated.
**Locations**
```sol
// contracts/StakedUSDe.sol#L245-L252
  function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {
      revert OperationNotAllowed();
    }
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
      revert OperationNotAllowed();
    }
  }
```
**Remediation**
If internal functions are not being called then remove them from the contract.