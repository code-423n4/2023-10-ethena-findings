## Reverting with incorrect error in EthenaMinting.sol


On line 345 of `EthenaMinting.sol` when checking if the `order.beneficiary` is the zero address, if the check fails the function is reverting with `InvalidAmount()` although it appears that it should be `InvalidZeroAddress()`.