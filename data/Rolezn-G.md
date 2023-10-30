Disclaimer: Several Gas findings now contain an <b>[Enhanced]</b> tag which indicate findings that have enhanced details which include per-instance foundry test gas reports and its gas usage. While some of the instances may be mentioned in the automated report, however, this report provides accurate gas-based test reports in order to help prioritize specific changes that can significantly reduce gas usage.

## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | Functions guaranteed to revert when called by normal users can be marked `payable` <b>[Enhanced]</b> | 12 | 29983 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Use do while loops instead of for loops <b>[Enhanced]</b> | 1 | 12759 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Using XOR (^) and AND (&) bitwise equivalents <b>[Enhanced]</b> | 1 | 400 |
| [GAS&#x2011;4](#GAS&#x2011;4) | `unchecked {}` can be used on the division of two `uints` in order to save gas <b>[Enhanced]</b> | 1 | 6816 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Use assembly to check for `address(0)` <b>[Enhanced]</b> | 1 | 10496 |
| [GAS&#x2011;6](#GAS&#x2011;6) | Using `delete` statement can save gas <b>[Enhanced]</b> | 1 | 8 |
| [GAS&#x2011;7](#GAS&#x2011;7) | `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops <b>[Enhanced]</b> | 1 | 3288 |

Total: 18 contexts over 7 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### <ins>Proof Of Concept</ins>

Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-29983}}$ gas usage (includes proof of forge test runs):

<details><summary>Contains 12 gas optimization instances</summary>

Summary: Overall gas change: -2874 (-0.007\%)

```solidity
function setMaxMintPerBlock(uint256 _maxMintPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L219

Summary: Overall gas change: -5675 (-0.014\%)

```solidity
function setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L224

Summary: Overall gas change: -8331 (-0.020\%)

```solidity
function disableMintRedeem() external onlyRole(GATEKEEPER_ROLE) payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L229

Summary: Overall gas change: -10990 (-0.026\%)

```solidity
function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L247

Summary: Overall gas change: -13689 (-0.033\%)

```solidity
function removeSupportedAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE) payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L259

Summary: Overall gas change: -16323 (-0.039\%)

```solidity
function removeCustodianAddress(address custodian) external onlyRole(DEFAULT_ADMIN_ROLE) payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L270

Summary: Overall gas change: -18971 (-0.045\%)

```solidity
function removeMinterRole(address minter) external onlyRole(GATEKEEPER_ROLE) payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L277

Summary: Overall gas change: -21627 (-0.052\%)

```solidity
function removeRedeemerRole(address redeemer) external onlyRole(GATEKEEPER_ROLE) payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L283

Summary: Overall gas change: -24400 (-0.058\%)

```solidity
function addSupportedAsset(address asset) public onlyRole(DEFAULT_ADMIN_ROLE) payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L290

Summary: Overall gas change: -27056 (-0.065\%)

```solidity
function addCustodianAddress(address custodian) public onlyRole(DEFAULT_ADMIN_ROLE) payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L298

Summary: Overall gas change: -27106 (-0.065\%)

```solidity
function setMinter(address newMinter) external onlyOwner payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/USDe.sol#L23

Summary: Overall gas change: -29983 (-0.072\%)

```solidity
function withdraw(address to, uint256 amount) external onlyStakingVault payable {
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/USDeSilo.sol#L28

</details>

#### <ins>Forge test</ins>

Running forge test comparison report:

<details>


Ran 11 test suites: 243 tests passed, 0 failed, 0 skipped (243 total tests)

testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 6 (0.001\%)) 

testFuzzCooldownAssets(uint256) (gas: -9 (-0.004\%)) 

testFuzzCooldownShares(uint256) (gas: -10 (-0.004\%)) 

testFairStakeAndUnstakePrices() (gas: -19 (-0.005\%))
 
testStakingAndUnstakingBeforeAfterReward() (gas: -20 (-0.006\%))
 
test_redeem_invalidNonce_revert() (gas: -19 (-0.006\%))
 
test_softBlacklist_withdraw_pass() (gas: -19 (-0.007\%))
 
testUSDeValuePerStUSDe() (gas: -39 (-0.007\%))
 
testStakeFlowCommonUser() (gas: -19 (-0.007\%))
 
testStakeUnstake() (gas: -19 (-0.008\%))
 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -60 (-0.009\%)) 

