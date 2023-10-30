# QA Report

Number of each QA issue type found.

| Id                                                                       | Issue                                                      |
|--------------------------------------------------------------------------|------------------------------------------------------------|
| [L-1](#l-1-usde-should-use-two-step-assignment-for-ownership)            | USDe should use two-step assignment for ownership          |
| [NC-1](#nc-1-natspec---incorrect-access-control-for-transferinrewards)   | NatSpec - Incorrect access control for transferInRewards   |
| [NC-2](#nc-2-natspec---incorrect-access-control-for-addtoblacklist)      | NatSpec - Incorrect access control for addToBlacklist      |
| [NC-3](#nc-3-natspec---incorrect-access-control-for-removefromblacklist) | NatSpec - Incorrect access control for removeFromBlacklist |
| [NC-4](#nc-4-unused-safeerc20-library)                                   | Unused SafeERC20 library                                   |



## Low Risk

### L-1 USDe should use two-step assignment for ownership

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L18-L21
``` Solidity
  constructor(address admin) ERC20("USDe", "USDe") ERC20Permit("USDe") {
    if (admin == address(0)) revert ZeroAddressException();
    _transferOwnership(admin);
  }
```

The constructor of `USDe` is directly assigning ownership to the input parameter address `admin`, bypassing the two-step
functionality inherited from `Ownable2Step`.

Direct ownership assigment has input risk, where if the input was incorrect, with there being no verification ownership
would be lost.
It's safer to implement a two-step process where the new address is first proposed, then later confirmed, 
allowing for more control and the chance to catch errors or malicious activity.

#### Recommended Mitigation
Use `Ownable2Step.transferOwnership` that requires the `admin` call `acceptOwnership` before ownership is actually 
transferred from `msg.sender`.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L18-L21
``` Solidity
  constructor(address admin) ERC20("USDe", "USDe") ERC20Permit("USDe") {
    if (admin == address(0)) revert ZeroAddressException();
-    _transferOwnership(admin);
+   transferOwnership(admin);
  }
```



## Non-Critical

### NC-1 NatSpec - Incorrect access control for transferInRewards
The NatSpec notice for the `extenral` function `StakedUSDe.transferInRewards` incorrectly states the access control as 
being the `owner`, where it is actually `REWARDER_ROLE`.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L86
``` Solidity
  /**
   * @notice Allows the owner to transfer rewards from the controller contract into this contract.
   * @param amount The amount of rewards to transfer.
   */
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
```

Although an `external` function, the reduction in access control only affects the owner, which will be an account under
the Ethena team's control (i.e. users would not call the function via Etherscan an expect correct execution,
as they would not be the owner).

#### Proof Of Concept
Add test as a member of the [Staking ACL tests](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/staking/StakedUSDe.ACL.t.sol)
and run with `forge test --match-test test_access_control_transferInRewards`
``` Solidty
bytes32 public constant REWARDER_ROLE = keccak256("REWARDER_ROLE");

  function test_access_control_transferInRewards() public {
    assertTrue(stakedUSDe.hasRole(DEFAULT_ADMIN_ROLE, owner));
    assertFalse(stakedUSDe.hasRole(REWARDER_ROLE, owner));

    vm.expectRevert(
      "AccessControl: account 0xe05fcc23807536bee418f142d19fa0d21bb0cff7 is missing role 0xbeec13769b5f410b0584f69811bfd923818456d5edcf426b0e31cf90eed7a3f6"
    );
    vm.prank(owner);
    stakedUSDe.transferInRewards(0 ether);
  }
```

#### Recommended Mitigation
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L86
``` Solidity
-   * @notice Allows the owner to transfer rewards from the controller contract into this contract.
+   * @notice Allows the rewarder to transfer rewards from the controller contract into this contract.
```


### NC-2 NatSpec - Incorrect access control for addToBlacklist
The NatSpec notice for the `external` function `StakedUSDe.addToBlacklist` incorrectly states the access control as
allowing `DEFAULT_ADMIN_ROLE`, where it only allows the blacklist managers.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L101-L109
``` Solidity
  /**
   * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to blacklist addresses.
   * @param target The address to blacklist.
   * @param isFullBlacklisting Soft or full blacklisting level.
   */
  function addToBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
```

Although an `external` function, the reduction in access control only affects the owner, which will be an account under
the Ethena team's control (i.e. users would not call the function via Etherscan an expect correct execution,
as they would not be the owner).

#### Proof Of Concept
Add test as a member of the [Staking ACL tests](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/staking/StakedUSDe.ACL.t.sol)
and run with `forge test --match-test test_access_control_addToBlacklist`
``` Solidty
  function test_access_control_addToBlacklist() public {
    assertTrue(stakedUSDe.hasRole(DEFAULT_ADMIN_ROLE, owner));
    assertFalse(stakedUSDe.hasRole(BLACKLIST_MANAGER_ROLE, owner));

    vm.expectRevert(
      "AccessControl: account 0xe05fcc23807536bee418f142d19fa0d21bb0cff7 is missing role 0xf988e4fb62b8e14f4820fed03192306ddf4d7dbfa215595ba1c6ba4b76b369ee"
    );
    vm.prank(owner);
    stakedUSDe.addToBlacklist(alice, true);
  }
```

#### Recommended Mitigation
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L102
``` Solidity
-   * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to blacklist addresses.
+   * @notice Allows blacklist managers to blacklist addresses.
```


### NC-3 NatSpec - Incorrect access control for removeFromBlacklist
The NatSpec notice for the `external` function `StakedUSDe.removeFromBlacklist` incorrectly states the access control as
allowing `DEFAULT_ADMIN_ROLE`, where it only allows the blacklist managers.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L115-L123
``` Solidity
  /**
   * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to un-blacklist addresses.
   * @param target The address to un-blacklist.
   * @param isFullBlacklisting Soft or full blacklisting level.
   */
  function removeFromBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
```

#### Proof Of Concept
Add test as a member of the [Staking ACL tests](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/staking/StakedUSDe.ACL.t.sol)
and run with `forge test --match-test test_access_control_removeFromBlacklist`
``` Solidty
  function test_access_control_removeFromBlacklist() public {
    assertTrue(stakedUSDe.hasRole(DEFAULT_ADMIN_ROLE, owner));
    assertFalse(stakedUSDe.hasRole(BLACKLIST_MANAGER_ROLE, owner));

    vm.expectRevert(
      "AccessControl: account 0xe05fcc23807536bee418f142d19fa0d21bb0cff7 is missing role 0xf988e4fb62b8e14f4820fed03192306ddf4d7dbfa215595ba1c6ba4b76b369ee"
    );
    vm.prank(owner);
    stakedUSDe.removeFromBlacklist(alice, true);
  }
```

#### Recommended Mitigation
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L116
``` Solidity
-   * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to un-blacklist addresses.
+   * @notice Allows blacklist managers to un-blacklist addresses.
```


### NC-4 Unused SafeERC20 library
In `USDeSilo.sol` the `SafeERC20` library is imported and used for the `IERC20` reference it holds, but no functions
of that library are being used.

The library and it's import can be safely removed, simplifying the code and reducing the size (and gas deployment cost) 
for the contract.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L5
``` Solidity
-   import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L13-L13
``` Solidity
-  using SafeERC20 for IERC20;
```

