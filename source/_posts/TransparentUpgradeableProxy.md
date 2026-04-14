---
title: How OpenZeppelin implements a transparent proxy
date: 2026-04-14 16:12:32
tags: tech
---

How OpenZeppelin implementats a transparent proxy?

**The best way to understand this, is to read the source code of OpenZeppelin's implementation**.



Understanding smart contract transparent upgradeable proxy is not very hard , here is all of it in the physical diagram :

<div style="text-align: center; margin: 5px 0;">
    <img src="https://img.learnblockchain.cn/2025/02/26/706568_13cb902dce8741acb57688dbe9f5ce40~mv2.jpg" 
        alt="hi hi hi"
        style="max-width: 100%; width: 500px; height: auto; border: 1px solid #e2e8f0; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.1);"
    >
</div>

**Proxy**

for `Proxy` , openzeppelin's implementated this component in /openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol ,  programmer can set the admin(owner) and the implementation contract when deploy this transparent proxy.

```solidity
contract TransparentUpgradeableProxy is ERC1967Proxy {
    constructor(
        address _logic,
        address admin_,
        bytes memory _data
    ) payable ERC1967Proxy(_logic, _data) {
        _changeAdmin(admin_);
    }
    ...
}
```

Admin(owner) can also be changed, contract even already had been deployed.

```solidity
function changeAdmin(address newAdmin) external virtual ifAdmin {
   _changeAdmin(newAdmin);
}
```

the crucial function is here, change implementation.  surely , only admin.

```solidity
function upgradeToAndCall(address newImplementation, bytes calldata data) external payable ifAdmin {
   _upgradeToAndCall(newImplementation, data, true);
}
```



**ProxyAdmin**

Actually, the `ProxyAdmin` contract is the real admin that valued in `Proxy` contract.

```solidity
contract ProxyAdmin is Ownable {

 	  // the admin(user) always call this function to upgrade implementation
	  function upgradeAndCall(
        TransparentUpgradeableProxy proxy,
        address implementation,
        bytes memory data
    ) public payable virtual onlyOwner {
    
    		// surprise!  this is the previous function just have seen in Proxy contract, here it is calling  
    		// Proxy contract for changing the implentation
        proxy.upgradeToAndCall{value: msg.value}(implementation, data);
    }
}
```



---

<div style="text-align: center; margin: 5px 0;">
    <img src="https://img.learnblockchain.cn/2025/02/26/706568_13cb902dce8741acb57688dbe9f5ce40~mv2.jpg" 
        alt="hi hi hi"
        style="max-width: 100%; width: 500px; height: auto; border: 1px solid #e2e8f0; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.1);"
    >
</div>

Finally , Owner can call the `Proxy`  directly for test or any thing , but can't directly upgrade , only upgrade via `ProxyAdmin`. Other users directly call Proxy.



