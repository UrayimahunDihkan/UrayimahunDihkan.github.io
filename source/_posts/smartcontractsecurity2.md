---
title: Smart contract security (II)
date: 2026-10-28 13:57:34
tags: tech
---



Security vulnerabilities are not only blame to the code itself, they're even pretty much guaranteed during wait in mempool.

- Sandwich Attack

  - A malicous bot searchs transactions those who is doing trade with the uniswap (in mempool), and before it happens, the bot gets the quote in advance and sends another transaction that can manipulate the price of pool with more gas, then the legal transaction sender does an unfavorable trade.

    <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 760px; margin: 0 auto; padding: 28px 24px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 14px; line-height: 1.6; box-shadow: 0 2px 12px rgba(0,0,0,0.4);">
      <div style="text-align: center; font-size: 18px; font-weight: bold; color: #ff7b72; margin-bottom: 8px;">🥪 Sandwich Attack</div>
      <div style="text-align: center; color: #8b949e; font-size: 12px; margin-bottom: 28px;">Mempool sniping and slippage exploitation</div>
      <div style="position: relative; padding-left: 30px; border-left: 2px solid #30363d; margin-left: 10px;">
        <div style="position: relative; margin-bottom: 24px;">
          <div style="color: #79c0ff; font-weight: bold; margin-bottom: 4px;">1.The user sends a transaction (Pending)</div>
          <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 8px;">A user wants to buy ETH with 1000 USDC, and the transaction enters the mempool.(mempool)</div>
          <div style="background: #161b22; border: 1px solid #30363d; border-radius: 8px; padding: 12px;">
            <div style="color: #8b949e; font-size: 11px; margin-bottom: 4px;">Mempool</div>
            <div style="color: #e6edf3;">👤 User ➔ <span style="color: #79c0ff;">Swap 1000 USDC for ETH</span> <span style="color: #8b949e;">(Gas: 20 Gwei)</span></div>
          </div>
        </div>
        <div style="position: relative; margin-bottom: 24px;">
          <div style="color: #ff7b72; font-weight: bold; margin-bottom: 4px;">2.Bots front-run the transaction (Front-run)</div>
          <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 8px;">The bot detects the transaction and immediately pays a higher gas fee to cut in line and buy a large amount of ETH, driving up the price.</div>
          <div style="background: #161b22; border: 1px solid #ff7b72; border-radius: 8px; padding: 12px; box-shadow: 0 0 8px rgba(255,123,114,0.2);">
            <div style="color: #ff7b72; font-size: 11px; margin-bottom: 4px;">Block N</div>
            <div style="color: #e6edf3;">🤖 Bot ➔ <span style="color: #ff7b72;">Buy 5000 USDC worth of ETH</span> <span style="color: #8b949e;">(Gas: 50 Gwei ⚡)</span></div>
            <div style="color: #ffa657; font-size: 12px; margin-top: 4px;">📈 As a result: the ETH in the pool decreases, and the price is driven up.</div>
          </div>
        </div>
        <div style="position: relative; margin-bottom: 24px;">
          <div style="color: #79c0ff; font-weight: bold; margin-bottom: 4px;">3.The user ends up executing at a worse price (Execution)</div>
          <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 8px;">The user's transaction gets packed right after, but by then the pool price has already been driven up, so the user gets filled at an extremely bad price (Take a huge slippage hit)</div>
          <div style="background: #161b22; border: 1px solid #30363d; border-radius: 8px; padding: 12px; border-left: 3px solid #ffa657;">
            <div style="color: #8b949e; font-size: 11px; margin-bottom: 4px;">Block N+1</div>
            <div style="color: #e6edf3;">👤 User ➔ <span style="color: #ffa657;">Swap 1000 USDC for ETH</span> <span style="color: #8b949e;">(Gas: 20 Gwei)</span></div>
            <div style="color: #ff7b72; font-size: 12px; margin-top: 4px;">📉 The result: they end up with very little ETH and take a massive hit.</div>
          </div>
        </div>
        <div style="position: relative;">
          <div style="color: #7ee787; font-weight: bold; margin-bottom: 4px;">4.The bot back-runs the transaction (Back-run)</div>
          <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 8px;">The bot immediately sells the ETH it just bought, captures the price difference, and completes a risk-free arbitrage</div>
          <div style="background: #161b22; border: 1px solid #7ee787; border-radius: 8px; padding: 12px; box-shadow: 0 0 8px rgba(126,231,135,0.1);">
            <div style="color: #7ee787; font-size: 11px; margin-bottom: 4px;">Block N+2</div>
            <div style="color: #e6edf3;">🤖 Bot ➔ <span style="color: #7ee787;">Sell ETH for USDC</span></div>
            <div style="color: #7ee787; font-size: 12px; margin-top: 4px;">💰 As a result, it sells at a high price and pockets the spread created by the user's slippage.</div>
          </div>
        </div>
      </div>
    </div>

