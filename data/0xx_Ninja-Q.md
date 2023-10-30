## **Wrong importing of the Openzepplin extension contract library**

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L11

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L6


Correct Import is
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";
```
## **The unimplemented interface that would lead to the code not working properly**

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L96

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L98

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L112

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L114

The `previewWithdraw`, `maxWithdraw`, `previewRedeem` and `maxRedeem`  are not called to the stakedUSDeV2 contract.

### Solution
```solidity
import "./interfaces/IERC4626Minimal.sol";
```
and implementing it into the contract

```solidity
contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe, IERC4626Minimal {
    //..code here
}

##** Conflicting uint control**
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L116

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L100

**Solution
```
cooldowns[owner].cooldownEnd = uint104(block.timestamp + cooldownDuration);

```

