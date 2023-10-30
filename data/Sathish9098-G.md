# GAS OPTIMIZATIONS

##

## [G-] Use a constant variable for MAX_COOLDOWN_DURATION(Save 1 SLOT: 2.1k Gas)

Since the MAX_COOLDOWN_DURATION value is never changed within the contract, using a constant instead of a state variable is more gas-efficient and saves approximately 20,000 gas.

This is because constants are stored in memory, while state variables are stored in storage. Reading from and writing to storage is more expensive in terms of gas than reading from and writing to memory.

```diff
FILE: 2023-10-ethena/contracts/StakedUSDeV2.sol

- 22:  uint24 public MAX_COOLDOWN_DURATION = 90 days;
+ 22:  uint24 public constant MAX_COOLDOWN_DURATION = 90 days;

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L22

##

## [G-] ``getUnvestedAmount()`` function should be cached 

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

## [G-] Refactor ``verifyOrder()`` function for more gas efficiency 

taker_order_hash and signer can be cached after function parameter checks to avoid unnecessary ``ECDSA.recover(taker_order_hash, signature.signature_bytes)`` external call  and ``hashOrder(order)`` function calls . if any reverts 

##

## [G-3] IF’s/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### Cheaper to check the function parameter before state variables [Saves 100 GAS]

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L342-L346


```diff
FILE: Breadcrumbs2023-10-ethena/contracts/EthenaMinting.sol

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


 


IS there any computation repeats ?

timeToExpiry only used if putOptionsRequired == true, it should be moved into if block

