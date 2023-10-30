## 1. No address zero check on on the `_to` address in the mint function
 In the `mint` function in contract `USDe.sol` there is no address(0) check to ensure that `USDe` is not minted to the zero address, which is basically burning the intended tokens to mint.
 ```solidity
 function mint(address to, uint256 amount) external {
    if (msg.sender != minter) revert OnlyMinter();
    _mint(to, amount);
  }
```
## 2. No checks to ensure that role is not granted to the zero address
 in the `_grantRole` function in `SingleAdminAccessControl.sol` is used by the `DEFAULT_ADMIN_ROLE` grant roles to `accounts`but does not ensure that the account to grant role is not the zero address before granting rolethis can lead to DoS and loss of gas.
```solidity
  function _grantRole(bytes32 role, address account) internal override {
    if (role == DEFAULT_ADMIN_ROLE) {
      emit AdminTransferred(_currentDefaultAdmin, account);
      _revokeRole(DEFAULT_ADMIN_ROLE, _currentDefaultAdmin);
      _currentDefaultAdmin = account;
      delete _pendingDefaultAdmin;
    }
    super._grantRole(role, account);
  }
```
## 3. Direct `minting` using the `mint` funntion by `MINTER` can break the `maximun_mint_per_block`
 As the there are no checks in the mint function that ensures that the `max_mint_per_block` is not exceeded. so if there is a direct mint using the `mint` function in `USDe.sol` it is very possible that the `max_mint_per_block` will be exceeded. Adding the `belowMaxRedeemPerBlock` mordifier to the mint function will eliminate this possible loophole. This will also act as a form of extra security if the `MINTER` role is breached as it limits the `amount` of `USDe` the malicious minter possibly mint in a short time. This also applies  to the `redeem` function in `USDe.sol`
```solidity
function mint(address to, uint256 amount) external {
    if (msg.sender != minter) revert OnlyMinter();
    _mint(to, amount);
  }
```
  See link [here](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L28)
## 4. function `previewRedeem` not specified in the `codebase`.
 The function `previewRedeem` is called in the `cooldownShares/cooldownAssets` functions, which presumably should be used to calculate the conversion of `shares` to `assets` to redeem, but this function is not implemented in the codebase `in scope` and `out of scope`, which can lead to unintended behaviour in the contract.
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









