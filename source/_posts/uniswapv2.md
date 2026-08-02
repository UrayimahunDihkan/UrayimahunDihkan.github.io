---
title: How the fee calculated in uniswap-v2?
date: 2026-07-30 16:37:36
tags: tech
---



Download uniswap whitepaper: https://app.uniswap.org/whitepaper.pdf

it's not very friendly to read, I learned so hard, then this is my summaries.



Origin AMM formula:

```math
(x_0+dx)(y_0-dy)=k=x_0y_0
=>dy=\frac{dx·y_0}{x_0+dx}
```

Assume the fee is 0.3% (in uniswapv-v2, this is a fixed fee), dx is the token you're trying to exchange, the fee will be charged on this dx, so we just change the dx of the formula above like list:

```math
=>[x_0+(1-0.003)dx][y_0-dy]=k=x_0y_0
=>dy=\frac{[(1-0.003)dx]·y_0}{x_0+[(1-0.003)dx]}
```

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
     |   ███     ███
     |   ███     ███
     |   ███     ███
     |   ███     ███
     +--+---+---+---+---> token
         USDT    DAI    
  ```

- After some swaps, USDT got 104, DAI got 98.

  ```ceylon
  Liquidity(√k)
     ^
     |   104
     |   ███     98
     |   ███     ███
     |   ███     ███
     |   ███     ███
     +--+---+---+---+---> token
         USDT    DAI    
  ```

- But immediately, they return back a balanced position. Bcz there usually are arbitrage bots are keep detecting whether there are an imbalance that can profit from , which brings the pool back to balance.

  ```ceylon
  Liquidity(√k)
     ^
     |   101     101
     |   ███     ███
     |   ███     ███
     |   ███     ███
     |   ███     ███
     +--+---+---+---+---> token
         USDT    DAI    
  ```

- Can easily see they've increased, say again: once the fee is taken out, it flows into the pool, not to a specific account, so the liquidity will increase as swaps accumulate.

