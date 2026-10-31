---
title: A use case for multi-sign wallet
date: 2026-08-27 21:37:20
tags: tech
---

Recently, I've developed an interesting business using multi-sign wallet. 

This project, requires tokens(ERC20) can be only minted by someone who approved by multiple administrators. I decided to implement those multi-approvals by administrators using multi-sign wallet.

1. First of all, there must be or we're implementing a contarct for `CRUD`ing to minters.

   <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
   <pre style="margin: 0;"><code><span style="color: #ff7b72;">contract</span> <span style="color: #ffa657;">AddressPrivileges</span> {
       <span style="color: #8b949e;">...</span>
       <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">addMinter</span>(<span style="color: #ff7b72;">address</span> _addMinter) <span style="color: #ff7b72;">public</span> <span style="color: #ff7b72;">validCall</span> <span style="color: #ff7b72;">returns</span> (<span style="color: #ffa657;">bool</span>) {<span style="color: #8b949e;">...</span>}
       <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">delMinter</span>(<span style="color: #ff7b72;">address</span> _delMinter) <span style="color: #ff7b72;">public</span> <span style="color: #ff7b72;">validCall</span> <span style="color: #ff7b72;">returns</span> (<span style="color: #ffa657;">bool</span>) {<span style="color: #8b949e;">...</span>}
       <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">isMinter</span>(<span style="color: #ff7b72;">address</span> account) <span style="color: #ff7b72;">public</span> <span style="color: #ff7b72;">view</span> <span style="color: #ff7b72;">returns</span> (<span style="color: #ffa657;">bool</span>) {<span style="color: #8b949e;">...</span>}
       <span style="color: #ff7b72;">modifier</span> <span style="color: #d2a8ff;">onlyMinter</span>() {<span style="color: #8b949e;">...</span>}
       <span style="color: #8b949e;">...</span>
   }</code></pre>
   </div>

2. Here is multiSignature.sol

   

   <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
   <pre style="margin: 0;"><code><span style="color: #ff7b72;">contract</span> <span style="color: #ffa657;">multiSignature</span> {
       <span style="color: #8b949e;">...</span>
       <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">createApplication</span>(<span style="color: #ff7b72;">address</span> to) <span style="color: #ff7b72;">external</span> <span style="color: #ff7b72;">returns</span>(<span style="color: #ffa657;">uint256</span>) {
           <span style="color: #ffa657;">bytes32</span> msghash = <span style="color: #d2a8ff;">getApplicationHash</span>(msg.sender, to);
           <span style="color: #ffa657;">uint256</span> index = signatureMap[msghash].length;
           signatureMap[msghash].<span style="color: #d2a8ff;">push</span>(<span style="color: #ffa657;">signatureInfo</span>(msg.sender, <span style="color: #ff7b72;">new</span> <span style="color: #ff7b72;">address</span>[](0)));
           <span style="color: #ff7b72;">emit</span> <span style="color: #d2a8ff;">CreateApplication</span>(msg.sender, to, msghash);
           <span style="color: #ff7b72;">return</span> index;
       }
       <span style="color: #ff7b72;">function</span> <span style="color: #d2a8ff;">signApplication</span>(<span style="color: #ffa657;">bytes32</span> msghash) <span style="color: #ff7b72;">external</span> <span style="color: #ff7b72;">onlyOwner</span> <span style="color: #ff7b72;">validIndex</span>(msghash, defaultIndex) {
       <span style="color: #ff7b72;">emit</span> <span style="color: #d2a8ff;">SignApplication</span>(msg.sender, msghash, defaultIndex);
       signatureMap[msghash][defaultIndex].signatures.<span style="color: #d2a8ff;">addWhiteListAddress</span>(msg.sender);
   }
   <span style="color: #8b949e;">...</span>
   }</code></pre>
   </div>

   

   When user clicks a button like `add minter` from FE, it calls `multiSignature.sol:createApplication` function first : `createApplication(${address of AddressPrivileges contract})` to hashes the ${address of `AddressPrivileges`} and users address , put the hash into a signatureMap. 

   Then administrators can see these pending approvals in their end. 

   Once they all approved the request from their site, FE calls `signApplication` to add their address to the signatureMap.signatureInfo.signatures. (signatures[] is empty before `signApplication` function, it was a  signatureMap[msghash] [empty] )

   

