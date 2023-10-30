## \[G‑01\] Save gas by preventing zero amount in mint()

```diff
function mint(address to, uint256 amount) external {
        if (msg.sender != minter) revert OnlyMinter();
+       require(amount != 0, "invalid amount);
       _mint(to, amount);
  }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L30

## \[G-02\] Use assembly to validate `msg.sender`

We can use assembly to efficiently validate `msg.sender` for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```
if (newAdmin == msg.sender) revert InvalidAdminChange();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L26

```
if (msg.sender != minter) revert OnlyMinter();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L29

```
if (msg.sender != _admin) {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L138

## \[Gas-03\] Compute known value `off-chain`

If You know what data to hash, there is no need to consume more computational power to hash it using keccak256 , you’ll end up consuming 2x amount of gas.

```
bytes32 private constant EIP712_REVISION = keccak256("1");
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L58

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L26C2-L32C99

## \[G‑04\] Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

[Reference.](https://code4rena.com/reports/2023-08-pooltogether#wardens:~:text=%5BG%E2%80%9102%5D%20Counting%20down%20in%20for%20statements%20is%20more%20gas%20efficient)

```
for (uint256 i = 0; i < _assets.length; i++) {
      addSupportedAsset(_assets[i]);
    }

    for (uint256 j = 0; j < _custodians.length; j++) {
      addCustodianAddress(_custodians[j]);
    }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126C4-L132C6

## \[G-05\] Amounts should be checked for 0 before calling a transfer

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```
function withdraw(address to, uint256 amount) external onlyStakingVault {
    USDE.transfer(to, amount);
  }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L29

```
IERC20(asset).safeTransfer(beneficiary, amount);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L408

## \[G-06\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

```
return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L452

```
silo = new USDeSilo(address(this), address(_asset));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L43

```
if (msg.sender != STAKING_VAULT) revert OnlyStakingVault();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L24