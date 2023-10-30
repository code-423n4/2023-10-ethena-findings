# 📜 Analysis Report- Ethena Labs 📜
![Ethena LOGO](https://github.com/0xWeb3boy/photo/assets/113019033/f20a0465-6292-4763-b853-56ddda43ef69?raw=true)
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |When reviewing the code, the approach I adopted  | The phases involved in my code review |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |



## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-10-brahma

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-10-brahma#building-and-running)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Brahma.fi](https://github.com/code-423n4/2023-10-brahma/blob/main/contracts/docs) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2023-10-brahma#building-slither-security-report)| The project does not currently have a slither result, a slither control was created from scratch |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-10-brahma#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-10-brahma#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-10-brahma#main-invariants)||



### Time spent:
22 hours