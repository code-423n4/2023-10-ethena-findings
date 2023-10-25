1. These revert messages are used but not defined:
https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/USDe.sol#L19

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/USDe.sol#L29

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/USDe.sol#L34

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/EthenaMinting.sol#L98

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/EthenaMinting.sol#L105

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/EthenaMinting.sol#L119-L121

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/EthenaMinting.sol#L169

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/EthenaMinting.sol#L171-L172

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/EthenaMinting.sol#L201

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/EthenaMinting.sol#L203

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/EthenaMinting.sol#L248

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/EthenaMinting.sol#L251

https://github.com/code-423n4/2023-10-ethena/blob/2eef9889496cb9af0a5042fbc599dba9c08d4002/contracts/EthenaMinting.sol#L260

ZeroAddressException
OnlyMinter
CantRenounceOwnership
MaxMintPerBlockExceeded
InvalidUSDeAddress
NoAssetsProvided
InvalidZeroAddress
InvalidOrder
InvalidRoute
Duplicate
InvalidAddress
TransferFailed
InvalidAssetAddress

This is a bug because when these require statements revert, it will revert with a generic "revert" message instead of a custom message.

Recommendation:

Define the custom error messages:
error ZeroAddressException();
error OnlyMinter(); 
error CantRenounceOwnership();