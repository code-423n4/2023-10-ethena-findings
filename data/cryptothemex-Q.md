## Impact
`_transferCollateral` in `EthenaMinting` contract does not check that arrays `addresses` and `ratios` are of same size before executing a loop on their values. A small difference in lengths of arrays `addresses` and `ratios` will run the loop and consume much more gas before reverting.

## Proof of Concept
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L413

## Tools Used
Manual Review

## Recommended Mitigation Steps
Check length of both arrays is equal or not before executing loop.