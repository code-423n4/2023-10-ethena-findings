| [L-01] | In ethenaMinting.sol EIP712 name/EIP712_REVISION not following eip712 standard for future implemation on 
           any other chain.
           Recommendation is to  make the EIP712 name/EIP712_REVISION immutable and give it value on constructor.

| [Q-01] | In ethenaMinting.sol::verifyRoute The description says "routes only used to mint" but cheking if 
          (orderType == OrderType.REDEEM) EthenaMinting.sol#L351-L374
           Recommendation is to check if (orderType == OrderType.MINT) instead cheking if (orderType == 
           OrderType.REDEEM) and returning true or just returning false.