# Lows

## [L-01] Using `SafeERC20` for `IERC20` but calling directly `transfer` method of `ERC20` inherited contract

Either remove `using SafeERC20 for IERC20` in [USDeSilo, line 13](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L13) and its import in [line 5](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L5), or call `USDE.safeTransfer` in [line 29](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L29)