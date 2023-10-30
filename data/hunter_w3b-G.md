# Gas Optimization For Ethena Labs Protocol

While striving for enhanced code clarity in the provided snippets, certain functions have been abbreviated to emphasize affected sections.

Developers should remain vigilant during the incorporation of these proposed modifications to avert potential vulnerabilities. Despite prior testing of the optimizations, developers bear the responsibility of conducting comprehensive reevaluation.

Conducting code reviews and additional testing is highly recommended to mitigate any plausible hazards that may arise from the refactoring endeavor.

These optimizations aim to reduce the gas costs associated with contract deployment and execution. Below is a summary of the key points and issues discussed:

# Summary

| Finding | Issues                                                                                                                         | Instances |
| :-----: | :----------------------------------------------------------------------------------------------------------------------------- | :-------: |
|    1    | Use assembly to validate msg.sender                                                                                            |     4     |
|    2    | Using a positive conditional flow to save a NOT opcode                                                                         |    11     |
|    3    | Access mappings directly rather than using accessor functions                                                                  |     2     |
|    4    | Use assembly for small keccak256 hashes, in order to save gas                                                                  |     2     |
|    5    | Using this to access functions result in an external call, wasting gas                                                         |     2     |
|    6    | Expensive operation inside a for loop                                                                                          |     2     |
|    7    | Cache external calls outside of loop to avoid re-calling function on each iteration                                            |     2     |
|    8    | Avoid having ERC20 token balances go to zero, always keep a small amount                                                       |     4     |
|    9    | Timestamps and block numbers in storage do not need to be uint256                                                              |     5     |
|   10    | Count from n to zero instead of counting from zero to n                                                                        |     2     |
|   11    | Use selfdestruct in the constructor if the contract is one-time use                                                            |     5     |
|   12    | Use transfer hooks for tokens instead of initiating a transfer from the destination smart contract                             |     3     |
|   13    | Use fallback or receive instead of deposit() when transferring Ether                                                           |     1     |
|   14    | Avoid contract calls by making the architecture monolithic                                                                     |     4     |
|   15    | Use one ERC1155 or ERC6909 token instead of several ERC20 tokens                                                               |     4     |
|   16    | Consider using alternatives to OpenZeppelin                                                                                    |     -     |
|   17    | Use vanity addresses (safely!)                                                                                                 |     1     |
|   18    | Using assembly to revert with an error message                                                                                 |    52     |
|   19    | Use assembly to reuse memory space when making more than one external call                                                     |     6     |
|   20    | It is sometimes cheaper to cache calldata                                                                                      |     2     |
|   21    | Avoid contract existence checks by using low level calls                                                                       |     8     |
|   22    | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant                       |    12     |
|   23    | `<X> += <Y>` costs more gas than `<X> = <X> + <Y>` for state variables                                                         |     4     |
|   24    | `++i/i++` should be unchecked{++i}/unchecked{i++} when it's not possible for them to overflow, as is the case when used in for |     4     |
|   25    | Should use arguments instead of state variable in emit to save some amount of gas                                              |     6     |
|   26    | Use bitmaps instead of bools when a significant amount of booleans are used                                                    |     3     |
|   27    | The result of function calls should be cached rather than re-calling the function                                              |     2     |
|   28    | A modifier used only once and not being inherited should be inlined to save gas                                                |     3     |
|   29    | abi.encode() is less efficient than abi.encodePacked()                                                                         |     3     |
|   30    | Using > 0 costs more gas than != 0                                                                                             |     3     |
|   31    | Use assembly to reuse memory space when making more than one external call                                                     |     9     |
|   32    | Short-circuit booleans                                                                                                         |    11     |
|   33    | Unnecessary casting as variable is already of the same type                                                                    |     5     |
|   34    | Use hardcode address instead address(this)                                                                                     |     5     |
|   35    | Use assembly for math (add, sub, mul, div)                                                                                     |    11     |
|   36    | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4                                          |     1     |
|   37    | `onlyRole(MINTER_ROLE)` no uses of the nonReentrant modifier                                                                   |     3     |
|   38    | State variables which are not modified within functions should be set as constants or immutable for values set at deployment   |     2     |

### These optimization techniques and recommendations can help developers reduce gas costs and improve the efficiency of the Ethena Labs Protocol contracts.It's essential for developers to exercise caution and conduct thorough testing and code reviews when implementing these optimizations to ensure the security and correctness of their contracts.

## [G-01] Use assembly to validate msg.sender

