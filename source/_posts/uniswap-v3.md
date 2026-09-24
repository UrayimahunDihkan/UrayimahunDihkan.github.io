---
title: Uniswap-V3 after V2
date: 2026-09-24 16:02:00
tags: tech
---

**Suppose** the real-time price of the ETH/DAI trading pair is 1500 DAI/ETH. 

Initial state: The pool holds 4,500 DAI and 3 ETH, with a price of 1,500 DAI/ETH.

After a price drop: ETH falls to 1,300 DAI/ETH, and arbitrageurs will buy ETH from the pool until the pool price also reaches 1,300.

Final state: The pool ends up with 4,192.54 DAI and 3.22 ETH.

> - The DAI balance in the pool decreased by 6.84%.
>
> - The ETH balance in the pool increased — from 3 to 3.22.
>
> If you're an LP and you put your money in, then when ETH's price drops, the pool will automatically "sell" some DAI to buy ETH. This process changes the composition of the assets you hold, and it usually results in **[impermanent loss](https://urayimahundihkan.github.io/2026/08/09/IL/)**.



[ Uniswap-V3 ]

**Uniswap V3 introduced concentrated liquidity, letting LPs choose their own price range. This gives them more control over their impermanent loss exposure — a narrower range means less IL risk, but requires more rebalancing.**

Scenerio:

**1. Create the pool (anyone can do this)**

- Pair: ETH/USDC
- Fee tier: 0.3%
- The pool starts with no liquidity

**2. LP A adds liquidity**

- Picks the range [1500, 2500]
- Deposits 1000 USDC + 0.5 ETH
- Gets an ERC721

**3. LP B adds liquidity**

- Picks a narrower range: [1800, 2200]
- Deposits 500 USDC + 0.25 ETH
- Gets another ERC721

**4. LP C adds liquidity**

- Picks a wider range: [1500, 3000]
- Deposits 2000 USDC + 1 ETH
- Gets a third ERC721

```
Price axis：
────────────────────────────────────────────────────────►
1000        1500          2000          2500        3000
             │             │             │
LP A:        [──────────1500~2500────────]
LP B:                 [1800~2200]
LP C:        [───────────1500~3000────────────────────]
```

LP A,B,C, they only earn fees within their own range.





