---
title: MQ
date: 2026-02-01 20:07:23
tags: tech
---

I think MQ is a genuis solution of non-real-time data requests between systems, especially in distributed systems for e-commerce, finance, big data. Here I am going to document some of my thoughts on MQ.

---

Most of typical financial process systems, the first level system which directly access by user and submit their financial applications , the financing request goes through several systems will eventually reach to bank center to do payment / settlement -- subtract / addition of account money,  between each of them, MQ plays a crucial role to ensure the reliability of the entire system via decoupling , traffic peaking, asynchronizing.

<div style="max-width:800px;margin:auto">
  <div style="display:flex;justify-content:center;align-items:center;margin-top:60px;background:#1a1a1a;padding:40px;border-radius:12px">
    <div style="background:#3498db;color:white;width:180px;height:80px;border-radius:6px;display:flex;flex-direction:column;justify-content:center;align-items:center;box-shadow:0 3px 6px rgba(0,0,0,0.3);font-weight:bold">
      <span style="font-size:14px;margin-top:5px;text-align:center">User TRade Financial Application</span>
    </div>
    <div style="font-size:14px;margin:0 20px;color:#cccccc">→<span style="font-size:14px;">MQ</span>→</div>
    <div style="background:#f39c12;color:white;width:180px;height:80px;border-radius:6px;display:flex;flex-direction:column;justify-content:center;align-items:center;box-shadow:0 3px 6px rgba(0,0,0,0.3);font-weight:bold">
      <span style="font-size:14px;margin-top:5px;text-align:center">Verification & Approving System</span>
    </div>
    <div style="font-size:14px;margin:0 20px;color:#cccccc">→<span style="font-size:14px;">MQ</span>→</div>
    <div style="background:#2ecc71;color:white;width:180px;height:80px;border-radius:6px;display:flex;flex-direction:column;justify-content:center;align-items:center;box-shadow:0 3px 6px rgba(0,0,0,0.3);font-weight:bold">
      <span style="font-size:14px;margin-top:5px;text-align:center">Bank Center System</span>
    </div>
  </div>
</div>

I enjoy the following structure, It is figure of Rocket-MQ which developed by Alibaba technology team , widely adopted by tech giants. 

<div align="center">
  <img src="https://pic1.zhimg.com/80/v2-79d63a48746b81bed6fb20769f7cfecc_1440w.webp?source=d16d100b" width="80%">
<div>

**·Producer**

Easy to understand, the Trade Financing Application System is a producer.

**·Consumer**

Bank Center System

**·Broker**

