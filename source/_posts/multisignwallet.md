---
title: A use case for multi-sign wallet
date: 2026-08-27 21:37:20
tags: tech
---

Recently, I've developed an interesting business using multi-sign wallet. 

This project, requires tokens(ERC20) can be only minted by someone who approved by multiple administrators. I decided to implement those multi-approvals by administrators using multi-sign wallet.

1. First of all, there must be or we're implementing a contarct for `CRUD`ing to minters.

   ```solidity
   contract AddressPrivileges {
   		...
     function addMinter(address _addMinter) public validCall returns (bool) {...}
     function delMinter(address _delMinter) public validCall returns (bool) {...}
     function isMinter(address account) public view returns (bool) {...}
     modifier onlyMinter() {...}
   		...
   }
   ```

2. Here is multiSignature.sol

   ```solidity
   contract multiSignature {
       ...
     function createApplication(address to) external returns(uint256) {
       bytes32 msghash = getApplicationHash(msg.sender,to);
       uint256 index = signatureMap[msghash].length;
       signatureMap[msghash].push(signatureInfo(msg.sender,new address[](0)));
       emit CreateApplication(msg.sender,to,msghash);
       return index;
     }
     
      function signApplication(bytes32 msghash) external onlyOwner 	validIndex(msghash,defaultIndex){
         emit SignApplication(msg.sender,msghash,defaultIndex);
         signatureMap[msghash][defaultIndex].signatures.addWhiteListAddress(msg.sender);
     }
       ...
   }
   ```

   When user clicks a button like `add minter` from FE, it calls `multiSignature.sol:createApplication` function first : `createApplication(${address of AddressPrivileges contract})` to hashes the ${address of `AddressPrivileges`} and users address , put the hash into a signatureMap. 

   Then administrators can see these pending approvals in their end. 

   Once they all approved the request from their site, FE calls `signApplication` to add their address to the signatureMap.signatureInfo.signatures. (signatures[] is empty before `signApplication` function, it was a  signatureMap[msghash] [empty] )

   

3. After approved by multi-sign wallets , now it's fine to call `addMinter` in `AddressPrivileges.sol`, bcz we can't pass the `validCall` check til multi-sign approval.

   ```solidity
       modifier validCall(){
           checkMultiSignature();
           _;
       }
   ```



I would not say too much to breakdown those source codes wrote in my style , it's not important , for now the matter is to get the mind. 

Cool.

