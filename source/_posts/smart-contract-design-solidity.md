---
title: Pattern design in solidity smart contracts
date: 2026-03-08 21:44:42
tags: web3
---



***Access control pattern***

```solidity
control Ownable {
	address public owner;
  constructor() {
  	owner = msg.sender;
  }
  modifier onlyOwner() {
   requier(msg.sender == owner , "You're not the owner");
   // other code
  }
}
```

<div style="margin-top: 25px; padding: 15px; background: #2d3748; border-radius: 6px; border-left: 4px solid #4CAF50; font-size: 13px; color: #e2e8f0; text-align: left;">
    <div style="padding: 10px; background: rgba(255,255,255,0.05); border-radius: 4px;">
        • It is a very simple access controlling implementation with very understandable codes. Once `onlyOwner` is called , will verify whether the caller is the owner of the contract (deployer)
    </div>
</div>

