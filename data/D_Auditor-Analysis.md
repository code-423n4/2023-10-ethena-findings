### General Overview
Ethena Labs emerges as a pioneering web3 protocol, introducing USDe, a synthetic dollar, with a fundamental goal of reducing dependence on conventional banking infrastructure. USDe represents a notable advancement in the world of decentralized finance, embodying key attributes of censorship resistance, scalability, and unwavering stability. This crypto-native solution is intriguingly transparent, backed by on-chain assets, and designed for seamless integration within the diverse DeFi ecosystem. To sustain its pegged stability, Ethena employs a robust combination of delta-hedging and arbitrage mechanisms, ensuring its position as a stalwart digital asset.

### Assets in Scope Definition

#### USDe
`USDe.sol` serves as Ethena's stablecoin contract, extending `ERC20Burnable`, `ERC20Permit`, and `Ownable2Step` from OpenZepplin. This contract encompasses essential functionalities like stablecoin minting and burning. Inherited from the Ownable2Step contract, it provides the USDe contract with the owner role, the `onlyOwner` modifier, and the `MINTER_ROLE`. The owner designates the `MINTER_ROLE` address, enabling the minting of USDe tokens in exchange for approved tokens to users. Notably, the `renounceOwnership()` function is disabled to prevent a compromised owner from disrupting the protocol by renouncing the role instead of transferring it.

#### Ethena Minting
`EthenaMinting.sol` stands as a pivotal component in the Ethena ecosystem, synergizing seamlessly with `USDe.sol` to facilitate the minting and redemption processes of USDe tokens. Within this contract, the `minter` variable in `USDe.sol` directly links to `EthenaMinting.sol`'s contract address, making it a core hub for user interactions.

For minting, users initiate the process by providing their order and signature parameters, which are then verified through an EIP 712 signature process. The `route` parameter, generated by Ethena, ensures the correct flow of collateral, guaranteeing its secure transfer to Ethena's custodian addresses. Notably, only addresses listed within `_custodianAddresses`, added by the `DEFAULT_ADMIN_ROLE`, are accepted, reinforcing security protocols.

Redemption operates similarly, with users validating their intentions via an EIP 712 signature, enabling them to redeem USDe tokens efficiently. Ethena ensures a reserve of $100k-$200k worth of collateral, facilitating smooth hot redemptions. For large redemptions, users can opt for gradual redemptions over multiple blocks or trade USDe on the open market.

Moreover, `EthenaMinting.sol` allows for the setting of delegated signers, beneficial for smart contracts. This feature enables the delegation of signing authority to specified EOA addresses, enhancing the flexibility of the redemption process. With meticulous attention to detail, this contract ensures secure and efficient interactions within the Ethena ecosystem, fostering user confidence and operational integrity.

#### USDe Staking and stUSDe
In the context of the protocol, `StakedUSDeV2.sol` plays a pivotal role in facilitating users' staking and yield-earning activities. Holders of the stablecoin, USDe, utilize this contract to stake their assets and receive stUSDe tokens in return, subsequently benefiting from the yield program.

In the protocol, yields are disbursed by individuals designated as `REWARDERs`. Their role is to transfer yields in USDe to the staking contract, thereby enhancing the value of stUSDe in relation to USDe. This approach is pivotal to ensure that yield distribution remains equitable and efficient.

A key modification in this contract is a departure from the ERC4626 standard. We have introduced a linear vesting mechanism for rewards, spanning over 8 hours. This change is instrumental in preventing users from potentially manipulating yield payments by front-running and quickly unwinding their positions. The implementation of the `REWARDER` role bolsters this safeguard, further securing the yield distribution process.

Additionally, we have incorporated an essential feature in the form of a 14-day cooldown period for unstaking stUSDe. From users' perspective, the process appears instant - stUSDe is immediately burned upon initiation of unstaking. However, in practice, the release of USDe takes place after the cooldown period concludes, serving as an additional security measure. Behind the scenes, during the stUSDe burning process, USDe funds are securely held in a separate silo contract. Subsequently, during withdrawal, the staking contract orchestrates a seamless transfer of user funds from the silo contract to their respective addresses. This cooldown period is configurable and can be tailored for durations of up to 90 days, ensuring flexibility and security for users.

