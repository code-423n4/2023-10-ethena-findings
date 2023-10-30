### [Low_01] Code Contradicting with Functions Natspec Comment 
```solidity
  /// @notice transfers an asset to a custody wallet
  function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {
    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress(); 
    if (asset == NATIVE_TOKEN) {
      (bool success,) = wallet.call{value: amount}("");
      if (!success) revert TransferFailed();
    } else {
      IERC20(asset).safeTransfer(wallet, amount);
    }
    emit CustodyTransfer(wallet, asset, amount);
```
As here we can see comment saying that asset transfer will occur to a `custody wallet`
There is a `EnumerableSet` named `_custodianAddresses` which keep tracks of `custody wallets`
If we see first `if` condition check 
   - inputed wallet address should not 0addr
   - OR
   - Wallet address is a Custody address

So caller can simply pass a address which is not 0addr and not included in  `_custodianAddresses` and if check succesfully failed and no reversal occur

Here we clearly see inputed wallet address not a custody address, but `code comment` says it should be a custody address

### Impact
Caller (MINTER ROLE) can transfer asset to any address other than `Custody Wallets`

### Code
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L248
```

### Mitigation
I believe there should be `&&` operator in that `If` condition check
OR
`!_custodianAddresses.contains(wallet)` check in `If` enough to ensure that inputed `wallet` is a custody wallet or not

### [Low_02] I think there should be a Order's `usde_amount` check on onChain
As we know user sign a order and send Order and signature off-chain, verification occur off-chain

Then MINTER_ROLE call `mint()` with parameters `order, router, signature`
```solidity
function mint(Order calldata order, Route calldata route, Signature calldata signature)
```
Then verification of order occurs
```solidity
  function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    if (order.beneficiary == address(0)) revert InvalidAmount();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount(); 
    if (block.timestamp > order.expiry) revert SignatureExpired();
    return (true, taker_order_hash);
  }
```
But problem is that onChain only checks occur that `order.usde_amount` and `order.collateral_amount` is not 0
If `MINTER_ROLE` gone malicious he can modify `order's usde_amount and collataral_amount` parameter and caller go for a significant amount of loss.

User will end in receiving less USDe amount and may result in giving more collateral

### Code
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L339-L348
```
### Mitigation
There should be onChain check which ensure `order.usde_amount` and `order.collateral_amount` passed by `MINTER_ROLE` actually same as value intended by Caller.



### [Low_03] During `unstaking()` method unstaked `usde` could send to a zeroAddress
```solidity
  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
      userCooldown.cooldownEnd = 0; // @audit delete
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets); 
    } else {
      revert InvalidCooldown();
    }
  }
```
Here we can see unstaked `USDe` will send to `receiver` from `silo` contract
But there no checks of `zero address` for `receiver`

May be mistakely token send to zero address 
### Code
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L78-L90
```
### Mitigation
There should be a zero address check for `receiver` in `unstake()`

### [Low_04] There should be `minAssetRequired` parameter in `cooldownShares()` while redeem shares into assets and starts a cooldown to claim

```solidity
  function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
    if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();

    uint256 assets = previewRedeem(shares); 

    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    cooldowns[owner].underlyingAmount += assets;

    _withdraw(_msgSender(), address(silo), owner, assets, shares);

    return assets;
  }
```
Here we can see `cooldownShares()` here is to redeem shares into assets and starts a cooldown to claim the converted underlying asset.

It takes `uint256 shares` parameter which then convert it into assets
```solidity
uint256 assets = previewRedeem(shares);
```
Then required accounting updates here

While converting shares -> assets may be goes under a malicious front-runn, as a result `cooldowns[owner].underlyingAmount` will incremented with a amount of assets that will less/significantly less than user intention.
### Code
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L111-L122
```
### Mitigation
There should be a other parameter in function which allow user to input atleast amount of assets he intended to receive while converting shares -> assets
If that amounts not matched then function should revert.

### [Low_05] importing `openzeppelin/contracts/access/Ownable2Step.sol` and using `_transferOwnership()`
`_transferOwnership` in `Ownable2Step.sol` is coded as follows
```solidity
    function _transferOwnership(address newOwner) internal virtual override {
        delete _pendingOwner;
        super._transferOwnership(newOwner);
    }
```
So basically its a single step Owner transfer.
Now a days its recommended to use 2-step process

So instead of using ` _transferOwnership()` try to use `transferOwnership()` from same contract which is as follows
```solidity
    function transferOwnership(address newOwner) public virtual override onlyOwner {
        _pendingOwner = newOwner;
        emit OwnershipTransferStarted(owner(), newOwner);
    }
```
```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol#L44-L47
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol#L35-L38
```
