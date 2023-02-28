---
description: >-
  This blog post explains re-entrancy attacks in smart contracts in an
  easy-to-understand way for beginners, so that they can get a good grasp of the
  concept. While not a professional explanation,......
cover: >-
  https://images.unsplash.com/photo-1578469645742-46cae010e5d4?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw1fHxreW90b3xlbnwwfHx8fDE2Nzc1NzA1MDc&ixlib=rb-4.0.3&q=80
coverY: 0
---

# What the Heck is Reentrancy? A Noob's Guide

I am writing this blog in the journey of going towards smart contract security, so mistakes can happen in the blog post Please let me know, if you spot any errors so that I can fix them and keep learning.

## Fallback() function

The solidity fallback function is executed if none of the other functions match the function identifier or <mark style="color:purple;">no data was provided with the function call</mark>

<pre class="language-solidity"><code class="lang-solidity"><strong>msg.sender.call.value(amount)()
</strong></code></pre>

In the above call function it was not sending any data to the `msg.sender`. if `msg.sender` is a contract, it will call its fallback function.

[Read more about fallback function](https://docs.soliditylang.org/en/v0.8.12/contracts.html#fallback-function)&#x20;

## Reentrancy?

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

A re-entry attack is a sort of security issue that can arise in smart contracts. It occurs when a malicious contract exploits a vulnerability in another contract by repeatedly calling the vuln function before the function completes.

{% code title="Vulnerable Contract" lineNumbers="true" %}
```solidity
function withdrawBalance() public{
        // send userBalance[msg.sender] ethers to msg.sender
        // if mgs.sender is a contract, it will call its fallback function
        if( ! (msg.sender.call.value(userBalance[msg.sender])() ) ){
            throw;
        }
        userBalance[msg.sender] = 0;
    }   
```
{% endcode %}

in the above `withdrawBalance()` the `userBalance[msg.sender]` is updating after the call function.

at line it sending ether with out any data to the msg.sender (malicious contract). now a fallback function is triggered in the maliciuous contract.

{% code title="Malicious contract (attacker)" lineNumbers="true" %}
```solidity
function () public payable{
    require(vulnerable_contract.call(bytes4(sha3("withdrawBalance()"))));
}
```
{% endcode %}

At line 2, the fallback function calls the `withdrawBalance()` function again before it finishes.

By repeatedly executing `withdrawBalance()` before it completes, `userBalance[msg.sender]` does not update, causing the funds in the vulnerable contract to drain.

see the screenshot to get picture of reentrancy attack, how it can happen and how it can be exploited.

<figure><img src=".gitbook/assets/reentrancy flow.png" alt=""><figcaption><p>reentrancy attack flow </p></figcaption></figure>
