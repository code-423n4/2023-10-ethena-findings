## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Avoid contract existence checks by using low level calls | 6 | - |
| [G-02] | keccak256() should only need to be called on a specific string literal once | 1 | - |
| [G-03] | Optimize Names to save Gas | 3 | - |
| [G-04] | USE A MORE RECENT VERSION OF SOLIDITY | 5 | - |
| [G-05] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= ) | 4 | - |
| [G-06] | Using fixed bytes is cheaper than using string | 1 | - |
| [G-07] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 4 | - |
| [G-08] | Can make the variable outside the loop to save gas | 1 | - |
| [G-09] | Should use arguments instead of state variable | 2 | - |
| [G-10] | Before transfer of  some functions, we should check some variables for possible gas save | 2 | - |
| [G-11] | Using storage instead of memory for structs/arrays saves gas | 2 | - |
| [G-12] | A modifier used only once and not being inherited should be inlined to save gas | 2 | - |
| [G-13] | Sort Solidity operations using short-circuit mode | 1 | - |
| [G-14] | Use hardcode address instead address(this) | 3 | - |
| [G-15] | Use assembly for math (add, sub, mul, div) | 3 | - |
| [G-16] | Non efficient zero initialization | 2 | - |
| [G-17] | State variable read in a loop | 1 | - |
| [G-18] | bytes.concat() can be used in place of abi.encodePacked | 1 | - |
| [G-19] | Use assembly to validate msg.sender | 2 | - |
| [G-20] | No need to evaluate all expressions to know if one of them is true | 2 | - |
| [G-21] | Pre-increment and pre-decrement are cheaper than +1 ,-1 | 2 | - |




## Gas Optimizations  

## [G-01] Avoid contract existence checks by using low level calls		

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.	

```solidity
file: /contracts/EthenaMinting.sol

///@audit ' !_custodianAddresses.contains(wallet) ' is low level call

248    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

253      IERC20(asset).safeTransfer(wallet, amount);

260    if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();

317    return ECDSA.toTypedDataHash(getDomainSeparator(), keccak256(encodeOrder(order)));

341    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);

407      if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L248


## [G-02] keccak256() should only need to be called on a specific string literal once

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once

```solidity
file: /contracts/EthenaMinting.sol

40   bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");

     /// @notice role enabling to invoke redeem
43   bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L40-L43


## [G-03] Optimize Names to save Gas

public/external function names and public member variable names can be optimized to save gas. See this link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted.

```solidity
file: /contracts/USDe.sol

15   contract USDe is Ownable2Step, ERC20Burnable, ERC20Permit, IUSDeDefinitions {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L15


```solidity
file: /contracts/EthenaMinting.sol

21   contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGuard {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L21


```solidity
file: /contracts/StakedUSDe.sol

21  contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, ERC4626, IStakedUSDe {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L21


## [G-04] USE A MORE RECENT VERSION OF SOLIDITY

To dayes the solidit version is 0.8.22

```solidity
file: /contracts/USDe.sol

2  pragma solidity 0.8.19;

///@audit and also other have , to should be change. 

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L2


## [G-05] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )

AVOID COMPOUND ASSIGNMENT OPERATOR IN STATE VARIABLES.
Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

```solidity
file: /contracts/EthenaMinting.sol

174    mintedPerBlock[block.number] += order.usde_amount;

205    redeemedPerBlock[block.number] += order.usde_amount;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L174


```solidity
file: /contracts/StakedUSDeV2.sol

101    cooldowns[owner].underlyingAmount += assets;

117    cooldowns[owner].underlyingAmount += assets;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L101


## [G-06] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
file: /contracts/EthenaMinting.sol

29    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L29


## [G-07] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use theright tool for the task at hand. There is a difference between constant variables and immutable variables, and they shouldeach be used in their appropriate contexts. constants should be used for literal values written into the code, and immutablevariables should be used for expressions, or values calculated in, or passed into the constructor. 


```solidity
file: /contracts/EthenaMinting.sol

28  bytes32 private constant EIP712_DOMAIN =
    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

  /// @notice route type
  bytes32 private constant ROUTE_TYPE = keccak256("Route(address[] addresses,uint256[] ratios)");

  /// @notice order type
  bytes32 private constant ORDER_TYPE = keccak256(
    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)"
37  );

49  bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L28-L37


## [G-08] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /contracts/EthenaMinting.sol

425      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L425


## [G-09] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   

```solidity
file: /contracts/USDe.sol

24    emit MinterUpdated(newMinter, minter);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L24


```solidity
file: /main/contracts/StakedUSDeV2.sol

133    emit CooldownDurationUpdated(previousDuration, cooldownDuration);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L133


## [G-10] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:


```solidity
file: /contracts/USDeSilo.sol

29    USDE.transfer(to, amount);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L29


```solidity
file: /contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96





## [G-11] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
file: /contracts/EthenaMinting.sol

113    address[] memory _assets,
114    address[] memory _custodians,

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L113-L114


## [G-12]  A modifier used only once and not being inherited should be inlined to save gas
 
```solidity
file: /contracts/USDeSilo.sol

23  modifier onlyStakingVault() {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L23


```solidity
file: /contracts/SingleAdminAccessControl.sol

17  modifier notAdmin(bytes32 role) {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L17


## [G-13] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows
 f(x) || g(y) 
 f(x) && g(y)
```

Instances:

```solidity
file: /contracts/StakedUSDe.sol

246    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L246


## [G-14] Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

 if the contract's address is needed in the code, it's more gas-efficient to hardcode the address as a constant rather than using the address(this) expression. This is because using address(this) requires additional gas consumption to compute the contract's address at runtime.

an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```
Instances:

```solidity
file: /contracts/EthenaMinting.sol

452    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L452


```solidity
file: /contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

167    return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96


## [G-15] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```solidity
file: /contracts/EthenaMinting.sol

425      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;

429    uint256 remainingBalance = amount - totalTransferred;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L429


```solidity
file: /contracts/StakedUSDe.sol

174    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L174


## [G-16] Non efficient zero initialization
 
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

```solidity
file: /contracts/EthenaMinting.sol

126    for (uint256 i = 0; i < _assets.length; i++) {

130    for (uint256 j = 0; j < _custodians.length; j++) {    

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126

## [G-17] State variable read in a loop

State variable reads and writes are more expensive than local variable reads and writes. Therefore, it is recommended to replace state variable reads and writes within loops with local variable reads and writes.

```solidity
file: /contracts/EthenaMinting.sol

364      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L364

## [G-18] bytes.concat() can be used in place of abi.encodePacked

Given concatenation is not going to be used for hashing bytes.concat is the preferred method to use as its more gas efficient

```solidity
file: /contracts/EthenaMinting.sol

49  bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L49

## [G-19] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file: /contracts/USDe.sol

29    if (msg.sender != minter) revert OnlyMinter();

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L29


```solidity
file: /contracts/SingleAdminAccessControl.sol

26    if (newAdmin == msg.sender) revert InvalidAdminChange();

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L26

## [G-20] No need to evaluate all expressions to know if one of them is true

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved

```solidity
file: /contracts/EthenaMinting.sol

291    if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {

342    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();    

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L291


```solidity
file: /contracts/StakedUSDe.sol

210    if (hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, receiver)) {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L210

## [G-21] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
file: /contracts/EthenaMinting.sol

126    for (uint256 i = 0; i < _assets.length; i++) {

363    for (uint256 i = 0; i < route.addresses.length; ++i) {    

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126

