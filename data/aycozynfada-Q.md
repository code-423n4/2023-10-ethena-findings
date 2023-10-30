# Event not emitted after sensitive actions in EthenaMinting.sol changes.


setMaxMintPerBlock()
setMaxRedeemPerBlock()
removeMinterRole()
disableMintRedeem()
and 
removeRedeemerRole() are all sensitive function calls in EthenaMinting.sol without a cocurrent event emitted

github link - https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L219-L232
              https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L276-L285
Mitigation
Events should the added to the above function calls

