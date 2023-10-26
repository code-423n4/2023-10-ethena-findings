# No `StakedUSDeV2::coolDownDuration` lower bound

In the `StakedUSDeV2::setCooldownDuration` function input is required to be lesser than the upper bound `MAX_COOLDOWN_DURATION` but doesn't check for a minimum value. This allows the contract to have a 1 second cooldown period, that will have identical impact as 0 value, and can lead to a unwanted behaviour.