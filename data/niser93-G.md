## Gas Analyisis Description
In order to report gas savings we used output from `forge snapshot` and computed difference between our and base code using `forge snapshot --diff`.

We also report gas savings of a single instance, of a single issue and the total, underlining that the final sum does not coincide with the sum of the individual parts.

|       |Issue  |Instances|Total Gas Saved|
|:-----:|:------|:-------:|:-------------:|
|[[GO-01](#go-01-useless-check-of-routeaddressesi)]|Useless check of `route.addresses[i]`|1|28514|
|[[GO-02](#go-02-avoid-useless-parameter-in-ethenamintingverifyroute)]|Avoid useless parameter in `EthenaMinting.verifyRoute()`|1|33886|
|[[GO-03](#go-03-reconding-ethenamintingverifyroute-in-order-to-have-different-conditional-path)]|Reconding `EthenaMinting.verifyRoute()` in order to have different conditional path|1|31763|
|[[GO-04](#go-04-avoid-checking-_supportedassetsaddasset-and-_custodianaddressesaddcustodian)]|Avoid checking `_supportedAssets.add(asset)` and `_custodianAddresses.add(custodian)`|1|7485|
|[[GO-05](#go-05-avoid-adding-value-that-is-always-equal-to-zero)]|Avoid adding value that is always equal to zero|1|19275|
|[[GO-06](#go-06-use-alternative-formulation-in-order-to-avoid-not-using-and-instead-of-or-or-vice-versa)]|Use alternative formulation in order to avoid NOT using AND instead of OR or vice versa|1|1299|
|[[GO-07](#go-07-avoid-defining-local-variable-that-are-used-once)]|Avoid defining local variable that are used once|6|6941|
|[[GO-08](#go-08-using-different-conditional-path)]|Using different conditional path|4|10637|
|                 |                     |          |      |
| OVERALL              |                | 16 | 111622 |
                
### [GO-01] Useless check of `route.addresses[i]`
                
The only function that add custodian to `_custodianAddresses` is [addCustodianAddress()](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L298C1-L303C4)  
```
function addCustodianAddress(address custodian) public onlyRole(DEFAULT_ADMIN_ROLE) {
    if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
        revert InvalidCustodianAddress();
    }
    emit CustodianAddressAdded(custodian);
} 
``` 

Through this function's checks, it is impossibile to add `address(0)` to `_custodianAddresses`.

This means that if `_custodianAddresses.contains(route.addresses[i])` is `true`, `route.addresses[i]` is not zero for sure.

So, this part of check at line 364 is useless: `route.addresses[i] == address(0)`

*Issue's instances:* 1

#### File: [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol)

*Instances:* 1    


```diff
File: EthenaMinting.sol


-  364        if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
+  364        if (!_custodianAddresses.contains(route.addresses[i]) || route.ratios[i] == 0)

```

```
testDelegateSuccessfulRedeem() (gas: -332 (-0.102%)) 
test_redeem_invalidNonce_revert() (gas: -332 (-0.107%)) 
test_admin_can_enable_redeem() (gas: -332 (-0.110%)) 
test_redeem() (gas: -332 (-0.113%)) 
test_fuzz_nextBlock_redeem_is_zero(uint256) (gas: -332 (-0.113%)) 
test_nativeEth_withdraw() (gas: -332 (-0.114%)) 
test_gatekeeper_cannot_enable_redeem_revert() (gas: -415 (-0.128%)) 
test_fuzz_nonAdmin_cannot_enable_redeem_revert(address) (gas: -415 (-0.129%)) 
test_redeem_notRedeemer_revert() (gas: -415 (-0.140%)) 
testDelegateFailureRedeem() (gas: -415 (-0.145%)) 
test_multiple_redeem() (gas: -664 (-0.147%)) 
test_fuzz_maxRedeem_perBlock_exceeded_revert(uint256) (gas: -415 (-0.153%)) 
test_sending_redeem_order_to_mint_revert() (gas: -415 (-0.163%)) 
testDelegateSuccessfulMint() (gas: -415 (-0.172%)) 
test_admin_can_enable_mint() (gas: -415 (-0.195%)) 
test_fuzz_mint_noSlippage(uint256) (gas: -415 (-0.198%)) 
test_fuzz_nextBlock_mint_is_zero(uint256) (gas: -415 (-0.205%)) 
test_mint() (gas: -415 (-0.210%)) 
test_unsupported_assets_ERC20_revert() (gas: -415 (-0.250%)) 
test_unsupported_assets_ETH_revert() (gas: -415 (-0.305%)) 
test_multiple_mints() (gas: -830 (-0.333%)) 
test_fuzz_multipleInvalid_custodyRatios_revert(uint256) (gas: -432 (-0.385%)) 
test_fuzz_singleInvalid_custodyRatio_revert(uint256) (gas: -415 (-0.416%)) 
testCorrectInitConfig() (gas: -15847 (-0.433%)) 
test_multipleValid_custodyRatios_addresses() (gas: -2939 (-0.856%))
```
```
Overall gas change: -28514 (-0.068%)
```
*Code link:* [[364](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L364)]



***
&nbsp;

                
### [GO-02] Avoid useless parameter in `EthenaMinting.verifyRoute()`
                
There are two kind of `OrderType`: `REDEEM` and `MINT`.

`EthenaMinting.verifyRoute()` is used only in `_mint()` (`_redeem()` doesn't use route). Due to this, inside `EthenaMinting.verifyRoute()` there is a check: if `orderType` is `OrderType.REDEEM`, return `true`.

It should be better and consumes less gas if parameter `orderType` is removed from `verifyRoute()` (and so, the check of `orderType` inside it).
We suggest also to think to remove `OrderType` enum at all.

*Issue's instances:* 1

#### File: [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol)

*Instances:* 1    


```diff
File: EthenaMinting.sol

-  351    function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
+  351    function verifyRoute(Route calldata route) public view override returns (bool) {
-  352      // routes only used to mint
-  353      if (orderType == OrderType.REDEEM) {
-  354        return true;
-  355      }
```

```
test_fuzz_not_gatekeeper_cannot_remove_minter_revert(address) (gas: 1 (0.002%)) 
test_renounceRole_nonAdminRole() (gas: -1 (-0.003%)) 
test_gatekeeper_can_remove_minter() (gas: 1 (0.005%)) 
test_fuzz_notMinter_cannot_mint(address) (gas: 9 (0.007%)) 
test_minter_canTransfer_custody() (gas: -11 (-0.009%)) 
test_fuzz_mint_maxMint_perBlock_exceeded_revert(uint256) (gas: -13 (-0.014%)) 
test_fuzz_not_gatekeeper_cannot_remove_redeemer_revert(address) (gas: 22 (0.035%)) 
test_fuzz_maxMint_perBlock_exceeded_revert(uint256) (gas: -35 (-0.038%)) 
testCanUndelegate() (gas: -51 (-0.043%)) 
testDelegateFailureMint() (gas: -52 (-0.047%)) 
test_grantRole_AdminRoleExternally() (gas: -22 (-0.051%)) 
testNonAdminCanRenounceRoles() (gas: -18 (-0.051%)) 
test_transferAdmin_notAdmin() (gas: -22 (-0.054%)) 
test_sending_mint_order_to_redeem_revert() (gas: -48 (-0.058%)) 
test_expired_orders_revert() (gas: -61 (-0.064%)) 
test_gatekeeper_cannot_enable_mint_revert() (gas: -101 (-0.068%)) 
test_fuzz_notAdmin_cannot_add_gatekeeper(address) (gas: -44 (-0.069%)) 
test_fuzz_notAdmin_cannot_add_minter(address) (gas: -44 (-0.069%)) 
test_fuzz_nonAdmin_cannot_enable_mint_revert(address) (gas: -101 (-0.069%)) 
test_gatekeeper_cannot_add_minters_revert() (gas: -44 (-0.070%)) 
test_gatekeeper_can_disable_mintRedeem() (gas: -79 (-0.070%)) 
test_fuzz_notAdmin_cannot_remove_gatekeeper(address) (gas: -66 (-0.072%)) 
test_fuzz_notAdmin_cannot_remove_minter(address) (gas: -66 (-0.072%)) 
test_cannot_add_asset_already_supported_revert() (gas: 55 (0.075%)) 
testDelegateSuccessfulRedeem() (gas: -278 (-0.085%)) 
test_fuzz_nonOwner_cannot_remove_supportedAsset_revert(address) (gas: 88 (0.086%)) 
test_nativeEth_withdraw() (gas: -259 (-0.089%)) 
test_redeem() (gas: -267 (-0.090%)) 
test_redeem_notRedeemer_revert() (gas: -275 (-0.093%)) 
testCanRepeatedlyTransferAdmin() (gas: -44 (-0.095%)) 
test_admin_can_enable_redeem() (gas: -302 (-0.100%)) 
test_admin_can_add_gatekeeper() (gas: -44 (-0.101%)) 
test_admin_can_add_minter() (gas: -44 (-0.101%)) 
test_grantRole_nonAdminRole() (gas: -44 (-0.102%)) 
test_fuzz_nextBlock_redeem_is_zero(uint256) (gas: -302 (-0.103%)) 
test_revokeRole_nonAdminRole() (gas: -35 (-0.103%)) 
test_gatekeeper_can_remove_redeemer() (gas: 22 (0.104%)) 
testDelegateSuccessfulMint() (gas: -253 (-0.105%)) 
test_gatekeeper_cannot_enable_redeem_revert() (gas: -341 (-0.105%)) 
test_fuzz_nonAdmin_cannot_enable_redeem_revert(address) (gas: -341 (-0.106%)) 
test_fuzz_not_gatekeeper_cannot_disable_mintRedeem_revert(address) (gas: -66 (-0.108%)) 
test_redeem_invalidNonce_revert() (gas: -340 (-0.110%)) 
test_sending_redeem_order_to_mint_revert() (gas: -288 (-0.113%)) 
testCanTransferOwnership() (gas: -70 (-0.114%)) 
test_fuzz_mint_noSlippage(uint256) (gas: -240 (-0.115%)) 
testDelegateFailureRedeem() (gas: -336 (-0.117%)) 
test_fuzz_maxRedeem_perBlock_exceeded_revert(uint256) (gas: -319 (-0.118%)) 
test_mint() (gas: -240 (-0.121%)) 
test_fuzz_nonOwner_cannot_add_supportedAsset_revert(address) (gas: 55 (0.124%)) 
testNewOwnerCanPerformOwnerActions() (gas: -110 (-0.125%)) 
test_add_and_remove_supported_asset() (gas: 71 (0.127%)) 
testOwnershipTransferRequiresTwoSteps() (gas: 64 (0.128%)) 
test_multiple_redeem() (gas: -587 (-0.130%)) 
testOldOwnerCantPerformOwnerActions() (gas: -132 (-0.140%)) 
test_admin_can_disable_redeem(bool) (gas: -22 (-0.140%)) 
test_fuzz_nextBlock_mint_is_zero(uint256) (gas: -284 (-0.140%)) 
testOldOwnerCantTransferOwnership() (gas: -132 (-0.143%)) 
test_admin_can_remove_gatekeeper() (gas: -53 (-0.148%)) 
test_admin_can_remove_minter() (gas: -53 (-0.148%)) 
test_admin_can_enable_mint() (gas: -328 (-0.154%)) 
test_unsupported_assets_ERC20_revert() (gas: -262 (-0.158%)) 
test_base_transferAdmin() (gas: -106 (-0.166%)) 
test_fuzz_maxRedeem_perBlock_setter(uint256) (gas: -44 (-0.172%)) 
test_cannot_removeAsset_not_supported_revert() (gas: 33 (0.172%)) 
test_unsupported_assets_ETH_revert() (gas: -240 (-0.176%)) 
test_admin_cannot_transfer_self() (gas: -44 (-0.204%)) 
test_multiple_mints() (gas: -546 (-0.219%)) 
testCancelTransferAdmin() (gas: -88 (-0.222%)) 
test_multipleValid_custodyRatios_addresses() (gas: -768 (-0.224%)) 
test_fuzz_multipleInvalid_custodyRatios_revert(uint256) (gas: -255 (-0.227%)) 
test_renounceRole_notAdmin() (gas: 43 (0.245%)) 
testAdminCanCancelTransfer() (gas: -106 (-0.259%)) 
test_fuzz_singleInvalid_custodyRatio_revert(uint256) (gas: -266 (-0.267%)) 
test_renounceRole_AdminRole() (gas: 43 (0.276%)) 
test_admin_can_disable_mint(bool) (gas: -44 (-0.279%)) 
test_renounceRole_forDifferentAccount() (gas: 43 (0.280%)) 
test_fuzz_maxMint_perBlock_setter(uint256) (gas: -66 (-0.280%)) 
testOwnershipCannotBeRenounced() (gas: 86 (0.359%)) 
testCorrectInitConfig() (gas: -24379 (-0.666%))
```

```
Overall gas change: -33886 (-0.081%)
```

*Code link:* [[351-355](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L351-L355)]



***
&nbsp;

                
### [GO-03] Reconding `EthenaMinting.verifyRoute()` in order to have different conditional path
                
It is possible to change `EthenaMinting.verifyRoute()` and have a different conditional path, in order to avoid some operation.

*Issue's instances:* 1

#### File: [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol)

*Instances:* 1    


```diff
File: EthenaMinting.sol

Current Code:
   356      uint256 totalRatio = 0;
-  357      if (route.addresses.length != route.ratios.length) {
-  358        return false;
-  359      }
-  360      if (route.addresses.length == 0) {
-  361        return false;
-  362      }
-  363      for (uint256 i = 0; i < route.addresses.length; ++i) {
-  364        if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
-  365        {
-  366          return false;
-  367        }
-  368        totalRatio += route.ratios[i];
-  369      }
-  370      if (totalRatio != 10_000) {
-  371        return false;
-  372      }
-  373      return true;
+  357      if (route.addresses.length == route.ratios.length && route.addresses.length != 0) {
+  358          for (uint256 i = 0; i < route.addresses.length; ++i) {
+  359              if (!_custodianAddresses.contains(route.addresses[i]) || route.ratios[i] == 0)
+  360              {
+  361                  return false;
+  362              }
+  363              totalRatio += route.ratios[i];
+  364          }
+  365          if (totalRatio == 10_000) {
+  366              return true;
+  367          }
+  368      }
```

```
testDelegateSuccessfulRedeem() (gas: -313 (-0.096%)) 
test_redeem_invalidNonce_revert() (gas: -312 (-0.101%)) 
test_admin_can_enable_redeem() (gas: -313 (-0.103%)) 
test_redeem() (gas: -312 (-0.106%)) 
test_fuzz_nextBlock_redeem_is_zero(uint256) (gas: -313 (-0.106%)) 
test_nativeEth_withdraw() (gas: -313 (-0.108%)) 
test_gatekeeper_cannot_enable_redeem_revert() (gas: -391 (-0.121%)) 
test_fuzz_nonAdmin_cannot_enable_redeem_revert(address) (gas: -391 (-0.122%)) 
test_redeem_notRedeemer_revert() (gas: -391 (-0.132%)) 
testDelegateFailureRedeem() (gas: -391 (-0.137%)) 
test_multiple_redeem() (gas: -626 (-0.139%)) 
test_fuzz_maxRedeem_perBlock_exceeded_revert(uint256) (gas: -391 (-0.144%)) 
test_sending_redeem_order_to_mint_revert() (gas: -391 (-0.153%)) 
testDelegateSuccessfulMint() (gas: -391 (-0.162%)) 
test_admin_can_enable_mint() (gas: -391 (-0.184%)) 
test_fuzz_mint_noSlippage(uint256) (gas: -391 (-0.187%)) 
test_fuzz_nextBlock_mint_is_zero(uint256) (gas: -391 (-0.193%)) 
test_mint() (gas: -391 (-0.198%)) 
test_unsupported_assets_ERC20_revert() (gas: -391 (-0.236%)) 
test_unsupported_assets_ETH_revert() (gas: -391 (-0.287%)) 
test_multiple_mints() (gas: -782 (-0.313%)) 
test_fuzz_multipleInvalid_custodyRatios_revert(uint256) (gas: -422 (-0.376%)) 
test_fuzz_singleInvalid_custodyRatio_revert(uint256) (gas: -424 (-0.425%)) 
testCorrectInitConfig() (gas: -19655 (-0.537%)) 
test_multipleValid_custodyRatios_addresses() (gas: -2895 (-0.843%)) 
```

```
Overall gas change: -31763 (-0.076%)
```

*Code link:* [[356-373](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L356-L373)]



***
&nbsp;

                
### [GO-04] Avoid checking `_supportedAssets.add(asset)` and `_custodianAddresses.add(custodian)`
                
There are two functions: `addSupportedAsset()` and `addCustodianAddress()`. They check these conditions, respectively: 

`!_supportedAssets.add(asset)` and `!_custodianAddresses.add(custodian)`.

Due to the fact that `supportedAssets` and `custodianAddresses` are defined as EnumerableSet ([lines #66 and #69 of EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L65-L69)), add function is defined inside
[openzeppelin lib #L243-L245](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/lib/openzeppelin-contracts/contracts/utils/structs/EnumerableSet.sol#L243-L245) and [openzeppelin lib #L169-L171](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/lib/openzeppelin-contracts/contracts/utils/structs/EnumerableSet.sol#L65-L75)
```
function _add(Set storage set, bytes32 value) private returns (bool) {
    if (!_contains(set, value)) {
        set._values.push(value);
        // The value is stored at length-1, but we add 1 to all indexes
        // and use 0 as a sentinel value
        set._indexes[value] = set._values.length;
        return true;
    } else {
        return false;
    }
}
```

We can observe that this function return false only if we try to add an element to a list that contains it.

It seems that it can avoid checking add return value and saving gas. When add an element returns false, it is not because that element
is not valid, but only because that element is still inside the set.

Of course, in order to report right gas savings, we had to disabled the specific test that check revert when a custodian address is added twice:
[EthenaMinting.core.t.sol#L390-L400](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/minting/tests/EthenaMinting.core.t.sol#L390-L400)
```
function test_cannot_add_asset_already_supported_revert() public {
    address asset = address(20);
    vm.expectEmit(true, false, false, false);
    emit AssetAdded(asset);
    vm.startPrank(owner);
    EthenaMintingContract.addSupportedAsset(asset);
    assertTrue(EthenaMintingContract.isSupportedAsset(asset));

    //vm.expectRevert(InvalidAssetAddress);
    //EthenaMintingContract.addSupportedAsset(asset);
}
```

*Issue's instances:* 1

#### File: [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol)

*Instances:* 1    


```diff
File: EthenaMinting.sol

Current Code:
   290    function addSupportedAsset(address asset) public onlyRole(DEFAULT_ADMIN_ROLE) {
-  291      if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {
+  292      if (asset == address(0) || asset == address(usde)) {
   292        revert InvalidAssetAddress();
   293      }
+  294      _supportedAssets.add(asset);
   294      emit AssetAdded(asset);
   295    }
   296  
   297    /// @notice Adds an custodian to the supported custodians list.
   298    function addCustodianAddress(address custodian) public onlyRole(DEFAULT_ADMIN_ROLE) {
-  299      if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
+  299      if (custodian == address(0) || custodian == address(usde)) {
   300        revert InvalidCustodianAddress();
   301      }
+  302      _custodianAddresses.add(custodian);
   302      emit CustodianAddressAdded(custodian);
   303    }
```

```
test_cannot_removeAsset_not_supported_revert() (gas: -1 (-0.005%)) 
test_multipleValid_custodyRatios_addresses() (gas: -20 (-0.006%)) 
test_minter_canTransfer_custody() (gas: -20 (-0.016%)) 
test_fuzz_nonOwner_cannot_remove_supportedAsset_revert(address) (gas: -20 (-0.020%)) 
test_add_and_remove_supported_asset() (gas: -16 (-0.029%)) 
testCorrectInitConfig() (gas: -2950 (-0.081%)) 
test_cannotAdd_USDe_revert() (gas: -17 (-0.086%)) 
test_cannotAdd_addressZero_revert() (gas: -17 (-0.108%)) 
test_cannot_add_asset_already_supported_revert() (gas: -4424 (-6.051%)) 
```

```
Overall gas change: -7485 (-0.018%)
```

*Code link:* [[290-303](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L290-L303)]



***
&nbsp;

                
### [GO-05] Avoid adding value that is always equal to zero
                
In `transferInRewards()`, there is an initial check:

```
StakedUSDe.sol#L90

if (getUnvestedAmount() > 0) revert StillVesting();
```

So `getUnvestedAmount()` must be zero, in order to go ahead.
This function is defined in [StakedUSDe.sol#L170-L181](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L170-L181):

```
/**
* @notice Returns the amount of USDe tokens that are unvested in the contract.
*/
function getUnvestedAmount() public view returns (uint256) {
    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;


    if (timeSinceLastDistribution >= VESTING_PERIOD) {
    return 0;
    }


    return ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) / VESTING_PERIOD;
}
```
This means that `getUnvestedAmount()` returns zero only if `vestingAmount` is zero or `timeSinceLastDistribution >= VESTING_PERIOD`.

In any case, `transferInRewards()` can run [line 91](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L91) only if `getUnvestedAmount()` is equal to zero.

```
StakedUSDe.sol#L91

uint256 newVestingAmount = amount + getUnvestedAmount();
```

So, we can avoid adding `getUnvestedAmount()`'s return value, because it must be zero.

Thanks to this, we can also avoid defining `newVestingAmount` local variable.

Moreover, we observer that 

```
emit RewardsReceived(amount, newVestingAmount);
```

emits always `emit RewardsReceived(amount, amount)` and we could saving other gas calling only 

```
emit RewardsReceived(amount);
```

*Issue's instances:* 1

#### File: [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol)

*Instances:* 1    


```diff
File: StakedUSDe.sol

Current Code:
   89    function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
   90      if (getUnvestedAmount() > 0) revert StillVesting();
-  91      uint256 newVestingAmount = amount + getUnvestedAmount();
   92  
-  93      vestingAmount = newVestingAmount;
+  93      vestingAmount = amount;
   94      lastDistributionTimestamp = block.timestamp;
   95      // transfer assets from rewarder to this contract
   96      IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
   97  
-  98      emit RewardsReceived(amount, newVestingAmount);
+  98      emit RewardsReceived(amount, amount);
   99    }
```

```
testFuzzCooldownAssetsUnstake(uint256) (gas: 9 (0.004%)) 
testFuzzCooldownAssets(uint256) (gas: 9 (0.004%)) 
testFuzzCooldownShares(uint256) (gas: 10 (0.004%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -542 (-0.077%)) 
testUSDeValuePerStUSDe() (gas: -535 (-0.101%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -531 (-0.101%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -530 (-0.101%)) 
test_constructor() (gas: -4608 (-0.120%)) 
testFairStakeAndUnstakePrices() (gas: -535 (-0.127%)) 
testUSDeValuePerStUSDe() (gas: -535 (-0.130%)) 
testUSDeValuePerStUSDe() (gas: -534 (-0.130%)) 
testFairStakeAndUnstakePrices() (gas: -534 (-0.149%)) 
testFairStakeAndUnstakePrices() (gas: -535 (-0.150%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -535 (-0.158%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -534 (-0.194%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -534 (-0.195%)) 
testOwnerCanChangeRewarder() (gas: -668 (-0.258%)) 
testOwnerCanChangeRewarder() (gas: -668 (-0.258%)) 
testOwnerCanChangeRewarder() (gas: -668 (-0.258%)) 
testCannotTransferRewardsWhileVesting() (gas: -535 (-0.359%)) 
testCannotTransferRewardsWhileVesting() (gas: -535 (-0.360%)) 
testFuzzNoJumpInVestedBalance(uint256) (gas: -535 (-0.374%)) 
testFuzzNoJumpInVestedBalance(uint256) (gas: -535 (-0.374%)) 
testFuzzNoJumpInVestedBalance(uint256) (gas: -535 (-0.374%)) 
testCanTransferRewardsAfterVesting() (gas: -802 (-0.407%)) 
testCanTransferRewardsAfterVesting() (gas: -802 (-0.407%)) 
testTransferRewardsFailsInsufficientBalance() (gas: -666 (-0.451%)) 
testTransferRewardsFailsInsufficientBalance() (gas: -666 (-0.451%)) 
testTransferRewardsFailsInsufficientBalance() (gas: -666 (-0.451%)) 
```

```
Overall gas change: -19275 (-0.046%)
```

*Code link:* [[89-99](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89-L99)]



***
&nbsp;

                
### [GO-06] Use alternative formulation in order to avoid NOT using AND instead of OR or vice versa
                
It's possible to change boolean formula in order to use AND instead of OR or vice versa.

In this way, ```>``` could be substituted with ```>=```, ```!=``` with ```==```

 In general

#### Two variables

```
(a || b) = !(!(a || b)) = !(!a && !b)
```

| a   | b   | (a or b)  | !a and !b  | !(!a and !b)|
|:---:|:---:|:---------:|:----------:|:------------:|
| 0  | 0  | 0         | 1          | 0            |
| 0  | 1  | 1         | 0          | 1            |
| 1  | 0  | 1         | 0          | 1            |
| 1  | 1  | 1         | 0          | 1            |


#### Three variables

```
(a || b || c) = !(!(a || b || c)) = !(!a && !b && !c)
```

| a   | b   | c   | (a or b or c)  | !a and !b and !c  | !(!a and !b and !c)|
|:---:|:---:|:---:|:--------------:|:-----------------:|:------------------:|
| 0  | 0  | 0 | 0              | 1                 | 0                  |
| 0  | 0  | 1 | 1              | 0                 | 1                  |
| 0  | 1  | 0 | 1              | 0                 | 1                  |
| 0  | 1  | 1 | 1              | 0                 | 1                  |
| 1  | 0  | 0 | 1              | 0                 | 1                  |
| 1  | 0  | 1 | 1              | 0                 | 1                  |
| 1  | 1  | 0 | 1              | 0                 | 1                  |
| 1  | 1  | 1 | 1              | 0                 | 1                  |

*Issue's instances:* 1

#### File: [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol)

*Instances:* 1    


```diff
File: StakedUSDe.sol

-  193      if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();
+  193      if (!(_totalSupply <= 0 || _totalSupply >= MIN_SHARES)) revert MinSharesViolation();
```

```
test_fullBlacklist_transferFrom_pass() (gas: -4 (-0.002%)) 
test_softBlacklist_transfer_pass() (gas: -4 (-0.002%)) 
test_redistributeLockedAmount() (gas: -5 (-0.002%)) 
test_redistributeLockedAmount() (gas: -5 (-0.002%)) 
test_softBlacklist_transferFrom_pass() (gas: -5 (-0.002%)) 
test_softBlacklist_transferFrom_pass() (gas: -5 (-0.002%)) 
test_fullBlacklist_withdraw_pass() (gas: -6 (-0.002%)) 
testBlacklistManagerCannotRedistribute() (gas: -6 (-0.002%)) 
testBlacklistManagerCannotRedistribute() (gas: -6 (-0.002%)) 
test_fullBlacklist_transferFrom_pass() (gas: -5 (-0.002%)) 
test_softFullBlacklist_transfer_pass() (gas: -6 (-0.002%)) 
test_softFullBlacklist_transfer_pass() (gas: -6 (-0.002%)) 
test_softBlacklist_transfer_pass() (gas: -5 (-0.003%)) 
testOwnerCanRescuestUSDe() (gas: -5 (-0.003%)) 
testOwnerCanRescuestUSDe() (gas: -5 (-0.003%)) 
testOwnerCanRescuestUSDe() (gas: -5 (-0.003%)) 
testCanBurnOnRedistribute() (gas: -5 (-0.003%)) 
testCanBurnOnRedistribute() (gas: -5 (-0.003%)) 
testOnlyRewarderCanReward() (gas: -6 (-0.003%)) 
testOnlyRewarderCanReward() (gas: -6 (-0.003%)) 
testOnlyRewarderCanReward() (gas: -6 (-0.003%)) 
test_softFullBlacklist_withdraw_pass() (gas: -10 (-0.003%)) 
test_fullBlacklist_withdraw_pass() (gas: -6 (-0.003%)) 
test_fullBlacklist_transfer_pass() (gas: -6 (-0.003%)) 
test_fullBlacklist_transfer_pass() (gas: -6 (-0.003%)) 
test_fullBlacklist_user_can_not_burn_and_donate_to_vault() (gas: -6 (-0.003%)) 
test_fullBlacklist_user_can_not_burn_and_donate_to_vault() (gas: -6 (-0.003%)) 
test_fullBlacklist_can_not_be_transfer_recipient() (gas: -9 (-0.003%)) 
testDonationAttack() (gas: -9 (-0.004%)) 
testDonationAttack() (gas: -9 (-0.004%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -12 (-0.004%)) 
testFairStakeAndUnstakePrices() (gas: -15 (-0.004%)) 
test_fullBlacklist_can_not_be_transfer_recipient() (gas: -10 (-0.004%)) 
testInitialStakeBelowMin() (gas: -6 (-0.004%)) 
testInitialStakeBelowMin() (gas: -6 (-0.004%)) 
testInitialStakeBelowMin() (gas: -6 (-0.004%)) 
testFairStakeAndUnstakePrices() (gas: -14 (-0.004%)) 
testOwnerCannotRescueUSDe() (gas: -6 (-0.004%)) 
testOwnerCannotRescueUSDe() (gas: -6 (-0.004%)) 
testOwnerCannotRescueUSDe() (gas: -6 (-0.004%)) 
testMintToDiffRecipient() (gas: -6 (-0.004%)) 
testMintToDiffRecipient() (gas: -6 (-0.004%)) 
testMintToDiffRecipient() (gas: -6 (-0.004%)) 
testInitialStake() (gas: -6 (-0.004%)) 
testInitialStake() (gas: -6 (-0.004%)) 
testInitialStake() (gas: -6 (-0.004%)) 
testUSDeValuePerStUSDe() (gas: -22 (-0.004%)) 
testFairStakeAndUnstakePrices() (gas: -15 (-0.004%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -12 (-0.004%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -23 (-0.004%)) 
test_softBlacklist_withdraw_pass() (gas: -12 (-0.004%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -12 (-0.004%)) 
test_softFullBlacklist_withdraw_pass() (gas: -12 (-0.005%)) 
testCantCooldownBelowMinShares() (gas: -12 (-0.005%)) 
testStakeFlowCommonUser() (gas: -12 (-0.005%)) 
testMintWithSlippageCheck(uint256) (gas: -9 (-0.005%)) 
testFuzzCooldownAssetsUnstake(uint256) (gas: -12 (-0.005%)) 
testStakeUnstake() (gas: -12 (-0.005%)) 
testFuzzCooldownAssets(uint256) (gas: -12 (-0.005%)) 
testMintWithSlippageCheck(uint256) (gas: -10 (-0.005%)) 
testMintWithSlippageCheck(uint256) (gas: -10 (-0.005%)) 
testFuzzCooldownShares(uint256) (gas: -12 (-0.005%)) 
testUSDeValuePerStUSDe() (gas: -22 (-0.005%)) 
testUSDeValuePerStUSDe() (gas: -22 (-0.005%)) 
test_softBlacklist_withdraw_pass() (gas: -12 (-0.006%)) 
testCantWithdrawBelowMinShares() (gas: -12 (-0.006%)) 
testCantWithdrawBelowMinShares() (gas: -12 (-0.006%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -41 (-0.006%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -32 (-0.006%)) 
testStakeFlowCommonUser() (gas: -12 (-0.006%)) 
testStakeUnstake() (gas: -12 (-0.006%)) 
testStakeUnstake() (gas: -12 (-0.006%)) 
test_constructor() (gas: -600 (-0.016%)) 
```

```
Overall gas change: -1299 (-0.003%)
```

*Code link:* [[193](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L193)]



***
&nbsp;

                
### [GO-07] Avoid defining local variable that are used once
                
Avoid defining a local variable can save gas. When code don't lose clarity, it could be better to write inline code.

*Issue's instances:* 6

#### File: [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol)

*Instances:* 2    


```diff
File: EthenaMinting.sol

-  381      mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
-  382      uint256 invalidator = invalidatorStorage[invalidatorSlot];
+  381      uint256 invalidator = _orderBitmaps[sender][invalidatorSlot];
```

```
test_multipleValid_custodyRatios_addresses() (gas: -17 (-0.005%)) 
test_gatekeeper_cannot_enable_redeem_revert() (gas: -17 (-0.005%)) 
test_fuzz_nonAdmin_cannot_enable_redeem_revert(address) (gas: -17 (-0.005%)) 
test_redeem_notRedeemer_revert() (gas: -17 (-0.006%)) 
testDelegateFailureRedeem() (gas: -17 (-0.006%)) 
test_fuzz_maxRedeem_perBlock_exceeded_revert(uint256) (gas: -17 (-0.006%)) 
test_sending_redeem_order_to_mint_revert() (gas: -17 (-0.007%)) 
testDelegateSuccessfulMint() (gas: -17 (-0.007%)) 
test_admin_can_enable_mint() (gas: -17 (-0.008%)) 
test_fuzz_mint_noSlippage(uint256) (gas: -17 (-0.008%)) 
testDelegateSuccessfulRedeem() (gas: -27 (-0.008%)) 
test_fuzz_nextBlock_mint_is_zero(uint256) (gas: -17 (-0.008%)) 
test_mint() (gas: -17 (-0.009%)) 
test_admin_can_enable_redeem() (gas: -27 (-0.009%)) 
test_redeem() (gas: -27 (-0.009%)) 
test_fuzz_nextBlock_redeem_is_zero(uint256) (gas: -27 (-0.009%)) 
test_nativeEth_withdraw() (gas: -27 (-0.009%)) 
test_unsupported_assets_ERC20_revert() (gas: -17 (-0.010%)) 
test_redeem_invalidNonce_revert() (gas: -34 (-0.011%)) 
test_multiple_redeem() (gas: -54 (-0.012%)) 
test_unsupported_assets_ETH_revert() (gas: -17 (-0.012%)) 
test_multiple_mints() (gas: -34 (-0.014%)) 
testCorrectInitConfig() (gas: -1200 (-0.033%)) 
```

```
Overall gas change: -1695 (-0.004%)
```

*Code link:* [[381-382](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L381-L382)]




***




```diff
File: EthenaMinting.sol

-  429      uint256 remainingBalance = amount - totalTransferred;
-  430      if (remainingBalance > 0) {
-  431        token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
+  430      if (amount > totalTransferred) {
+  431         token.safeTransferFrom(benefactor, addresses[addresses.length - 1], amount - totalTransferred);
   432      }
```

```
testCorrectInitConfig() (gas: -600 (-0.016%)) 
testDelegateSuccessfulRedeem() (gas: -63 (-0.019%)) 
test_redeem_invalidNonce_revert() (gas: -63 (-0.020%)) 
test_admin_can_enable_redeem() (gas: -63 (-0.021%)) 
test_redeem() (gas: -63 (-0.021%)) 
test_fuzz_nextBlock_redeem_is_zero(uint256) (gas: -63 (-0.021%)) 
test_nativeEth_withdraw() (gas: -63 (-0.022%)) 
test_multipleValid_custodyRatios_addresses() (gas: -79 (-0.023%)) 
test_gatekeeper_cannot_enable_redeem_revert() (gas: -79 (-0.024%)) 
test_fuzz_nonAdmin_cannot_enable_redeem_revert(address) (gas: -79 (-0.025%)) 
test_redeem_notRedeemer_revert() (gas: -79 (-0.027%)) 
testDelegateFailureRedeem() (gas: -79 (-0.028%)) 
test_multiple_redeem() (gas: -126 (-0.028%)) 
test_fuzz_maxRedeem_perBlock_exceeded_revert(uint256) (gas: -79 (-0.029%)) 
test_sending_redeem_order_to_mint_revert() (gas: -79 (-0.031%)) 
testDelegateSuccessfulMint() (gas: -79 (-0.033%)) 
test_admin_can_enable_mint() (gas: -79 (-0.037%)) 
test_fuzz_mint_noSlippage(uint256) (gas: -79 (-0.038%)) 
test_fuzz_nextBlock_mint_is_zero(uint256) (gas: -79 (-0.039%)) 
test_mint() (gas: -79 (-0.040%)) 
test_multiple_mints() (gas: -158 (-0.063%)) 
```

```
Overall gas change: -2210 (-0.005%)
```

*Code link:* [[429-432](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L429-L432)]



***
&nbsp;

#### File: [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol)

*Instances:* 2    


```diff
File: StakedUSDe.sol

-  111      bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
-  112      _grantRole(role, target);
+  111      _grantRole(isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE, target);
```

```
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 4 (0.001%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -6 (-0.001%)) 
testBlacklistManagerCannotRedistribute() (gas: -13 (-0.005%)) 
testBlacklistManagerCannotRedistribute() (gas: -13 (-0.005%)) 
testNewOwnerCanPerformOwnerActions() (gas: -13 (-0.012%)) 
testBlacklistManagerCanUnblacklist() (gas: -21 (-0.022%)) 
testBlacklistManagerCanUnblacklist() (gas: -21 (-0.022%)) 
testBlacklistManagerCanBlacklist() (gas: -26 (-0.023%)) 
testBlacklistManagerCanBlacklist() (gas: -26 (-0.023%)) 
test_constructor() (gas: -1000 (-0.026%)) 
```

```
Overall gas change: -1135 (-0.003%)
```

*Code link:* [[111-112](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L111-L112)]




***




```diff
File: StakedUSDe.sol

-  125      bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
-  126      _revokeRole(role, target);
+  125      _revokeRole(isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE, target);
```

```
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 5 (0.001%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -7 (-0.001%)) 
testBlacklistManagerCanUnblacklist() (gas: -21 (-0.022%)) 
testBlacklistManagerCanUnblacklist() (gas: -21 (-0.022%)) 
test_constructor() (gas: -1000 (-0.026%)) 
```

```
Overall gas change: -1044 (-0.002%)
```

*Code link:* [[125-126](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L125-L126)]



***
&nbsp;

#### File: [StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol)

*Instances:* 2    


```diff
File: StakedUSDeV2.sol

-  95    function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {
+  95      function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256 shares) {
   96      if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();
   97  
-  98           uint256 shares = previewWithdraw(assets);
+  98           shares = previewWithdraw(assets);
   99  
   100          cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
   101          cooldowns[owner].underlyingAmount += assets;
   102  
   103          _withdraw(_msgSender(), address(silo), owner, assets, shares);
   104  
-  105          return shares;
   106    }
```

```
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -2 (-0.000%)) 
test_fullBlacklist_withdraw_pass() (gas: -3 (-0.001%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 7 (0.001%)) 
test_softBlacklist_withdraw_pass() (gas: -4 (-0.001%)) 
testStakeFlowCommonUser() (gas: -4 (-0.002%)) 
testFuzzCooldownAssetsUnstake(uint256) (gas: -4 (-0.002%)) 
testFuzzCooldownAssets(uint256) (gas: -4 (-0.002%)) 
test_softFullBlacklist_withdraw_pass() (gas: -7 (-0.002%)) 
test_constructor() (gas: -400 (-0.010%)) 
```

```
Overall gas change: -421 (-0.001%)
```

*Code link:* [[95-106](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L95-L106)]




***




```diff
File: StakedUSDeV2.sol

-  111    function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
+  111      function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256 assets) {
   112      if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();
   113  
-  114          uint256 assets = previewRedeem(shares);
+  114          assets = previewRedeem(shares);
   115  
   116          cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
   117          cooldowns[owner].underlyingAmount += assets;
   118  
   119          _withdraw(_msgSender(), address(silo), owner, assets, shares);
   120  
-  121          return assets;
   122    }
```

```
testFuzzCooldownShares(uint256) (gas: -1 (-0.000%)) 
testFuzzCooldownAssetsUnstake(uint256) (gas: 2 (0.001%)) 
testFuzzCooldownAssets(uint256) (gas: 2 (0.001%)) 
testFairStakeAndUnstakePrices() (gas: -4 (-0.001%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 6 (0.001%)) 
testCantCooldownBelowMinShares() (gas: -3 (-0.001%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -4 (-0.001%)) 
testUSDeValuePerStUSDe() (gas: -8 (-0.002%)) 
testStakeUnstake() (gas: -4 (-0.002%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -17 (-0.002%)) 
test_constructor() (gas: -400 (-0.010%)) 
```

```
Overall gas change: -431 (-0.001%)
```

*Code link:* [[111-122](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L111-L122)]


***
&nbsp;

                
### [GO-08] Using different conditional path
                
Many times, a different conditional path could lead to saving gas.

For example, it could permit to use `>=` instead of `>` or avoid using `NOT` operator.

*Issue's instances:* 4

#### File: [StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol)

*Instances:* 3    


```diff
File: StakedUSDeV2.sol

   95    function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {
-  96      if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();
+  96      if (assets <= maxWithdraw(owner)) {
   97  
   98      uint256 shares = previewWithdraw(assets);
   99  
   100     cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
   101     cooldowns[owner].underlyingAmount += assets;
   102  
   103     _withdraw(_msgSender(), address(silo), owner, assets, shares);
   104  
   105     return shares;
+  106     }
+  107     revert ExcessiveWithdrawAmount();
   106   }
```

```
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 1 (0.000%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -9 (-0.001%)) 
test_softFullBlacklist_withdraw_pass() (gas: 5 (0.001%)) 
test_fullBlacklist_withdraw_pass() (gas: -4 (-0.002%)) 
testFuzzCooldownShares(uint256) (gas: 5 (0.002%)) 
test_softBlacklist_withdraw_pass() (gas: 9 (0.003%)) 
testStakeFlowCommonUser() (gas: 9 (0.004%)) 
testFuzzCooldownAssetsUnstake(uint256) (gas: 13 (0.005%)) 
test_constructor() (gas: -200 (-0.005%)) 
testFuzzCooldownAssets(uint256) (gas: 13 (0.006%)) 
```

```
Overall gas change: -158 (-0.000%)
```

*Code link:* [[95-106](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L95-L106)]




***




```diff
File: StakedUSDeV2.sol

   111    function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
-  112      if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();
+  112      if (shares <= maxRedeem(owner)){
   113  
   114      uint256 assets = previewRedeem(shares);
   115  
   116      cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
   117      cooldowns[owner].underlyingAmount += assets;
   118  
   119      _withdraw(_msgSender(), address(silo), owner, assets, shares);
   120  
   121      return assets;
+  122      }
+  123      revert ExcessiveRedeemAmount();
   122    }
```

```
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 4 (0.001%)) 
testFuzzCooldownAssetsUnstake(uint256) (gas: 2 (0.001%)) 
testFuzzCooldownAssets(uint256) (gas: 2 (0.001%)) 
testCantCooldownBelowMinShares() (gas: -4 (-0.002%)) 
testFairStakeAndUnstakePrices() (gas: 9 (0.002%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: 8 (0.002%)) 
testUSDeValuePerStUSDe() (gas: 17 (0.003%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 24 (0.003%)) 
testStakeUnstake() (gas: 9 (0.004%)) 
test_constructor() (gas: -200 (-0.005%)) 
testFuzzCooldownShares(uint256) (gas: 12 (0.005%)) 
```

```
Overall gas change: -117 (-0.000%)
```

*Code link:* [[111-122](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L111-L122)]




***




```diff
File: StakedUSDeV2.sol

Current Code:
   126    function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
-  127      if (duration > MAX_COOLDOWN_DURATION) {
+  127      if (duration <= MAX_COOLDOWN_DURATION) {
-  128        revert InvalidCooldown();
-  129      }
   130  
   131      uint24 previousDuration = cooldownDuration;
   132      cooldownDuration = duration;
   133      emit CooldownDurationUpdated(previousDuration, cooldownDuration);
   134    }
+  134    else revert InvalidCooldown();
   135  }
```

```
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 4 (0.001%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -7 (-0.001%)) 
testFuzzCooldownAssetsUnstake(uint256) (gas: 9 (0.004%)) 
testFuzzCooldownAssets(uint256) (gas: 9 (0.004%)) 
testFuzzCooldownShares(uint256) (gas: 10 (0.004%)) 
testSetCooldown_error_gt_max() (gas: -2 (-0.013%)) 
test_fails_v2_if_set_duration_zero() (gas: -4 (-0.017%)) 
testSetCooldown_fuzz(uint24) (gas: -4 (-0.018%)) 
testSetCooldown_zero() (gas: -4 (-0.018%)) 
test_constructor() (gas: -10218 (-0.267%)) 
```

```
Overall gas change: -10207 (-0.024%)
```

*Code link:* [[126-1234](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L126-L1234)]



***
&nbsp;

#### File: [USDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol)

*Instances:* 1    


```diff
File: USDe.sol

-  28    function mint(address to, uint256 amount) external {
-  29      if (msg.sender != minter) revert OnlyMinter();
-  30      _mint(to, amount);
+  29      if (msg.sender == minter) _mint(to, amount);
+  30      else revert OnlyMinter();
-  31    }
```

```
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -1 (-0.000%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 1 (0.000%)) 
test_softFullBlacklist_withdraw_pass() (gas: -1 (-0.000%)) 
test_multipleValid_custodyRatios_addresses() (gas: -1 (-0.000%)) 
testDelegateSuccessfulRedeem() (gas: -1 (-0.000%)) 
test_gatekeeper_cannot_enable_redeem_revert() (gas: -1 (-0.000%)) 
test_fuzz_nonAdmin_cannot_enable_redeem_revert(address) (gas: -1 (-0.000%)) 
test_admin_can_enable_redeem() (gas: -1 (-0.000%)) 
test_redeem_notRedeemer_revert() (gas: -1 (-0.000%)) 
test_fuzz_nextBlock_redeem_is_zero(uint256) (gas: -1 (-0.000%)) 
test_nativeEth_withdraw() (gas: -1 (-0.000%)) 
testDelegateFailureRedeem() (gas: -1 (-0.000%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -1 (-0.000%)) 
test_softBlacklist_withdraw_pass() (gas: -1 (-0.000%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -1 (-0.000%)) 
test_fuzz_maxRedeem_perBlock_exceeded_revert(uint256) (gas: -1 (-0.000%)) 
test_softFullBlacklist_withdraw_pass() (gas: -1 (-0.000%)) 
test_fullBlacklist_withdraw_pass() (gas: -1 (-0.000%)) 
test_fullBlacklist_can_not_be_transfer_recipient() (gas: -1 (-0.000%)) 
testCantCooldownBelowMinShares() (gas: -1 (-0.000%)) 
testStakeFlowCommonUser() (gas: -1 (-0.000%)) 
test_sending_redeem_order_to_mint_revert() (gas: -1 (-0.000%)) 
testFuzzCooldownAssetsUnstake(uint256) (gas: 1 (0.000%)) 
testBlacklistManagerCannotRedistribute() (gas: -1 (-0.000%)) 
testBlacklistManagerCannotRedistribute() (gas: -1 (-0.000%)) 
testStakeUnstake() (gas: -1 (-0.000%)) 
test_softFullBlacklist_transfer_pass() (gas: -1 (-0.000%)) 
test_softFullBlacklist_transfer_pass() (gas: -1 (-0.000%)) 
testDelegateSuccessfulMint() (gas: -1 (-0.000%)) 
testFuzzCooldownAssets(uint256) (gas: 1 (0.000%)) 
test_redistributeLockedAmount() (gas: -1 (-0.000%)) 
test_redistributeLockedAmount() (gas: -1 (-0.000%)) 
test_multiple_redeem() (gas: -2 (-0.000%)) 
test_softBlacklist_transferFrom_pass() (gas: -1 (-0.000%)) 
test_softBlacklist_transferFrom_pass() (gas: -1 (-0.000%)) 
test_admin_can_enable_mint() (gas: -1 (-0.000%)) 
test_softBlacklist_withdraw_pass() (gas: -1 (-0.000%)) 
testCantWithdrawBelowMinShares() (gas: -1 (-0.000%)) 
test_fuzz_mint_noSlippage(uint256) (gas: -1 (-0.000%)) 
testCantWithdrawBelowMinShares() (gas: -1 (-0.000%)) 
testUSDeValuePerStUSDe() (gas: -2 (-0.000%)) 
test_fuzz_nextBlock_mint_is_zero(uint256) (gas: -1 (-0.000%)) 
test_fullBlacklist_transferFrom_pass() (gas: -1 (-0.000%)) 
test_softBlacklist_transfer_pass() (gas: -1 (-0.001%)) 
testOwnerCanRescuestUSDe() (gas: -1 (-0.001%)) 
testOwnerCanRescuestUSDe() (gas: -1 (-0.001%)) 
testOwnerCanRescuestUSDe() (gas: -1 (-0.001%)) 
test_mint() (gas: -1 (-0.001%)) 
testCanTransferRewardsAfterVesting() (gas: -1 (-0.001%)) 
test_fullBlacklist_withdraw_pass() (gas: -1 (-0.001%)) 
testStakeFlowCommonUser() (gas: -1 (-0.001%)) 
testMintWithSlippageCheck(uint256) (gas: -1 (-0.001%)) 
testMintWithSlippageCheck(uint256) (gas: -1 (-0.001%)) 
testMintWithSlippageCheck(uint256) (gas: -1 (-0.001%)) 
testStakeUnstake() (gas: -1 (-0.001%)) 
test_fullBlacklist_transfer_pass() (gas: -1 (-0.001%)) 
test_fullBlacklist_transfer_pass() (gas: -1 (-0.001%)) 
testStakeUnstake() (gas: -1 (-0.001%)) 
testCanBurnOnRedistribute() (gas: -1 (-0.001%)) 
testCanBurnOnRedistribute() (gas: -1 (-0.001%)) 
test_fullBlacklist_user_can_not_burn_and_donate_to_vault() (gas: -1 (-0.001%)) 
test_fullBlacklist_user_can_not_burn_and_donate_to_vault() (gas: -1 (-0.001%)) 
testFairStakeAndUnstakePrices() (gas: -2 (-0.001%)) 
testUSDeValuePerStUSDe() (gas: -3 (-0.001%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -2 (-0.001%)) 
testInitialStakeBelowMin() (gas: -1 (-0.001%)) 
testInitialStakeBelowMin() (gas: -1 (-0.001%)) 
testInitialStakeBelowMin() (gas: -1 (-0.001%)) 
testOwnerCannotRescueUSDe() (gas: -1 (-0.001%)) 
testOwnerCannotRescueUSDe() (gas: -1 (-0.001%)) 
testOwnerCannotRescueUSDe() (gas: -1 (-0.001%)) 
testMintToDiffRecipient() (gas: -1 (-0.001%)) 
testMintToDiffRecipient() (gas: -1 (-0.001%)) 
testMintToDiffRecipient() (gas: -1 (-0.001%)) 
testInitialStake() (gas: -1 (-0.001%)) 
testInitialStake() (gas: -1 (-0.001%)) 
testInitialStake() (gas: -1 (-0.001%)) 
testCannotTransferRewardsWhileVesting() (gas: -1 (-0.001%)) 
testCannotTransferRewardsWhileVesting() (gas: -1 (-0.001%)) 
testTransferRewardsFailsInsufficientBalance() (gas: -1 (-0.001%)) 
testTransferRewardsFailsInsufficientBalance() (gas: -1 (-0.001%)) 
testTransferRewardsFailsInsufficientBalance() (gas: -1 (-0.001%)) 
test_softBlacklist_deposit_reverts() (gas: -1 (-0.001%)) 
test_softBlacklist_deposit_reverts() (gas: -1 (-0.001%)) 
testFuzzNoJumpInVestedBalance(uint256) (gas: -1 (-0.001%)) 
testFuzzNoJumpInVestedBalance(uint256) (gas: -1 (-0.001%)) 
testFuzzNoJumpInVestedBalance(uint256) (gas: -1 (-0.001%)) 
testFairStakeAndUnstakePrices() (gas: -3 (-0.001%)) 
test_fullBlacklist_deposit_reverts() (gas: -1 (-0.001%)) 
test_fullBlacklist_deposit_reverts() (gas: -1 (-0.001%)) 
testUSDeValuePerStUSDe() (gas: -3 (-0.001%)) 
test_fullBlacklist_can_not_be_transfer_recipient() (gas: -2 (-0.001%)) 
testOwnerCanChangeRewarder() (gas: -2 (-0.001%)) 
testOwnerCanChangeRewarder() (gas: -2 (-0.001%)) 
testOwnerCanChangeRewarder() (gas: -2 (-0.001%)) 
testDonationAttack() (gas: -2 (-0.001%)) 
testDonationAttack() (gas: -2 (-0.001%)) 
test_multiple_mints() (gas: -2 (-0.001%)) 
testFairStakeAndUnstakePrices() (gas: -3 (-0.001%)) 
testFuzzCooldownShares(uint256) (gas: 2 (0.001%)) 
testOnlyRewarderCanReward() (gas: -2 (-0.001%)) 
testOnlyRewarderCanReward() (gas: -2 (-0.001%)) 
testOnlyRewarderCanReward() (gas: -2 (-0.001%)) 
test_softFullBlacklist_deposit_reverts() (gas: -2 (-0.001%)) 
test_softFullBlacklist_deposit_reverts() (gas: -2 (-0.001%)) 
testCanTransferRewardsAfterVesting() (gas: -2 (-0.001%)) 
testTransferRewardsFailsZeroAmount() (gas: -1 (-0.001%)) 
testTransferRewardsFailsZeroAmount() (gas: -1 (-0.001%)) 
testTransferRewardsFailsZeroAmount() (gas: -1 (-0.001%)) 
testCannotStakeWithoutApproval() (gas: -1 (-0.001%)) 
testCannotStakeWithoutApproval() (gas: -1 (-0.001%)) 
testCannotStakeWithoutApproval() (gas: -1 (-0.001%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -8 (-0.001%)) 
testNewMinterCanMint() (gas: -1 (-0.001%)) 
testMinterCanMint() (gas: -1 (-0.002%)) 
test_role_authorization() (gas: 2 (0.002%)) 
testOldMinterCantMint() (gas: 1 (0.003%)) 
testOwnerCantMint() (gas: 1 (0.006%)) 
testMinterCantMintToZeroAddress() (gas: -1 (-0.008%)) 
```

```
Overall gas change: -135 (-0.000%)
```

*Code link:* [[28-31](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L28-L31)]



***
&nbsp;
