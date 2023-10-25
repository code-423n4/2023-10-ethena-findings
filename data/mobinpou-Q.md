**[link to code](https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/USDe.sol#L23)** 
we can delete this function and change the constructor like this: 
```
constructor(address admin, address ethenaMintingAddress) ERC20("USDe", "USDe") ERC20Permit("USDe") {
    if (admin == address(0) || ethenaMintingAddress == address(0)) revert ZeroAddressException();
    _transferOwnership(admin);
    emit MinterUpdated(ethenaMintingAddress, minter);
    minter = ethenaMintingAddress;
  }```

because you said The minter address is the only address that has the ability to mint USDe. This minter address has one of the most powerful non-owner permissions, the ability to create an unlimited amount of USDe. It will always be pointed to the EthenaMinting.sol contract.
so we when deploy the contract can give the address of EthenaMinting contract and it don't need to can change and you say allways the EthenaMinting address so if can change it maybe change to another address and this may issue for contract if address of the EthenaMinting changed we need again deploy this contract with new address