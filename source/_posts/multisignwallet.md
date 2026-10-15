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

