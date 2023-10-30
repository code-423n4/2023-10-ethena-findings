# [N-01] Comments inconsistent with the actual functionality in **transferInRewards** function 

### Summary
The comments on the function states that the function **transferInRewards** allows the owner to transfer rewards from the controller contract into this contract but the actual implementation gives permisson to **REWARDER_ROLE** to do that. 

### Vulnerability Details
I have confirmed with the sponsers that the functionality is correct, just the comments on the functions are misleading. 

### Impact
The current comments can be misleading.

### Proof of Concept 

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L86-L99

### Recommended Mitigation Steps
Change the comments on the function to match the functionality 