test_fuzz_maxRedeem_perBlock_exceeded_revert(uint256) (gas: -24 (-0.009\%)) 

test_fullBlacklist_withdraw_pass() (gas: -24 (-0.009\%))
 
testFuzzCooldownAssetsUnstake(uint256) (gas: -28 (-0.011\%)) 

test_softFullBlacklist_withdraw_pass() (gas: -39 (-0.011\%))
 
test_admin_can_enable_redeem() (gas: -38 (-0.013\%))
 
test_multipleValid_custodyRatios_addresses() (gas: -48 (-0.014\%))
 
test_unsupported_assets_ERC20_revert() (gas: -24 (-0.014\%))
 
test_gatekeeper_can_disable_mintRedeem() (gas: -24 (-0.021\%))
 
test_gatekeeper_cannot_enable_redeem_revert() (gas: -72 (-0.022\%))
 
test_fuzz_nonAdmin_cannot_enable_redeem_revert(address) (gas: -72 (-0.022\%)) 

test_admin_can_enable_mint() (gas: -48 (-0.023\%))
 
test_fuzz_nonMinter_cannot_transferCustody_revert(address) (gas: -24 (-0.025\%)) 

test_gatekeeper_cannot_enable_mint_revert() (gas: -48 (-0.032\%))
 
test_fuzz_nonAdmin_cannot_enable_mint_revert(address) (gas: -48 (-0.033\%)) 

testNewMinterCanMint() (gas: -24 (-0.033\%))
 
test_fuzz_not_gatekeeper_cannot_remove_minter_revert(address) (gas: -24 (-0.038\%)) 

test_fuzz_not_gatekeeper_cannot_remove_redeemer_revert(address) (gas: -24 (-0.038\%)) 

test_minter_canTransfer_custody() (gas: -48 (-0.038\%))
 
test_fuzz_not_gatekeeper_cannot_disable_mintRedeem_revert(address) (gas: -24 (-0.039\%)) 

testNewOwnerCanPerformOwnerActions() (gas: -19 (-0.044\%))
 
testOldOwnerCantSetMinter() (gas: -19 (-0.047\%))
 
test_fuzz_nonOwner_cannot_remove_supportedAsset_revert(address) (gas: -48 (-0.047\%)) 

test_fuzz_nonOwner_cannot_add_supportedAsset_revert(address) (gas: -24 (-0.054\%)) 

test_cannot_add_asset_already_supported_revert() (gas: -48 (-0.066\%))
 
test_constructor() (gas: -2600 (-0.068\%))
 
test_add_and_remove_supported_asset() (gas: -38 (-0.068\%))
 
testCantInitWithNoOwner() (gas: 82 (0.073\%))
 
testOldMinterCantMint() (gas: -24 (-0.074\%))
 
test_fuzz_maxRedeem_perBlock_setter(uint256) (gas: -24 (-0.094\%)) 

test_fuzz_maxMint_perBlock_setter(uint256) (gas: -24 (-0.102\%)) 

test_gatekeeper_can_remove_minter() (gas: -24 (-0.114\%))
 
test_gatekeeper_can_remove_redeemer() (gas: -24 (-0.114\%))
 
testOnlyOwnerCanSetMinter() (gas: -24 (-0.118\%))
 
test_cannotAdd_USDe_revert() (gas: -24 (-0.121\%))
 
test_cannot_removeAsset_not_supported_revert() (gas: -24 (-0.125\%))
 
test_admin_can_disable_mint(bool) (gas: -24 (-0.152\%)) 

test_admin_can_disable_redeem(bool) (gas: -24 (-0.153\%)) 

test_cannotAdd_addressZero_revert() (gas: -24 (-0.153\%))
 
testCorrectInitConfig() (gas: -26073 (-0.712\%))
 
Overall gas change: -29983 (-0.072\%)
</details>



#### <ins>Invalid instances</ins>

Reports that contain these instances should be invalidated as they either break the foundry test runs, increase the overall gas usage or provide 0 gas optimization:

