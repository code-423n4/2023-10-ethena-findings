# 1. Codebase

## 1.1 overview
Ethena is developing a synthetic dollar, USDe. This coin aims to be stable, permissionless, and censorship-resistant.
Ethena tries to create a stablecoin that doesn't rely on a centralized environment or banking system, like USDC.

This is obtained using stETH as underlying asset. Due to the volatility of this asset, USDe needs mechanisms to ensure peg stability and programmatic delta-neutral hedges concerning the underlying collateral assets.
These mechanisms are based on perpetuals, as explained [here](https://ethena-labs.gitbook.io/ethena-labs/10CaMBZwnrLWSUWzLS2a/solution-overview/underlying-derivatives/futures-vs-perpetuals)
Anyway, these mechanisms are out of scope, and only involved contracts inside the contest are the following:

* *USDe.sol* where is coded ERC20 token stablecoin
* *EthenaMinting.sol* where are described minting and redemption actions. This contract should be the minter defined inside USDe.sol
* *StakedUSDe.sol*. This contract extends ERC4626 and permits users to stake USDe and receive stUSDe, which increases in value
* *StakedUSDeV2.sol* extends StakedUSDe, adding redemption cooldown
* *USDeSilo.sol* where USDe are held during the cooldown period
* *SingleAdminAccessControl.sol* that extends OpenZeppelin's Access Control to manage project governance

## USDe.sol
This contract is a simple representation of the USDe token, extending ERC20Burnable.
It contains few lines of code and seems that it has no critical behaviors.
We want to underline that `renounceOwnership()` is disabled. This means that this contract will have only one owner, that will not change.
Moreover, the owner is the only account that can update the MINTER address, the only address that can mint USDe.
This was reported also by [automatic findings - 4naly3er](https://github.com/code-423n4/2023-10-ethena/blob/main/4naly3er-report.md#M-1), but we
want to stress this. Maybe, this is the most important contract of the project and it needs a different governance strategy.
USDe extends Ownable2Step and permits to transfer of ownership, but if the owner is compromised, the whole project will be compromised (it will able to set minter=address(0)).

We also underline the necessity of having two steps in order to define a newMinter, as the project does for other critical roles.
Finally, we found a little correction in lines 29-30 in order to save gas (we reported it in our gas report).

## EthenaMinting.sol
This contract should be the only one that can mint `USDe`. By design, the `minter` defined inside the `USDe` contract should be `EthenaMinting`.
EthenaMinting uses SafeERC20 for IERC20 and EnumerableSet, in order to avoid duplicates inside some lists.
Furthermore, it defines two modifiers. We think that these could be avoided, due to the fact that are used only once, and implement a nontrivial logic
that should be inside the functions that contain them inside their definitions.

Inside the constructor, we saw a little issue: this function needs to change DEFAULT_ADMIN_ROLE twice.
This happens because addSupportedAsset(), addCustodianAddress(), _setMaxMintPerBlock(), and _setMaxRedeemPerBlock() have to be called when msg.sender has DEFAULT_ADMIN_ROLE role and because developers would give the possibility to set an admin different from msg.sender.
Anyway, we think that this adds a useless level of complexity and danger. A mistake passing the _admin parameter could lead to deploying the contract once again.
We suggest removing the _admin parameter and building the contract using msg.sender which would be the admin.

Furthermore, we have found a lot of code that wastes gas. We reported this in the specific report. Anyway, we want to stress that gas wasting could
lead to a bad opinion of the project, which instead is based on trust. Especially, it could happen due to the fact that contracts will rely on L1.
Useless variables, functions' parameters, or checks could create bad effects.

Moreover, there are some parts of the protocol that should be more clear, because they will be part of the user experience. Using of beneficiary, benefactor, 
routes and nonce need better documentation (that is, anyway, very good)

## StakedUSDe.sol and StakedUSDeV2.sol
These contracts permit to stake USDe. StakedUSDe extends ERC4626, and StakedUSDeV2 extends StakedUSDe, adding the cooldown concept, in order to mitigate loss
if some part of the protocol is compromised.

We found some critical behaviors.
transferInRewards() permits to owner to transfer rewards. We don't understand the mechanisms behind calls of this function.
It permits adding asset liquidity to the contract (USDe). Due to the getUnvestedAmount() limitation (it is different from 0 only if vestingAmount is zero 
or VESTING_PERIOD is ended), adding liquidity could be done only at certain times.
If there is a problem, neither owner can add USDe. Furthermore, arbitrary calls of this function could modify seriously the contract behavior
and also lead to a lack of trust with respect to the project.
We also shared with the sponsor that line #91 is useless, due to the fact that getUnvestedAmount() must be zero at that point.
This is another example of dirty code, that wastes gas and shows the rest of old functionalities.

Finally, we found some gas savings and reported them in the gas report. We reiterate what we said for EthenaMinting.sol.

## USDeSilo.sol
This contract is a simple container from USDe during the redemption cooldown period.
There is no issues according to us. Anyway, we recommend using a mutable staking_vault address with proper governance, in order to manage
the case that the original one is compromised. Furthermore, it could be better to have multiple addresses and a route logic.

## SingleAdminAccessControl.sol
This function represents the Governance core. It permits managing roles and there is a two-step implementation in order to transfer the admin role.
We didn't find any problem with this contract.


# 2. Centralization risks
We have already talked about centralization risk. This was also reported inside [automatic findings - 4naly3er](https://github.com/code-423n4/2023-10-ethena/blob/main/4naly3er-report.md#M-1).
In this project, where developers made so effort to have a good role governance, it should be better to improve this inside USDe.sol and USDeSilo.sol.

# 3. USDe basic idea
The idea of a stablecoin based on stETH and perpetuals seems fascinating. We understand how USDe tries to maintain the delta neutrality.
Anyway, we fear the fact that users' found will be vested in positions that do not belong strictly to Ethena project. This could move
the stability of USDe on a field that could be not fully controlled.

We are really curious about the future of Ethena and USDe, and we hope that it will be able to do what it sets out to do.

# 4. Gas Wasting
We found many slots in order to save gas. This is not good, because like we said before, Ethena will deploy on layer 1, which has higher fees.
We think that users will only appreciate it if developers will make an effort to save gas.
We reported a deep gas analysis, that we hope will be useful.

# 4. Time Spent
* Day 1: Study of Ethena basic concepts
* Day 2: Study of neutral delta idea and perps.
* Day 3: Visual inspection and analysis of SingleAdminAccessControl.sol.
* Day 4: Visual inspection and analysis of USDe.sol and EthenaMinting.sol. 
* Day 5: Visual inspection and analysis of StakedUSDe.sol
* Day 6: Visual inspection and analysis of StakedUSDeV2.sol and USDeSilo.sol
* Day 7: Report writing

Time spent: 35 hours

### Time spent:
35 hours