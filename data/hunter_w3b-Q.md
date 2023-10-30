## [L-01] Overreliance on ERC20 interface without Pausable token functionality - no pause minting/transfers in emergency

The token fails to inherit from Pausable and lacks functionality to pause minting and transfers in an emergency situation.

The USDe contract inherits from ERC20 and ERC20Burnable interfaces to define its core token functionality. However, it does not also inherit from Pausable which would allow an authorized party to pause all minting, burning and transfers on the token in an emergency situation.

Without a Pausable implementation, there is no built-in mechanism to freeze the contract and prevent further activity if a vulnerability is discovered or the system requires downtime. This poses an important security and risk management shortcoming

A malicious actor could potentially exploit a bug or vulnerability to rapidly mint new tokens, transfer value out of the contract, or otherwise manipulate the token supply. Without pausability, there would be no built-in standardized method for contract administrators to freeze the system and contain the issue.

This risks undermining the stability and integrity of the USDe token in an emergency scenario where quick action is required. It also does not follow best practice standards for token implementations.

```solidity
File: contracts/USDe.sol

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";
import "@openzeppelin/contracts/access/Ownable2Step.sol";
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L4-L7

## [L-02] No events on critical actions like minting - lack of transparency into token supply changes

Critical actions like minting lack emitted events, reducing transparency into token supply changes.

While the USDe contract implements the ERC20 standard, it is missing crucial events for minting actions. Whenever new tokens are minted via the mint() function, no corresponding Transfer or Mint event is emitted.

Without events tracking minting, there is no programmatic or on-chain way to verify changes to the total supply over time. External observers have reduced visibility into minting activity and changes to circulating supply.

```solidity
File: contracts/USDe.sol

  function mint(address to, uint256 amount) external {
    if (msg.sender != minter) revert OnlyMinter();
    _mint(to, amount);
  }

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L28-L32

## [L-03] Delegated signatures have no expiration, risk of indefinite delegation.

The delegated signature functionality lacks an expiration, allowing indefinite delegation of signing permissions.

The EthenaMinting contract allows contracts to delegate their signing capabilities to an external account via the setDelegatedSigner() function. However, there is no specified expiration set on these delegations.

Once a delegation is made, the external account has indefinite powers to sign orders on behalf of the contract. The contract owner has no built-in ability to later revoke these permissions.

```solidity
File:  contracts/EthenaMinting.sol

  function setDelegatedSigner(address _delegateTo) external {
    delegatedSigner[_delegateTo][msg.sender] = true;
    emit DelegatedSignerAdded(_delegateTo, msg.sender);
  }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L235-L238

The lack of expirations on delegated signatures centralizes long-term signing control in the delegated account. If compromised, it could be used to automatically sign fraudulent orders over a lengthy period.

## [L-04] Route verification only checks arrays match, not actual logic.

The route verification logic only checks array lengths match but does not validate the actual route details meet requirements.

The verifyRoute() function checks that the route address and ratio arrays passed to mint() match lengths. However, it does not implement the full route verification logic.

Such logic should check things like: addresses are valid custodians, ratios sum to 100%, no address receives 0%, etc.

Without this, any route could be passed even if the ratios/addresses do not actually represent a valid custody transfer path.

Impact: This overly permissive validation allows fraudulent or invalid routes to be used for minting. It undermines the intended asset transfer guarantees that routes are expected to provide. It risks assets being incorrectly transferred or even stolen if invalid routes can still trigger minting. This reduces integrity of the custody mechanism.

```solidity
File: contracts/EthenaMinting.sol

function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
    // routes only used to mint
    if (orderType == OrderType.REDEEM) {
      return true;
    }
    uint256 totalRatio = 0;
    if (route.addresses.length != route.ratios.length) {
      return false;
    }
    if (route.addresses.length == 0) {
      return false;
    }
    for (uint256 i = 0; i < route.addresses.length; ++i) {
      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
      {
        return false;
      }
      totalRatio += route.ratios[i];
    }
    if (totalRatio != 10_000) {
      return false;
    }
    return true;
  }
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L351C1-L374C4
