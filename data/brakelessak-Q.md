## MAX_COOLDOWN_DURATION not marked as constant

Relevant files: StakedUSDeV2.sol
Relevant lines: L22

The state variable `MAX_COOLDOWN_DURATION` can be declared as `immutable` or `constant`.

```solidity
uint24 public MAX_COOLDOWN_DURATION = 90 days;
```

```solidity
uint24 public constant MAX_COOLDOWN_DURATION = 90 days;
```