<details><summary>Contains 12 invalid gas optimization instances</summary>

Summary: Error (6959): Overriding function changes state mutability from "nonpayable" to "payable".

```solidity
File: StakedUSDe.sol

function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
```

Summary: Overall gas change: 135268 (0.323\%)

```solidity
File: StakedUSDe.sol

function addToBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
```

Summary: Overall gas change: 135497 (0.323\%)

```solidity
File: StakedUSDe.sol

function removeFromBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
```

Summary: Error (6959): Overriding function changes state mutability from "nonpayable" to "payable".

```solidity
File: StakedUSDe.sol

function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

Summary: Overall gas change: 135476 (0.323\%)

```solidity
File: StakedUSDe.sol

function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

Summary: Error (6959): Overriding function changes state mutability from "nonpayable" to "payable".

```solidity
File: EthenaMinting.sol

function mint(Order calldata order, Route calldata route, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(MINTER_ROLE)
    belowMaxMintPerBlock(order.usde_amount)
  {
```

Summary: Error (6959): Overriding function changes state mutability from "nonpayable" to "payable".

```solidity
File: EthenaMinting.sol

function redeem(Order calldata order, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(REDEEMER_ROLE)
    belowMaxRedeemPerBlock(order.usde_amount)
  {
```

Summary: Error (6959): Overriding function changes state mutability from "nonpayable" to "payable".

```solidity
File: StakedUSDeV2.sol

function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

Summary: Error (9680): State mutability already specified as "view".

```solidity
File: USDe.sol

function renounceOwnership() public view override onlyOwner {
```

Summary: Overall gas change: 132407 (0.316\%)

```solidity
File: SingleAdminAccessControl.sol

function transferAdmin(address newAdmin) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

Summary: Error (6959): Overriding function changes state mutability from "nonpayable" to "payable".

```solidity
File: SingleAdminAccessControl.sol

function grantRole(bytes32 role, address account) public override onlyRole(DEFAULT_ADMIN_ROLE) notAdmin(role) {
```

Summary: Error (6959): Overriding function changes state mutability from "nonpayable" to "payable".

```solidity
File: SingleAdminAccessControl.sol

function revokeRole(bytes32 role, address account) public override onlyRole(DEFAULT_ADMIN_ROLE) notAdmin(role) {
```

</details>





### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

#### <ins>Proof Of Concept</ins>

Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-12759}}$ gas usage (includes proof of forge test runs):

Summary: Overall gas change: -12759 (-0.030\%)

```solidity
uint256 i = 0;
do {

      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
     ++i;
} while ( i < addresses.length);
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L414

#### <ins>Forge test</ins>

Omitting 0 gas test results for readability..

Running forge test comparison report:

<details>


Ran 11 test suites: 243 tests passed, 0 failed, 0 skipped (243 total tests)

testDelegateSuccessfulRedeem() (gas: -35 (-0.011\%))
 
test_redeem_invalidNonce_revert() (gas: -35 (-0.011\%))
 
test_admin_can_enable_redeem() (gas: -35 (-0.012\%))
 
test_redeem() (gas: -35 (-0.012\%))
 
test_fuzz_nextBlock_redeem_is_zero(uint256) (gas: -35 (-0.012\%)) 

test_nativeEth_withdraw() (gas: -35 (-0.012\%))
 
test_gatekeeper_cannot_enable_redeem_revert() (gas: -44 (-0.014\%))
 
test_fuzz_nonAdmin_cannot_enable_redeem_revert(address) (gas: -44 (-0.014\%)) 

test_redeem_notRedeemer_revert() (gas: -44 (-0.015\%))
 
testDelegateFailureRedeem() (gas: -44 (-0.015\%))
 
test_multiple_redeem() (gas: -70 (-0.016\%))
 
test_fuzz_maxRedeem_perBlock_exceeded_revert(uint256) (gas: -44 (-0.016\%)) 

test_sending_redeem_order_to_mint_revert() (gas: -44 (-0.017\%))
 
testDelegateSuccessfulMint() (gas: -44 (-0.018\%))
 
test_admin_can_enable_mint() (gas: -44 (-0.021\%))
 
test_fuzz_mint_noSlippage(uint256) (gas: -44 (-0.021\%)) 

test_fuzz_nextBlock_mint_is_zero(uint256) (gas: -44 (-0.022\%)) 

test_mint() (gas: -44 (-0.022\%))
 
test_multipleValid_custodyRatios_addresses() (gas: -78 (-0.023\%))
 
test_multiple_mints() (gas: -88 (-0.035\%))
 
testCorrectInitConfig() (gas: -11829 (-0.323\%))
 
Overall gas change: -12759 (-0.030\%)
</details>



### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-400}}$ gas usage (includes proof of forge test runs):

Summary: Overall gas change: -400 (-0.001\%)

```solidity
if ((nonce ^ 0 == 0)) revert InvalidNonce();
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L378

