## [G-01] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate
We can combine multiple mappings below into structs. This will result in cheaper storage reads since
multiple mappings are accessed in functions and those values are now occupying the same storage
slot, meaning the slot will become warm after the first SLOAD. In addition, when writing to and reading
from the struct values we will avoid a Gsset (20000 gas) and Gcoldsload (2100 gas) since multiple
struct values are now occupying the same slot.

```
 /// @notice user deduplication
  mapping(address => mapping(uint256 => uint256)) private _orderBitmaps;

  /// @notice USDe minted per block
  mapping(uint256 => uint256) public mintedPerBlock;
  /// @notice USDe redeemed per block
  mapping(uint256 => uint256) public redeemedPerBlock;

  /// @notice For smart contracts to delegate signing to EOA address
  mapping(address => mapping(address => bool)) public delegatedSigner;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L77C2-L86C71


## [G-02] Make for loop unchecked
The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is
limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time
and gas to let the for loop overflow.

```
  for (uint256 i = 0; i < addresses.length; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L424C3-L428C6

# [G-03] Use double if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2
if statements.

```
 if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
      uint256 amountToDistribute = balanceOf(from);
      _burn(from, amountToDistribute);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L149C4-L151C39

```
  if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();
  }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L193C3-L195C1

```
 if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {
      revert OperationNotAllowed();
    }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L246C4-L248C6

## [G-04] Caching global variables is expensive than using the variable itself

```
  _chainId = block.chainid;
    _domainSeparator = _computeDomainSeparator();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L142C3-L144C1

```
  lastDistributionTimestamp = block.timestamp;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L94C3-L94C49

## [G-05] Use a more recent version of solidity
Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining 
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads 
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings 
Use asolidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call
has a return value.

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L2C1-L2C24

```
pragma solidity 0.8.19;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L2C24-L2C24

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L2

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L2

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L2

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L2

## [G-06] Avoid unnecessary read of array length in for loops can save gas

```
  for (uint256 i = 0; i < _assets.length; i++) {
      addSupportedAsset(_assets[i]);
    }

    for (uint256 j = 0; j < _custodians.length; j++) {
      addCustodianAddress(_custodians[j]);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126C3-L131C43

```
 for (uint256 i = 0; i < addresses.length; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L424C4-L428C6

## [G-07] Use hardcode address instead address(this)
Instead of using address(this) , it is more gas-efficient to pre-calculate and use the hardcoded
address . Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824

```
 if (address(this).balance < amount) revert InvalidAmount();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L403C6-L403C66


```
   IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96C2-L96C73

```
 return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();
  }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L167C4-L168C4

```
  silo = new USDeSilo(address(this), address(_asset));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L43C3-L43C57

## [G-08] `variable == false` instead of `!variable`.
a bit cheapier when you replace:

```
  if (!success) revert TransferFailed();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L251C6-L251C45

```
 if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();
    emit AssetRemoved(asset);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L260C4-L261C30

```
 if (!_custodianAddresses.remove(custodian)) revert InvalidCustodianAddress();
    emit CustodianAddressRemoved(custodian);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L271C4-L272C45

## [G-09] Expressions for constant values such as a call to keccak256() ,
should use immutable rather than constant

```
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

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L27C2-L59C1

```
 /* ------------- CONSTANTS ------------- */
  /// @notice The role that is allowed to distribute rewards to this contract
  bytes32 private constant REWARDER_ROLE = keccak256("REWARDER_ROLE");
  /// @notice The role that is allowed to blacklist and un-blacklist addresses
  bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");
  /// @notice The role which prevents an address to stake
  bytes32 private constant SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
  /// @notice The role which prevents an address to transfer, stake, or unstake. The owner of the contract can redirect address staking balance if an address is in full restricting mode.
  bytes32 private constant FULL_RESTRICTED_STAKER_ROLE = keccak256("FULL_RESTRICTED_STAKER_ROLE");
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L24C2-L32C99

## [G-10] abi.encode() is less efficient than abi.encodePacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost.

```
 return abi.encode(
      ORDER_TYPE,
      order.order_type,
      order.expiry,
      order.nonce,
      order.benefactor,
      order.beneficiary,
      order.collateral_asset,
      order.collateral_amount,
      order.usde_amount
    );
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L321C4-L321C23

```
  return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L335C3-L335C66

```
 return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
  }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L452C4-L453C4