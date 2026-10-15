---
title: Smart contract security (I)
date: 2026-10-15 14:19:39
tags: tech
---



Let's review this piece of code and see what vulnerabilities might be hiding in there.

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
<pre style="margin: 0;"><code><span style="color: #8b949e;">// SPDX-License-Identifier: MIT</span>
<span style="color: #ff7b72;">pragma</span> solidity <span style="color: #79c0ff;">^0.8.0</span>;
<span style="color: #ff7b72;">interface</span> <span style="color: #ffa657;">IERC20</span> {
    <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">approve</span>(<span style="color: #ff7b72;">address</span> spender, <span style="color: #ffa657;">uint256</span> amount) <span style="color: #ff7b72;">external</span> <span style="color: #ff7b72;">returns</span> (<span style="color: #ffa657;">bool</span>);
    <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">transferFrom</span>(<span style="color: #ff7b72;">address</span> from, <span style="color: #ff7b72;">address</span> to, <span style="color: #ffa657;">uint256</span> amount) <span style="color: #ff7b72;">external</span> <span style="color: #ff7b72;">returns</span> (<span style="color: #ffa657;">bool</span>);
    <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">transfer</span>(<span style="color: #ff7b72;">address</span> recipient, <span style="color: #ffa657;">uint256</span> amount) <span style="color: #ff7b72;">external</span> <span style="color: #ff7b72;">returns</span> (<span style="color: #ffa657;">bool</span>);
}
<span style="color: #ff7b72;">interface</span> <span style="color: #ffa657;">IAaveLendingPool</span> {
    <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">deposit</span>(<span style="color: #ff7b72;">address</span> asset, <span style="color: #ffa657;">uint256</span> amount, <span style="color: #ff7b72;">address</span> onBehalfOf, <span style="color: #ffa657;">uint16</span> referralCode) <span style="color: #ff7b72;">external</span>;
}
<span style="color: #ff7b72;">contract</span> <span style="color: #ffa657;">YieldAggregator</span> {
    <span style="color: #ff7b72;">address</span> <span style="color: #ff7b72;">public</span> owner;
    <span style="color: #ffa657;">IERC20</span> <span style="color: #ff7b72;">public</span> <span style="color: #ff7b72;">immutable</span> usdc;
    <span style="color: #ffa657;">IAaveLendingPool</span> <span style="color: #ff7b72;">public</span> aave;
    <span style="color: #ff7b72;">mapping</span>(<span style="color: #ff7b72;">address</span> =&gt; <span style="color: #ffa657;">uint256</span>) <span style="color: #ff7b72;">public</span> deposits;
    <span style="color: #ff7b72;">constructor</span>(<span style="color: #ff7b72;">address</span> _usdc, <span style="color: #ff7b72;">address</span> _aave) {
        owner = msg.sender;
        usdc = <span style="color: #ffa657;">IERC20</span>(_usdc);
        aave = <span style="color: #ffa657;">IAaveLendingPool</span>(_aave);
    }
    <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">deposit</span>(<span style="color: #ffa657;">uint256</span> amount) <span style="color: #ff7b72;">external</span> {
        <span style="color: #ff7b72;">require</span>(amount &gt; 0, <span style="color: #a5d6ff;">"Amount must be &gt; 0"</span>);
        usdc.<span style="color: #d2a8ff;">transferFrom</span>(msg.sender, <span style="color: #ff7b72;">address</span>(<span style="color: #ff7b72;">this</span>), amount);
        usdc.<span style="color: #d2a8ff;">approve</span>(<span style="color: #ff7b72;">address</span>(aave), amount);
        aave.<span style="color: #d2a8ff;">deposit</span>(<span style="color: #ff7b72;">address</span>(usdc), amount, msg.sender, 0); 
        deposits[msg.sender] += amount;
    }
    <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">setAave</span>(<span style="color: #ff7b72;">address</span> _aave) <span style="color: #ff7b72;">external</span> {
        <span style="color: #ff7b72;">require</span>(msg.sender == owner, <span style="color: #a5d6ff;">"Not owner"</span>);
        aave = <span style="color: #ffa657;">IAaveLendingPool</span>(_aave);
    }
    <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">emergencyWithdraw</span>(<span style="color: #ff7b72;">address</span> token, <span style="color: #ffa657;">uint256</span> amount) <span style="color: #ff7b72;">external</span> {
        <span style="color: #ff7b72;">require</span>(msg.sender == owner, <span style="color: #a5d6ff;">"Not owner"</span>);
        <span style="color: #ffa657;">IERC20</span>(token).<span style="color: #d2a8ff;">transfer</span>(owner, amount);
    }
}</code></pre>
</div>

<br>

<br>

<br>

<div style="display: flex; align-items: center; gap: 12px; max-width: 720px; margin: 24px auto; font-family: 'SF Mono', Consolas, monospace; font-size: 13px; color: #8b949e;">
  <div style="flex: 1; border-top: 1px solid #30363d;"></div>
  <span style="white-space: nowrap;">Time to think</span>
  <div style="flex: 1; border-top: 1px solid #30363d;"></div>
</div>

<br>

<br>

<br>

<br>

<br>

<br>

<br>



time's up.

