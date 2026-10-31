---
title: This is how flash loan arbitrages
date: 2026-10-18 13:53:43
tags: tech
---

At first, a non-developer intuitively thought of a complete contract transaction as a single fast 'action', but indeed a contract  "transaction" actually wraps many "internal actions", sometimes they're hundreds. They're all executed synchronously , one by one , in a single thread, before the transaction finishes. 

That's exactly what a flash loan is: doing multiple actions in sequence within one transaction to carry out a speculative trade.

When you hit "confirm" in your wallet and send a transaction, you're not just submitting a simple action to Ethereum — you're submitting an entire program execution.

---

Now, with this in mind , here's the break down of flash loan's entire program:

1. Call Aave's `flashLoan()` function to borrow funds.
2. Once the funds arrive, Aave triggers the callback function in your contract (where you write your arbitrage logic).
3. Your contract buys on Uniswap.
4. Your contract sells on SushiSwap.
5. Your contract transfers the borrowed amount plus the fee back to Aave.
6. The transaction ends, and the final state is settled (packaged into a block).

>In traditional finance, if you borrow $10 million with 10x leverage and lose $1 million after trading, and you can't pay it back, the bank has to sue you and freeze your assets. 

**But in blockchain smart contracts, there's only one fallback mechanism for all of this: revert.**

Powerful undo button.

Here is break down of lose money:

1. Your contract sells on SushiSwap.
2. It turns out , didn't just make no money, you actually lost 100k.
3. Program execute to the last step: `require({Repayment amount > 10million},"...");`
4. You only have 9.9million left, `require` fails and triggers **revert**.
5. Miracle happens: it rolls back everything that happened in the transaction, 10million eventually gets back to Aave. Example of revert:
   - Your account initially had 0 USDC
   - Aave pool's total amount: 10million USDC
   - After losing money and require fails , atomicly reverts to original states: Aave(1000USDC) & your account(0USDC).
   - The only loss is the gas fee.

---

I'd call it money printing machine.

<br>

<br>

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 20px 0; color: #8b949e; font-size: 13px; border-bottom: 1px solid #30363d; padding-bottom: 8px;">
  Updated on <span style="color: #e6edf3; font-weight: 600;">Oct 31, 2026</span>
