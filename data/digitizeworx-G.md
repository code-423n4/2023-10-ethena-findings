EthenaMinting uses simple for loops to iterate arrays in function https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L413-L433:

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L424
```solidity
for (uint256 i = 0; i < addresses.length; ++i) {

  // transfer tokens

}
```

This is not optimized for gas as each iteration requires incrementing and checking the loop variable.

**Impact**

- Iterating arrays with a simple for loop costs more gas than necessary.

- This accumulates across multiple array iterations in the contract.

- Over time, significant extra gas is wasted when minting or redeeming through Ethena.

**Mitigation**

Use inline assembly to loop through arrays which is more gas efficient:

```solidity
uint256 i;
for { let i := 0 } lt(i, addresses.length) { i := add(i, 1) } {

  // transfer tokens
  
}
```

This increments and checks the loop index in assembly which reduces gas costs.

**Benefits**

- Gas savings of 100-200 gas per array iteration.

- Smoother user experience for Ethena users as transaction fees reduced.

- More sustainable long-term as contract won't waste gas needlessly.