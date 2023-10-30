1. Overview of Ethena Labs

Ethena Labs is a stablecoin issuance and staking platform. The project primarily introduces the stablecoin USDe, which is minted by staking stETH (potentially including more asset types in the future) while further utilizing the staked assets to open corresponding long and short positions on a decentralized perpetual exchange to hedge, ensuring price support for USDe in volatile markets. Furthermore, the platform supports staking USDe to receive stUSDe, which is a share in an ERC4626 vault, and further places the earnings obtained through the decentralized perpetual exchange into the vault to continuously accumulate stUSDe in users' holdings. The rewards distribution from the USDe staking contract is released periodically in a linear manner, and there is a cool down period for withdrawing staked USDe from the vault.

The project's components include:

`SingleAdminAccessControl.sol`: This contract is used for initialization and transferring `DEFAULT_ADMIN_ROLE`, as well as granting or revoking any other roles to an address by `DEFAULT_ADMIN_ROLE`.

`EthenaMinting.sol`: This contract mints or redeems USDe tokens based on signature and order information. During minting, received stETH is distributed proportionally to different `_custodianAddresses` for further investment. The contract limits the maximum quantity of minting and redeeming in each block and ensures that each order is not executed repeatedly through nonce verification.

`StakedUSDe.sol`: Inheriting from ERC4626, this contract allows users to stake USDe and receive corresponding stUSDe. The `REWARDER_ROLE` regularly distributes USDe to stakers in the vault as rewards.

`StakedUSDeV2.sol`: This contract inherits `StakedUSDe.sol` and introduces a cooldown period for users who wish to unstake. During the cooldown period, USDe is temporarily transferred to the `USDeSilo` contract.

`USDeSilo.sol`: This contract is used to store USDe transferred during the cooldown period and return it to users upon expiration. Only the staking vault contract can call this contract.

`USDe.sol`: This is the project's ERC20 stablecoin, which can only be called by the `EthenaMinting` contract.

2. My Thoughts

Stablecoins are an essential component in the ecosystem of every public chains. They typically come in two main forms: algorithmic stablecoins and collateral-backed stablecoins. Algorithmic stablecoins often rely on user-initiated arbitrage to peg their value, making their stability vulnerable during volatile market conditions. Collateral-backed stablecoins, on the other hand, are a more common form of stablecoin, where the stability of their value is maintained by collateral in the form of other stablecoins or fiat currencies.

Ethena Labs, as a blockchain project on the Ethereum mainnet, leverages the current focus on the liquid staking derivatives by using stETH as collateral to create the stablecoin USDe. This approach attracts ETH stakers to immediately convert their staked ETH into USDe while still retaining the right to redeem stETH, which significantly increases the capital utilization in the Ethereum. Ethena also allows for long and short positions through decentralized perpetual exchange, ensuring the stability of the total value of stETH in a volatile market. In addition, stakeholders can further stake USDe to mine and accumulate earnings, increasing the total locked-in USDe and contributing to the stability of the stablecoin's price.

In summary, Ethena Labs follows the current trends on the liquid staking derivatives and offers various mechanisms to peg the price of USDe issued on the platform. It has great potential as a stablecoin ecosystem and contributes to improving the capital efficiency in the Ethereum.

3. Audit Approach

The following is my main audit process:

Step 1: Read the project background, familiarize myself with the code structure of the project contracts, understand the inheritance relationships of the project contracts, and get a rough idea of the function of each function in the code. Annotate the code during this process. (Approximately 3 hours)

Step 2: Iteratively read the code implementation details, mark any code that raises questions, and stay updated with the Discord group and project materials to ensure an understanding of the project's economic model. Findings are cross-checked with the results from bots. (Approximately 9 hours)

Step 3: Confirm the openzeppelin code versions on which the project relies and review the corresponding openzeppelin code to check for any omissions. (Approximately 1 hour)

Step 4: Validate all identified issues by writing test cases or proof of concept for attack steps in a foundry. (Approximately 2 hours)

Step 5: Write findings and audit report. (Approximately 1 hour)

4. Possible Risks

During my audit, I identified two systemic risks in the project:

4.1 Price Issues for stETH

In this project, the quantity of USDe corresponding to stETH in an order is predetermined, but the mechanism for determining the price is not reflected in the contract. Since the price of stETH is constantly fluctuating, the process from obtaining a price oracle quote to executing an order and then transferring stETH to the corresponding perpetual exchange may not occur in a single transaction. This creates the potential for significant deviations between the executed price and the actual price. This could allow users to exploit price differences, particularly when stETH experiences significant price drops, potentially resulting in losses for the contract.

4.2 Risks to Decentralized Perpetual Exchange

In the case of severe market price fluctuations, decentralized perpetual exchange platforms like GMX may have their own systemic risks. The cost of going long or short may not be the same, so the project needs to continuously monitor the status of the exchange and optimize the hedging strategy.

5. Time Spent

The total time spent on the analysis was approximately 3 days, totaling around 16 hours.

### Time spent:
16 hours