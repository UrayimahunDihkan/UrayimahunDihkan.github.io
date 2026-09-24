---
title: Uniswap-V3 after V2
date: 2026-09-24 16:02:00
tags: tech
mathjax: true
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

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; background: #0d1117; border-radius: 16px; padding: 28px 32px; max-width: 720px; color: #e6edf3; font-size: 14px; line-height: 1.6; box-shadow: 0 2px 12px rgba(0,0,0,0.4);">
  <div style="margin-bottom: 12px; color: #e6edf3;">Price axis:</div>
  <div style="position: relative; width: 70%; height: 20px; border-bottom: 1.5px solid #e6edf3; margin-bottom: 28px;">
    <span style="position: absolute; right: -2px; bottom: -8px; font-size: 16px; line-height: 0; color: #e6edf3;">►</span>
  </div>
  <div style="position: relative; width: 70%; height: 20px; margin-bottom: 20px;">
    <span style="position: absolute; left: 0%;   transform: translateX(-50%); color: #e6edf3;">1000</span>
    <span style="position: absolute; left: 25%;  transform: translateX(-50%); color: #e6edf3;">1500</span>
    <span style="position: absolute; left: 50%;  transform: translateX(-50%); color: #e6edf3;">2000</span>
    <span style="position: absolute; left: 75%;  transform: translateX(-50%); color: #e6edf3;">2500</span>
    <span style="position: absolute; left: 100%; transform: translateX(-50%); color: #e6edf3;">3000</span>
  </div>
  <div style="position: relative; width: 70%; height: 18px; margin-bottom: 12px;">
    <span style="position: absolute; left: 25%;  transform: translateX(-50%); color: #e6edf3;">│</span>
    <span style="position: absolute; left: 50%;  transform: translateX(-50%); color: #e6edf3;">│</span>
    <span style="position: absolute; left: 75%;  transform: translateX(-50%); color: #e6edf3;">│</span>
  </div>
  <div style="position: relative; width: 70%; height: 28px; display: flex; align-items: center;">
    <span style="position: absolute; left: -68px; width: 60px; text-align: right; color: #e6edf3;">LP A:</span>
    <div style="position: absolute; left: 25%; width: 50%; display: flex; align-items: center; justify-content: space-between; color: #7ee787;">
      <span>[</span>
      <span style="flex: 1; border-top: 1.5px solid #7ee787; margin: 0 4px;"></span>
      <span>1500~2500</span>
      <span style="flex: 1; border-top: 1.5px solid #7ee787; margin: 0 4px;"></span>
      <span>]</span>
    </div>
  </div>
  <div style="position: relative; width: 70%; height: 28px; display: flex; align-items: center;">
    <span style="position: absolute; left: -68px; width: 60px; text-align: right; color: #e6edf3;">LP B:</span>
    <div style="position: absolute; left: 40%; width: 20%; display: flex; align-items: center; justify-content: center; gap: 6px; color: #79c0ff;">
      <span>[</span>
      <span>1800~2200</span>
      <span>]</span>
    </div>
  </div>
  <div style="position: relative; width: 70%; height: 28px; display: flex; align-items: center;">
    <span style="position: absolute; left: -68px; width: 60px; text-align: right; color: #e6edf3;">LP C:</span>
    <div style="position: absolute; left: 25%; width: 75%; display: flex; align-items: center; justify-content: space-between; color: #ffa657;">
      <span>[</span>
      <span style="flex: 1; border-top: 1.5px solid #ffa657; margin: 0 4px;"></span>
      <span>1500~3000</span>
      <span style="flex: 1; border-top: 1.5px solid #ffa657; margin: 0 4px;"></span>
      <span>]</span>
    </div>
  </div>
</div>

LP A,B,C, they only earn fees within their own range.

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; padding: 24px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 14px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4);">
  <table style="width: 100%; border-collapse: collapse;">
    <thead>
      <tr style="border-bottom: 1.5px solid #e6edf3;">
        <th style="text-align: center; padding: 8px 12px; font-weight: 600; color: #e6edf3;">Price range</th>
        <th style="text-align: center; padding: 8px 12px; font-weight: 600; color: #e6edf3;">Live LP</th>
        <th style="text-align: center; padding: 8px 12px; font-weight: 600; color: #e6edf3;">total liquidity</th>
      </tr>
    </thead>
    <tbody>
      <tr style="border-bottom: 1px solid #30363d;">
        <td style="padding: 8px 12px; text-align: center; color: #e6edf3;">1000~1500</td>
        <td style="padding: 8px 12px; text-align: center; color: #7ee787;">LP A</td>
        <td style="padding: 8px 12px; text-align: center; color: #e6edf3;">100</td>
      </tr>
      <tr style="border-bottom: 1px solid #30363d;">
        <td style="padding: 8px 12px; text-align: center; color: #e6edf3;">1500~1800</td>
        <td style="padding: 8px 12px; text-align: center; color: #7ee787;">LP A + LP B</td>
        <td style="padding: 8px 12px; text-align: center; color: #e6edf3;">300</td>
      </tr>
      <tr style="border-bottom: 1px solid #30363d;">
        <td style="padding: 8px 12px; text-align: center; color: #e6edf3;">1800~2000</td>
        <td style="padding: 8px 12px; text-align: center; color: #7ee787;">LP A + LP B + LP C</td>
        <td style="padding: 8px 12px; text-align: center; color: #e6edf3;">600</td>
      </tr>
      <tr style="border-bottom: 1px solid #30363d;">
        <td style="padding: 8px 12px; text-align: center; color: #e6edf3;">2000~2200</td>
        <td style="padding: 8px 12px; text-align: center; color: #7ee787;">LP B + LP C</td>
        <td style="padding: 8px 12px; text-align: center; color: #e6edf3;">500</td>
      </tr>
      <tr>
        <td style="padding: 8px 12px; text-align: center; color: #e6edf3;">2200~2500</td>
        <td style="padding: 8px 12px; text-align: center; color: #7ee787;">LP B</td>
        <td style="padding: 8px 12px; text-align: center; color: #e6edf3;">200</td>
      </tr>
    </tbody>
  </table>