You can use inline assembly to validate msg.sender and potentially save gas. Here's an example of how to do it:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ValidateSender {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function validateSender() public view returns (bool) {
        bool isValid;

        assembly {
            // Retrieve the current sender address
            let sender := caller()

            // Load the owner's address from storage
            let ownerAddress := sload(owner.slot)

            // Compare the sender with the owner's address
            isValid := eq(sender, ownerAddress)
        }

        return isValid;
    }
}
```

The validateSender function uses inline assembly to compare msg.sender (retrieved with caller()) with the owner's address stored in storage (retrieved with sload). If they match, isValid is set to true, indicating that the sender is the owner. Using inline assembly for this simple comparison can potentially save gas compared to a standard Solidity statement due to the reduced gas overhead of inline assembly.

```solidity
File: contracts/USDe.sol

29    if (msg.sender != minter) revert OnlyMinter();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L29

```solidity
File: contracts/EthenaMinting.sol

138    if (msg.sender != _admin) {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L138

```solidity
File: contracts/USDeSilo.sol

24    if (msg.sender != STAKING_VAULT) revert OnlyStakingVault();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L24

```solidity
File: contracts/SingleAdminAccessControl.sol

32    if (msg.sender != _pendingDefaultAdmin) revert NotPendingAdmin();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L32

## [G-02] Using a positive conditional flow to save a NOT opcode

In the code snippet below, the second function avoids an unnecessary negation. In theory, the extra ! increases the computational cost. But as we noted at the top of the article, you should benchmark both methods because the compiler is can sometimes optimize this.

```solidity
function cond() public {
    if (!condition) {
        action1();
    }
    else {
        action2();
    }
}

function cond() public {
    if (condition) {
        action2();
    }
    else {
        action1();
    }
}
```

```solidity
File: contracts/EthenaMinting.sol

171    if (!verifyRoute(route, order.order_type)) revert InvalidRoute();

172    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();

203    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();

251      if (!success) revert TransferFailed();

260    if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();

271    if (!_custodianAddresses.remove(custodian)) revert InvalidCustodianAddress();

342    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();

364      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)

405      if (!success) revert TransferFailed();

407      if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();

421    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L171

## [G-03] Access mappings directly rather than using accessor functions

When you have a mapping, accessing its values through accessor functions involves an additional layer of indirection, which can incur some gas cost. This is because accessing a value from a mapping typically involves two steps: first, locating the key in the mapping, and second, retrieving the corresponding value.

```solidity
File:

98    if (mintedPerBlock[block.number] + mintAmount > maxMintPerBlock) revert MaxMintPerBlockExceeded();

105    if (redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L98

## [G-04] Use assembly for small keccak256 hashes, in order to save gas

If the arguments to the encode call can fit into the scratch space (two words or fewer), then it's more efficient to use assembly to generate the hash.

Using assembly to hash up to 96 bytes of data

```soliditycontract ExpensiveHasher {
    bytes32 public hash;
    struct Values {
        uint256 a;
        uint256 b;
        uint256 c;
    }
    Values values;

    // cost: 113155function setOnchainHash(Values calldata _values) external {
        hash = keccak256(abi.encode(_values));
        values = _values;
    }
}


contract CheapHasher {
    bytes32 public hash;
    struct Values {
        uint256 a;
        uint256 b;
        uint256 c;
    }
    Values values;

    // cost: 112107
    function setOnchainHash(Values calldata _values) external {
        assembly {
            // cache the free memory pointer because we are about to override it
            let fmp := mload(0x40)

            // use 0x00 to 0x60
            calldatacopy(0x00, 0x04, 0x60)
            sstore(hash.slot, keccak256(0x00, 0x60))

            // restore the cache value of free memory pointer
            mstore(0x40, fmp)
        }

        values = _values;
    }
}

```

In the above example, similar to the first one, we use assembly to store values in the first 96 bytes of memory which saves us 1,000+ gas. Also notice that in this instance, because we still break back into solidity code, we cached and updated our free memory pointer at the start and end of our assembly block. This is to make sure that the solidity compiler’s assumptions on what is stored in memory remains compatible.

```solidity
File: contracts/EthenaMinting.sol

317    return ECDSA.toTypedDataHash(getDomainSeparator(), keccak256(encodeOrder(order)));


452    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L317

## [G-05] Using this to access functions result in an external call, wasting gas

External calls have an overhead of 100 gas, which can be avoided by not referencing the function using this. Contracts are allowed to override their parents' functions and change the visibility from external to public, so make this change if it's required in order to call the function internally.

