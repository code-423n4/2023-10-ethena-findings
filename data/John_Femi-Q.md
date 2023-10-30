## Title
Add Pausable Feature

## Impact
In a high stake contract like this that also connects to external features like perps, it is advisable to add pausable feature as an additional security measure incase anything goes wrong and user intends to drain funds across several blocks

## Tools Used
Manual Review

## Recommended Mitigation Steps
it is advisable to add pausable feature as an additional security measure incase anything goes wrong and user intends to drain funds across several blocks



## Title
Add Check for address(this).balance

## Impact
In the `transferToCustody` function, there needs to be an extra check to see that the balance of address is greater than intended withdraw amount, this could save gas rather than 63/64 gas used to call transfer on the address and still revert in the end.

## Proof of Concept
The call function in the line `https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L250` will attempt to transfer tokens to thewallet without checking if enough balance is available before attempting this and most gas would be used to do this, causing user to losing some money in the end

## Tools Used
Manual Review

## Recommended Mitigation Steps
Add an extra check for address(this).balance > amount