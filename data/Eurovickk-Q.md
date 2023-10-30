https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol

1)Line 199: if (address(_usde) == address(0)) revert InvalidUSDeAddress(); is better to use require inestead of revert.
2)Line 339: The verifyOrder function does not check the order.nonce has a unique value.
mapping(address => uint256) private usedNonces;
function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {    
    if (usedNonces[order.benefactor] >= order.nonce) {
        revert NonceAlreadyUsed();
    }    
    usedNonces[order.benefactor] = order.nonce;    
}

3)The verifyNonce function can't manage a data overflow from invalidatorBit.

