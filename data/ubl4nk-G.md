## After a hard-fork the `domainSeparator` need to be calculated everytime and it leads to waste more gas than before
`EthenaMinting` is saving the `chainid` in constuctor and computes the _domainSeparator also in constructor:
```solidity
constructor() {
  // ...
 _chainId = block.chainid;
 _domainSeparator = _computeDomainSeparator();
  // ...
}
```

And function `getDomainSeparator` checks the blockchain's chainid is changed or not:
```solidity
function getDomainSeparator() public view returns (bytes32) {
    if (block.chainid == _chainId) {
      return _domainSeparator;
    }
    return _computeDomainSeparator();
  }
```
If it is not changed, it returns previously calculated _domainSeparator, if it is changed it everytime goes to _computeDomainSeparator and re-calculates the domainSeparator. 
**In this way all the functions that are using `getDomainSeprator` will waste more gas than before.**

Recommendation: If a hard-fork is happended, simply update these two state variables (_chainId and _domainSeparator) as you did in constructor.

