# Add salt to `domainSeperator`

## Impact
The salt parameter in itself is not mandatory nor does it have any impact in this contract as the `domainSeparator` has all the fields that make it unique.
However, according to the EIP712 documentation, including the salt field adds another layer of protection to the contract making the `domainSeparator` even more unique.