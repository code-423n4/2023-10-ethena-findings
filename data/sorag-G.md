# x += y costs more gas than x = x + y for state variables
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L174
```diff
- mintedPerBlock[block.number] += order.usde_amount;
+mintedPerBlock[block.number] = mintedPerBlock[block.number] + order.usde_amount;
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L205
```diff
- redeemedPerBlock[block.number] += order.usde_amount;
+ redeemedPerBlock[block.number] = redeemedPerBlock[block.number]+order.usde_amount;
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L368C1-L368C37

```diff
- totalRatio += route.ratios[i];
+ totalRatio = totalRatio+ route.ratios[i];
```