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
