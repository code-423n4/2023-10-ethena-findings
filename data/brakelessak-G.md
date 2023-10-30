## Pre-hash constants prior to deployment for gas savings

Relevant Files: EthenaMinting.sol / StakedUSDe.sol

Relevant Lines: L29 - 61 / L25 - 32

In EthenaMinting.sol and StakedUSDe.sol, several constants can be pre-hashed prior to deployment for a few gas savings.

Tool Used - Manual Review + Foundry Chisel

EthenaMinting.sol → L29 - 61

```solidity
/// @notice EIP712 domain
bytes32 private constant EIP712_DOMAIN =
  keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

/// @notice route type
bytes32 private constant ROUTE_TYPE = keccak256("Route(address[] addresses,uint256[] ratios)");

/// @notice order type
bytes32 private constant ORDER_TYPE = keccak256(
  "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)"
);

/// @notice role enabling to invoke mint
bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");

/// @notice role enabling to invoke redeem
bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");

/// @notice role enabling to disable mint and redeem and remove minters and redeemers in an emergency
bytes32 private constant GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");

/// @notice EIP712 domain hash
bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));

/// @notice address denoting native ether
address private constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

/// @notice EIP712 name
bytes32 private constant EIP_712_NAME = keccak256("EthenaMinting");

/// @notice holds EIP712 revision
bytes32 private constant EIP712_REVISION = keccak256("1");
```

```solidity
/// @notice EIP712 domain
/// @dev keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)")
bytes32 private constant EIP712_DOMAIN = 0x8b73c3c69bb8fe3d512ecc4cf759cc79239f7b179b0ffacaa9a75d522b39400f;

/// @notice route type
/// @dev keccak256("Route(address[] addresses,uint256[] ratios)")
bytes32 private constant ROUTE_TYPE = 0x39a103cb1f6bbcbbc2861a37eea5beb4ee817caf9963b8481d6ad1d956e953ee;

/// @notice order type
/// @dev keccak256("Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)")
bytes32 private constant ORDER_TYPE = 0xadd81430b566ae2d49ebbbd6d0bbf5717dd8781caa842362ba2b8bf25c0288b5;

/// @notice role enabling to invoke mint
/// @dev keccak256("MINTER_ROLE")
bytes32 private constant MINTER_ROLE = 0x9f2df0fed2c77648de5860a4cc508cd0818c85b8b8a1ab4ceeef8d981c8956a6;

/// @notice role enabling to invoke redeem
/// @dev keccak256("REDEEMER_ROLE")
bytes32 private constant REDEEMER_ROLE = 0x44ac9762eec3a11893fefb11d028bb3102560094137c3ed4518712475b2577cc;

/// @notice role enabling to disable mint and redeem and remove minters and redeemers in an emergency
/// @dev keccak256("GATEKEEPER_ROLE")
bytes32 private constant GATEKEEPER_ROLE = 0x3c63e605be3290ab6b04cfc46c6e1516e626d43236b034f09d7ede1d017beb0c;

/// @notice EIP712 domain hash
bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));

/// @notice address denoting native ether
address private constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

/// @notice EIP712 name
/// @dev keccak256("EthenaMinting")
bytes32 private constant EIP_712_NAME = 0xdfc7060bb2e53279833759ad83d246a6016adac9d59a3fc1c0c23c3d495e137e;

/// @notice holds EIP712 revision
/// @dev keccak256("1")
bytes32 private constant EIP712_REVISION = 0xc89efdaa54c0f20c7adf612882df0950f5a951637e0307cdcb4c672f298b8bc6;
```

StakedUSDe.sol → L25 - 32

```solidity
/// @notice The role that is allowed to distribute rewards to this contract
bytes32 private constant REWARDER_ROLE = keccak256("REWARDER_ROLE");
/// @notice The role that is allowed to blacklist and un-blacklist addresses
bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");
/// @notice The role which prevents an address to stake
bytes32 private constant SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
/// @notice The role which prevents an address to transfer, stake, or unstake. The owner of the contract can redirect address staking balance if an address is in full restricting mode.
bytes32 private constant FULL_RESTRICTED_STAKER_ROLE = keccak256("FULL_RESTRICTED_STAKER_ROLE");
```

```solidity
/// @notice The role that is allowed to distribute rewards to this contract
/// @dev keccak256("REWARDER_ROLE")
bytes32 private constant REWARDER_ROLE = 0xbeec13769b5f410b0584f69811bfd923818456d5edcf426b0e31cf90eed7a3f6;
/// @notice The role that is allowed to blacklist and un-blacklist addresses
/// @dev keccak256("BLACKLIST_MANAGER_ROLE")
bytes32 private constant BLACKLIST_MANAGER_ROLE = 0xf988e4fb62b8e14f4820fed03192306ddf4d7dbfa215595ba1c6ba4b76b369ee;
/// @notice The role which prevents an address to stake
/// @dev keccak256("SOFT_RESTRICTED_STAKER_ROLE")
bytes32 private constant SOFT_RESTRICTED_STAKER_ROLE = 0x8f7080408a06296c6347c87c115ad99669141ae35eae974c12dff8bd01680cb6;
/// @notice The role which prevents an address to transfer, stake, or unstake. The owner of the contract can redirect address staking balance if an address is in full restricting mode.
/// @dev keccak256("FULL_RESTRICTED_STAKER_ROLE")
bytes32 private constant FULL_RESTRICTED_STAKER_ROLE = 0x0a4af4bcc1942295207d9f047442ebdae6170a6e324850f758b14cf99b65c3bd;
```

## Loops can be optimized in EthenaMinting constructor()

Relevant Files: EthenaMinting.sol

Relevant Lines: L126 - 132

The loops in the `constructor()` method can be optimized using unchecked increment.

```solidity
for (uint i; i < _assets.length;) {   
  addSupportedAsset(_assets[i]);
  unchecked { ++i; }         
}

for (uint j; j < _custodians.length;) {  
  addCustodianAddress(_custodians[j]);      
  unchecked { ++j; }     
}
```

## Redundant addition in transferInRewards()

Relevant Files: StakedUSDe.sol

Relevant Lines: L89 - 99

In `transferInRewards()`, for the logic to be executed, the unvested amount must be 0.

Therefore, the addition that takes place after the initial check is redundant and will always be `amount + 0`.

This addition can be skipped entirely and the `vestingAmount` can be set directly with `amount`.

```solidity
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```

```solidity
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();

    vestingAmount = amount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```

