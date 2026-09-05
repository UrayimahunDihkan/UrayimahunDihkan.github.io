---
title: How OpenZeppelin implements a transparent proxy
date: 2026-04-14 16:12:32
tags: tech
---

How OpenZeppelin implementats a transparent proxy?



Understanding smart contract transparent upgradeable proxy is not very hard , here is all of it in the physical diagram :

<div style="text-align: center; margin: 5px 0;">
    <img src="https://img.learnblockchain.cn/2025/02/26/706568_13cb902dce8741acb57688dbe9f5ce40~mv2.jpg" 
        alt="hi hi hi"
        style="max-width: 100%; width: 500px; height: auto; border: 1px solid #e2e8f0; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.1);"
    >
</div>
The main challenge with the proxy pattern is function selector collisions. If the proxy contract and the implementation contract both expose functions with the same name and parameter types, the proxy won't know which one to route to. The transparent proxy pattern resolves this issue by differentiating the admin address and normal users — so the proxy knows whose calls to handle locally and which ones to forward.

- Calls from the admin are executed directly in the proxy contract
- Calls from regular users are forwarded to the implementation contract



Lets implement.

```solidity
contract TransparentProxy is Proxy {

    /* "The transparent proxy uses fixed storage slots for the implementation address and admin address. These special storage positions follow the EIP‑1967 standard, ensuring they don't conflict with the logic contract's storage layout."
    */
    
    bytes32 private constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;
    bytes32 private constant _ADMIN_SLOT = 0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103;


    constructor(address _initImplementation, address _initAdmin) {
    
        // The proxy contract uses inline assembly to manipulate storage directly
        // This avoids using regular Solidity storage variables, which helps prevent storage collisions
        
        assembly {
            sstore(_IMPLEMENTATION_SLOT, _initImplementation)
        }
      
        assembly {
            sstore(_ADMIN_SLOT, _initAdmin)
        }
    }
    
    // other...
}
```

The `_implementation()` function — inherited from the `Proxy` contract, and it's responsible for returning the implementation contract's address that stored in the `_IMPLEMENTATION_SLOT`

```solidity
function _implementation() internal view override returns (address) {
    address impl;
    assembly {
        impl := sload(_IMPLEMENTATION_SLOT)
    }
    return impl;
}
```

If a user calls a function that the proxy doesn't have, the `fallback` function kicks in and forwards it to the implementation contract. `fallback` defined in the `Proxy` base contract.

The proxy contract provides an admin-only upgrade function: 

```solidity
contract TransparentProxy is Proxy {

	...
	function upgradeTo(address _newImplementation) external {
    require(msg.sender == admin(), "Only admin can upgrade");
    assembly {
        sstore(_IMPLEMENTATION_SLOT, _newImplementation)
    }
	}
	...
	
}
```



Implementation contracts:

(In the proxy pattern, the logic contract's constructor gets bypassed. To compensate, we use an `initialize` function provided by the `Initializable` contract, which does the same job as the constructor does)

```solidity
contract ImplementationContractV1 is Initializable, OwnableUpgradeable {
    uint256 public value; //slot0

    function setValue(uint256 _value) public virtual {
        value = _value;
    }

    function getValue() public view virtual returns (uint256) {
        return value;
    }

    function initialize(address initialOwner) public initializer {
        __Ownable_init(initialOwner);
    }
}
```

```solidity
contract ImplementationContractV2 is TransparentLogicV1 {
    uint256 public newValue; // 新增状态变量,slot1

    function setValue(uint256 _newValue) public virtual override {
        newValue = _newValue;
    }

    function getValue() public view virtual override returns (uint256) {
        return value + newValue;
    }
}
```



Here's the typical workflow for a transparent proxy:

1. Deploy the logic contract — ImplementationContractV1
2. Deploy the transparent proxy contract, pointing it to the ImplementationContractV1's address
3. Interact with the proxy contract address using ImplementationContractV1's ABI
4. When an upgrade is needed, deploy the new logic contract — ImplementationContractV2
5. The admin calls the `upgradeTo` function on the proxy, pointing it to ImplementationContractV2
6. Continue using the same proxy address, but now interact with it using ImplementationContractV2's ABI



