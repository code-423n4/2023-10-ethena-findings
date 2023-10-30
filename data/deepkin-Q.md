## EthenaMinting.sol
### Two Admin _grantRole in one constructor
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L124
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L138-L140

Make no sense because contract inherited from SingleAdminAccessControl, that will have only one. And it's better for readability to set Admin in one "if" block.

#### Reccomendation 
Move in scope of one "if-else" block.

### No protection of adding StackedUSDe(V2) to _supportedAssets. 

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L290-L295
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L126-L128

It's possible to add StackedUSDe(V2) asset for EthenaMinting._supportedAssets. This can lead to endless minting of USDe and StackedUSDe(V2) tokens.

#### Recommendation
Add check for StackedUSDe(V2).

### Mint / burn inconsistency for native Eth.
There is no information about how contract and mint logic should work with native Eth, but both receive() and EthenaMinting._transferToBeneficiary() support it. 
Currently there is no options to mint() with native Eth, but contract still can burn() during EthenaMinting.redeem()
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L194-L216

#### Recommendation
Make flows consistent to be sure that end logic wouldn't be broken.


## StakedUSDe.sol
### Unnecessary "getUnvestedAmount()" call
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L90-L91

With current implementation getUnvestedAmount() always will be null, so there is no reason for "amount + getUnvestedAmount()".

#### Recommendation
Remove unnecessary call.

### Incorrect blocklist code documentation
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L101-L113
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L115-L127

Both addToBlacklist() and removeFromBlacklist() have code documentation with similar context:
*Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to ...*
But with current implementation only blacklist manager can call both functions.
I report it as QA because from what I saw there is no additional documentation on that in Gitbook or C4 audit description.

#### Recommendation
Fix code documentation.

### It's possible to block future(pending) Admin and there will be no option to unblock
Marked as low because it shouldn't really break anything, but this flow still can be a risk in case if Ethena pending Admin will be a sender or receiver in assets transfer flow.
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L106-L109
In addToBlacklist() function we can see check notOwner(target). So contract have protection to not block an Admin of that contract. 
But because the fact that SingleAdminAccessControl use Transfer/Accept pattern it's still possible to make a mistake and block future(pending) Admin. There is no check during Admin acceptance.

And even more important because of a similar notOwner(target) check in removeFromBlacklist() function.
That literally blocks any way to unblock blocked admin.
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L120C3-L123

#### Recommendation
Recheck if the blocking or unblocking of the Admin leads to any errors that are outside the scope of this audit. Fix removeFromBlacklist() function.

### StakedUSDeV2.sol

#### Disabling cooldown with setCooldownDuration(0) will be ignored by unstake().
Value of current cooldown isn't checked in unstake(). In case when we disable cooldown with setCooldownDuration(0) users will still need to wait for previous cooldown if request for withdraw have been submitted(cooldowns[owner]). 
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L78-L90

#### Recommendation
Extend unstake() logic with checks for disabled cooldowns.

### MAX_COOLDOWN_DURATION should be constant
`public MAX_COOLDOWN_DURATION = 90 days` should be constant

### USDeSilo.sol
#### Unused SafeERC20 library
Library added but not used.
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L13

### USDe.sol
#### deprecated-ERC20Permit import
Deprecated since 4.9.0 https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0, use the  proper one ERC20Permit. 
Similar dependency used in StackedUSDe.sol