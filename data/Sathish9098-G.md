# GAS OPTIMIZATIONS

##

## [G-1] Use a constant variable for MAX_COOLDOWN_DURATION(Save 1 SLOT: 2.1k Gas)

Since the MAX_COOLDOWN_DURATION value is never changed within the contract, using a constant instead of a state variable is more gas-efficient and saves approximately 20,000 gas.

This is because constants are stored in memory, while state variables are stored in storage. Reading from and writing to storage is more expensive in terms of gas than reading from and writing to memory.

```diff
FILE: 2023-10-ethena/contracts/StakedUSDeV2.sol

- 22:  uint24 public MAX_COOLDOWN_DURATION = 90 days;
+ 22:  uint24 public constant MAX_COOLDOWN_DURATION = 90 days;

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L22

##

## [G-2] State variable can be packed within fewer slots (Save 1 SLOT: 2.1k Gas)

It is generally safe to change uint256 to uint128 for variables that represent the maximum amount of something that can be minted or redeemed in a block on the Ethereum blockchain. The maximum amount of something that can be minted or redeemed in a block is unlikely to exceed 2^128 (340 undecillion).

Here are some specific examples of how this change has been implemented in existing protocols:

- Aave uses uint128 for its maxSupply variables.
- Compound uses uint128 for its maxBorrowRatePerBlock and maxSupply variables.
- Uniswap V3 uses uint128 for its maxTick and minTick variables.

```diff
FILE: 2023-10-ethena/contracts/EthenaMinting.sol

 /// @notice max minted USDe allowed per block
- 89:  uint256 public maxMintPerBlock;
+ 89:  uint128 public maxMintPerBlock;
90:  /// @notice max redeemed USDe allowed per block
- 91:  uint256 public maxRedeemPerBlock;
+ 91:  uint128 public maxRedeemPerBlock;

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L89-L91

##

## [G-3] ``getUnvestedAmount()`` function should be cached 

it is a good practice to cache the ``getUnvestedAmount()`` call in this case. This is because the ``getUnvestedAmount()`` function may be expensive to compute, and caching the result can save gas.

```diff
FILE:Breadcrumbs2023-10-ethena/contracts/StakedUSDe.sol

 function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
+   uint256 getUnvestedAmount_ = getUnvestedAmount() ;
-    if (getUnvestedAmount() > 0) revert StillVesting();
+    if (getUnvestedAmount_ > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();
+    uint256 newVestingAmount = amount + getUnvestedAmount_;

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L90-L91

##

## [G-4] Refactor ``verifyOrder()`` function for more gas efficiency 

Saves ``2100 GAS ``
To improve gas efficiency, the taker_order_hash and signer can be cached after the function parameter checks. This avoids unnecessary external calls to ECDSA.recover() and the function call to hashOrder(). However, if any of the following checks revert:

- order.beneficiary == address(0)
- order.collateral_amount == 0
- order.usde_amount == 0
- block.timestamp > order.expiry

Then the caching of taker_order_hash and signer is a waste of gas. Therefore, the code should be refactored to avoid caching in these cases. This saves gas 

```diff
FILE: Breadcrumbs2023-10-ethena/contracts/EthenaMinting.sol

 /// @notice assert validity of signed order
  function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
-    bytes32 taker_order_hash = hashOrder(order);
-    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
-    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    if (order.beneficiary == address(0)) revert InvalidAmount();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
    if (block.timestamp > order.expiry) revert SignatureExpired();
+    bytes32 taker_order_hash = hashOrder(order);
+    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
+    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    return (true, taker_order_hash);
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L339-L348

##

## [G-5] Use sort solidity operations

Check parameters first then check external calls and state variables in If and Require checks when using || and && Operations

```diff
FILE: 2023-10-ethena/contracts/EthenaMinting.sol

- 364: if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
+ 364: if ( route.addresses[i] == address(0) || route.ratios[i] == 0 || !_custodianAddresses.contains(route.addresses[i]))

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L364

##

## [G-6] IF’s/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### Cheaper to check the function parameter before state variables [Saves 100 GAS]

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L342-L346


```diff
FILE: 2023-10-ethena/contracts/EthenaMinting.sol

function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
-    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    if (order.beneficiary == address(0)) revert InvalidAmount();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
    if (block.timestamp > order.expiry) revert SignatureExpired();
+    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    return (true, taker_order_hash);
  }

```

##

## [G-7] Don't declare the variables Inside the for loop. Declare outside and use inside the loop 

```diff
FILE: 2023-10-ethena/contracts/EthenaMinting.sol

IERC20 token = IERC20(asset);
    uint256 totalTransferred = 0;
+      uint256 amountToTransfer;
    for (uint256 i = 0; i < addresses.length; ++i) {
-      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
+      amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L422-L428




 