</details>

#### <ins>Forge test</ins>

Omitting 0 gas test results for readability..

Running forge test comparison report:

<details>


Ran 11 test suites: 243 tests passed, 0 failed, 0 skipped (243 total tests)
testCorrectInitConfig() (gas: -400 (-0.011\%))
 
Overall gas change: -400 (-0.001\%)
</details>




#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

#### <ins>Invalid instances</ins>

Reports that contain these instances should be invalidated as they either break the foundry test runs, increase the overall gas usage or provide 0 gas optimization:

<details><summary>Contains 15 invalid gas optimization instances</summary>

Summary: Error (2271): Built-in binary operator ^ cannot be applied to types address and type(address). Arithmetic operations on addresses are not supported. Convert to integer first before using them.

```solidity
File: EthenaMinting.sol

if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();
```

Summary: Error (2271): Built-in binary operator ^ cannot be applied to types address and address. Arithmetic operations on addresses are not supported. Convert to integer first before using them.

```solidity
File: EthenaMinting.sol

if (asset == NATIVE_TOKEN) {
```

Summary: Error (2271): Built-in binary operator ^ cannot be applied to types address and type(address). Arithmetic operations on addresses are not supported. Convert to integer first before using them.

```solidity
File: EthenaMinting.sol

if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {
```

Summary: Error (2271): Built-in binary operator ^ cannot be applied to types address and type(address). Arithmetic operations on addresses are not supported. Convert to integer first before using them.

```solidity
File: EthenaMinting.sol

if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
```

Summary: Overall gas change: 1143 (0.003\%)

```solidity
File: EthenaMinting.sol

if (block.chainid == _chainId) {
```

Summary: Error (2271): Built-in binary operator ^ cannot be applied to types address and address. Arithmetic operations on addresses are not supported. Convert to integer first before using them.

```solidity
File: EthenaMinting.sol

if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
```

Summary: Error (2271): Built-in binary operator ^ cannot be applied to types address and type(address). Arithmetic operations on addresses are not supported. Convert to integer first before using them.

```solidity
File: EthenaMinting.sol

if (order.beneficiary == address(0)) revert InvalidAmount();
```

Summary: Overall gas change: 0 (0.000\%)

```solidity
File: EthenaMinting.sol

if (order.collateral_amount == 0) revert InvalidAmount();
```

Summary: Overall gas change: 0 (0.000\%)

```solidity
File: EthenaMinting.sol

if (order.usde_amount == 0) revert InvalidAmount();
```

Summary: Error (2271): Built-in binary operator ^ cannot be applied to types enum IEthenaMinting.OrderType and enum IEthenaMinting.OrderType. No matching user-defined operator found.

```solidity
File: EthenaMinting.sol

if (orderType == OrderType.REDEEM) {
```

Summary: Overall gas change: 0 (0.000\%)

```solidity
File: EthenaMinting.sol

if (route.addresses.length == 0) {
```

Summary: Error (2271): Built-in binary operator ^ cannot be applied to types address and address. Arithmetic operations on addresses are not supported. Convert to integer first before using them.

```solidity
File: EthenaMinting.sol

if (asset == NATIVE_TOKEN) {
```

Summary: Error (2271): Built-in binary operator ^ cannot be applied to types address and address. Arithmetic operations on addresses are not supported. Convert to integer first before using them.

```solidity
File: EthenaMinting.sol

if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
```

