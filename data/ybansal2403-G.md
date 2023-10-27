# Summary

Some optimizations were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.19`, `optimizer enabled`, and `2000 runs`. Optimizations that were not benchmarked are explained properly. The following command was used to run the tests for the benchmarks below: `forge test` and the gas report was generated using `forge test --gas-report`

## Gas Optimizations

| Number        | Issue                                                                                          | Instances |
| ------------- | :--------------------------------------------------------------------------------------------- | :-------: |
| [G-01](#g-01) | State variables that are used multiple times in a function should be cached in stack variables |     2     |
| [G-02](#g-02) | Use `do while` loops instead of `for loops`                                                    |     2     |
| [G-03](#g-03) | Do not calculate constants                                                                     |    13     |
| [G-04](#g-04) | Use hardcode address instead of address(this)                                                  |     3     |

<a name='g-01'></a>

## [G-01] State variables that are used multiple times in a function should be cached in stack variables

_The following instances are missed in the automated report_

When performing multiple operations on a state variable in a function, it is recommended to cache it first. Either multiple reads or multiple writes to a state variable can save gas by caching it on the stack. Caching of a state variable replaces each Gwarmaccess `100 gas` with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. Saves 100 gas per instance.

Total Instances: `2`

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L100C1-L101C49

Gas savings for `StakedUSDeV2.cooldownAssets`, obtained via protocol's tests:

|        | Min | Avg   | Median | Max   | # calls |
| ------ | --- | ----- | ------ | ----- | ------- |
| Before | 630 | 53646 | 70591  | 87449 | 9       |
| After  | 630 | 26250 | 39100  | 47497 | 9       |

```solidity
File: contracts/StakedUSDeV2.sol

//@audit  cooldowns[owner] at line 100,101
100: cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
101:    cooldowns[owner].underlyingAmount += assets;

```

```diff
diff --git a/contracts/StakedUSDeV2.sol b/contracts/StakedUSDeV2.sol
index df2bb48..d59e506 100644
--- a/contracts/StakedUSDeV2.sol
+++ b/contracts/StakedUSDeV2.sol
@@ -96,9 +96,9 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {
     if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();

     uint256 shares = previewWithdraw(assets);
-
-    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
-    cooldowns[owner].underlyingAmount += assets;
+   UserCooldown memory cooldown_owner =  cooldowns[owner];
+   cooldown_owner.cooldownEnd = uint104(block.timestamp) + cooldownDuration;
+   cooldown_owner.underlyingAmount +=  assets;

     _withdraw(_msgSender(), address(silo), owner, assets, shares);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L116C1-L117C49

Gas savings for `StakedUSDeV2.cooldownShares`, obtained via protocol's tests:

|        | Min | Avg   | Median | Max   | # calls |
| ------ | --- | ----- | ------ | ----- | ------- |
| Before | 631 | 58837 | 69459  | 87674 | 12      |
| After  | 631 | 29628 | 38047  | 47720 | 10      |

```solidity
File: contracts/StakedUSDeV2.sol

//@audit  cooldowns[owner] at line 116,117
116:    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
117:    cooldowns[owner].underlyingAmount += assets;

```

```diff
diff --git a/contracts/StakedUSDeV2.sol b/contracts/StakedUSDeV2.sol
index df2bb48..ad717c9 100644
--- a/contracts/StakedUSDeV2.sol
+++ b/contracts/StakedUSDeV2.sol
@@ -112,9 +112,9 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {
     if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();

     uint256 assets = previewRedeem(shares);
-
-    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
-    cooldowns[owner].underlyingAmount += assets;
+    UserCooldown memory cooldown_owner =  cooldowns[owner];
+    cooldown_owner.cooldownEnd = uint104(block.timestamp) + cooldownDuration;
+    cooldown_owner.underlyingAmount += assets;

     _withdraw(_msgSender(), address(silo), owner, assets, shares);

```

<a name='g-02'></a>

## [G-02] Use `do while` loops instead of `for loops`

A do while loop will cost less gas since the condition is not being checked for the first iteration. Other optimizations includes caching length, preicrementing,non-initialisation of variables with default values and using unchecked blocks

Total Instances: `2`

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126C5-L128C6

\_Gas Savings for ` contracts/EthenaMinting.sol:EthenaMinting contract`, obtained via protocol's tests:

|        | Deployment Cost |
| ------ | --------------- |
| Before | 3576457         |
| After  | 3575976         |

```solidity
File: contracts/EthenaMinting.sol
126:  for (uint256 i = 0; i < _assets.length; i++) {
127:        addSupportedAsset(_assets[i]);
128:      }

```

```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..3a9b2b7 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -122,10 +122,14 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     usde = _usde;

     _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
-
-    for (uint256 i = 0; i < _assets.length; i++) {
+    uint256 i;
+    uint256 _assets_length = _assets.length;
+    do {
       addSupportedAsset(_assets[i]);
-    }
+      unchecked{
+        ++i;
+      }
+    }while(i < _assets_length);

     for (uint256 j = 0; j < _custodians.length; j++) {
       addCustodianAddress(_custodians[j]);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L130C5-L132C6

\_Gas Savings for ` contracts/EthenaMinting.sol:EthenaMinting contract`,obtained via protocol's tests:

|        | Deployment Cost |
| ------ | --------------- |
| Before | 3576457         |
| After  | 3576361         |

```solidity
File: contracts/EthenaMinting.sol
130:  for (uint256 j = 0; j < _custodians.length; j++) {
131:      addCustodianAddress(_custodians[j]);
132:    }

```

```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..f96eda3 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -127,9 +127,14 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
       addSupportedAsset(_assets[i]);
     }

