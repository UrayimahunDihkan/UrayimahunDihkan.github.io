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


In Apache RocketMQ, there are usually **three message sending modes** we are using:

1. Synchronous send

   the `producer.sendMessage()` code line wait there til the broker receives the message and returns a confirmation. This mode offers a good reliability.

2. Asynchronous send

   the `producer.sendMessage()` code line won't be blocked, means I just need to send the message out , and continue to run , don't block. So there is always a callback, is provided to handle the send result (success or exception) asynchronously. Very applies to High-throughput scenarios where low-latency is needed , but delivery assurance is still important.

3. One-way send

   just only to send out the message, but i don't care about whether it received. Typical features : no return , no callback.

4. Ordered messages

   If the consumer needs to receive messages as the order that they were sent, usually use the orderd message mode that implemented by sending only 1 message queue in a topic.

5. Delayed message

   Send message after a certain time.

6. Transactional message

   A bit complex, but it can be a very elegant and ingenious solution for handling time out orders without payment. (will write a essay to record if I got spare time)

The above methods are relatively easy to be implemented, by an everage programmer. I prefer to dive into its design.



**This is the figure of Rocket-MQ :**

<div align="center">
  <img src="https://pic1.zhimg.com/80/v2-79d63a48746b81bed6fb20769f7cfecc_1440w.webp?source=d16d100b" width="80%">
<div>

**NameServer**:  think of it as kind of DNS for producers and consumers , brokers register themselves with nameservers, producers query nameservers to find which brokers host which topics, usually we implement multiple nameservers and they don't need to sync up each other (eventually consistent) .

**Broker**:  imagine as a "post office" , producer send message from left side, consumer receive from right side. Broker persists messages to disk, maintains manages topics and their message queues.

<blockquote style="text-align: left;">
  <h3>How do they work together ?</h3>
  1. Broker → Nameserver: "Hi, I'm Broker-A hosting Topic-1, Topic-2"<br>
  2. Producer → Nameserver: "Where's Topic-1?"<br>
  3. Nameserver → Producer: "Broker-A has it at 192.168.1.10:10911"<br>
  4. Producer → Broker-A: Sends messages to Topic-1
</blockquote>

