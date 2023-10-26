# No Zero address check for the custodian which holds the stETH token

### Impact
***The issue I mentioned regarding the custodian contract not checking for an address of 0 (zero) in its constructor could potentially affect the behavior of the system. In Ethereum, an address of 0 represents an invalid or non-existent address, often used as a placeholder or to indicate the absence of a valid address.***

##### Recommendation
>Check if the custodian is `zero address`.