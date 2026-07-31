---
title: How the fee calculated in uniswap-v2?
date: 2026-07-30 16:37:36
tags: tech
---

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

