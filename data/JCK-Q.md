## LOW Risk

| Number | Issue | Instances |
|--------|-------|-----------|
|[L-01]| For loops in public or external functions should be avoided due to high gas costs and possible DOS  | 3 |
|[L-02]| Use descriptive constant instead of 0 as a parameter  | 2  |
|[L-03]| Revert on transfer to the zero address  | 4  |
|[L-04]| Functions calling contracts with transfer hooks are missing reentrancy guards  | 3 |
|[L-05]| Lack of User Validation in the EthenaMinting.sol  | 2 |
|[L-06]| Lack of Limitations in the removeSupportedAsset Function  | 1 |
|[L-07]| When removing _supportedAssets _supportedAssets.remove(asset) should be set to 0 for inactive  | 1 |

## [L‑01] For loops in public or external functions should be avoided due to high gas costs and possible DOS

In Solidity, for loops can potentially cause Denial of Service (DoS) attacks if not handled carefully. DoS attacks can occur when an attacker intentionally exploits the gas cost of a function, causing it to run out of gas or making it too expensive for other users to call. Below are some scenarios where for loops can lead to DoS attacks: Nested for loops can become exceptionally gas expensive and should be used sparingly

There are 3 instances of this issue:

```solidity
file: main/contracts/EthenaMinting.sol

126   for (uint256 i = 0; i < _assets.length; i++) {
            addSupportedAsset(_assets[i]);
        }
    
130   for (uint256 j = 0; j < _custodians.length; j++) {
            addCustodianAddress(_custodians[j]);
        }

424   for (uint256 i = 0; i < addresses.length; ++i) {
            uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
            token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
            totalTransferred += amountToTransfer;
        }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126-L128

## [L‑02] Use descriptive constant instead of 0 as a parameter

Passing 0 or 0x0 as a function argument can sometimes result in a security issue(e.g. passing zero as the slippage parameter). A historical example is the infamous 0x0 address bug where numerous tokens were lost. This happens because 0 can be interpreted as an uninitialized address, leading to transfers to the 0x0 address, effectively burning tokens. Moreover, 0 as a denominator in division operations would cause a runtime exception. It's also often indicative of a logical error in the caller's code.

Consider using a constant variable with a descriptive name, so it's clear that the argument is intentionally being used, and for the right reasons..

```solidity
file:  main/contracts/EthenaMinting.sol

230    _setMaxMintPerBlock(0);

231    _setMaxRedeemPerBlock(0);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L230

## [L‑03] Revert on transfer to the zero address

It's good practice to revert a token transfer transaction if the recipient's address is the zero address. This can prevent unintentional transfers to the zero address due to accidental operations or programming errors. Many token contracts implement such a safeguard, such as

```solidity
file:  main/contracts/EthenaMinting.sol

426   token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L426

```solidity
file: main/contracts/StakedUSDe.sol

96   IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

140  IERC20(token).safeTransfer(to, amount);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96


```solidity
file: main/contracts/USDeSilo.sol

29   USDE.transfer(to, amount);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L29

## [L-04] Functions calling contracts with transfer hooks are missing reentrancy guards

Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to read-only reentrancies with no way to protect against it, except by block-listing the whole protocol.

```solidity
file: main/contracts/EthenaMinting.sol

408   IERC20(asset).safeTransfer(beneficiary, amount);

426  token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L408

```solidity
file: main/contracts/StakedUSDe.sol

140    IERC20(token).safeTransfer(to, amount);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L140

## [L-05] Lack of User Validation in the EthenaMinting.sol


### Vulnerability details

the setDelegatedSigner function lacks crucial checks to verify the validity and authenticity of the user attempting to delegate authority to another address. 

Unauthorized Delegation:

Users, denoted by msg.sender, can freely invoke the setDelegatedSigner function to designate any Ethereum address as a delegated signer without undergoing any validation or authentication. This absence of validation exposes the system to the possibility of unauthorized delegation, deviating from the intended security model.
Delegating signing authority to an unverified or unauthorized address introduces security vulnerabilities. Malicious actors may exploit this deficiency to impersonate authorized signers or engage in fraudulent activities, potentially jeopardizing the overall security of the system.

Loss of Control:

The absence of proper user validation deprives the contract owner or administrators of effective control over the delegation of signing authority. This lack of control can result in a disorderly and potentially chaotic delegation system.

```solidity
file:  main/contracts/EthenaMinting.sol