```solidity
File: contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

167    return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96

## [G-06] Expensive operation inside a for loop

Performing an expensive operation inside a for loop in Solidity can be a gas-inefficient practice, as it can significantly increase the transaction cost.

```solidity
File: contracts/EthenaMinting.sol

363    for (uint256 i = 0; i < route.addresses.length; ++i) {
364      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
365      {
366        return false;
367      }
368      totalRatio += route.ratios[i];
369    }




424    for (uint256 i = 0; i < addresses.length; ++i) {
425      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
426      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
427      totalTransferred += amountToTransfer;
428    }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L363-L369

## [G-07] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALL that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

Cache \_custodianAddresses.contains(route.addresses[i]) outside of loop to save 1 STATICCALL per loop iteration

```solidity
File:

363    for (uint256 i = 0; i < route.addresses.length; ++i) {
364      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
365      {
366        return false;
367      }
368      totalRatio += route.ratios[i];
369    }



424    for (uint256 i = 0; i < addresses.length; ++i) {
425      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
426      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
427      totalTransferred += amountToTransfer;
428    }
```

## [G-08] Avoid having ERC20 token balances go to zero, always keep a small amount

This is related to the avoiding zero writes section above, but it’s worth calling out separately because the implementation is a bit subtle.

If an address is frequently emptying (and reloading) it’s account balance, this will lead to a lot of zero to one writes.

```solidity
File: contracts/EthenaMinting.sol

253      IERC20(asset).safeTransfer(wallet, amount);

408      IERC20(asset).safeTransfer(beneficiary, amount);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L253

```solidity
File: contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

140    IERC20(token).safeTransfer(to, amount);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96

## [G-09] Timestamps and block numbers in storage do not need to be uint256

A timestamp of size uint48 will work for millions of years into the future. A block number increments once every 12 seconds. This should give you a sense of the size of numbers that are sensible.

```solidity
File:  contracts/StakedUSDe.sol

45  uint256 public lastDistributionTimestamp;

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L45

```solidity
File: contracts/StakedUSDeV2.sol

100    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;

116    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L100

```solidity
File: contracts/EthenaMinting.sol

81  mapping(uint256 => uint256) public mintedPerBlock;

83  mapping(uint256 => uint256) public redeemedPerBlock;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L81

## [G-10] Count from n to zero instead of counting from zero to n

When setting a storage variable to zero, a refund is given, so the net gas spent on counting will be less if the final state of the storage variable is zero.

```solidity
File: contracts/StakedUSDeV2.sol

83      userCooldown.cooldownEnd = 0;

84      userCooldown.underlyingAmount = 0;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L83

## [G-11] Use selfdestruct in the constructor if the contract is one-time use

Sometimes, contracts are used to deploy several contracts in one transaction, which necessitates doing it in the constructor.

If the contract’s only use is the code in the constructor, then selfdestructing at the end of the operation will save gas.

Although selfdestruct is set for removal in an upcoming hardfork, it will still be supported in the constructor per EIP 6780

```solidity
File: contracts/USDe.sol

18  constructor(address admin) ERC20("USDe", "USDe") ERC20Permit("USDe") {

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L18

```solidity
File: contracts/EthenaMinting.sol

111  constructor(

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L111

```solidity
File: contracts/StakedUSDe.sol

70  constructor(IERC20 _asset, address _initialRewarder, address _owner)
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L70

```solidity
File: contracts/StakedUSDeV2.sol

42  constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L42

```solidity
File: contracts/USDeSilo.sol

18  constructor(address stakingVault, address usde) {

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L18

## [G-12] Use transfer hooks for tokens instead of initiating a transfer from the destination smart contract

Let’s say you have contract A which accepts token B (an NFT or an ERC1363 token). The naive workflow is as follows:

msg.sender approves contract A to accept token B

msg.sender calls contract A to transfer tokens from msg.sender to A

Contract A then calls token B to do the transfer

Token B does the transfer, and calls onTokenReceived() in contract A

Contract A returns a value from onTokenReceived() to token B

Token B returns execution to contract A

This is very inefficient. It’s better for msg.sender to call contract B to do a transfer which calls the tokenReceived hook in contract A.

Note that:

All ERC1155 tokens include a transfer hook

safeTransfer and safeMint in ERC721 have a transfer hook

ERC1363 has transferAndCall

ERC777 has a transfer hook but has been deprecated. Use ERC1363 or ERC1155 instead if you need fungible tokens

If you need to pass arguments to contract A, simply use the data field and parse that in contract A.

```solidity
File: contracts/EthenaMinting.sol

426      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);

