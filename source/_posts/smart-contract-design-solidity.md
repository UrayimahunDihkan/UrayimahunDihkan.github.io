---
title: Pattern design in solidity smart contracts
date: 2026-03-08 21:44:42
tags: web3
---



***Access control pattern***

This is a relatively simple Access Controlling Pattern implemented with very understandable codes. Once `onlyOwner` is called , will verify whether the caller is the owner of the contract (deployer)

<div style="background: #2d3748; color: #e2e8f0; padding: 16px; border-radius: 6px; font-family: monospace; font-size: 14px; white-space: pre-wrap; border-left: 4px solid #4CAF50; margin: 10px 0;">
  control Ownable {
    address public owner;
    constructor() {
      owner = msg.sender;
    }
    modifier onlyOwner() {
     requier(msg.sender == owner , "You're not the owner");
     // ...
    }
  }
</div>



This is another one, can take care more complicated scenarios , and that is implemented by ***openzeppelin***. we can declare roles for entitlements , then it'll be what's your role that's what you're allowed to do.

<div style="background: #2d3748; color: #e2e8f0; padding: 16px; border-radius: 6px; font-family: monospace; font-size: 14px; white-space: pre-wrap; border-left: 4px solid #4CAF50; margin: 10px 0;">
  import "@openzeppelin/contracts/access/AccessControl.sol"
  contract Mycontract is AccessControl {
     byte32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        // ...
    }
  }
</div>

---

***Withdrawal Pattern***

