## Summary<a name="Summary">

### Gas Issues
| |Issue|number of Instances
|-|:-|:-|
| [G&#x2011;01](#G&#x2011;01) | use simple uint variable instead for mapping | 1 

## Gas Issues:

### <a href="#Summary">[G&#x2011;01]</a><a name="G&#x2011;01"> use simple uint variable instead for mapping
    
the two storage variables `mintedPerBlock` and `redeemedPerBlock` are responsible for holding how much usde was minted and redeemed per each block.
they are used to check if the minted and redeemed amount per block is not exceeding the limit. but a simple uint variable holding how much was minted and redeemed , that gets reset for each new block by holding the blockNumber in a separate variable can achieve the same. it would be much more optimal as it wont hold how much was minted and redeemed for each block.

- *EthenaMinting.sol* ( [#L81](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L81-L83)):
    
```solidity=81
mapping(uint256 => uint256) public mintedPerBlock;
  /// @notice USDe redeemed per block
  mapping(uint256 => uint256) public redeemedPerBlock;
```