-    for (uint256 j = 0; j < _custodians.length; j++) {
+    uint256 j;
+    uint256 _custodians_length = _custodians.length;
+    do {
       addCustodianAddress(_custodians[j]);
-    }
+      unchecked{
+        ++j;
+      }
+    }while(j < _custodians_length);

     // Set the max mint/redeem limits per block
     _setMaxMintPerBlock(_maxMintPerBlock);

```

<a name='g-03'></a>

## [G-03] Do not calculate constants

The reason for this is that constant variables are evaluated at runtime and their value is included in the bytecode of the contract. This means that any expensive operations performed as part of the constant expression, such as a call to keccak256(), will be executed every time the contract is deployed, even if the result is always the same. This can result in higher gas costs.

In contrast, immutable variables are evaluated at compilation time, and their values are included in the bytecode of the contract as constants. This means that any expensive operations performed as part of the immutable expression are only executed once, when the contract is compiled, and the result is reused every time the contract is deployed. This can result in lower gas costs compared to using constant variables.

Let's consider an example to illustrate this. Suppose we want to store the hash of a string as a constant value in our contract. We could do this using a constant variable, like so:

```solidity
bytes32 constant MY_HASH = keccak256("my string");
```

Alternatively, we could use an immutable variable, like so:

```solidity
bytes32 immutable MY_HASH = keccak256("my string");
```

Total Instances: `13`

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L27C3-L58C61

```solidity
File:src/WildcatSanctionsSentinel.sol

27:/// @notice EIP712 domain
28:  bytes32 private constant EIP712_DOMAIN =
29:    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
30:
31:  /// @notice route type
32:  bytes32 private constant ROUTE_TYPE = keccak256("Route(address[] addresses,uint256[] ratios)");
33:
34:  /// @notice order type
35:  bytes32 private constant ORDER_TYPE = keccak256(
36:    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)"
37:  );
38:
39:  /// @notice role enabling to invoke mint
40:  bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");
41:
42:  /// @notice role enabling to invoke redeem
43:  bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");
44:
45:  /// @notice role enabling to disable mint and redeem and remove minters and redeemers in an emergency
46:  bytes32 private constant GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");
47:
48:  /// @notice EIP712 domain hash
4950::  bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));
50:
51:  /// @notice address denoting native ether
52:  address private constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;
53:
54:  /// @notice EIP712 name
55:  bytes32 private constant EIP_712_NAME = keccak256("EthenaMinting");
56:
57:  /// @notice holds EIP712 revision
58:  bytes32 private constant EIP712_REVISION = keccak256("1");

```

```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..ce3d5a2 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -25,37 +25,34 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
   /* --------------- CONSTANTS --------------- */

   /// @notice EIP712 domain
-  bytes32 private constant EIP712_DOMAIN =
-    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
+  bytes32 private immutable EIP712_DOMAIN;

   /// @notice route type
-  bytes32 private constant ROUTE_TYPE = keccak256("Route(address[] addresses,uint256[] ratios)");
+  bytes32 private immutable ROUTE_TYPE;

   /// @notice order type
-  bytes32 private constant ORDER_TYPE = keccak256(
-    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)"
-  );
+  bytes32 private immutable ORDER_TYPE;

   /// @notice role enabling to invoke mint
-  bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");
+  bytes32 private immutable MINTER_ROLE;

   /// @notice role enabling to invoke redeem
-  bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");
+  bytes32 private immutable REDEEMER_ROLE;

   /// @notice role enabling to disable mint and redeem and remove minters and redeemers in an emergency
-  bytes32 private constant GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");
+  bytes32 private immutable GATEKEEPER_ROLE;

   /// @notice EIP712 domain hash
-  bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));
+  bytes32 private immutable EIP712_DOMAIN_TYPEHASH;

   /// @notice address denoting native ether
   address private constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

   /// @notice EIP712 name
