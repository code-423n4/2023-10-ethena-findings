# G-1. `+=` Costs More Gas Than `=+` For State Variables

When you use the += operator on a state variable, the EVM has to perform three operations:

Load the current value of the state variable.
Add the new value to it.
Then store the result back in the state variable.
On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations:

Load the current value of the state variable.
Add the new value to it.
Better use =+ For State Variables.

Instances :

File : contracts/EthenaMinting.sol

174 :     mintedPerBlock[block.number] += order.usde_amount;

205 :     redeemedPerBlock[block.number] += order.usde_amount;

368 :       totalRatio += route.ratios[i];

427 :       totalTransferred += amountToTransfer;

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L174

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L205

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L368

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L427


File : contracts/StakedUSDeV2.sol

101 :     cooldowns[owner].underlyingAmount += assets;

117 :     cooldowns[owner].underlyingAmount += assets;

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L101

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L117C25-L117C25


# G-2. Use `assembly` to validate `msg.sender`

We can use assembly to efficiently validate msg.sender for the functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

Instances : 

File : contracts/EthenaMinting.sol

138 :     if (msg.sender != _admin) {

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L138

File : contracts/SingleAdminAccessControl.sol

26 :     if (newAdmin == msg.sender) revert InvalidAdminChange();

32 :     if (msg.sender != _pendingDefaultAdmin) revert NotPendingAdmin();

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L26

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L32

File : contracts/USDe.sol

29 :     if (msg.sender != minter) revert OnlyMinter();

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L29


File : contracts/USDeSilo.sol

24 :     if (msg.sender != STAKING_VAULT) revert OnlyStakingVault();

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L24


# G-3. Gas saving is achieved by removing the `delete` keyword

The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

Instance : 

File : contracts/SingleAdminAccessControl.sol

77 :       delete _pendingDefaultAdmin;

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L77

# G-4.  OR in `if`-condition can be rewritten to two single `if` conditions

Refactoring the if-condition in a way it won’t be containing the || operator will save more gas.

Reference : https://code4rena.com/reports/2023-08-shell#g-05-or-in-if-condition-can-be-rewritten-to-two-single-if-conditions

Instances : 

File : contracts/EthenaMinting.sol

248 :     if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

342 :     if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();

421 :     if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();


https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L248

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L342

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L421


File : contracts/StakedUSDe.sol

210 :     if (hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, receiver)) {

232 :     if (hasRole(FULL_RESTRICTED_STAKER_ROLE, caller) || hasRole(FULL_RESTRICTED_STAKER_ROLE, receiver)) {

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L210

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L232


# G-5. Using a positive conditional flow to save a NOT opcode

Estimated savings: 3 gas

Instances : 

File : contracts/EthenaMinting.sol

251 :       if (!success) revert TransferFailed();

405 :       if (!success) revert TransferFailed();

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L251

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L405


# G-6. Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

Instances : 

File : contracts/EthenaMinting.sol

425 :       uint256 amountToTransfer = (amount * ratios[i]) / 10_000;

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L425


# G-7. Use `abi.encodePacked()` not `abi.encode()`

`abi.encode` pads extra null bytes at the end of the call data which is normally unnecessary. In general, `abi.encodePacked` is more gas-efficient.

Instances : 

File : contracts/EthenaMinting.sol

321-330 :     return abi.encode(
      ORDER_TYPE,
      order.order_type,
      order.expiry,
      order.nonce,
      order.benefactor,
      order.beneficiary,
      order.collateral_asset,
      order.collateral_amount,
      order.usde_amount
    );
	
335 :     return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);

452 :     return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));


https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L321C3-L331C7

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L335

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L452
