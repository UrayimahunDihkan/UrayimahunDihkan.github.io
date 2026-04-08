---
title: Improve the gas efficiency in solidity (Summary)
date: 2026-04-08 15:57:34
tags: tech
---

1. **Can optimize the sort of variables in storage <u>if it's possible</u>**

   Before change the sort :

   ```solidity
   uint16 public a;    
   uint256 public x;  
   uint32 public b;
   ```

   <div style="text-align: center;">
       <img src="https://static.wixstatic.com/media/706568_2796450a514943efa19e2eea4b986099~mv2.png" alt="Storage" width="450" height="100">
   </div>

   After change the sort:

   ```solidity
   uint256 public x; 
   uint16 public a; 
   uint32 public b;
   ```

   <div style="text-align: center;">
       <img src="https://static.wixstatic.com/media/706568_3fc7736a7d974e738cd5b74810cba347~mv2.png" alt="Storage" width="450" height="150">
   </div>

   ***Enhanced a lot , this is good.***

2. aaa

   

3. Ccc