431      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L426

```solidity
File: contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96

## [G-13] Use fallback or receive instead of deposit() when transferring Ether

Similar to above, you can “just transfer” ether to a contract and have it respond to the transfer instead of using a payable function. This of course, depends on the rest of the contract’s architecture.

Example Deposit in AAVE

```solidity
contract AddLiquidity{

    receive() external payable {
      IWETH(weth).deposit{msg.value}();
      AAVE.deposit(weth, msg.value, msg.sender, REFERRAL_CODE)
    }
}

```

The fallback function is capable of receiving bytes data which can be parsed with abi.decode. This servers as an alternative to supplying arguments to a deposit function.

```solidity
File: contracts/StakedUSDe.sol

203  function _deposit(address caller, address receiver, uint256 assets, uint256 shares)

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L203

## [G-14] Avoid contract calls by making the architecture monolithic

Contract calls are expensive, and the best way to save gas on them is by not using them at all. There is a natural tradeoff with this, but having several contracts that talk to each other can sometimes increase gas and complexity rather than manage it.

```solidity
File: contracts/EthenaMinting.sol

250      (bool success,) = wallet.call{value: amount}("");

404      (bool success,) = (beneficiary).call{value: amount}("");

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L250

```solidity
File: contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

140    IERC20(token).safeTransfer(to, amount);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96

## [G-15] Use one ERC1155 or ERC6909 token instead of several ERC20 tokens

This was the original intent of the ERC1155 token. Each individual token behaves like and ERC20, but only one contract needs to be deployed.

The drawback of this approach is that the tokens will not be compatible with most DeFi swapping primitives.

ERC1155 uses callbacks on all of the transfer methods. If this is not desired, ERC6909 can be used instead.

```solidity
File: contracts/USDe.sol

4   import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L4

```solidity
File: contracts/EthenaMinting.sol

10  import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L10

```solidity
File: contracts/StakedUSDe.sol

9   import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L9

```solidity
File: contracts/USDeSilo.sol

4   import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L4

## [G-16] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are [Solmate](https://github.com/transmissions11/solmate) and [Solady](https://github.com/Vectorized/solady).

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.

## [G-17] Use vanity addresses (safely!)

It is cheaper to use vanity addresses with leading zeros, this saves calldata gas cost.

A good example is OpenSea Seaport contract with this address: 0x00000000000000ADc04C56Bf30aC9d3c0aAF14dC.

This will not save gas when calling the address directly. However, if that contract’s address is used as an argument to a function, that function call will cost less gas due to having more zeros in the calldata.

This is also true of passing EOAs with a lot of zeros as a function argument – it saves gas for the same reason.

Just be aware that there have been hacks from generating vanity addresses for wallets with insufficiently random private keys. This is not a concert for smart contracts vanity addresses created with finding a salt for create2, because smart contracts do not have private keys.

```solidity
File: contracts/EthenaMinting.sol

52  address private constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L52

## [G-18] Using assembly to revert with an error message

When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Here’s an example;

```solidity
// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

// calling restrictedAction(2) with a non-owner address: 23734

contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

```solidity
File: contracts/USDe.sol

19    if (admin == address(0)) revert ZeroAddressException();

29    if (msg.sender != minter) revert OnlyMinter();

34    revert CantRenounceOwnership();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L19

```solidity
File: contracts/EthenaMinting.sol

98    if (mintedPerBlock[block.number] + mintAmount > maxMintPerBlock) revert MaxMintPerBlockExceeded();

