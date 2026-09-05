---
title: How OpenZeppelin implements a transparent proxy
date: 2026-04-14 16:12:32
tags: tech
---

The main challenge with the proxy pattern is function selector collisions. If the proxy contract and the implementation contract both expose functions with the same name and parameter types, the proxy won't know which one to route to. The transparent proxy pattern resolves this issue by differentiating the admin address and normal users — so the proxy knows whose calls to handle locally and which ones to forward.

- Calls from the admin are executed directly in the proxy contract
- Calls from regular users are forwarded to the implementation contract



Lets implement.

```solidity
contract TransparentProxy is Proxy {

    /* The transparent proxy uses fixed storage slots for 
    the implementation address and admin address. These special 
    storage positions follow the EIP‑1967 standard, ensuring 
    they don't conflict with the logic contract's storage layout.
    */
    
    bytes32 private constant _IMPLEMENTATION_SLOT = 0x...;
    bytes32 private constant _ADMIN_SLOT = 0x...;


    constructor(address _initImplementation, address _initAdmin) {
    
        /* The proxy contract uses inline assembly to manipulate 
        	 storage directly. This avoids using regular Solidity 
        	 storage variables, which helps prevent storage collisions
        */
        
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



Clarification:

- The proxy contract doesn't implement any business logic. It simply uses `delegatecall` to delegate all function calls to the logic contract
- The proxy contract stores the implementation address, and during an upgrade, it updates that address 
- The proxy uses `delegatecall` to essentially pull the logic contract's code into its own context and run it as if it were its own，so the storage always stays in the proxy contract, not in the logic contract
