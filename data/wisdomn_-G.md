Cache addresses[addresses.length - 1] outside the token transfer in function [_transferCollateral](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L431) to save gas. Going through the array of addresses during a transfer will waste gas. Note that this finding is different from " Cache array length outside of loop" which was submitted by [4naly3er](https://github.com/code-423n4/2023-10-ethena/blob/main/4naly3er-report.md#GAS-2)

```
    if (remainingBalance > 0) {
      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
    }
```