-  bytes32 private constant EIP_712_NAME = keccak256("EthenaMinting");
+  bytes32 private immutable EIP_712_NAME;

   /// @notice holds EIP712 revision
-  bytes32 private constant EIP712_REVISION = keccak256("1");
+  bytes32 private immutable EIP712_REVISION;

   /* --------------- STATE VARIABLES --------------- */

@@ -119,6 +116,27 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     if (address(_usde) == address(0)) revert InvalidUSDeAddress();
     if (_assets.length == 0) revert NoAssetsProvided();
     if (_admin == address(0)) revert InvalidZeroAddress();
+    EIP712_DOMAIN =
+    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
+
+    ROUTE_TYPE = keccak256("Route(address[] addresses,uint256[] ratios)");
+
+    ORDER_TYPE = keccak256(
+    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)"
+  );
+
+    MINTER_ROLE = keccak256("MINTER_ROLE");
+
+    REDEEMER_ROLE = keccak256("REDEEMER_ROLE");
+
+    GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");
+
+    EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));
+
+    EIP_712_NAME = keccak256("EthenaMinting");
+
+    EIP712_REVISION = keccak256("1");
+
     usde = _usde;

     _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
@@ -317,7 +335,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     return ECDSA.toTypedDataHash(getDomainSeparator(), keccak256(encodeOrder(order)));
   }

-  function encodeOrder(Order calldata order) public pure returns (bytes memory) {
+  function encodeOrder(Order calldata order) public view returns (bytes memory) {
     return abi.encode(
       ORDER_TYPE,
       order.order_type,
@@ -331,7 +349,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     );
   }

-  function encodeRoute(Route calldata route) public pure returns (bytes memory) {
+  function encodeRoute(Route calldata route) public view returns (bytes memory) {
     return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);
   }

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L26C3-L32C99

```solidity
File:contracts/StakedUSDe.sol

26:bytes32 private constant REWARDER_ROLE = keccak256("REWARDER_ROLE");
27:  /// @notice The role that is allowed to blacklist and un-blacklist addresses
28:  bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");
29:  /// @notice The role which prevents an address to stake
30:  bytes32 private constant SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
31:  /// @notice The role which prevents an address to transfer, stake, or unstake. The owner of the contract can redirect address staking balance if an address is in full restricting mode.
32:  bytes32 private constant FULL_RESTRICTED_STAKER_ROLE = keccak256("FULL_RESTRICTED_STAKER_ROLE");

```

```diff
diff --git a/contracts/StakedUSDe.sol b/contracts/StakedUSDe.sol
index 0a56a7d..0d92bb2 100644
--- a/contracts/StakedUSDe.sol
+++ b/contracts/StakedUSDe.sol
@@ -23,13 +23,13 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E

   /* ------------- CONSTANTS ------------- */
   /// @notice The role that is allowed to distribute rewards to this contract
-  bytes32 private constant REWARDER_ROLE = keccak256("REWARDER_ROLE");
+  bytes32 private immutable REWARDER_ROLE;
   /// @notice The role that is allowed to blacklist and un-blacklist addresses
-  bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");
+  bytes32 private immutable BLACKLIST_MANAGER_ROLE;
   /// @notice The role which prevents an address to stake
-  bytes32 private constant SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
+  bytes32 private immutable SOFT_RESTRICTED_STAKER_ROLE;
   /// @notice The role which prevents an address to transfer, stake, or unstake. The owner of the contract can redirect address staking balance if an address is in full restricting mode.
-  bytes32 private constant FULL_RESTRICTED_STAKER_ROLE = keccak256("FULL_RESTRICTED_STAKER_ROLE");
+  bytes32 private immutable FULL_RESTRICTED_STAKER_ROLE;
   /// @notice The vesting period of lastDistributionAmount over which it increasingly becomes available to stakers
   uint256 private constant VESTING_PERIOD = 8 hours;
   /// @notice Minimum non-zero shares amount to prevent donation attack
@@ -75,6 +75,10 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
     if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {
       revert InvalidZeroAddress();
     }
+    REWARDER_ROLE = keccak256("REWARDER_ROLE");
+    BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");
+    SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
+    FULL_RESTRICTED_STAKER_ROLE = keccak256("FULL_RESTRICTED_STAKER_ROLE");

     _grantRole(REWARDER_ROLE, _initialRewarder);
     _grantRole(DEFAULT_ADMIN_ROLE, _owner);

```

<a name='g-04'></a>

## [G-04] Use hardcode address instead of address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;

    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");

        // Do something
    }
}
```

Total Instances: `3`

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L167

```solidity
File: contracts/StakedUSDe.sol

96: IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

167: return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L43C5-L43C57

```solidity
File: contracts/StakedUSDeV2.sol

43: silo = new USDeSilo(address(this), address(_asset));

```
