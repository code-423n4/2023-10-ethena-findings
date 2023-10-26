
**EthenaMinting.sol**
- L49 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.

- L135/136/97/104 - In the constructor, when defining the variables maxMintPerBlock and maxRedeemPerBlock they could be zero, therefore, the modifiers belowMaxMintPerBlock() and belowMaxRedeemPerBlock() will always revert and generate a DoS in mint and redeem, until The DEFAULT_ADMIN_ROLE changes the values, therefore it would be advisable in the constructor to add validations so that the initial values ​​are != 0x.