Summary: Error (2271): Built-in binary operator ^ cannot be applied to types address and address. Arithmetic operations on addresses are not supported. Convert to integer first before using them.

```solidity
File: SingleAdminAccessControl.sol

if (newAdmin == msg.sender) revert InvalidAdminChange();
```

Summary: Overall gas change: 2647 (0.006\%)

```solidity
File: SingleAdminAccessControl.sol

if (role == DEFAULT_ADMIN_ROLE) {
```

</details>




### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> `unchecked {}` can be used on the division of two `uints` in order to save gas

Make such found divisions are unchecked when ensured it is safe to do so.

#### <ins>Proof Of Concept</ins>


Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-6186}}$ gas usage (includes proof of forge test runs):

Summary: Overall gas change: -6186 (-0.015\%)

```solidity
uint256 amountToTransfer;
unchecked {
	amountToTransfer = (amount * ratios[i]) / 10_000;
}
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L425

#### <ins>Forge test</ins>

Omitting 0 gas test results for readability..

Running forge test comparison report:

<details>


Ran 11 test suites: 243 tests passed, 0 failed, 0 skipped (243 total tests)

testDelegateSuccessfulRedeem() (gas: -99 (-0.030\%))
 
test_redeem_invalidNonce_revert() (gas: -99 (-0.032\%))
 
test_admin_can_enable_redeem() (gas: -99 (-0.033\%))
 
test_redeem() (gas: -99 (-0.034\%))
 
test_fuzz_nextBlock_redeem_is_zero(uint256) (gas: -99 (-0.034\%)) 

test_nativeEth_withdraw() (gas: -99 (-0.034\%))
 
test_gatekeeper_cannot_enable_redeem_revert() (gas: -124 (-0.038\%))
 
test_fuzz_nonAdmin_cannot_enable_redeem_revert(address) (gas: -124 (-0.039\%)) 

test_redeem_notRedeemer_revert() (gas: -124 (-0.042\%))
 
testDelegateFailureRedeem() (gas: -124 (-0.043\%))
 
test_multiple_redeem() (gas: -198 (-0.044\%))
 
test_fuzz_maxRedeem_perBlock_exceeded_revert(uint256) (gas: -124 (-0.046\%)) 

test_sending_redeem_order_to_mint_revert() (gas: -124 (-0.049\%))
 
testDelegateSuccessfulMint() (gas: -124 (-0.051\%))
 
test_admin_can_enable_mint() (gas: -124 (-0.058\%))
 
test_fuzz_mint_noSlippage(uint256) (gas: -124 (-0.059\%)) 

test_fuzz_nextBlock_mint_is_zero(uint256) (gas: -124 (-0.061\%)) 

test_mint() (gas: -124 (-0.063\%))
 
testCorrectInitConfig() (gas: -3410 (-0.093\%))
 
test_multiple_mints() (gas: -248 (-0.099\%))
 
test_multipleValid_custodyRatios_addresses() (gas: -372 (-0.108\%))
 
Overall gas change: -6186 (-0.015\%)
</details>



### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Use assembly to check for `address(0)`

#### <ins>Proof Of Concept</ins>


Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-10496}}$ gas usage (includes proof of forge test runs):

Summary: Overall gas change: -10496 (-0.025\%)

```solidity
assembly {
	if iszero(wallet) {
		mstore(0x00, "AddressZero")
		revert(0x00, 0x20)
	}
}
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L248

</details>

#### <ins>Forge test</ins>

Omitting 0 gas test results for readability..

Running forge test comparison report:

<details>


Ran 11 test suites: 243 tests passed, 0 failed, 0 skipped (243 total tests)
test_minter_canTransfer_custody() (gas: -267 (-0.214\%))
 
testCorrectInitConfig() (gas: -10229 (-0.279\%))
 
Overall gas change: -10496 (-0.025\%)
</details>



#### <ins>Invalid instances</ins>

Reports that contain these instances should be invalidated as they either break the foundry test runs, increase the overall gas usage or provide 0 gas optimization:

Summary: FAIL

```solidity
File: StakedUSDe.sol

if (to != address(0)) _mint(to, amountToDistribute);
```

