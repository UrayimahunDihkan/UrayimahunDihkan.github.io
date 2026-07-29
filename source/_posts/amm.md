---
title: How AMM module works? [x*y=k]
date: 2026-07-20 19:07:51
tags: tech
---

All swapping pools between two currncies mostly base on the **x·y=k** equation. Usually, x: number of TokenA, y: number of TokenB. (Initial number of TokenA and TokenB are based on the exact price when the pool was created). 

Here are two senators for buy and sell,

1. User sells ETH for getting USDC:  (x+dx)·(y-dy)=k
2. User sells USDC for getting ETH:  (x-dx)·(y+dy)=k

Let's assume there is a pool, and there are 10ETH and 20,000USDC in it, then the k=10*20,000=200,000

1. User sells 5 ETH for getting USDC:  (10+5)·(20,000-dy)=200,000  dy=6666.67USDC

2. User sells 10,000USDC for getting ETH: (10-dx)·(20000+10,000)=200,000  dx=3.33ETH

3. What if I am trying to sell 10,000ETH for USDC? Calculate it, for fun ^^

   (10+10,000)·(20000-dy)=200,000 dy=19998USDC

>- Sell 5ETH get 6666.67USDC , then 6666.67/5 = 1333.33 USDC/ETH
>
>- Sell 10,000ETH get 19998USDC , then 19998/10,000 = 2 USDC/ETH
>
>  What is this?!! So, The bigger you try to swap , the worse the value you get. But don't worry, usually for a pool that size, an everage ordinary trade barely moves the needle.

<div style="margin-top: 25px; padding: 15px; background: #2d3748; border-radius: 6px; border-left: 4px solid #4CAF50; font-size: 13px; color: #e2e8f0; text-align: left;">
    <div style="font-weight: bold; margin-bottom: 10px; color: #ffffff; font-size: 14px;">Question：</div>
    <div style="margin-bottom: 15px; padding: 10px; background: rgba(255,255,255,0.05); border-radius: 4px;">
        • And what if the price fluctuates out of the pool?
    </div>
    <div style="padding: 10px; background: rgba(76, 175, 80, 0.1); border-radius: 4px; border-left: 3px solid #4CAF50;">
      No worries, there usually is an arbitrage bot that monitors price differences between the pool and the outside world, once there's a price gap for speculation, the bot immediately sells the token which went up, this trick helps keep the pool price aligned with the broader market.
    </div>
</div>


---

**Concepts**


- **Slippage**: this is a concept about how much difference are there between expected price and buy-in price. 

  - buy-in price: 1333.33USDC/ETH (just the value we calculated base on x*y=k law)

  - expected price: current market price that we ususally query from chain network, assume it 1340

    - Slippage = ({buy-in price} - {expected price} ) / {expected price} * 100%

    >- Slippage for 1st bought (1333.33USDC/ETH) :  (1333.33-1340)/1340 * 100% = -0.5% 
    >- Slippage for 2nd bought (2USDC/ETH) : (2-1340)/1340 * 100%=-99.9% 

  Slippage is a key matric that tells you how cost-effective the transaction is.

  

- **Liquidity**

  x·y=k, Liquidity=√k

- **Liquidity provider (LP)**

  Sometimes you may hear someone says "I earned a lot doing LP" , it means being a liquidity provider putting your tokens in pool for lending them to borrowers , you'll get interest.

  - Add liquidity 
    - First add: tokens' price are got from chain-net
    -  After first add: tokens' price are not got from chain-net, price keep with equation  **(x+dy)/(y+dy)=x/y**

  - Excess liquidity 
    - tokens' price are not got from chain-net, price keep with equation  **(x-dy)/(y-dy)=x/y**

  - > No matter add or excess, the key principle is to don't affect the price in pool. that's why the equations are like  **(xdy)/(y+dy)=x/y**, **(x-dy)/(y-dy)=x/y**



---

**Advice for investors**

1. Check the real‑time slippage first, then decide whether to trade	

2. If you're a whale, choose a larger pool

