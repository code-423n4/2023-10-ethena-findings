
Gas optimization No : 1

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L111-L122

The state variable ```cooldownDuration``` reads from state 3 times. 

```jsx
function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (duration > MAX_COOLDOWN_DURATION) {
      revert InvalidCooldown();
    }

    uint24 previousDuration = cooldownDuration;
    cooldownDuration = duration;
    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
  }
}
```

This can be fixing as follows :

```jsx
function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (duration > MAX_COOLDOWN_DURATION) {
      revert InvalidCooldown();
    }
    uint24 _cooldownDuration = cooldownDuration
    uint24 previousDuration = _cooldownDuration;
    _cooldownDuration = duration;
    emit CooldownDurationUpdated(previousDuration, _cooldownDuration);
  }
}
```
Gas optimization No : 2

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L72-L81

The state variable ```_currentDefaultAdmin``` reads from state 3 times. 


```jsx

function _grantRole(bytes32 role, address account) internal override {
    if (role == DEFAULT_ADMIN_ROLE) {
      emit AdminTransferred(_currentDefaultAdmin, account);
      _revokeRole(DEFAULT_ADMIN_ROLE, _currentDefaultAdmin);
      _currentDefaultAdmin = account;
      delete _pendingDefaultAdmin;
    }
    super._grantRole(role, account);
  }
```

It can fix like as follows :

```jsx
function _grantRole(bytes32 role, address account) internal override {
    if (role == DEFAULT_ADMIN_ROLE) {
address currentDefaultAdmin_ =_currentDefaultAdmin;

      emit AdminTransferred(currentDefaultAdmin_, account);
      _revokeRole(DEFAULT_ADMIN_ROLE, currentDefaultAdmin_);
      currentDefaultAdmin_ = account;
      delete _pendingDefaultAdmin;
    }
    super._grantRole(role, account);
  }
```