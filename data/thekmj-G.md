| |Issue|Instances| Gas saved|
|-|:-|:-:|:-:|
| [G-01] | Pre-calculated constants can be hardcoded | 2 | - |
| [G-02] | `MAX_COOLDOWN_DURATION` can be made constant | 1 | 2100 |
| [G-03] | `mintedPerBlock` and `redeemedPerBlock` can be implemented in a way that re-uses storage, instead of allocating anew | 2 | 30400 |
| [G-03A] | `maxMintPerBlock` can be storage-optimized with smaller data types | 2 | 4000 |

## [G-01] Pre-calculated constants can be hardcoded

All of the constants within `StakedUSDe` and `EthenaMinting` can be hardcoded and commented, instead of calculating on deployment. This saves deployment gas by saving numerous `abi.encodePacked()` calls and `keccak256()` calls.

Instances:
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L26-L32
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L27-L58

## [G-02] `MAX_COOLDOWN_DURATION` can be made constant

In `StakedUSDeV2`, `MAX_COOLDOWN_DURATION` is set as a storage variable. However, since it cannot be modified, using a constant will save approx. $2100$ gas on every accesses.

## [G-03] `mintedPerBlock` and `redeemedPerBlock` can be implemented in a way that re-uses storage, instead of allocating anew

*Note: We analyze the gas saving for `mintedPerBlock` only. `redeemedPerBlock` can be applied with the exact same techniques described here.*

The `mintedPerBlock` mapping stores the amount of minted USDe per block. We first analyze the gas usage:
- For each `mint()` operation, the modifier `belowMaxMintPerBlock` accesses the storage twice (once for the mapping, once for `maxMintPerBlock`), incurring $2100 * 2 = 4200$ gas.
- The mapping has to be updated for each mint operation. Writing a storage variable from zero to non-zero costs $21000$ gas on a **GSSET**.

Therefore the first redeem for each block will cost at least **25200** gas. We propose a more gas-efficient logic as follow:

- Instead of using a mapping `mintedPerBlock`, use two storage variables `lastMintedBlock` and `lastMintedBlockAmount`. They can be packed into a single storage slot. 
```solidity
uint32 public lastMintedBlock;
uint224 public lastMintedBlockAmount;
```
- The idea is that, to check for minting limit, we only need the info from the previously minted block, instead of for all of the blocks. Since `lastMintedBlock` (and therefore the storage slot) will only be zero on the first ever mint, we eliminate the need for **GSSET** operations by writing into non-zero slots only.

Modify the `belowMaxMintPerBlock()` modifier as follow:

```solidity
modifier belowMaxMintPerBlock(uint256 mintAmount) {
    if (block.number != lastMintedBlock) {
        lastMintedBlock = block.number;
        lastMintedBlockAmount = 0;
    }
    if (lastMintedBlockAmount + mintAmount > maxMintPerBlock) {
        revert MaxMintPerBlockExceeded();
    }
    _;
}
```

The impact is that, for the first mint of each block, converts one **GSSET** operation (21000 gas each) into two **GSRESET** operations (2900 gas each), for **15200 gas** saved. 

The information on amount minted per past block can be retrieved through events or forking if ever needed.

The exact same logic can be applied to `redeem()`, further saving gas for each redeem.

## [G-03A] `maxMintPerBlock` can be storage-optimized with smaller data types

*This finding builds upon [G-03]. Again, same logic can be applied to `redeem()`.*

The maximum supply of USDe cannot realistically exceed `uint112` data type, given $18$ decimals. Therefore `maxMintPerBlock` can be packed together with the two storage variables above, as shown:

```solidity
uint112 public maxMintPerBlock;
uint32 public lastMintedBlock;
uint112 public lastMintedBlockAmount;
```

This reduces the number of storage slots that must be read by one. Converts one **Gcoldsload** (2100 gas each) to one **Gwarmaccess** (100 gas). Thereby saving **2000 gas** per mint/redeem.