105    if (redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();

119    if (address(_usde) == address(0)) revert InvalidUSDeAddress();

120    if (_assets.length == 0) revert NoAssetsProvided();

121    if (_admin == address(0)) revert InvalidZeroAddress();

169    if (order.order_type != OrderType.MINT) revert InvalidOrder();

171    if (!verifyRoute(route, order.order_type)) revert InvalidRoute();

172    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();

201    if (order.order_type != OrderType.REDEEM) revert InvalidOrder();

203    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();

248    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

251      if (!success) revert TransferFailed();

260    if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();

271    if (!_custodianAddresses.remove(custodian)) revert InvalidCustodianAddress();

292      revert InvalidAssetAddress();

300      revert InvalidCustodianAddress();

342    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();

343    if (order.beneficiary == address(0)) revert InvalidAmount();

344    if (order.collateral_amount == 0) revert InvalidAmount();

345    if (order.usde_amount == 0) revert InvalidAmount();

346    if (block.timestamp > order.expiry) revert SignatureExpired();

378    if (nonce == 0) revert InvalidNonce();

383    if (invalidator & invalidatorBit != 0) revert InvalidNonce();

403      if (address(this).balance < amount) revert InvalidAmount();

405      if (!success) revert TransferFailed();

407      if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();

421    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L98

```solidity
File: contracts/StakedUSDe.sol

51    if (amount == 0) revert InvalidAmount();

57    if (target == owner()) revert CantBlacklistOwner();

76      revert InvalidZeroAddress();

90    if (getUnvestedAmount() > 0) revert StillVesting();

139    if (address(token) == asset()) revert InvalidToken();

157      revert OperationNotAllowed();

193    if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();


211      revert OperationNotAllowed();

233      revert OperationNotAllowed();

247      revert OperationNotAllowed();

250      revert OperationNotAllowed();

258    revert OperationNotAllowed();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L51

```solidity
File: contracts/StakedUSDeV2.sol

28    if (cooldownDuration != 0) revert OperationNotAllowed();

34    if (cooldownDuration == 0) revert OperationNotAllowed();

88      revert InvalidCooldown();

96    if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();

112    if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();

128      revert InvalidCooldown();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L28

```solidity
File: contracts/USDeSilo.sol

24    if (msg.sender != STAKING_VAULT) revert OnlyStakingVault();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L24

```solidity
File: contracts/SingleAdminAccessControl.sol

18    if (role == DEFAULT_ADMIN_ROLE) revert InvalidAdminChange();

26    if (newAdmin == msg.sender) revert InvalidAdminChange();

32    if (msg.sender != _pendingDefaultAdmin) revert NotPendingAdmin();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L18

## [G-19] Use assembly to reuse memory space when making more than one external call

An operation that causes the solidity compiler to expand memory is making external calls. When making external calls the compiler has to encode the function signature of the function it wishes to call on the external contract alongside it’s arguments in memory. As we know, solidity does not clear or reuse memory memory so it’ll have to store these data in the next free memory pointer which expands memory further.

With inline assembly, we can either use the scratch space and free memory pointer offset to store this data (as above) if the function arguments do not take up more than 96 bytes in memory. Better still, if we are making more than one external call we can reuse the same memory space as the first calls to store the new arguments in memory without expanding memory unnecessarily. Solidity in this scenario would expand memory by as much as the returned data length is. This is because the returned data is stored in memory (in most cases). If the return data is less than 96 bytes, we can use the scratch space to store it to prevent expanding memory.

See the example below;

```solidity
contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}


contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}


contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

```solidity
File: contracts/EthenaMinting.sol

248    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

253      IERC20(asset).safeTransfer(wallet, amount);


407      if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();
408      IERC20(asset).safeTransfer(beneficiary, amount);


426      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
431      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L248

## [G-20] It is sometimes cheaper to cache calldata

Although the calldataload instruction is a cheap opcode, the solidity compiler will sometimes output cheaper code if you cache calldataload. This will not always be the case, so you should test both possibilities.

```solidity
contract LoopSum {
    function sumArr(uint256[] calldata arr) public pure returns (uint256 sum) {
        uint256 len = arr.length;
        for (uint256 i = 0; i < len; ) {
            sum += arr[i];
            unchecked {
                ++i;
            }
        }
    }
}
```

```solidity
File: contracts/EthenaMinting.sol

413  function _transferCollateral(
414    uint256 amount,
415    address asset,
416    address benefactor,
417    address[] calldata addresses,
418    uint256[] calldata ratios
419  ) internal {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L413-L419

## [G-21] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File: contracts/EthenaMinting.sol

178    usde.mint(order.beneficiary, order.usde_amount);

206    usde.burnFrom(order.benefactor, order.usde_amount);

253      IERC20(asset).safeTransfer(wallet, amount);

408      IERC20(asset).safeTransfer(beneficiary, amount);

426      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);

431      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L178

```solidity
File: contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

140    IERC20(token).safeTransfer(to, amount);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96

## [G-22] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

The immutable keyword was introduced to optimize the storage of constant values, especially those that can be computed at compile-time, like the result of keccak256(). When you use immutable, the value is computed at compile-time and included directly in the contract's bytecode. It becomes a part of the contract's storage and is available without any runtime computation. This approach reduces gas when accessing the constant value because there's no need to compute it at runtime. So by using immutable, you save gas because you eliminate the need for runtime computation of constant values. The value is readily available in the contract's bytecode.

```solidity
File: contracts/EthenaMinting.sol

28  bytes32 private constant EIP712_DOMAIN =
29    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

32  bytes32 private constant ROUTE_TYPE = keccak256("Route(address[] addresses,uint256[] ratios)");


35  bytes32 private constant ORDER_TYPE = keccak256(
36    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)"
37  );


40  bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");

43  bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");

46  bytes32 private constant GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");

49  bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));

55  bytes32 private constant EIP_712_NAME = keccak256("EthenaMinting");

58  bytes32 private constant EIP712_REVISION = keccak256("1");
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L28

```solidity
File: contracts/StakedUSDe.sol

26  bytes32 private constant REWARDER_ROLE = keccak256("REWARDER_ROLE");

30  bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");

32  bytes32 private constant SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L26

## [G-23] `<X> += <Y>` costs more gas than `<X> = <X> + <Y>` for state variables

`<x> = <x> + <y>` is often more gas-efficient than `<x> += <y>` for state variables. The += operator can incur higher gas costs because it involves both a read and a write operation on the state variable. In contrast, using `<x> = <x> + <y>` explicitly reads and updates the state variable in a more optimized manner, reducing gas consumption.

```solidity
File: contracts/EthenaMinting.sol

174    mintedPerBlock[block.number] += order.usde_amount;

205    redeemedPerBlock[block.number] += order.usde_amount;

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L174

```solidity
File: contracts/StakedUSDeV2.sol

101    cooldowns[owner].underlyingAmount += assets;

117    cooldowns[owner].underlyingAmount += assets;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L101

## [G-24] `++i/i++` should be unchecked{++i}/unchecked{i++} when it's not possible for them to overflow, as is the case when used in for and while-loops

When you're sure that an operation won't lead to overflow or underflow, you can use unchecked arithmetic to potentially save gas and increase efficiency. For example, in cases where ++i and i++ won't result in overflow or underflow, you can use unchecked{++i} and unchecked{i++} to indicate that these operations are safe.

```solidity
File: contracts/EthenaMinting.sol

126    for (uint256 i = 0; i < _assets.length; i++) {

130    for (uint256 j = 0; j < _custodians.length; j++) {

363    for (uint256 i = 0; i < route.addresses.length; ++i) {

424    for (uint256 i = 0; i < addresses.length; ++i) {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126

## [G-25] Should use arguments instead of state variable in emit to save some amount of gas

```solidity
File: contracts/EthenaMinting.sol

439    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);