235    function setDelegatedSigner(address _delegateTo) external {
    delegatedSigner[_delegateTo][msg.sender] = true;
    emit DelegatedSignerAdded(_delegateTo, msg.sender);
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L235-L238

Impact of the Vulnerability:
compromised system integrity, a heightened potential for fraudulent activities, and a lack of accountability in the delegation process. Unauthorized delegations may disrupt the system's expected behavior and security, potentially leading to financial or reputational damage.

Proof of Concept:
To address these Vulnerabilitys, it is strongly recommended to enhance the setDelegatedSigner function with robust user authentication and access control mechanisms. These measures could include requiring users to provide authentication, such as a valid signature and ensuring that they possess the necessary privileges to delegate signing authority. Access control can be implemented through role-based permissions or other appropriate methods, enabling better control and oversight. Additionally, the introduction of an approval process or a whitelist for delegated signers can fortify the security and governance of the delegation system, mitigating the identified Vulnerabilitys.


in unstake()  it's essential to clarify who the staker is, how assets are obtained for staking, and implement appropriate access control mechanisms for the withdraw function. Access control can be enforced through role-based permissions or other authentication methods to ensure that only authorized users can execute withdrawal operations. Additionally, proper documentation and comments in the code should clarify the intended behavior of the unstake function and the associated withdraw function to avoid misunderstandings and ensure secure operation.


```solidity
file: main/contracts/StakedUSDeV2.sol

78  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L78-L90


## [L-06] Lack of Limitations in the removeSupportedAsset Function

Vulnerability details:

The removeSupportedAsset function, as presented, lacks any limitations or checks on the quantity of assets that can be removed. This omission introduces a substantial Vulnerability: users with the DEFAULT_ADMIN_ROLE can potentially remove numerous assets simultaneously or even remove all supported assets. The associated Vulnerability are as follows:

Mass Removal of Assets:
Users endowed with the DEFAULT_ADMIN_ROLE have the capability to initiate the removal of a considerable number of assets in a single transaction. This creates the Vulnerability of a massive asset removal event, potentially disrupting the system and adversely affecting users who rely on these assets.

Loss of Functionality:
In the event that all supported assets are removed, the system may suffer a profound loss of functionality. Such a scenario could render the system unusable and lead to severe inconvenience for its users, resulting in significant disruptions.

Lack of Control:
The absence of limitations or checks within the function exacerbates the Vulnerability of a loss of control and oversight over asset removals. This lack of control can lead to unintended consequences, disputes, and disruptions within the ecosystem.

```solidity
file: contracts/EthenaMinting.sol

259  function removeSupportedAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();
    emit AssetRemoved(asset);
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L259-L262

Impact of the Vulnerability:
loss of key functionality, and potential disputes or user dissatisfaction.

Proof of Concept:
To address these Vulnerability, it is highly recommended to implement appropriate limitations and controls on asset removal within the removeSupportedAsset function. This may involve setting a maximum limit on the number of assets that can be removed in a single transaction or over a specific timeframe. Additionally, consider the implementation of governance mechanisms or the requirement of multi-signature approvals for asset removals. These measures are essential to ensure that asset removals are carried out in a controlled and deliberate manner, thus reducing the Vulnerability of mass or unauthorized removals and safeguarding the system's stability and functionality.


## [L-07] When removing _supportedAssets _supportedAssets.remove(asset) should be set to 0 for inactive

Function removeSupportedAsset() removes a _supportedAssets if _supportedAssets.remove(asset) however it does not set  _supportedAssets.remove of the _supportedAssets to 0 for inactive. In  EnumerableSet.AddressSet state it says if a _supportedAssets is inactive activationRound should be 0.


```solidity
file: main/contracts/EthenaMinting.sol

259   function removeSupportedAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();
    emit AssetRemoved(asset);
  } 

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L259-L261