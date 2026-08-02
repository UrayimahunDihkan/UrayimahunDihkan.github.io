---
title: How the fee calculated in uniswap-v2?
date: 2026-07-30 16:37:36
mathjax: true
tags: tech
---



$$ E =$2.72² 

Download uniswap whitepaper: https://app.uniswap.org/whitepaper.pdf

it's not very friendly to read, I learned so hard, then this is my summaries.



Origin AMM formula:


$$
(x_0 + \mathrm{d}x)(y_0 - \mathrm{d}y) = k = x_0 y_0
\Longrightarrow \mathrm{d}y = \frac{\mathrm{d}x \cdot y_0}{x_0 + \mathrm{d}x}
$$
Assume the fee is 0.3% (in uniswapv-v2, this is a fixed fee), dx is the token you're trying to exchange, the fee will be charged on this dx, so we just change the dx of the formula above like list:
$$
[x_0 + (1 - 0.003)dx][y_0 - dy] = k = x_0 y_0
\Rightarrow dy = \frac{[(1 - 0.003)dx] \cdot y_0}{x_0 + [(1 - 0.003)dx]}
$$
this is the dy we correctly will get. And, this formula actually is what the `getAmountOut` function does, look:

```sol
function getAmountOut(...) {
	...
	uint amountInWithFee = amountIn.mul(997);  //1-0.003=0.997
	...
}
```





Once the fee is taken out, it flows into the pool, not to a specific account, so the liquidity will increase as swaps accumulate. 

Here is the breakdown of the flow.

- Here is pool for USDT/DAI, initially their liquidity are: (100USDT and 100 DAI, they're woth same)

  ```ceylon
  Liquidity(√k)
     ^
     |   100	   100
     |   ██      ██
     |   ██      ██
     |   ██      ██
     |   ██      ██
     +--+---+---+---+---> token
         USDT    DAI    
  ```

- After some swaps, USDT got 104, DAI got 98.

  ```ceylon
  Liquidity(√k)
     ^
     |   104
     |   ██      98
     |   ██      ██
     |   ██      ██
     |   ██      ██
     +--+---+---+---+---> token
         USDT    DAI    
  ```

- But immediately, they return back a balanced position. Bcz there usually are arbitrage bots are keep detecting whether there are an imbalance that can profit from , which brings the pool back to balance.

  ```ceylon
  Liquidity(√k)
     ^
     |   101     101
     |   ██      ██
     |   ██      ██
     |   ██      ██
     |   ██      ██
     +--+---+---+---+---> token
         USDT    DAI    
  ```

- Can easily see they've increased, say again: once the fee is taken out, it flows into the pool, not to a specific account, so the liquidity will increase as swaps accumulate.

  ​	pool liquidity

  - <div style="display:flex; align-items:flex-end; height:230px; width: 350px; gap:50px; padding-left: 30px; border-left:2px solid #333; border-bottom:2px solid #333; background:rgba(33, 150, 243, 0.1);">
      <div style="text-align:center;">
        <div style="width:60px; height:100px; background:#2196F3; border-radius:4px 4px 0 0;"></div>
      </div>
    	<div style="text-align:center;">→</div>
      <div style="text-align:center;">
        <div style="width:60px; height:50px; background:#FF9800; border:2px dashed #ffffff; border-radius:4px 4px 0 0;">
        	<span style="font-size: 10px;">increased liquidity</span>
        </div>
        <div style="width:60px; height:100px; background:#FF9800; border-radius:0;"></div>
      </div>
    </div>

    The growth in liquidity , that's exactly where LPs earn, as the pool growth , so do their profits.

    But, where does the project team who writes contracts benefit from?

    - in uniswap-v2's code project team doesn't charge fee for themselves by default. 

  