446    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L439

```solidity
File: contracts/SingleAdminAccessControl.sol

28    emit AdminTransferRequested(_currentDefaultAdmin, newAdmin);

74      emit AdminTransferred(_currentDefaultAdmin, account);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L28

```solidity
File: contracts/StakedUSDeV2.sol

133    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L133

```solidity
File: contracts/USDe.sol

24    emit MinterUpdated(newMinter, minter);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L24

## [G-26] Use bitmaps instead of bools when a significant amount of booleans are used

A common pattern, especially in airdrops, is to mark an address as “already used” when claiming the airdrop or NFT mint.

However, since it only takes one bit to store this information, and each slot is 256 bits, that means one can store a 256 flags/booleans with one storage slot.

```solidity
File: contracts/EthenaMinting.sol

86  mapping(address => mapping(address => bool)) public delegatedSigner;

236    delegatedSigner[_delegateTo][msg.sender] = true;

242    delegatedSigner[_removedSigner][msg.sender] = false;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L86

## [G-27] The result of function calls should be cached rather than re-calling the function

Caching the result of function calls is a common gas optimization technique in Solidity. When a function's result doesn't change between multiple uses within the same transaction or block, caching can save gas by preventing redundant computations.

```solidity
File: contracts/StakedUSDe.sol

149    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {

210    if (hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, receiver)) {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L149

## [G-28] A modifier used only once and not being inherited should be inlined to save gas

Inlining a modifier that is used only once and not inherited can help save gas by reducing the overhead of function call and stack management associated with applying a modifier. Inlining essentially means directly including the code of the modifier within the function where it's used. This is a manual optimization that can be applied in cases where the modifier is simple and only used in one place.

```solidity
File: contracts/EthenaMinting.sol

// @audit  This modifier is only used in mint() function it's gas saving to use inline
97  modifier belowMaxMintPerBlock(uint256 mintAmount) {
98    if (mintedPerBlock[block.number] + mintAmount > maxMintPerBlock) revert MaxMintPerBlockExceeded();
99    _;
100  }



