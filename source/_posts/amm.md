---
title: How AMM module works? [x*y=k]
date: 2026-07-20 19:07:51
tags: tech
---

All swapping pools between two currncies mostly base on the **x·y=k** equation. Usually, x: number of ETH, y: number of USDC. (Initial number of ETH and USDC are based on the exact price when the pool was created). k: liquidity.

Here are two senators for buy and sell,

1. User sells ETH for getting USDC:  (x+dx)·(y-dy)=k
2. User sells USDC for getting ETH:  (x-dx)·(y+dy)=k

Let's assume there is a pool, and there are 10ETH and 20,000USDC in it, then the k=10*20,000=200,000

1. User sells 5 ETH for getting USDC:  (10+5)·(20,000-dy)=200,000  dy=6666.67USDC

2. User sells 10,000USDC for getting ETH: (10-dx)·(20000+10,000)=200,000  dx=3.33ETH

3. What if I am trying to sell 10,000ETH for USDC? Calculate it, for fun ^^

   (10+10,000)·(20000-dy)=200,000 dy=19998USDC

>- Sell 5ETH get 6666.67USDC , then 6666.67/5 = 1,333.33 USDC/ETH
>
>- Sell 10,000ETH get 19998USDC , then 19998/10,000 = 2 USDC/ETH
>
>  What is this?!! So, The bigger you try to swap , the worse the value you get, I think no one do this.

<div style="margin-top: 25px; padding: 15px; background: #2d3748; border-radius: 6px; border-left: 4px solid #4CAF50; font-size: 13px; color: #e2e8f0; text-align: left;">
    <div style="font-weight: bold; margin-bottom: 10px; color: #ffffff; font-size: 14px;">Question：</div>
    <div style="margin-bottom: 15px; padding: 10px; background: rgba(255,255,255,0.05); border-radius: 4px;">
        • And what if the price fluctuates out of the pool?
    </div>
    <div style="padding: 10px; background: rgba(76, 175, 80, 0.1); border-radius: 4px; border-left: 3px solid #4CAF50;">
      No worries, there usually is an arbitrage bot that monitors price differences between the pool and the outside world, once there's a price gap for speculation, the bot immediately sells the token which went up, this trick helps keep the pool price aligned with the broader market.
    </div>
</div>

