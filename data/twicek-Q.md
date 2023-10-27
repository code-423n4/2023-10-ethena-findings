# Manual findings

## [L-01] `verifyOrder` return value is not checked in `EthenaMinting.sol.mint` and `EthenaMinting.sol.redeem`

`verifyOrder` returns `true`:

[EthenaMinting.sol#L339-L348](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L339-L348)
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

But this return value is not checked:

[EthenaMinting.sol#L169-L171](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L169-L171)
```solidity
    if (order.order_type != OrderType.MINT) revert InvalidOrder();
    verifyOrder(order, signature);
    if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
```

## Recommendations
Consider adding a defensive check in `EthenaMinting.sol.mint` and `EthenaMinting.sol.redeem` to check the return value of `verifyOrder`.


## [L-02] Users with the `FULL_RESTRICTED_STAKER_ROLE` role can deposit

The README states:
> `FULL_RESTRICTED_STAKER_ROLE` - address with this role can't burn his stUSDe tokens to unstake his USDe tokens, neither to transfer stUSDe tokens. His balance can be manipulated by the admin of StakedUSDe

A user who gets blacklisted with the `FULL_RESTRICTED_STAKER_ROLE` role without knowing it risks depositing collateral in the contract that he will not be able to recover until his restrictions are removed.

[StakedUSDe.sol#L203-L215](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L203-L215)
```solidity
  function _deposit(address caller, address receiver, uint256 assets, uint256 shares)
    internal
    override
    nonReentrant
    notZero(assets)
    notZero(shares)
  {
    if (hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, receiver)) {
      revert OperationNotAllowed();
    }
    super._deposit(caller, receiver, assets, shares);
    _checkMinShares();
  }
```

## Recommendations
Consider preventing users with the `FULL_RESTRICTED_STAKER_ROLE` role from depositing.

## [L-03] `transfer` return value is not checked when transferring USDe from the `USDeSilo` contract

[USDeSilo.sol#L28-L30](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L28-L30)
```solidity
  function withdraw(address to, uint256 amount) external onlyStakingVault {
    USDE.transfer(to, amount);
  }
```
## Recommendations
Consider adding a defensive check to verify the return value of `transfer`.

# Automated findings (not in the bot-report)

## [L-04] External call recipient may consume all transaction gas
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use `addr.call{gas: <amount>}("")` or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

```solidity
File: contracts/EthenaMinting.sol
404:       (bool success,) = (beneficiary).call{value: amount}("");
```
*GitHub*: [404](https://github.com/code-423n4/2023-09-maia/blob/main/contracts/EthenaMinting.sol#L404) 

## [L-05] Consider implementing two-step procedure for updating protocol addresses
A copy-paste error or a typo may end up bricking protocol functionality, or sending tokens to an address with no known private key. Consider implementing a two-step procedure for updating protocol addresses, where the recipient is set as pending, and must 'accept' the assignment by making an affirmative call. A straightforward way of doing this would be to have the target contracts implement [EIP-165](https://eips.ethereum.org/EIPS/eip-165), and to have the 'set' functions ensure that the recipient is of the right interface type.

```solidity
File: contracts/USDe.sol
23:   function setMinter(address newMinter) external onlyOwner {
```
*GitHub*: [23](https://github.com/code-423n4/2023-09-maia/blob/main/contracts/USDe.sol#L23) 

## [L-06] `receive()`/`payable fallback()` function does not authorize requests
Having no access control on the function (e.g. `require(msg.sender == address(weth))`) means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue mistakenly-sent Ether.

```solidity
File: contracts/EthenaMinting.sol
153:   receive() external payable {
```
*GitHub*: [153](https://github.com/code-423n4/2023-09-maia/blob/main/contracts/EthenaMinting.sol#L153)