3. After approved by multi-sign wallets , now it's fine to call `addMinter` in `AddressPrivileges.sol`, bcz we can't pass the `validCall` check til multi-sign approval.

   <div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
   <pre style="margin: 0;"><code><span style="color: #ff7b72;">modifier</span> <span style="color: #d2a8ff;">validCall</span>() {
       <span style="color: #d2a8ff;">checkMultiSignature</span>();
       <span style="color: #ff7b72;">_</span>;
   }</code></pre>
   </div>

I would not say too much to breakdown those source codes wrote in my style , it's not important , for now the matter is to get the mind. 

Cool.

<br>

<br>

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 20px 0; color: #8b949e; font-size: 13px; border-bottom: 1px solid #30363d; padding-bottom: 8px;">
  Updated on <span style="color: #e6edf3; font-weight: 600;">Oct 31, 2026</span>
</div>

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 760px; margin: 0 auto; padding: 28px 24px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 14px; line-height: 1.6; box-shadow: 0 2px 12px rgba(0,0,0,0.4);">
  <div style="text-align: center; font-size: 18px; font-weight: bold; color: #d2a8ff; margin-bottom: 8px;">🛡️ Multi-sig Wallet: Core Logic of signatureMap</div>
  <div style="text-align: center; color: #8b949e; font-size: 12px; margin-bottom: 28px;">How to control high-privilege operations via an on-chain state machine</div>
  <div style="background: rgba(210,168,255,0.05); border: 1px solid #d2a8ff; border-radius: 8px; padding: 16px; margin-bottom: 24px;">
    <div style="color: #d2a8ff; font-size: 13px; margin-bottom: 8px; font-weight: bold;">💡 The Core State (On-chain Ledger):</div>
    <code style="color: #e6edf3; font-size: 14px; background: #161b22; padding: 4px 8px; border-radius: 4px; display: inline-block; margin-bottom: 8px;">
      mapping(bytes32 =&gt; SignatureInfo) public signatureMap;
    </code>
    <div style="color: #c9d1d9; font-size: 12px;">
      This mapping is the absolute core. It tracks the approval status of <em>every</em> request using a unique <code>msghash</code>.
    </div>
  </div>
  <div style="background: #161b22; border: 1px solid #30363d; border-radius: 12px; padding: 24px 20px; margin-bottom: 24px;">
    <div style="display: flex; align-items: flex-start; margin-bottom: 24px;">
      <div style="background: #79c0ff; color: #0d1117; font-weight: bold; border-radius: 50%; width: 24px; height: 24px; display: flex; align-items: center; justify-content: center; flex-shrink: 0; margin-right: 16px; margin-top: 2px;">1</div>
      <div style="flex: 1;">
        <div style="color: #e6edf3; font-weight: bold; margin-bottom: 4px;">Create Request (createApplication)</div>
        <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 12px;">A user calls the contract. The contract hashes the target address and initiator to create a unique <code style="background:#30363d; padding:2px 4px; border-radius:4px; font-size:12px;">msghash</code>. It then initializes an empty signature array in the Map.</div>
        <div style="background: #0d1117; border: 1px dashed #79c0ff; border-radius: 8px; padding: 12px;">
          <div style="color: #8b949e; font-size: 11px; margin-bottom: 8px;">📦 signatureMap state change:</div>
          <div style="color: #79c0ff; font-size: 13px;">signatureMap[msghash] = {</div>
          <div style="color: #e6edf3; font-size: 13px; padding-left: 20px;">applicant: msg.sender, <br> signatures: <span style="color: #ff7b72;">[]</span> <span style="color: #8b949e;">// Empty array</span></div>
          <div style="color: #79c0ff; font-size: 13px;">}</div>
        </div>
      </div>
    </div>
    <div style="display: flex; align-items: flex-start; margin-bottom: 24px;">
      <div style="background: #d2a8ff; color: #0d1117; font-weight: bold; border-radius: 50%; width: 24px; height: 24px; display: flex; align-items: center; justify-content: center; flex-shrink: 0; margin-right: 16px; margin-top: 2px;">2</div>
      <div style="flex: 1;">
        <div style="color: #e6edf3; font-weight: bold; margin-bottom: 4px;">Admin Approval (signApplication)</div>
        <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 12px;">Multiple administrators call the contract. The contract looks up the <code>msghash</code> in the map and appends their addresses to the signatures array.</div>
        <div style="background: #0d1117; border: 1px dashed #d2a8ff; border-radius: 8px; padding: 12px;">
          <div style="color: #8b949e; font-size: 11px; margin-bottom: 8px;">📦 signatureMap state change:</div>
          <div style="color: #d2a8ff; font-size: 13px;">signatureMap[msghash] = {</div>
          <div style="color: #e6edf3; font-size: 13px; padding-left: 20px;">applicant: msg.sender, <br> signatures: <span style="color: #7ee787;">[admin1, admin2, admin3...]</span> <span style="color: #8b949e;">// Array grows</span></div>
          <div style="color: #d2a8ff; font-size: 13px;">}</div>
        </div>
      </div>
    </div>
    <div style="display: flex; align-items: flex-start;">
      <div style="background: #7ee787; color: #0d1117; font-weight: bold; border-radius: 50%; width: 24px; height: 24px; display: flex; align-items: center; justify-content: center; flex-shrink: 0; margin-right: 16px; margin-top: 2px;">3</div>
      <div style="flex: 1;">
        <div style="color: #e6edf3; font-weight: bold; margin-bottom: 4px;">Execute Action (validCall)</div>
        <div style="color: #c9d1d9; font-size: 13px; margin-bottom: 12px;">User calls <code style="background:#30363d; padding:2px 4px; border-radius:4px; font-size:12px;">addMinter</code>. The <code style="background:#30363d; padding:2px 4px; border-radius:4px; font-size:12px;">modifier validCall</code> intercepts and queries the Map to verify if approvals are sufficient.</div>
        <div style="background: #0d1117; border: 1px solid #7ee787; border-radius: 8px; padding: 12px; box-shadow: 0 0 8px rgba(126,231,135,0.1);">
          <div style="color: #7ee787; font-size: 13px; margin-bottom: 4px;">✅ checkMultiSignature() verification pass</div>
          <div style="color: #e6edf3; font-size: 12px;">If <code style="color:#7ee787">signatureMap[msghash].signatures.length &gt;= threshold</code></div>
          <div style="color: #e6edf3; font-size: 12px; margin-top: 8px;">➔ Verification passes, executes logic: <code style="color:#7ee787">addMinter(_addMinter)</code></div>
        </div>
      </div>
    </div>
  </div>
  <div style="background: rgba(210,168,255,0.1); border-left: 4px solid #d2a8ff; padding: 16px; border-radius: 0 8px 8px 0;">
    <div style="color: #d2a8ff; font-weight: bold; margin-bottom: 8px;">💡 Why is signatureMap the core?</div>
    <div style="color: #c9d1d9; font-size: 13px; line-height: 1.7;">
      It simplifies the complex process of 'multi-sig' into an on-chain <strong>state machine</strong>.<br><br>
      1. <strong>create</strong> registers the state (generating an empty array).<br>
      2. <strong>sign</strong> modifies the state (pushing addresses).<br>
      3. <strong>validCall</strong> reads the state (checking the array length).<br><br>
      Without this mapping, approvals would be scattered and impossible to verify securely.
    </div>
  </div>
</div>