// @audit  This modifier is only used in redeem() function it's gas saving to use inline
104  modifier belowMaxRedeemPerBlock(uint256 redeemAmount) {
105    if (redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();
106    _;
107  }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L97-L107

```solidity
File: contracts/USDeSilo.sol

// @audit This modifier is only used in this withdraw() function it's gas saving to use inline
23  modifier onlyStakingVault() {
24    if (msg.sender != STAKING_VAULT) revert OnlyStakingVault();
25    _;
26  }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L23-L26

## [G-29] abi.encode() is less efficient than abi.encodePacked()

abi.encode() is more gas-efficient when encoding function calls according to the Ethereum ABI. It includes the necessary function selectors and parameter encoding.abi.encodePacked() is more efficient for simply concatenating raw data without ABI-related encoding, making it a preferred choice when ABI structure isn't required.Your choice between the two functions should be based on whether you need ABI compliance for contract interactions or a more lightweight data concatenation.

```solidity
File: contracts/EthenaMinting.sol

321    return abi.encode(

335    return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);

425    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L321

## [G-30] Using > 0 costs more gas than != 0

Using != 0 is more gas-efficient than > 0 when checking if a value is non-zero because the != comparison directly checks for inequality, resulting in lower gas costs.

```solidity
File: contracts/EthenaMinting.sol

430    if (remainingBalance > 0) {

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L430

```solidity
File: contracts/StakedUSDe.sol

90    if (getUnvestedAmount() > 0) revert StillVesting();

193    if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L90

## [G-31] Use assembly to reuse memory space when making more than one external call

An operation that causes the solidity compiler to expand memory is making external calls. When making external calls the compiler has to encode the function signature of the function it wishes to call on the external contract alongside it’s arguments in memory. As we know, solidity does not clear or reuse memory memory so it’ll have to store these data in the next free memory pointer which expands memory further.

With inline assembly, we can either use the scratch space and free memory pointer offset to store this data (as above) if the function arguments do not take up more than 96 bytes in memory. Better still, if we are making more than one external call we can reuse the same memory space as the first calls to store the new arguments in memory without expanding memory unnecessarily. Solidity in this scenario would expand memory by as much as the returned data length is. This is because the returned data is stored in memory (in most cases). If the return data is less than 96 bytes, we can use the scratch space to store it to prevent expanding memory.

See the example below;

```solidity
contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}


contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}


contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

```solidity
File:  contracts/EthenaMinting.sol

248    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

250      (bool success,) = wallet.call{value: amount}("");

253      IERC20(asset).safeTransfer(wallet, amount);




404      (bool success,) = (beneficiary).call{value: amount}("");

407      if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();

408      IERC20(asset).safeTransfer(beneficiary, amount);




421    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();

426      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);

431      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);



```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L248

## [G-32] Short-circuit booleans

In Solidity, when you evaluate a boolean expression (e.g the || (logical or) or && (logical and) operators), in the case of || the second expression will only be evaluated if the first expression evaluates to false and in the case of && the second expression will only be evaluated if the first expression evaluates to true. This is called short-circuiting.

For example, the expression require(msg.sender == owner || msg.sender == manager) will pass if the first expression msg.sender == owner evaluates to true. The second expression msg.sender == manager will not be evaluated at all.

However, if the first expression msg.sender == owner evaluates to false, the second expression msg.sender == manager will be evaluated to determine whether the overall expression is true or false. Here, by checking the condition that is most likely to pass firstly, we can avoid checking the second condition thereby saving gas in majority of successful calls.

This is similar for the expression require(msg.sender == owner && msg.sender == manager). If the first expression msg.sender == owner evaluates to false, the second expression msg.sender == manager will not be evaluated because the overall expression cannot be true. For the overall statement to be true, both side of the expression must evaluate to true. Here, by checking the condition that is most likely to fail firstly, we can avoid checking the second condition thereby saving gas in majority of call reverts.

Short-circuiting is useful and it’s recommended to place the less expensive expression first, as the more costly one might be bypassed. If the second expression is more important than the first, it might be worth reversing their order so that the cheaper one gets evaluated first.

```solidity
File:  contracts/EthenaMinting.sol

248    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

