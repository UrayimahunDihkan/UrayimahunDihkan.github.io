---
title: UUPS and transparent proxy
date: 2026-04-14 16:12:32
tags: tech
---



**UUPS**



<div style="text-align: center;">
  <img src="https://pic1.zhimg.com/v2-df1a25fb069c7c31cc524f257c4e10d8_r.jpg" style="width: 70%; max-width: 600px; border-radius: 8px; box-shadow: 0 2px 12px rgba(0,0,0,0.4);">
</div>



We chose UUPS over the transparent proxy because UUPS puts the upgrade logic in the implementation contract, which makes the proxy smaller and cheaper on gas. The tradeoff is that the implementation must inherit `UUPSUpgradeable` and implement `_authorizeUpgrade`.

---





**transparent proxy**

<div style="text-align: center;">
  <img src="https://pic4.zhimg.com/v2-4d0596ffccfbb8475a387d5fcdc5add7_r.jpg" style="width: 70%; max-width: 600px; border-radius: 8px; box-shadow: 0 2px 12px rgba(0,0,0,0.4);">
</div>