</div>

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 820px; margin: 0 auto; padding: 28px 24px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 14px; line-height: 1.6; box-shadow: 0 2px 12px rgba(0,0,0,0.4);">
  <div style="text-align: center; font-size: 18px; font-weight: bold; color: #79c0ff; margin-bottom: 8px;">⚡ Flash Loan Arbitrage: The Magic of Atomicity</div>
  <div style="text-align: center; color: #8b949e; font-size: 12px; margin-bottom: 28px;">Multiple actions in one transaction, with a powerful "undo" button.</div>
  <div style="background: #161b22; border: 1px solid #30363d; border-radius: 12px; padding: 20px; margin-bottom: 24px;">
    <div style="color: #e6edf3; font-weight: bold; margin-bottom: 12px;">📦 What is a "Transaction"?</div>
    <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 12px;">
      When you click "confirm", you are not just sending a single action. You are submitting an <strong>entire program execution</strong>. All internal actions execute synchronously, in a single thread, before the transaction finishes.
    </div>
    <div style="background: #0d1117; border-left: 3px solid #79c0ff; padding: 10px 14px; color: #8b949e; font-size: 13px; border-radius: 0 4px 4px 0;">
      This is exactly what a Flash Loan is: doing multiple actions in sequence within ONE transaction to carry out a speculative trade.
    </div>
  </div>
  <div style="display: flex; gap: 16px; margin-bottom: 24px; flex-wrap: wrap;">
    <div style="flex: 1; min-width: 300px; background: #161b22; border: 1px solid #30363d; border-top: 3px solid #7ee787; border-radius: 8px; padding: 20px;">
      <div style="color: #7ee787; font-weight: bold; margin-bottom: 16px; display: flex; align-items: center;">
        <span style="background: #7ee787; color: #0d1117; border-radius: 50%; width: 20px; height: 20px; display: inline-flex; align-items: center; justify-content: center; margin-right: 8px; font-size: 12px;">✓</span>
        Successful Arbitrage
      </div>
      <div style="position: relative; padding-left: 16px; border-left: 1px solid #30363d;">
        <div style="margin-bottom: 12px; position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #7ee787;"></div>
          <span style="color: #e6edf3;">1. Call Aave <code>flashLoan()</code></span>
        </div>
        <div style="margin-bottom: 12px; position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #7ee787;"></div>
          <span style="color: #e6edf3;">2. Receive funds, trigger callback</span>
        </div>
        <div style="margin-bottom: 12px; position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #7ee787;"></div>
          <span style="color: #e6edf3;">3. Buy on Uniswap</span>
        </div>
        <div style="margin-bottom: 12px; position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #7ee787;"></div>
          <span style="color: #e6edf3;">4. Sell on SushiSwap</span>
        </div>
        <div style="margin-bottom: 12px; position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #7ee787;"></div>
          <span style="color: #e6edf3;">5. Repay Aave (Principal + Fee)</span>
        </div>
        <div style="position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #7ee787;"></div>
          <span style="color: #7ee787; font-weight: bold;">6. Keep the profit!</span>
        </div>
      </div>
    </div>
    <div style="flex: 1; min-width: 300px; background: #161b22; border: 1px solid #30363d; border-top: 3px solid #ff7b72; border-radius: 8px; padding: 20px;">
      <div style="color: #ff7b72; font-weight: bold; margin-bottom: 16px; display: flex; align-items: center;">
        <span style="background: #ff7b72; color: #0d1117; border-radius: 50%; width: 20px; height: 20px; display: inline-flex; align-items: center; justify-content: center; margin-right: 8px; font-size: 12px;">✗</span>
        Trade Loss (Revert)
      </div>
      <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 16px;">
        What if you borrow $10M, but lose $100k due to bad market conditions?
      </div>
      <div style="position: relative; padding-left: 16px; border-left: 1px solid #30363d;">
        <div style="margin-bottom: 12px; position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #8b949e;"></div>
          <span style="color: #8b949e;">1. Buy on Uniswap</span>
        </div>
        <div style="margin-bottom: 12px; position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #8b949e;"></div>
          <span style="color: #8b949e;">2. Sell on SushiSwap (Loses $100k)</span>
        </div>
        <div style="margin-bottom: 12px; position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #ff7b72;"></div>
          <span style="color: #ff7b72;">3. Contract only has $9.9M left</span>
        </div>
        <div style="margin-bottom: 12px; position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #ff7b72;"></div>
          <span style="color: #ff7b72;">4. <code>require(repay &gt;= 10M)</code> fails!</span>
        </div>
        <div style="position: relative;">
          <div style="position: absolute; left: -21px; top: 4px; width: 8px; height: 8px; border-radius: 50%; background: #ff7b72;"></div>
          <span style="color: #ff7b72; font-weight: bold;">5. Trigger <code>revert</code> (Undo)</span>
        </div>
      </div>
    </div>
  </div>
  <div style="background: rgba(126,231,135,0.1); border: 1px solid #7ee787; border-radius: 12px; padding: 20px; box-shadow: 0 0 15px rgba(126,231,135,0.05);">
    <div style="color: #7ee787; font-weight: bold; margin-bottom: 12px; font-size: 15px;">🪄 The Powerful Undo Button (Atomicity)</div>
    <div style="color: #c9d1d9; font-size: 13px; line-height: 1.7; margin-bottom: 16px;">
      In traditional finance, a failed trade means bankruptcy and lawsuits. In blockchain, it just triggers a <code>revert</code>.
    </div>
    <div style="display: flex; align-items: center; justify-content: space-around; background: #0d1117; border: 1px dashed #30363d; border-radius: 8px; padding: 16px;">
      <div style="text-align: center;">
        <div style="color: #8b949e; font-size: 12px; margin-bottom: 4px;">Aave Pool</div>
        <div style="color: #e6edf3; font-weight: bold;">10M USDC</div>
      </div>
      <div style="color: #ff7b72; font-weight: bold; font-size: 20px;">➔</div>
      <div style="text-align: center;">
        <div style="color: #ff7b72; font-size: 12px; margin-bottom: 4px;">Loss</div>
        <div style="color: #ff7b72; font-weight: bold;">-$100k</div>
      </div>
      <div style="color: #79c0ff; font-weight: bold; font-size: 20px;">➔</div>
      <div style="text-align: center;">
        <div style="color: #7ee787; font-size: 12px; margin-bottom: 4px;">After Revert</div>
        <div style="color: #7ee787; font-weight: bold;">10M USDC <br><span style="font-size:11px;font-weight:normal;">(State Restored)</span></div>
      </div>
    </div>
    <div style="text-align: center; margin-top: 16px; color: #8b949e; font-size: 12px;">
      The only loss is the <strong>Gas fee</strong>. Everything else is magically rolled back to the original state.
    </div>
  </div>
</div>
