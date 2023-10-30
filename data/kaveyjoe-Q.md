## Q>01 No Check for order.expiry in mint and redeem Functions
**Description**
While there is a check for order.expiry being in the past in the verifyOrder function, there is no similar check in the mint and redeem functions. This could potentially allow expired orders to be processed.

**Impact**
Allowing expired orders to be processed can result in unexpected behavior, potentially leading to loss of assets or misuse of the contract.

**Proof of Concept**
Suppose an order is created with an expiry date set in the past.
If the mint or redeem function is called with this expired order, it will be processed without any checks for expiry.
**Recommendation**
Add a check for order.expiry in both the mint and redeem functions. Ensure that expired orders are rejected and appropriate error handling is implemented.

## Q>2: No Event Emission on _setMaxMintPerBlock and _setMaxRedeemPerBlock
**Description**
While the changes to maxMintPerBlock and maxRedeemPerBlock are logged, there are no events being emitted to notify external systems of these changes. It might be beneficial to emit events for transparency and monitoring purposes.

**Impact**
The absence of events for these changes may lead to a lack of transparency and monitoring capabilities for external systems. It may be challenging to track and verify modifications to these critical parameters.

**Proof of Concept**
Observe that there are no events emitted in the _setMaxMintPerBlock and _setMaxRedeemPerBlock functions.
**Recommendation**
Add event emissions in the _setMaxMintPerBlock and _setMaxRedeemPerBlock functions to notify external systems of changes to maxMintPerBlock and maxRedeemPerBlock. This enhances transparency and facilitates monitoring by external parties.

## Q>3: No Return Value for belowMaxMintPerBlock and belowMaxRedeemPerBlock
**Description**
The belowMaxMintPerBlock and belowMaxRedeemPerBlock modifiers are used in the mint and redeem functions, but they don't return anything. Depending on the context, it might be beneficial to add a return value or revert message for clarity.

**Impact**
Without a return value or revert message, it may be unclear why a transaction fails when the belowMaxMintPerBlock or belowMaxRedeemPerBlock modifiers are used.

**Proof of Concept**
Observe that there is no return value or revert message specified in the belowMaxMintPerBlock and belowMaxRedeemPerBlock modifiers.
**Recommendation**
Consider adding a return value or revert message to the belowMaxMintPerBlock and belowMaxRedeemPerBlock modifiers for clarity. This will provide a clear indication of why a transaction may fail when these modifiers are in use.