In conclusion, `StakedUSDeV2.sol` stands as a critical component within the protocol, diligently working to provide a seamless and secure staking and yield-earning experience for valued users.

### **Approach to Vulnerability Assessment**

My methodology for identifying vulnerabilities in the Ethena Labs protocol is a multi-step, detail-oriented process that ensures a comprehensive evaluation. Here's a concise report of my approach:

1. **In-Depth Protocol Understanding**: I initiate the assessment by thoroughly understanding the protocol. This step involves unraveling the core functionality of each function, identifying essential functions, and determining the necessary states for their execution.

2. **Review of Previous Audits and Code Comments**: I extensively study previous audit reports and code comments. This provides a valuable reference for recognizing vulnerabilities and understanding conditions outlined in prior assessments. I aim to align my findings with those of previous auditors.

3. **Bot Race Findings**: I examine Bot Race findings, though I do this after my manual code review. This process enables me to compare my discoveries with those generated through automated analysis and to validate my findings.

4. **Threat Model Construction**: A crucial element of my approach is the creation of a threat model. This model encompasses various scenarios and conditions, extending beyond expected cases outlined by the protocol or functions.

5. **Meticulous Manual Code Review**: I systematically review the code, validating the threats identified in my model. This hands-on inspection is vital for detecting vulnerabilities and confirming which potential threats are substantiated in the code.

In summary, my approach involves a structured process, beginning with a deep understanding of the protocol, followed by the review of previous audits, cross-referencing findings with Bot Race results, constructing a comprehensive threat model, and, finally, meticulous code scrutiny. This approach ensures a thorough assessment of the protocol's security.

### Codebase quality analysis and Architecture Recommendations
The current state of the codebase looks promising. While there are a few redundancies scattered throughout, the overall foundation appears robust. The concepts and user experience are straightforward.

Considering the number of audits the code has undergone, it's advisable to continue regular audits and maintain up-to-date security measures. Even with the existing audits, there were still some redundancies found in the code, indicating the importance of ongoing vigilance.

```solidity

    function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting(); //if there are rewards to be vested
    
    uint256 newVestingAmount = amount + getUnvestedAmount();

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```
The function 'stakedUSDe::transferInRewards()' contains redundant lines of code. Given the existing check that ensures 'getUnvestedAmount' returns zero, adding zero to the 'amount' to obtain 'newVestingAmount' seems unnecessary. Redundancy appears to be a recurring issue within the codebase.

```solidity
  function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
    if (orderType == OrderType.REDEEM) {
      return true;
    }
    uint256 totalRatio = 0;
    if (route.addresses.length != route.ratios.length) {
      return false;
    }
    if (route.addresses.length == 0) {
      return false;
    }
    
    for (uint256 i = 0; i < route.addresses.length; ++i) {
      //ensures that each route address is not a zero address, 0 ratioed and a custdian address
      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
      {
        return false;
      }

      //if the above is not valid, then the sum of the route ratios is calculated
      totalRatio += route.ratios[i];
    }
```
In 'stakedUSDe::redeem()', the line checking 'orderType == redeem' is superfluous since the redeem function doesn't call 'verifyOrder'. It's evident that there are highly skilled and meticulous auditors reviewing the code. This report won't extensively discuss the findings, but it's clear there are a few improvements that can be implemented in the codebase.

### ADDENDUM
In conclusion, the most significant risk in relation to Ethena Labs and within the scope of this audit appears to be the risk of centralization. The protocol relies on multiple authorized roles, and both the protocol and users must place trust in the integrity of these roles to prevent compromise. While the protocol has taken commendable steps to mitigate these risks through extensive and specific tests in the test suite, the potential for compromised roles remains a concern. The protocol must take all necessary precautions to guard against such compromises, as the repercussions of such events could be severe. Although I have identified certain issues in my findings, I have chosen not to elaborate on them in this report.

### Time spent:
36 hours