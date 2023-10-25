1. These revert messages are used but not defined:

ZeroAddressException
OnlyMinter
CantRenounceOwnership

This is a bug because when these require statements revert, it will revert with a generic "revert" message instead of a custom message.

Recommendation:

Define the custom error messages:
error ZeroAddressException();
error OnlyMinter(); 
error CantRenounceOwnership();