---
title: Smart contract security (II)
date: 2026-10-28 13:57:34
tags: tech
---



Security vulnerabilities are not only blame to the code itself, they're even pretty much guaranteed during wait in mempool.

- Sandwich Attack

  - A malicous bot searchs transactions those who is doing trade with the uniswap (in mempool), and before it happens, the bot gets the quote in advance and sends another transaction that can manipulate the price of pool with more gas, then the legal transaction sender does an unfavorable trade.

- Transaction Front Running Attack

  - A malicous bot searchs a profitable transaction in the mempool, and submit the transaction again with higher gas fee , that intercepts and steals your profits

    - for example, you got an arbitraging chance , utilizing a price gap between DEX_A and DEX_B , buy from one and sell to another , you can make a benefit. but when your transaction waiting in the mempool, malicious bot will copy the transaction and re-send with a higher gas price , so that steal your benefit.

  - How to prevent? 

    1. The most commonly used one is Flashbots Protect RPC. You can just customize RPC to [https://rpc.flashbots.net](https://rpc.flashbots.net/) in your wallet's network settings. these nodes runs with private mempool.

    2. Don't let your transaction look like a sitting duck in the mempool

       - Control the slippage that you can accept, don't set it too high

       - Set proxy to the uniswap call, so that the bot only see you're calling an obscure contract. (personally , I recommend this trick)

    

I know this much so far , and continuesly focusing security on web3. I also prefer to prectical implementation of guard against these attacks , will share.
