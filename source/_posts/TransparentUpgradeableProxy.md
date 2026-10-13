---
title: UUPS and transparent proxy
date: 2026-04-14 16:12:32
tags: tech
---



**UUPS**

```mermaid
sequenceDiagram
    participant Script as script
    participant V2 as MultiSignatureV2
    participant Proxy as Proxy (proxyAddress)
    participant Impl as MultiSignature (old impl)

    Note over Script: new MultiSignatureV2()
    Script->>V2: deploy（execute constructor）

    Note over Script: impl=MultiSignature(proxyAddress)
    Script->>Script: cast proxy address to implementation contract type

    Note over Script: impl.upgradeTo(V2)
    Script->>Proxy: upgradeTo(V2)
    Proxy->>Impl: delegatecall upgradeTo(V2)
    Impl->>Proxy: modify the implementation address slot = V2
    Proxy-->>Script: Upgrade complete
```

We chose UUPS over the transparent proxy because UUPS puts the upgrade logic in the implementation contract, which makes the proxy smaller and cheaper on gas. The tradeoff is that the implementation must inherit `UUPSUpgradeable` and implement `_authorizeUpgrade`.

---





**transparent proxy**

```mermaid
sequenceDiagram
    participant Admin as Admin
    participant ProxyAdmin as ProxyAdmin
    participant Proxy as Proxy
    participant LogicV1 as LogicV1
    participant LogicV2 as LogicV2

    Note over Proxy,LogicV2: initial state：proxy points to V1

    Admin->>ProxyAdmin: upgrade(proxy, logicV2Address)
    ProxyAdmin->>Proxy: update implementation address
    Proxy->>Proxy: implementation = LogicV2
    ProxyAdmin-->>Admin: Upgrade complete

    Note over Proxy,LogicV2: after update：proxy points to V2

    rect rgb(200, 245, 200)
        Note over Proxy,LogicV2: user interaction
        Proxy->>LogicV2: delegatecall(burn, 100)
        LogicV2-->>Proxy: executes in the proxy's storage
    end
```



