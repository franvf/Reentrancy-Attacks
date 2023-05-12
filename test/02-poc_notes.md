## The vulnerability
A typical withdrawal function is programmed on the victim's smart contract, but this time it is protected with a mutex (noon reentrant). But this contract is still vulnerable to reentrancy attacks because another function (transferTo) can be called from an external call before updating the "effects" of withdraw function. So we can withdraw our funds and, in the external call, execute the function transferTo. Therefore we can transfer our funds to a different address controlled by us while we also get back our funds.

Trace:
We have two attack smart contracts (A and B) with one exploit function in each one. This function calls vulnerable.deposit, vulnerable.withdraw, and just in case the vulnerable contract has more ETH than the sidekick contract, we call the other attacker exploit function (the sidekick). Then we execute a recursive call. The exploit function steps are the next ones:

1. SC A has 1 ETH, which is deposited into Vulnerable contract. Now the variables trace is:
    - Vulnerable balance: 11 ETH (Started with 10 ETH)
    - Vulnerable balances mapping: balance[attackerA] = 1 ETH
    - Vulnerable balances mapping: balance[attackerB] = 0 ETH
    - Attacker A balance: 0 ETH (Started with 1 ETH)
    - Attacker B balance: 1 ETH (Initial value)

2. SC A withdraws its ETH from the Vulnerable contract (I'm taking into account that not the entire withdraw function is being executed, just until "call"). Now the variables trace is:
    - Vulnerable balance: 10 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 1 ETH (Not updated at this point)
    - Vulnerable balances mapping: balance[attackerB] = 0 ETH
    - Attacker A balance: 1 ETH 
    - Attacker B balance: 1 ETH 

3. Because non Checks-Effects-Interactions principle is followed, the vulnerable contract balance mapping is not updated yet, so, for now, attackerA still has the ETH in its possession. Now, we, the attackers, can take advantage of it to call the transferTo function in our receive function, which is executed before withdrawal finalization. Therefore we can move our balance to another account we manage and keep the ETH in our possession. After this process, withdraw function is finalized. Now the variables trace is:

    - Vulnerable balance: 10 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 0 ETH 
    - Vulnerable balances mapping: balance[attackerB] = 1 ETH
    - Attacker A balance: 1 ETH 
    - Attacker B balance: 1 ETH 

4. Notice that we are in a similar state at this point to the initial one. But the difference is that our contract B has an ETH in the vulnerable contract (but also, our two attacker contracts keep their initial ETH). So we can call it again. To do so, from the exploit function in contract A, we call the exploit function of contract B, which starts depositing its ETH. Now the variables trace is:

    - Vulnerable balance: 11 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 0 ETH 
    - Vulnerable balances mapping: balance[attackerB] = 2 ETH
    - Attacker A balance: 1 ETH 
    - Attacker B balance: 0 ETH 

5. Now contract B withdraws 2 ETH from the vulnerable contract. Now the variables trace (Until call) is:

    - Vulnerable balance: 9 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 0 ETH 
    - Vulnerable balances mapping: balance[attackerB] = 2 ETH (Not updated yet)
    - Attacker A balance: 1 ETH 
    - Attacker B balance: 2 ETH 

6. Before finalizing the withdrawal, contract B calls the transferTo function and transfers its balance to attack A contract. Now the variables trace is:

    - Vulnerable balance: 9 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 2 ETH 
    - Vulnerable balances mapping: balance[attackerB] = 0 ETH 
    - Attacker A balance: 1 ETH 
    - Attacker B balance: 2 ETH 

7. Withdraw function finishes, and contract B calls contract A exploit function, which starts depositing just an ETH into the vulnerable contract. Now the variables trace is:

    - Vulnerable balance: 9 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 3 ETH 
    - Vulnerable balances mapping: balance[attackerB] = 0 ETH 
    - Attacker A balance: 0 ETH 
    - Attacker B balance: 2 ETH 

8. Contract A withdraw its balance. Now the variables trace (Until call) is:

    - Vulnerable balance: 7 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 3 ETH (Not updated yet)
    - Vulnerable balances mapping: balance[attackerB] = 0 ETH 
    - Attacker A balance: 3 ETH 
    - Attacker B balance: 2 ETH 

9. Contract A calls transferTo before finalizing the withdrawal function, and transfer its balance to the attack contract B. Now the variables trace is:

    - Vulnerable balance: 7 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 0 ETH
    - Vulnerable balances mapping: balance[attackerB] = 3 ETH 
    - Attacker A balance: 3 ETH 
    - Attacker B balance: 2 ETH 

10. Contract A withdraw function finishes, and contract A calls the exploit function at contract B because the vulnerable contract balance is greater than the contract B balance. The exploit function starts depositing 1 ETH into the vulnerable SC. Now the variables trace is:

    - Vulnerable balance: 7 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 0 ETH
    - Vulnerable balances mapping: balance[attackerB] = 4 ETH 
    - Attacker A balance: 3 ETH 
    - Attacker B balance: 1 ETH 

11. Contract B withdraw its balance. Now the variables trace (Until call) is: 

    - Vulnerable balance: 4 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 0 ETH
    - Vulnerable balances mapping: balance[attackerB] = 4 ETH (Not updated yet)
    - Attacker A balance: 3 ETH 
    - Attacker B balance: 5 ETH 

12.  Contract B calls transferTo before finalizing the withdraw function and transferring its balance to the attack contract. A. Now the variables trace is:

    - Vulnerable balance: 4 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 4 ETH
    - Vulnerable balances mapping: balance[attackerB] = 0 ETH 
    - Attacker A balance: 3 ETH 
    - Attacker B balance: 5 ETH 

13. Contract B's withdraw function finishes, and contract B calls the exploit function at contract A because the vulnerable contract's balance is greater than contract A balance. The exploit function starts depositing 1 ETH into the vulnerable SC. Now the variables trace is:

    - Vulnerable balance: 5 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 5 ETH
    - Vulnerable balances mapping: balance[attackerB] = 0 ETH 
    - Attacker A balance: 2 ETH 
    - Attacker B balance: 5 ETH 

14. Contract A withdraw its balance. Now the variables trace (Until call) is: 

    - Vulnerable balance: 0 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 5 ETH (Not updated yet)
    - Vulnerable balances mapping: balance[attackerB] = 0 ETH 
    - Attacker A balance: 7 ETH 
    - Attacker B balance: 5 ETH 

15. Contract A calls transferTo before finalizing the withdraw function, and transfer its balance to attack contract B. Now the variables trac, and the final state is:
   
    - Vulnerable balance: 0 ETH 
    - Vulnerable balances mapping: balance[attackerA] = 0 ETH 
    - Vulnerable balances mapping: balance[attackerB] = 5 ETH 
    - Attacker A balance: 7 ETH 
    - Attacker B balance: 5 ETH 

16.Contract A withdraw function finish, but now the contract B exploit function is not called because the vulnerable contract balance is 0, so we can't withdraw any ETH. Now all the recursive functions finalize, meaning that all exploit functions initiated finish.