291    if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {

299    if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {

364      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)

421    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L248

```solidity
File: contracts/StakedUSDe.sol

75    if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {

149    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {

193    if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();

246    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {

210    if (hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, receiver)) {

232    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, caller) || hasRole(FULL_RESTRICTED_STAKER_ROLE, receiver)) {

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L75

## [G-33] Unnecessary casting as variable is already of the same type

### StakedUSDe.sol.rescueTokens(): token should not be cast to address as it’s declared as an address

```solidity
File: contracts/StakedUSDe.sol

139    if (address(token) == asset()) revert InvalidToken();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L139C1-L140C1

StakedUSDeV2.sol.cooldownAssets(): silo should not be cast to address as it’s declared as an address

```solidity
File: contracts/StakedUSDeV2.sol

103    _withdraw(_msgSender(), address(silo), owner, assets, shares);

119    _withdraw(_msgSender(), address(silo), owner, assets, shares);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L103

```solidity
File: contracts/EthenaMinting.sol

// @audit usde  unnecessary casting the usde
291    if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {

299    if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L291C1

## [G-34] Use hardcode address instead address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

```solidity
File: contracts/EthenaMinting.sol

403      if (address(this).balance < amount) revert InvalidAmount();

452    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L403

```solidity
File: contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

167    return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L167

```solidity
File: contracts/StakedUSDeV2.sol

43    silo = new USDeSilo(address(this), address(_asset));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L43

## [G-35] Use assembly for math (add, sub, mul, div)

Using assembly for math operations like addition, subtraction, multiplication, and division in Solidity can be more gas-efficient in certain cases. Here's an example of how to perform these operations using inline assembly:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MathOperations {
    function add(uint256 a, uint256 b) public pure returns (uint256 result) {
        assembly {
            result := add(a, b)
        }
    }

    function subtract(uint256 a, uint256 b) public pure returns (uint256 result) {
        assembly {
            result := sub(a, b)
        }
    }

    function multiply(uint256 a, uint256 b) public pure returns (uint256 result) {
        assembly {
            result := mul(a, b)
        }
    }

    function divide(uint256 a, uint256 b) public pure returns (uint256 result) {
        assembly {
            result := div(a, b)
        }
    }
}

```

In this example, we've defined functions for addition, subtraction, multiplication, and division using inline assembly. The add, sub, mul, and div assembly instructions perform these mathematical operations

```solidity
File: contracts/EthenaMinting.sol

98    if (mintedPerBlock[block.number] + mintAmount > maxMintPerBlock) revert MaxMintPerBlockExceeded();

105    if (redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();

425      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;

429    uint256 remainingBalance = amount - totalTransferred;

431      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L98

```solidity
File: contracts/StakedUSDe.sol

91    uint256 newVestingAmount = amount + getUnvestedAmount();

167    return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();

174    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

180    return ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) / VESTING_PERIOD;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91

```solidity
File: contracts/StakedUSDeV2.sol

100    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;

116    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L100

## [G-36] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

It's recommended to use the bytes.concat() function instead of abi.encodePacked() for concatenating byte arrays. bytes.concat() is a more gas-efficient and safer way to concatenate byte arrays, and it's considered a best practice in newer Solidity versions.

```solidity
File: contracts/EthenaMinting.sol

49  bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L49

## [G-37] `onlyRole(MINTER_ROLE)` no uses of the nonReentrant modifier

### onlyRole(MINTER_ROLE) modifier is used if this is trusted there is no need of nonReentrant modifier this save significant amount of gas

```solidity
File: contracts/EthenaMinting.sol

162  function mint(Order calldata order, Route calldata route, Signature calldata signature)
163    external
164    override
165    nonReentrant
166    onlyRole(MINTER_ROLE)
167    belowMaxMintPerBlock(order.usde_amount)
168  {



194  function redeem(Order calldata order, Signature calldata signature)
195    external
196    override
197    nonReentrant
198    onlyRole(REDEEMER_ROLE)
199    belowMaxRedeemPerBlock(order.usde_amount)
200  {


247  function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L162-L168

## [G-38] State variables which are not modified within functions should be set as constants or immutable for values set at deployment

Variables that are never updated should be immutable or constant
In Solidity, variables which are not intended to be updated should be constant or immutable.

This is because constants and immutable values are embedded directly into the bytecode of the contract which they are defined and does not use storage because of this.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract Constants {
    uint256 constant MAX_UINT256 = 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;

    function get_max_value() external pure returns (uint256) {
        return MAX_UINT256;
    }
}

// This uses more gas than the above contract
contract NoConstants {
    uint256 MAX_UINT256 = 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;

    function get_max_value() external view returns (uint256) {
        return MAX_UINT256;
    }
}
```

This saves a lot of gas as we do not make any storage reads which are costly.

```solidity
File: contracts/EthenaMinting.sol

66  EnumerableSet.AddressSet internal _supportedAssets;

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L66

```solidity
File: contracts/StakedUSDeV2.sol

22  uint24 public MAX_COOLDOWN_DURATION = 90 days;

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L22