</div>









---

**Tick**

- Another concept incented by V3, is Tick — price is discretized into ticks, and each tick corresponds to a specific price, during trading, the price moves between ticks. the formula is: 

$$
Price = 1.0001^{tick}
$$

​	

>- tick = 0 → price = 1
>- tick = 1 → price = 1.0001
>- tick = 100 → price = 1.0001^100 ≈ 1.01005
>- tick = -100 → price = 1.0001^-100 ≈ 0.99005
>
>But, Why 0.0001 ? 
>
>​	A 0.01% price change — that's precise enough.
>
>example:
>
>- LP wants to provide liquidity in the range [1500, 2500]
>  - Convert to ticks:
>    tickLower = tick(1500) ≈ -23028
>    tickUpper = tick(2500) ≈ 19570



- Each tick has its own liquidity

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 28px 32px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 14px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4);">
  <div style="margin-bottom: 12px; color: #e6edf3;">价格轴（从低到高）：</div>
  <div style="position: relative; height: 20px; border-bottom: 1.5px solid #e6edf3; margin-bottom: 24px;">
    <span style="position: absolute; right: -2px; bottom: -8px; font-size: 16px; line-height: 0; color: #e6edf3;">►</span>
  </div>
  <div style="display: flex; justify-content: space-around; align-items: flex-end;">
    <div style="text-align: center;">
      <div style="margin-bottom: 4px; color: #e6edf3;">tick A</div>
      <div style="color: #e6edf3;">│</div>
      <div style="color: #e6edf3;">▼</div>
      <div style="display: inline-block; width: 36px; height: 60px; border: 1.5px solid #7ee787; border-radius: 4px; margin: 6px 0;"></div>
      <div style="color: #7ee787;">100 USDC</div>
    </div>
    <div style="text-align: center;">
      <div style="margin-bottom: 4px; color: #e6edf3;">tick B</div>
      <div style="color: #e6edf3;">│</div>
      <div style="color: #e6edf3;">▼</div>
      <div style="display: inline-block; width: 36px; height: 60px; border: 1.5px solid #79c0ff; border-radius: 4px; margin: 6px 0;"></div>
      <div style="color: #79c0ff;">500 USDC</div>
    </div>
    <div style="text-align: center;">
      <div style="margin-bottom: 4px; color: #e6edf3;">tick C</div>
      <div style="color: #e6edf3;">│</div>
      <div style="color: #e6edf3;">▼</div>
      <div style="display: inline-block; width: 36px; height: 60px; border: 1.5px solid #ffa657; border-radius: 4px; margin: 6px 0;"></div>
      <div style="color: #ffa657;">200 USDC</div>
    </div>
    <div style="text-align: center;">
      <div style="margin-bottom: 4px; color: #e6edf3;">tick D</div>
      <div style="color: #e6edf3;">│</div>
      <div style="color: #e6edf3;">▼</div>
      <div style="display: inline-block; width: 36px; height: 60px; border: 1.5px solid #d2a8ff; border-radius: 4px; margin: 6px 0;"></div>
      <div style="color: #d2a8ff;">800 USDC</div>
    </div>
  </div>
</div>

<div style="margin-top: 25px; padding: 15px; background: #2d3748; border-radius: 6px; border-left: 4px solid #4CAF50; font-size: 13px; color: #e2e8f0; text-align: left;">
    <div style="font-weight: bold; margin-bottom: 10px; color: #ffffff; font-size: 14px;">Question：</div>
    <div style="margin-bottom: 15px; padding: 10px; background: rgba(255,255,255,0.05); border-radius: 4px;">
        为什么每个 tick 要单独记录流动性？
    </div>
    <div style="padding: 10px; background: rgba(76, 175, 80, 0.1); border-radius: 4px; border-left: 3px solid #4CAF50;">
      因为不同的 LP 选择的区间不同，导致在不同价格区间，活跃的流动性总量是不一样的。V3并不直接记录"每个价格区间的总流动性"，而是记录每个tick 上的"流动性变化量"（liquidityNet）
    </div>
</div>

Scenario of a real trade within V3:

1. The current price is between tick 1800 and 2000
   → Active liquidity = 600
2. Someone buys ETH, and the price moves from 1900 to 2100
   → The price crosses tick 2000 (where LP A's range ends)
   → Active liquidity = 600 - 100 = 500
   → Continue calculating with 500
3. The price keeps rising to 2300
   → It crosses tick 2200 (where LP C's range ends)
   → Active liquidity = 500 - 300 = 200

key takeaway: **each tick records the change in liquidity.**