- **We usually pay special attention to external contract calls , for checking reentrancy vulnerabilities [★★★★★★]** 

  <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
  <pre style="margin: 0;"><code><span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">deposit</span>(<span style="color: #ffa657;">uint256</span> amount) <span style="color: #ff7b72;">external</span> {
      <span style="color: #ff7b72;">require</span>(amount &gt; 0, <span style="color: #a5d6ff;">"Amount must be &gt; 0"</span>);
      usdc.<span style="color: #d2a8ff;">transferFrom</span>(msg.sender, <span style="color: #ff7b72;">address</span>(<span style="color: #ff7b72;">this</span>), amount);  <span style="color: #8b949e;">// ← call externals</span>
      usdc.<span style="color: #d2a8ff;">approve</span>(<span style="color: #ff7b72;">address</span>(aave), amount);
      aave.<span style="color: #d2a8ff;">deposit</span>(<span style="color: #ff7b72;">address</span>(usdc), amount, msg.sender, 0);     <span style="color: #8b949e;">// ← call externals</span>
  deposits[msg.sender] += amount;  <span style="color: #8b949e;">// ← Status updates come last</span>
  }</code></pre>
  </div>

  

  >Why external contract calls could be exposed to reentrancy attacks?
  >
  >EVM is single-threaded, and each call to external contracts runs synchronously. If the external contract is a bad guy, it calls the `deposit` function again when it is called by `transferFrom`, then we are getting back to the `deposit` function again, that's causing `transferFrom` to run again, it's transferring again and again and again... till it cleans up all of your money.

  So, this is the fix:

  <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
  <pre style="margin: 0;"><code><span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">deposit</span>(<span style="color: #ffa657;">uint256</span> amount) <span style="color: #ff7b72;">external</span> <span style="color: #ff7b72;">nonReentrant</span> {
      <span style="color: #ff7b72;">require</span>(amount &gt; 0, <span style="color: #a5d6ff;">"Amount must be &gt; 0"</span>);
      deposits[msg.sender] += amount;  <span style="color: #8b949e;">// ← Update the state first</span>
      usdc.<span style="color: #d2a8ff;">transferFrom</span>(msg.sender, <span style="color: #ff7b72;">address</span>(<span style="color: #ff7b72;">this</span>), amount);
      usdc.<span style="color: #d2a8ff;">approve</span>(<span style="color: #ff7b72;">address</span>(aave), amount);
      aave.<span style="color: #d2a8ff;">deposit</span>(<span style="color: #ff7b72;">address</span>(usdc), amount, msg.sender, 0);
  }</code></pre>
  </div>	

  Either add a `nonReentrant` modifier or swap these two line, we do both. (nonReentrant modifier requires _status != _ENTERED , to prevent reentrancy )

<br>

- **Single point of failure (SPOF) [★★★★★★]**

  <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
  <pre style="margin: 0;"><code><span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">setAave</span>(<span style="color: #ff7b72;">address</span> _aave) <span style="color: #ff7b72;">external</span> {
      <span style="color: #ff7b72;">require</span>(msg.sender == owner, <span style="color: #a5d6ff;">"Not owner"</span>);
      aave = <span style="color: #ffa657;">IAaveLendingPool</span>(_aave);  <span style="color: #8b949e;">// ← this can change address by owner alone</span>
  }</code></pre>
  </div>

  ​	The owner alone can change some critical variables ，better not to allow this.

  ​	Advice: consider using multi-sig

  <br>

- **backdoor [★★★★★★]**

  <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
  <pre style="margin: 0;"><code><span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">emergencyWithdraw</span>(<span style="color: #ff7b72;">address</span> token, <span style="color: #ffa657;">uint256</span> amount) <span style="color: #ff7b72;">external</span> {
      <span style="color: #ff7b72;">require</span>(msg.sender == owner, <span style="color: #a5d6ff;">"Not owner"</span>);
      <span style="color: #ffa657;">IERC20</span>(token).<span style="color: #d2a8ff;">transfer</span>(owner, amount);  <span style="color: #8b949e;">// ← rug pull</span>
  }</code></pre>
  </div>

  ​	

   1. It's called an 'emergency withdrawal' , but it's really a rug pull backdoor. 

   2. Users fully trust the owner, but they have no protection

      Fix: Might add this line: `require(token != address(usdc), "Cannot withdraw USDC");` to protect USDCs. Anyway, the fix always depends on the situation.

<br>

- **Compatibility [★★☆☆☆☆]**

  - <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px 24px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 14px; line-height: 1.8; box-shadow: 0 2px 12px rgba(0,0,0,0.4);"><pre style="margin: 0; white-space: pre-wrap;"><code><span style="color: #79c0ff;">usdc</span>.<span style="color: #d2a8ff;">transferFrom</span>(<span style="color: #ffa657;">msg.sender</span>, <span style="color: #ffa657;">address</span>(<span style="color: #79c0ff;">this</span>), <span style="color: #ffa657;">amount</span>);
    <span style="color: #8b949e;">// ← The return value of `transferFrom` isn't handled</span></code></pre></div>

    Some ERC-20 tokens don't return a bool from `transferFrom`, and they won't revert even if the transfer fails, for example the early USDT didn't return a bool from `transfer` or `transferFrom` , recommend using SafeERC20 instead.

    <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
    <pre style="margin: 0;"><code><span style="color: #ff7b72;">using</span> <span style="color: #ffa657;">SafeERC20</span> <span style="color: #ff7b72;">for</span> <span style="color: #ffa657;">IERC20</span>;
    usdc.<span style="color: #d2a8ff;">safeTransferFrom</span>(msg.sender, <span style="color: #ff7b72;">address</span>(<span style="color: #ff7b72;">this</span>), amount);</code></pre>
    </div>



<br>

- **Design defect [★★☆☆☆☆]**

  You're only depositing money, and there are no withdrawals for users. I think this it is a design flaw. 

  We need to be sure to provide a solution to withdraw funds that got stock somewhere , that convinces users to trust the contract.