- Transaction Front Running Attack

  - A malicous bot searchs a profitable transaction in the mempool, and submit the transaction again with higher gas fee , that intercepts and steals your profits

    - for example, you got an arbitraging chance , utilizing a price gap between DEX_A and DEX_B , buy from one and sell to another , you can make a benefit. but when your transaction waiting in the mempool, malicious bot will copy the transaction and re-send with a higher gas price , so that steal your benefit.

    <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 760px; margin: 0 auto; padding: 28px 24px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 14px; line-height: 1.6; box-shadow: 0 2px 12px rgba(0,0,0,0.4);">
      <div style="text-align: center; font-size: 18px; font-weight: bold; color: #ff7b72; margin-bottom: 8px;">🏃 Transaction Front-Running</div>
      <div style="text-align: center; color: #8b949e; font-size: 12px; margin-bottom: 28px;">Front-run using gas fees to steal arbitrage profits</div>
      <div style="position: relative; padding-left: 30px; border-left: 2px solid #30363d; margin-left: 10px;">
        <div style="position: relative; margin-bottom: 24px;">
          <div style="color: #79c0ff; font-weight: bold; margin-bottom: 4px;">1.Identify an arbitrage opportunity</div>
          <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 8px;">You notice a price gap between DEX A and DEX B. By buying low and selling high, you can capture a risk-free arbitrage.</div>
          <div style="background: #161b22; border: 1px solid #30363d; border-radius: 8px; padding: 12px;">
            <div style="color: #8b949e; font-size: 11px; margin-bottom: 4px;">Off-chain calculation</div>
            <div style="color: #e6edf3;">👤 You ➔ <span style="color: #79c0ff;">DEX A buy ➔ DEX B sell</span> <span style="color: #8b949e;">(expected profit: +100 USDC)</span></div>
          </div>
        </div>
        <div style="position: relative; margin-bottom: 24px;">
          <div style="color: #79c0ff; font-weight: bold; margin-bottom: 4px;">2.Send transaction (Pending)</div>
          <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 8px;">You submit a transaction, and now it's sitting in the node's mempool, waiting to be packaged into a block</div>
          <div style="background: #161b22; border: 1px solid #30363d; border-radius: 8px; padding: 12px;">
            <div style="color: #8b949e; font-size: 11px; margin-bottom: 4px;">Mempool</div>
            <div style="color: #e6edf3;">👤 You ➔ <span style="color: #79c0ff;">Arbitrage TX</span> <span style="color: #8b949e;">(Gas: 20 Gwei)</span></div>
          </div>
        </div>
        <div style="position: relative; margin-bottom: 24px;">
          <div style="color: #ff7b72; font-weight: bold; margin-bottom: 4px;">3.The bot sniffs it out and front-runs it (Front-run)</div>
          <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 8px;">The bot spots your transaction, replicates your logic, and pays a massive gas fee to front-run you.</div>
          <div style="background: #161b22; border: 1px solid #ff7b72; border-radius: 8px; padding: 12px; box-shadow: 0 0 8px rgba(255,123,114,0.2);">
            <div style="color: #ff7b72; font-size: 11px; margin-bottom: 4px;">Block N (It gets packed into the block before yours)</div>
            <div style="color: #e6edf3;">🤖 Bot ➔ <span style="color: #ff7b72;">Copy & Paste TX</span> <span style="color: #8b949e;">(Gas: 100 Gwei ⚡)</span></div>
            <div style="color: #ffa657; font-size: 12px; margin-top: 4px;">💸 As a result, the bot eats the spread and takes the 100 USDC profit.</div>
          </div>
        </div>
        <div style="position: relative;">
          <div style="color: #8b949e; font-weight: bold; margin-bottom: 4px;">4.The profit has been front-run and stolen(Execution)</div>
          <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 8px;">Your transaction is then packed into a block, but since the profit margin has already been squeezed dry by the bot, it either turns out to be unprofitable (and fails), or you even end up losing the gas fee.</div>
          <div style="background: #161b22; border: 1px solid #30363d; border-radius: 8px; padding: 12px; border-left: 3px solid #8b949e;">
            <div style="color: #8b949e; font-size: 11px; margin-bottom: 4px;">Block N+1 (included in a subsequent block)</div>
            <div style="color: #e6edf3;">👤 You ➔ <span style="color: #8b949e;">Arbitrage TX Reverted / No Profit</span></div>
            <div style="color: #8b949e; font-size: 12px; margin-top: 4px;">😭 Result: You not only make zero profit, but you also end up paying the gas fee for nothing.</div>
          </div>
        </div>
      </div>
    </div>
  
  - How to prevent? 
  
    1. The most commonly used one is Flashbots Protect RPC. You can just customize RPC to [https://rpc.flashbots.net](https://rpc.flashbots.net/) in your wallet's network settings. these nodes runs with private mempool.
  
    2. Don't let your transaction look like a sitting duck in the mempool
  
       - Control the slippage that you can accept, don't set it too high
  
       - Set proxy to the uniswap call, so that the bot only see you're calling an obscure contract. (personally , I recommend this trick)
  
    

I know this much so far , and continuesly focusing security on web3. I also prefer to prectical implementation of guard against these attacks , will share.
