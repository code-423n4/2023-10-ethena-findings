## [G-01] Set the constructor as payable
One can save circa 10 opcodes and a bit of gas if the constructors are set to payable.
But, there are risks due to payable constructors ability to take ETH when deploying.
**Locations**
```txt
/contracts/USDe.sol#L18-L21
/contracts/USDeSilo.sol#L18-L21
/contracts/EthenaMinting.sol#L111-L146
/contracts/StakedUSDeV2.sol#L42-L45
/contracts/StakedUSDe.sol#L70-L81
```
**Remediation**
Set the constructors to payable which will save gas. 
Check it will not lead to bad effects within the instance of an upgrade pattern.