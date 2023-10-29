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
## [G-02] ABI Encode costs more gas than ABI Encode Packed
This contract utilises abi.encode() within the method called encodeOrder. 
Within abi.encode(), the main types are padded to 32 bytes with dynamic arrays including their length, but abi.encodePacked() just utilises the minimum needed memory to encode this data.
**Location**
```txt
/contracts/EthenaMinting.sol#L321-L321
/contracts/EthenaMinting.sol#L335-L335
/contracts/EthenaMinting.sol#L452-L452
```
**Remediation**
If not required, then utilise abi.encodePacked() rather than abi.encode().