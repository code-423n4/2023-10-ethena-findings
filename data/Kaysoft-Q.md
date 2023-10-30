## [NC-1] Avoid use of deparacated draft openzeppelin libraries

File: https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L11

The `stakeUSDe.sol` contract imports `draft-ERC20Permit.sol` which is deparated as indicated in the imported file.
```
import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol"
```
As you can see in the draft-ERC20Permit.sol file, there is a comment which reads "This file is deprecated. 
```
draft-ERC20Permit.sol
// SPDX-License-Identifier: MIT
// OpenZeppelin Contracts (last updated v4.9.0) (token/ERC20/extensions/draft-ERC20Permit.sol)

pragma solidity ^0.8.0;

// EIP-2612 is Final as of 2022-11-01. This file is deprecated.

import "./ERC20Permit.sol";
```
Consider importing `ERC20Permit.sol` instead of the draft `draft-ERC20Permit.sol` from openzeppelin.

## [NC-2] `InvalidCooldown()` `error` should be changed to `CoolDownNotCompleted()`

The name `CoolDownNotCompleted()` describes the error better.
file: https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L88

```
function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }
```
## [NC-3] The stablecoin USDe.sol should have blacklist and pausable mechanism like USDC too.

File: https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L15
- The blacklists helps to blacklist sanctioned addresseses associated with crime and funding terrorism.
- The Pausable mechanism allows to protocol to pause the system if there is a case of depeg or similar risk.

## [NC-4] `cooldownDuration` is set to  `MAX_COOLDOWN_DURATION` (90days) in the constructor.

File: https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L42C3-L46C1
```
constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {
    silo = new USDeSilo(address(this), address(_asset));
    cooldownDuration = MAX_COOLDOWN_DURATION;
  }
```
From the documentation on the contest page, the cool down period will be initially set to 14 days but it is set to 90days above in the constructor.
Here is a quote from the texts on the contest page: 
"There's also an additional change to add a 14 day cooldown period on unstaking stUSDe."

