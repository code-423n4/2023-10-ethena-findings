## [G-01] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

Consider using abi.encodePacked() here:

Use abi.encodePacked() where possible to save gas

There are 3 instances of this issue in 1 file:

```
File: EthenaMinting.sol

321: return abi.encode(

335: return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);

452: return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```
    diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
    index 32da3a5..8dcf40a 100644
    --- a/contracts/EthenaMinting.sol
    +++ b/contracts/EthenaMinting.sol
    @@ -318,7 +318,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
       }

       function encodeOrder(Order calldata order) public pure returns (bytes memory) {
    -    return abi.encode(
    +    return abi.encodePacked(
           ORDER_TYPE,
           order.order_type,
           order.expiry,
    @@ -332,7 +332,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
       }

       function encodeRoute(Route calldata route) public pure returns (bytes memory) {
    -    return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);
    +    return abi.encodePacked(ROUTE_TYPE, route.addresses, route.ratios);
       }

       /// @notice assert validity of signed order
    @@ -449,6 +449,6 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
       /// @notice Compute the current domain separator
       /// @return The domain separator for the token
       function _computeDomainSeparator() internal view returns (bytes32) {
    -    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
    +    return keccak256(abi.encodePacked(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
       }
     }

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-02] Using XOR (^) and AND (&) bitwise equivalents

On Remix, given only uint256 types, the following are logical equivalents, but don’t cost the same amount of gas:

(a != b || c != d || e != f) costs 571
((a ^ b) | (c ^ d) | (e ^ f)) != 0 costs 498 (saving 73 gas)
Consider rewriting as following to save gas:

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.
Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.

Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas:

However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values

There is 1 instance of this issue in 1 file:

```
File: StakedUSDe.sol

75: if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {
```

    diff --git a/contracts/StakedUSDe.sol b/contracts/StakedUSDe.sol
    index 0a56a7d..627362f 100644
    --- a/contracts/StakedUSDe.sol
    +++ b/contracts/StakedUSDe.sol
    @@ -72,7 +72,7 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
         ERC4626(_asset)
         ERC20Permit("stUSDe")
       {
    -    if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {
    +    if (((_owner ^ address(0)) & (_initialRewarder ^ address(0)) & (address(_asset) ^ address(0))) == 0) {
           revert InvalidZeroAddress();
         }

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol

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

#### Gas Report

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

## [G-03] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

There are 6 instances of this issue in 2 files:

```
File: EthenaMinting.sol

174: mintedPerBlock[block.number] += order.usde_amount;

205: redeemedPerBlock[block.number] += order.usde_amount;

368: totalRatio += route.ratios[i];

427: totalTransferred += amountToTransfer;
```

    diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
    index 32da3a5..0cd1c35 100644
    --- a/contracts/EthenaMinting.sol
    +++ b/contracts/EthenaMinting.sol
    @@ -171,7 +171,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
         if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
         if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
         // Add to the minted amount in this block
    -    mintedPerBlock[block.number] += order.usde_amount;
    +    mintedPerBlock[block.number] = mintedPerBlock[block.number] + order.usde_amount;
         _transferCollateral(
           order.collateral_amount, order.collateral_asset, order.benefactor, route.addresses, route.ratios
         );
    @@ -202,7 +202,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
         verifyOrder(order, signature);
         if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
         // Add to the redeemed amount in this block
    -    redeemedPerBlock[block.number] += order.usde_amount;
    +    redeemedPerBlock[block.number] = redeemedPerBlock[block.number] + order.usde_amount;
         usde.burnFrom(order.benefactor, order.usde_amount);
         _transferToBeneficiary(order.beneficiary, order.collateral_asset, order.collateral_amount);
         emit Redeem(
    @@ -365,7 +365,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
           {
             return false;
           }
    -      totalRatio += route.ratios[i];
    +      totalRatio = totalRatio + route.ratios[i];
         }
         if (totalRatio != 10_000) {
           return false;
    @@ -424,7 +424,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
         for (uint256 i = 0; i < addresses.length; ++i) {
           uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
           token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
    -      totalTransferred += amountToTransfer;
    +      totalTransferred = totalTransferred + amountToTransfer;
         }
         uint256 remainingBalance = amount - totalTransferred;
         if (remainingBalance > 0) {

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol

```
File: StakedUSDeV2.sol

101: cooldowns[owner].underlyingAmount += assets;

117: cooldowns[owner].underlyingAmount += assets;
```

    diff --git a/contracts/StakedUSDeV2.sol b/contracts/StakedUSDeV2.sol
    index df2bb48..c39ca51 100644
    --- a/contracts/StakedUSDeV2.sol
    +++ b/contracts/StakedUSDeV2.sol
    @@ -98,7 +98,7 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {
         uint256 shares = previewWithdraw(assets);

         cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    -    cooldowns[owner].underlyingAmount += assets;
    +    cooldowns[owner].underlyingAmount = cooldowns[owner].underlyingAmount + assets;

         _withdraw(_msgSender(), address(silo), owner, assets, shares);

    @@ -114,7 +114,7 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {
         uint256 assets = previewRedeem(shares);

         cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    -    cooldowns[owner].underlyingAmount += assets;
    +    cooldowns[owner].underlyingAmount = cooldowns[owner].underlyingAmount + assets;

         _withdraw(_msgSender(), address(silo), owner, assets, shares);

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.add();
            c1.add1();
        }
    }

    contract Contract0 {

        uint8 num1 = 1;

        function add() public{
            num1 += 1;
        }

    }

    contract Contract1 {

        uint8 num1 = 1;

        function add1() public{
            num1 = num1 + 1;
        }

    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 67017                                     | 268             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add                                       | 5405            | 5405 | 5405   | 5405 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 70623                                     | 286             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add1                                      | 5363            | 5363 | 5363   | 5363 | 1       |

## [G-04] Instead of counting down in *for* statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 4 instances of this issue in 1 file:

```
File: EthenaMinting.sol

126: for (uint256 i = 0; i < _assets.length; i++) {

130: for (uint256 j = 0; j < _custodians.length; j++) {

363: for (uint256 i = 0; i < route.addresses.length; ++i) {

424: for (uint256 i = 0; i < addresses.length; ++i) {
```
    diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
    index 32da3a5..4afba06 100644
    --- a/contracts/EthenaMinting.sol
    +++ b/contracts/EthenaMinting.sol
    @@ -123,11 +123,11 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu

         _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);

    -    for (uint256 i = 0; i < _assets.length; i++) {
    +    for (uint256 i = _assets.length - 1; i >= 0; i--) {
           addSupportedAsset(_assets[i]);
         }

    -    for (uint256 j = 0; j < _custodians.length; j++) {
    +    for (uint256 j = _custodians.length - 1; j >= 0; j--) {
           addCustodianAddress(_custodians[j]);
         }

    @@ -360,7 +360,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
         if (route.addresses.length == 0) {
           return false;
         }
    -    for (uint256 i = 0; i < route.addresses.length; ++i) {
    +    for (uint256 i = route.addresses.length - 1; i >= 0; --i) {
           if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
           {
             return false;
    @@ -421,7 +421,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
         if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
         IERC20 token = IERC20(asset);
         uint256 totalTransferred = 0;
    -    for (uint256 i = 0; i < addresses.length; ++i) {
    +    for (uint256 i = addresses.length - 1; i >= 0; --i) {
           uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
           token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
           totalTransferred += amountToTransfer;

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }


    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }


    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-05] Revert Transaction as soon as possible

Always try reverting transactions as early as possible when using require statements . In case a transaction revert occurs, the user will pay the gas up until the revert was executed

There is 1 instance of this issue in 1 file:

```
File: EthenaMinting.sol

339: function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
340:   bytes32 taker_order_hash = hashOrder(order);
341:   address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
342:   if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
343:   if (order.beneficiary == address(0)) revert InvalidAmount();
344:   if (order.collateral_amount == 0) revert InvalidAmount();
345:   if (order.usde_amount == 0) revert InvalidAmount();
```

    diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
    index 32da3a5..e114105 100644
    --- a/contracts/EthenaMinting.sol
    +++ b/contracts/EthenaMinting.sol
    @@ -337,12 +337,12 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu

       /// @notice assert validity of signed order
       function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    -    bytes32 taker_order_hash = hashOrder(order);
    -    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    -    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
         if (order.beneficiary == address(0)) revert InvalidAmount();
         if (order.collateral_amount == 0) revert InvalidAmount();
         if (order.usde_amount == 0) revert InvalidAmount();
    +    bytes32 taker_order_hash = hashOrder(order);
    +    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    +    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
         if (block.timestamp > order.expiry) revert SignatureExpired();
         return (true, taker_order_hash);
       }

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol