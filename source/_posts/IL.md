---
title: Impermanent Loss (IL)
date: 2026-08-09 17:00:37
tags: tech
---

Once we do LP, impermanent loss (LP) is pretty much guaranteed, it's not a matter of if, but how much. 

There's a vivid breakdown for much better understanding:

Scenerio 1:

- Suppose we got a pool with initial liquidity of 100DAI and 1ETH 
  - now the `LP's position value` is 200DAI (100DAI+1ETH)
- ETH goes up
  - The bot keeps arbitraging until the price gap disappears, as a result, the amount of ETH in the pool had dropped from 1 to 0.83
    - x·y=k,  100·1=120*0.83
    - PriceOfEth=120/0.83=144.58
  - Now, `LP's position value` got
    - 120DAI + 0.83* 144.58DAI= 240DAI
    - we earned 240DAI-200DAI=40DAI
    - But, the position of not being LP's value is 4DAI more:
      - 100DAI + 1*144.58DAI=244.58DAI

**Conclusion:**

>If ETH rises, it's profitable but you don't capture the full gain.

---



Scenario 2:

- Suppose we got a pool with initial liquidity of 100DAI and 1ETH 
  - now the `LP's position value` is 200DAI (100DAI+1ETH)
- ETH drops
  - The bot keeps arbitraging until the price gap disappears, as a result, the amount of ETH in the pool had risen from 1 to 1.25
    - x·y=k,  100·1=80*1.25
    - PriceOfEth=80/1.25=64
  - Now, `LP's position value` got:
    - 80DAI + 1.25*64DAI=160DAI
    - we lost 200DAI-160DAI=40DAI
    - But, the position of not being LP's value:
      - 100DAI + 64*1DAI = 164DAI

**Conclusion:**

>If ETH drops, we all lose money no matter whether you're an LP or not, LP lose a bit more.

---

