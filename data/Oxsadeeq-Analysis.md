

 Analysis Report of Ethena

### Overview of Ethena Protocol

Ethena is a yield farming protocol designed to attract common DeFi users seeking to generate yields. The protocol offers two main products: a permissionless stablecoin, USDe, and a yield-generating token, StUSDe.

1. **USDe:** Ethena's USDe is a stablecoin fully backed by trustless crypto assets, ensuring censorship resistance.

2. **StUSDe:** This token represents a staked amount of USDe, primarily for generating yields and for use in DeFi money markets. It has some centralized aspects to prevent exploits and money laundering.

### Approach for Reviewing the Codebase

We initially assessed the codebase scope as a blueprint for analyzing and reviewing the protocol.

**Scope:**

1. **USDe.sol:** Ethena's EIP20 compliant stablecoin, inheriting ERC20Burnable, ERC20Permit, and Ownable2Step. It includes a minter address to mint tokens to a specified target address.

2. **EthenaMinting.sol:** The contract where users can mint and redeem USDe tokens, also serving as the minter address in the USDe contracts.

3. **StakedUSDe.sol:** Implements the ERC4626 (Vault standard), allowing users to stake USDe and receive StUSDe as shares. It is an extension of the StakedUSDeV2.sol contract.

4. **StakedV2USDeV2.sol:** Extends the StakedUSDe contract functionalities, including a cooldown period for users to unstake and wait for cooldown exhaustion before claiming their underlying tokens.

5. **USDeSilo.sol:** Serves as a vault contract to temporarily hold USDe during the redemption cooldown.

6. **SingleAdminAccessControl.sol:** Extends the OpenZeppelin AccessControl.sol with additional functionalities.

### Audit and Approach

In conducting the contract audit, we utilized various techniques, including:

1. Employing automated tools like Slither and Echidna to scan the codebase.
2. Implementing a top-down approach to understand the developer's perspective and the rationale behind specific design patterns.
3. Crafting tests to stress the system's invariants and subject the design patterns to rigorous scrutiny.
4. Engaging in manual analysis to identify common coding errors.
5. Concluding with a manual comparison between the codebase and the corresponding documentation.

### Codebase Quality Analysis

The codebase adheres to the most recent industry standards, and the analysis reveals:

1. Well-documented and easily readable code, facilitating smooth navigation and comprehension for auditors.
2. Customized code design prioritizing minimizing transaction gas costs.
3. Integration of widely recognized libraries and utilities to enhance the security of the protocol.

### Systemic Risk

- The protocol implements a novel mitigation technique to limit attackers during a black swan event, while also deterring large traders and arbitrageurs responsible for stabilizing asset prices.
- Smart contracts can contain vulnerabilities exploitable by malicious entities, leading to potential asset losses or system manipulation. Mitigating measures are strongly advised post-audit.

### Recommendations

- Using multisig wallets for highly privileged roles to increase the difficulty of compromise.
- Easing restrictions to attract larger traders and implementing an off-chain component to monitor the protocol during black swan events.

### Conclusions

Ethena protocol demonstrates an intriguing solution and design for yield farming and a permissionless stablecoin. The efforts of the Ethena team in developing the codebase are commendable. However, it's crucial to acknowledge the significant risks associated with smart contracts in web3 projects, especially the vulnerability of privileged address wallets to potential malicious actors. The future success of the Ethena protocol relies on effectively mitigating and managing system threats. The Ethena team must remain updated on the latest risks within the evolving realm of web3.

### Time spent:
23 hours