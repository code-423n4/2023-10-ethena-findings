# `_domainSeparator` on the spot every time the function is called may result in higher gas costs

## Summary
***The getDomainSeparator function is a view function, meaning it doesn't modify the blockchain state, but it still consumes gas for computation. Calculating _domainSeparator on the spot every time the function is called may result in higher gas costs compared to returning a pre-computed value.***

#### Recommendation
>
 ```
bytes32 private _cachedDomainSeparator;
uint256 private _cachedChainId;

function getDomainSeparator() public view returns (bytes32) {
    if (block.chainid == _cachedChainId) {
        return _cachedDomainSeparator;
    }

    // If chainid has changed, calculate the domain separator
    bytes32 computedDomainSeparator = _computeDomainSeparator();
    
    // Cache the computed domain separator and the current chainid
    _cachedDomainSeparator = computedDomainSeparator;
    _cachedChainId = block.chainid;

    return computedDomainSeparator;
}
```