The Constructor of StakedUSDe contract validate the arguments for zero address check. but it does not check `address(assets)` must be contract address.

```
constructor(IERC20 _asset, address _initialRewarder, address _owner)
    ERC20("Staked USDe", "stUSDe")
    ERC4626(_asset)
    ERC20Permit("stUSDe")
  {
     constructor(IERC20 _asset, address _initialRewarder, address _owner)
    ERC20("Staked USDe", "stUSDe")
    ERC4626(_asset)
    ERC20Permit("stUSDe")
  {
   @> if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {
      revert InvalidZeroAddress();
    }
     ...
```
we recoment add yul code for address contract check like:
```
  uint32 size;
  assembly {
    size := extcodesize(_asset)
  }
if(size<=0) revert InvalidAsset();

```