https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol

1)In the mint function, the increment of mintedPerBlock and the call to _transferCollateral are executed before the verification of conditions. This could potentially lead to unnecessary gas consumption in case the verification fails.

pragma solidity ^0.8.0;

contract MintProofOfConcept {
    uint256 public mintedPerBlock;
    address public usde;
    constructor(address _usde) {
        usde = _usde;
    }
    function mint(uint256 amount) public {
        // Check if the amount is positive
        if (amount <= 0) {
            revert("Invalid amount");
        }
        // Increment mintedPerBlock
        mintedPerBlock += amount;
        // Simulate a token transfer
        // In a real contract, you would transfer the token here
        // Here, we'll just emit an event to simulate the transfer
        emit Minted(amount);
    }
    event Minted(uint256 amount);
}

2)In the redeem function, incrementing redeemedPerBlock before verifying the conditions can result in unnecessary gas consumption if the condition check fails. 
function redeem(Order calldata order, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(REDEEMER_ROLE)
    belowMaxRedeemPerBlock(order.usde_amount)
{
    if (order.order_type != OrderType.REDEEM) revert InvalidOrder();    
    // Verify the validity of the order and the signature
    (bool isValid, bytes32 orderHash) = verifyOrder(order, signature);
    if (!isValid) revert InvalidOrder();
    // Increment redeemedPerBlock after verifying the order
    redeemedPerBlock[block.number] += order.usde_amount;
    // Perform the redemption and asset transfer to the beneficiary
    usde.burnFrom(order.benefactor, order.usde_amount);
    _transferToBeneficiary(order.beneficiary, order.collateral_asset, order.collateral_amount);
    emit Redeem(
        msg.sender,
        order.benefactor,
        order.beneficiary,
        order.collateral_asset,
        order.collateral_amount,
        order.usde_amount
    );
}