Summary: Error (8198): Identifier "order.beneficiary" not found.

```solidity
File: EthenaMinting.sol

if (order.beneficiary == address(0)) revert InvalidAmount();
```



### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-8}}$ gas usage (includes proof of forge test runs):

Summary: Overall gas change: -8 (-0.000\%)

```solidity
delete(userCooldown.cooldownEnd);

```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/StakedUSDeV2.sol#L83

</details>

#### <ins>Forge test</ins>

Omitting 0 gas test results for readability..

Running forge test comparison report:

<details>


Ran 11 test suites: 243 tests passed, 0 failed, 0 skipped (243 total tests)

testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 3 (0.000\%)) 

testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: 4 (0.001\%)) 

testFuzzCooldownAssetsUnstake(uint256) (gas: -5 (-0.002\%)) 

testFuzzCooldownAssets(uint256) (gas: -5 (-0.002\%)) 

testFuzzCooldownShares(uint256) (gas: -5 (-0.002\%)) 

Overall gas change: -8 (-0.000\%)
</details>



#### <ins>Invalid instances</ins>

Reports that contain these instances should be invalidated as they either break the foundry test runs, increase the overall gas usage or provide 0 gas optimization:

Summary: Error (7576): Undeclared identifier.

```solidity
File: EthenaMinting.sol

uint256 totalRatio = 0;
```

Summary: Error (7576): Undeclared identifier.

```solidity
File: EthenaMinting.sol

uint256 totalTransferred = 0;
```

Summary: Overall gas change: 6 (0.000\%)

```solidity
File: StakedUSDeV2.sol

userCooldown.underlyingAmount = 0;
```





### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

#### <ins>Proof Of Concept</ins>


Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-3288}}$ gas usage (includes proof of forge test runs):

Summary: Overall gas change: -3288 (-0.008\%)

```solidity
    for (uint256 i = 0; i < addresses.length; ) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    	unchecked {++i;}
}
```

https://github.com/code-423n4/2023-10-ethena/tree/main/contracts/EthenaMinting.sol#L424

</details>

#### <ins>Forge test</ins>

Omitting 0 gas test results for readability..

Running forge test comparison report:

<details>


Ran 11 test suites: 243 tests passed, 0 failed, 0 skipped (243 total tests)

testDelegateSuccessfulRedeem() (gas: -53 (-0.016\%))
 
test_redeem_invalidNonce_revert() (gas: -52 (-0.017\%))
 
test_admin_can_enable_redeem() (gas: -53 (-0.018\%))
 
test_redeem() (gas: -52 (-0.018\%))
 
test_fuzz_nextBlock_redeem_is_zero(uint256) (gas: -53 (-0.018\%)) 

test_nativeEth_withdraw() (gas: -53 (-0.018\%))
 
test_gatekeeper_cannot_enable_redeem_revert() (gas: -66 (-0.020\%))
 
test_fuzz_nonAdmin_cannot_enable_redeem_revert(address) (gas: -66 (-0.021\%)) 

test_redeem_notRedeemer_revert() (gas: -66 (-0.022\%))
 
testDelegateFailureRedeem() (gas: -66 (-0.023\%))
 
test_multiple_redeem() (gas: -106 (-0.024\%))
 
test_fuzz_maxRedeem_perBlock_exceeded_revert(uint256) (gas: -66 (-0.024\%)) 

test_sending_redeem_order_to_mint_revert() (gas: -66 (-0.026\%))
 
testDelegateSuccessfulMint() (gas: -66 (-0.027\%))
 
test_admin_can_enable_mint() (gas: -66 (-0.031\%))
 
test_fuzz_mint_noSlippage(uint256) (gas: -66 (-0.032\%)) 

test_fuzz_nextBlock_mint_is_zero(uint256) (gas: -66 (-0.033\%)) 

test_mint() (gas: -66 (-0.033\%))
 
testCorrectInitConfig() (gas: -1810 (-0.049\%))
 
test_multiple_mints() (gas: -132 (-0.053\%))
 
test_multipleValid_custodyRatios_addresses() (gas: -198 (-0.058\%))
 
Overall gas change: -3288 (-0.008\%)
</details>