## Introduction
These are my solutions to two of the four problems @[jcsc-security] exposed in his Reentrancy Workshop. Thanks to @[jcsc-security] for letting me share these solutions.

### 02-xFunction (Cross Function attack)
This SC is vulnerable to a reentrancy problem, but it is not the typical reentrancy attack where the attackers call the same function over and over again. Here, we, the attackers, must divide our efforts because the main function (withdraw) is supposed to be protected against reentrancy problems. But there is another function, transferTo, which we can call from our attack smart contract to modify our balance's view before updating it in the withdraw function. A more detailed explanation of this problem is exposed [here](test/02-poc_notes.md)

### 03-xContract (Cross Contract attack)
In this case, we have a very similar attack, but instead of calling a function in the same contract to take advantage of the vulnerability, we will need to call a function in a different smart contract. In this case, a detailed explanation is not necessary because the contract states change similarly to the last case.

