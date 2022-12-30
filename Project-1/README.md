# Akbank-Practicum-Project-1
Hands on Task (Beginner Level): Build and Deploy an Ether-Store Smart Contract with Remix IDE

![image](https://user-images.githubusercontent.com/85889196/192043159-6f118c21-3120-4f1a-9d74-b37e5632b461.png)


```Solidity
// SPDX-License-Identifier: MIT

pragma solidity >=0.7.0 <0.9.0;


contract Ether{

    address public owner;
    uint256 public balance;

    constructor(){
        owner = msg.sender;
    }
    // Function that adding entered value on account's balance
    receive() payable external{
        balance += msg.value;
    }
    
    // Function that withdraw an entered amount of money. The owner must be msg.sender and balance greater than entered amount
    function withdraw(uint amount, address payable destination) public {
        require(msg.sender == owner, "Only owner can withdraw");
        require(amount <= balance, "Your balance is not enough");
        destination.transfer(amount);
        balance-=amount;
    }
```