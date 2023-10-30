#L1

##Severity: Low

##Add length checks in _transferCollateral function

###File: https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol

###Original code:

```solidity
function _transferCollateral(
    uint256 amount,
    address asset,
    address benefactor,
    address[] calldata addresses,
    uint256[] calldata ratios
  ) internal {
    // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
    IERC20 token = IERC20(asset);
    uint256 totalTransferred = 0;
```

###Optimized code:

```solidity
internal {
    if (addresses.length != ratios.length) revert InvalidInput("Length mismatch");
    // ... (rest of the code remains the same)
  }
```
In the _transferCollateral function, there should be a check to ensure that the lengths of the addresses and ratios arrays are equal.


#L2

##Severity: Low

##rescueTokens function can be more optimized

###File: https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L138


###Original code:
```solidity
function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (address(token) == asset()) revert InvalidToken();
    IERC20(token).safeTransfer(to, amount);
  }
```
###Optimized code:
```solidity
function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
  // ... (previous code)
  uint256 contractBalance = IERC20(token).balanceOf(address(this));
  if (amount > contractBalance) revert InsufficientBalance();
  // continues
}
```
In the rescueTokens function, ensure that the contract has enough balance of the token before rescuing.


#L3

##Severity: Low


###File: https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L19

###Original code:
```solidity
constructor(address stakingVault, address usde) {
    STAKING_VAULT = stakingVault;
    USDE = IERC20(usde);
  }
```

###Optimized code:
```solidity
constructor(address stakingVault, address usde) {
        require(stakingVault != address(0), "InvalidStakingVaultAddress");
        require(usde != address(0), "InvalidUSDeAddress");
        STAKING_VAULT = stakingVault;
        USDE = IERC20(usde);
    }
```    
Add checks to ensure the staking vault and USDe addresses provided in the constructor are not zero addresses.
