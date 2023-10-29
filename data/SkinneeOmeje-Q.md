In the EthenaMinting.sol.
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L343
## Title
wrong customError usuage
## Impact
This custom error name in the verifyOrder is meant to be InvalidAddress not InvalidAmount
## Tools Used
Manual review
## Recommended Mitigation Steps
change the InvalidAmount customError to InvalidAddress