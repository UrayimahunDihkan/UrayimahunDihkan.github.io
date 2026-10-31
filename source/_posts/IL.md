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





<br>

<br>

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 20px 0; color: #8b949e; font-size: 13px; border-bottom: 1px solid #30363d; padding-bottom: 8px;">
  Updated on <span style="color: #e6edf3; font-weight: 600;">Oct 31, 2026</span>
</div>

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 860px; margin: 0 auto; padding: 40px 24px; background: linear-gradient(135deg, #090a0f 0%, #1b223c 100%); border-radius: 20px; color: #e6edf3; font-size: 14px; line-height: 1.6; box-shadow: inset 0 0 0 1px rgba(255,255,255,0.05);">
  <div style="text-align: center; margin-bottom: 32px;">
    <h1 style="color: #e6edf3; margin: 0 0 8px 0; font-size: 24px; text-shadow: 0 2px 4px rgba(0,0,0,0.4);">💧 Impermanent Loss (IL)</h1>
    <div style="color: #a1aab5; font-size: 13px;">Once we do LP, it's not a matter of IF, but HOW MUCH.</div>
  </div>
  <div style="display: flex; gap: 20px; flex-wrap: wrap; margin-bottom: 24px;">
    <div style="flex: 1; min-width: 320px; background: rgba(255, 255, 255, 0.03); border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 16px; padding: 24px; backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px); box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.3);">
      <div style="display: inline-block; background: rgba(126, 231, 135, 0.15); border: 1px solid rgba(126, 231, 135, 0.3); color: #7ee787; padding: 4px 10px; border-radius: 20px; font-size: 12px; font-weight: bold; margin-bottom: 16px;">
        Scenario 1: ETH Goes UP 📈
      </div>
      <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 16px;">
        <strong>Initial Pool:</strong> 100 DAI + 1 ETH = 200 DAI<br>
        Arbitrageurs step in as ETH price rises. Pool becomes <strong>120 DAI + 0.83 ETH</strong>.
      </div>
      <div style="background: rgba(0,0,0,0.2); border-radius: 8px; padding: 12px; font-size: 12px; color: #a1aab5; margin-bottom: 16px;">
        <div style="margin-bottom: 4px;">Formula: <span style="color: #79c0ff;">x · y = k</span></div>
        <div>100 · 1 = 120 · 0.83</div>
        <div style="margin-top: 4px;">New Price = 120 / 0.83 = <span style="color: #e3b341;">144.58 DAI</span></div>
      </div>
      <div style="border-top: 1px dashed rgba(255,255,255,0.1); padding-top: 16px; margin-bottom: 12px;">
        <div style="display: flex; justify-content: space-between; margin-bottom: 8px;">
          <span style="color: #8b949e;">LP Position Value:</span>
          <span style="color: #e6edf3;">120 + (0.83 × 144.58) = <strong>240 DAI</strong></span>
        </div>
        <div style="display: flex; justify-content: space-between; margin-bottom: 8px;">
          <span style="color: #8b949e;">Not being LP:</span>
          <span style="color: #79c0ff;">100 + (1 × 144.58) = <strong>244.58 DAI</strong></span>
        </div>
      </div>
      <div style="background: rgba(255, 123, 114, 0.1); border-left: 3px solid #ff7b72; padding: 10px; border-radius: 0 6px 6px 0; font-size: 12px; color: #ff7b72;">
        <strong>Conclusion:</strong> Profitable (+40 DAI), but you underperform HODL by 4.58 DAI.
      </div>
    </div>
    <div style="flex: 1; min-width: 320px; background: rgba(255, 255, 255, 0.03); border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 16px; padding: 24px; backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px); box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.3);">
      <div style="display: inline-block; background: rgba(255, 123, 114, 0.15); border: 1px solid rgba(255, 123, 114, 0.3); color: #ff7b72; padding: 4px 10px; border-radius: 20px; font-size: 12px; font-weight: bold; margin-bottom: 16px;">
        Scenario 2: ETH Goes DOWN 📉
      </div>
      <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 16px;">
        <strong>Initial Pool:</strong> 100 DAI + 1 ETH = 200 DAI<br>
        Arbitrageurs step in as ETH price drops. Pool becomes <strong>80 DAI + 1.25 ETH</strong>.
      </div>
      <div style="background: rgba(0,0,0,0.2); border-radius: 8px; padding: 12px; font-size: 12px; color: #a1aab5; margin-bottom: 16px;">
        <div style="margin-bottom: 4px;">Formula: <span style="color: #79c0ff;">x · y = k</span></div>
        <div>100 · 1 = 80 · 1.25</div>
        <div style="margin-top: 4px;">New Price = 80 / 1.25 = <span style="color: #e3b341;">64 DAI</span></div>
      </div>
      <div style="border-top: 1px dashed rgba(255,255,255,0.1); padding-top: 16px; margin-bottom: 12px;">
        <div style="display: flex; justify-content: space-between; margin-bottom: 8px;">
          <span style="color: #8b949e;">LP Position Value:</span>
          <span style="color: #e6edf3;">80 + (1.25 × 64) = <strong>160 DAI</strong></span>
        </div>
        <div style="display: flex; justify-content: space-between; margin-bottom: 8px;">
          <span style="color: #8b949e;">Not being LP:</span>
          <span style="color: #79c0ff;">100 + (1 × 64) = <strong>164 DAI</strong></span>
        </div>
      </div>
      <div style="background: rgba(255, 123, 114, 0.1); border-left: 3px solid #ff7b72; padding: 10px; border-radius: 0 6px 6px 0; font-size: 12px; color: #ff7b72;">
        <strong>Conclusion:</strong> Everyone loses (-40 DAI), but LP loses an extra 4 DAI compared to HODL.
      </div>
    </div>
  </div>
  <div style="background: rgba(210, 168, 255, 0.05); border: 1px solid rgba(210, 168, 255, 0.2); border-radius: 12px; padding: 16px 20px; backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px);">
    <div style="color: #d2a8ff; font-weight: bold; font-size: 13px; margin-bottom: 4px;">🎯 The Core Takeaway</div>
    <div style="color: #a1aab5; font-size: 13px;">
      Impermanent Loss is the difference between holding your tokens in a wallet vs. providing them to a liquidity pool. It happens because the AMM automatically rebalances the pool against the market price. The bigger the price change, the bigger the loss relative to simply HODLing.
    </div>
  </div>
</div>

