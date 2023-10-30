0. USDe::setMinter() - Add `payable` modifier so its cheaper for `onlyOwner` to call

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L23

1. SingleAdminAccessControl::transferAdmin() - can add `payable` to make it cheaper to call for onlyRole.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L25

