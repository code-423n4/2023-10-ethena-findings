https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L413-L433

code from L423-L432 replaces as below, the following code can reduce one iteration of the loop, which can save gas consumption.

```
    uint256 len = addresses.length;
    uint256 totalTransferred = 0;
    for (uint256 i = 0; i < len - 1; ) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
      unchecked {++i};
    }
    uint256 remainingBalance = amount - totalTransferred;
    token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
```