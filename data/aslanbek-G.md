# [G-01] Remove onlyOwner modifier from renounceOwnership

[USDe.sol#L33-L35](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L33-L35) 

The function reverts anyways. Removing the modifier saves 1600 gas on deployment.

# [G-02] Use hardcoded value instead of retrieving it from calldata 
[EthenaMinting.sol#L171](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L171)

`order.order_type` is guaranteed to be `OrderType.MINT` thanks to the check at line 169.

```diff
    if (order.order_type != OrderType.MINT) revert InvalidOrder();
    verifyOrder(order, signature);
-   if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
+   if (!verifyRoute(route, OrderType.MINT)) revert InvalidRoute();
```

```diff
  | contracts/EthenaMinting.sol:EthenaMinting contract |                 |       |        |        |         |
  |----------------------------------------------------|-----------------|-------|--------|--------|---------|
  | Deployment Cost                                    | Deployment Size |       |        |        |         |
- | 3576457                                            | 18793           |       |        |        |         |
+ | 3575057                                            | 18786           |       |        |        |         |
  | Function Name                                      | min             | avg   | median | max    | # calls |
- | mint                                               | 4657            | 80639 | 123779 | 195520 | 38      |
+ | mint                                               | 4657            | 78735 | 123632 | 195373 | 39      |
```
# [G-03] Remove adding zero
[StakedUSDe.sol#L90-L91](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L90-L91)

```diff
    if (getUnvestedAmount() > 0) revert StillVesting();
-   uint256 newVestingAmount = amount + getUnvestedAmount();
+   uint256 newVestingAmount = amount;
```
# [G-04] Redundant address casting 
```diff
  function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
-   if (address(token) == asset()) revert InvalidToken();
+   if (token == asset()) revert InvalidToken();
```
```diff
  | contracts/EthenaMinting.sol:EthenaMinting contract |                 |       |        |        |         |
  |----------------------------------------------------|-----------------|-------|--------|--------|---------|

  | Function Name                                      | min             | avg   | median | max    | # calls |

- | mint                                               | 4657            | 80639 | 123779 | 195520 | 38      |
+ | mint                                               | 4657            | 80552 | 123779 | 195520 | 38      |


  | contracts/StakedUSDe.sol:StakedUSDe contract |                 |       |        |       |         |
  |----------------------------------------------|-----------------|-------|--------|-------|---------|

  | Function Name                                | min             | avg   | median | max   | # calls |

- | deposit                                      | 16246           | 62953 | 75786  | 78986 | 38      |
+ | deposit                                      | 16246           | 62946 | 75786  | 78986 | 38      |

- | mint                                         | 15450           | 45612 | 45612  | 75775 | 2       |
+ | mint                                         | 15215           | 45495 | 45495  | 75775 | 2       |

- | totalAssets                                  | 1550            | 1740  | 1883   | 1883  | 14      |
+ | totalAssets                                  | 1550            | 1716  | 1716   | 1883  | 14      |


  | contracts/StakedUSDeV2.sol:StakedUSDeV2 contract |                 |       |        |       |         |
  |--------------------------------------------------|-----------------|-------|--------|-------|---------|

  | Function Name                                    | min             | avg   | median | max   | # calls |

- | cooldownShares                                   | 631             | 58837 | 69459  | 87674 | 12      |
+ | cooldownShares                                   | 631             | 58817 | 69342  | 87674 | 12      |
```
# [G-05] use uint256 instead of uint104

[IStakedUSDeCooldown.sol#L8](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IStakedUSDeCooldown.sol#L8)

```diff
struct UserCooldown {
-   uint104 cooldownEnd;
+   uint256 cooldownEnd;
    uint256 underlyingAmount;
}
```

Remove downcasting:

[StakedUSDeV2.sol#L100](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L100)

[StakedUSDeV2.sol#L116](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L116)

Change uint104 to uint256:

[StakedUSDeV2.blacklist.t.sol#L95](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/staking/StakedUSDeV2.blacklist.t.sol#L95)

[StakedUSDeV2.cooldownEnabled.t.sol#L88](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/staking/StakedUSDeV2.cooldownEnabled.t.sol#L88)

[StakedUSDeV2.cooldownEnabled.t.sol#L110](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/staking/StakedUSDeV2.cooldownEnabled.t.sol#L110)

```diff
  | contracts/StakedUSDe.sol:StakedUSDe contract |                 |       |        |       |         |
  |----------------------------------------------|-----------------|-------|--------|-------|---------|
  | Deployment Cost                              | Deployment Size |       |        |       |         |
  | 3215153                                      | 17592           |       |        |       |         |
  | Function Name                                | min             | avg   | median | max   | # calls |

- | mint                                         | 15450           | 45612 | 45612  | 75775 | 2       |
+ | mint                                         | 15215           | 45495 | 45495  | 75775 | 2       |



  | contracts/StakedUSDeV2.sol:StakedUSDeV2 contract |                 |       |        |       |         |
  |--------------------------------------------------|-----------------|-------|--------|-------|---------|
  | Deployment Cost                                  | Deployment Size |       |        |       |         |
- | 3773992                                          | 20505           |       |        |       |         |
+ | 3725335                                          | 20262           |       |        |       |         |
  | Function Name                                    | min             | avg   | median | max   | # calls |

- | balanceOf                                        | 569             | 1140  | 569    | 2569  | 98      |
+ | balanceOf                                        | 569             | 1194  | 569    | 2569  | 96      |

- | cooldownAssets                                   | 630             | 53751 | 71061  | 87449 | 9       |
+ | cooldownAssets                                   | 630             | 53651 | 71019  | 87396 | 9       |

- | cooldownShares                                   | 631             | 58817 | 69342  | 87674 | 12      |
+ | cooldownShares                                   | 631             | 57526 | 69966  | 87621 | 10      |

- | cooldowns                                        | 762             | 1047  | 762    | 4762  | 14      |
+ | cooldowns                                        | 738             | 1071  | 738    | 4738  | 12      |

- | deposit                                          | 16246           | 64141 | 75786  | 78986 | 58      |
+ | deposit                                          | 16246           | 65158 | 76517  | 78986 | 56      |

- | totalAssets                                      | 1572            | 1738  | 1738   | 1905  | 24      |
+ | totalAssets                                      | 1572            | 1766  | 1905   | 1905  | 24      |

- | unstake                                          | 5289            | 19778 | 22152  | 22152 | 14      |
+ | unstake                                          | 5177            | 19622 | 22062  | 22062 | 12      |

  | contracts/USDe.sol:USDe contract |                 |       |        |       |         |
  |----------------------------------|-----------------|-------|--------|-------|---------|
  | Deployment Cost                  | Deployment Size |       |        |       |         |
  | 1589682                          | 8940            |       |        |       |         |

- | approve                          | 2649            | 24229 | 24649  | 24649 | 150     |
+ | approve                          | 2649            | 24224 | 24649  | 24649 | 148     |

- | balanceOf                        | 580             | 1209  | 580    | 2580  | 394     |
+ | balanceOf                        | 580             | 1222  | 580    | 2580  | 386     |

- | mint                             | 564             | 40774 | 48790  | 48790 | 167     |
+ | mint                             | 564             | 40966 | 48790  | 48790 | 165     |

- | transfer                         | 2453            | 19922 | 21469  | 24966 | 52      |
+ | transfer                         | 2453            | 19860 | 21469  | 24966 | 48      |

- | transferFrom                     | 2700            | 15448 | 20451  | 22051 | 123     |
+ | transferFrom                     | 2700            | 15629 | 20451  | 22051 | 121     |



  | contracts/USDeSilo.sol:USDeSilo contract |                 |       |        |       |         |
  |------------------------------------------|-----------------|-------|--------|-------|---------|
  | Deployment Cost                          | Deployment Size |       |        |       |         |
  | 107348                                   | 783             |       |        |       |         |
  | Function Name                            | min             | avg   | median | max   | # calls |
- | withdraw                                 | 3768            | 16911 | 18935  | 18935 | 14      |
+ | withdraw                                 | 3768            | 16573 | 18935  | 18935 | 12      |
```
