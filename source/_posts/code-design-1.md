---
title: Code pattern design（一）
date: 2024-09-22 10:45:15
tags: design
---

**Business：**

As the diagram below shows, suppose we receive a requirement: listen to MySQL data changes via Canal, and sync those changes to other systems in real time — like Elasticsearch, MQ messages, or triggering business logic. What I'm trying to show is a solid, extensible base built with the right design patterns and strong open-closed compliance for all these scenarios.

![img](https://pica.zhimg.com/v2-074a1b61dd58c8b97c9256a675f6739c_1440w.jpg)

**design：**

　　In this project, Spring Boot will definitely have a Listener as the main entry point. As soon as Canal detects even the slightest change in MySQL, we'll be able to catch it in this Listener and take the appropriate action based on the event. Here's the pseudocode:

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
<pre style="margin: 0;"><code><span style="color: #8b949e;">/**
Data change monitoring business
* @param beforeColumns Data before the update
* @param afterColumns Data after the update
* @param tableName Table name
* @param eventType Operation type [insert, update, delete]
* @param timestamp Timestamp when the data changed
 */</span>
<span style="color: #ff7b72;">public class</span> <span style="color: #ffa657;">CanalListener</span> {
    <span style="color: #ff7b72;">public void</span> <span style="color: #d2a8ff;">listen</span>(<span style="color: #ffa657;">List</span>&lt;<span style="color: #ffa657;">CanalEntry.Column</span>&gt; beforeColumns, <span style="color: #ffa657;">List</span>&lt;<span style="color: #ffa657;">CanalEntry.Column</span>&gt; afterColumns, <span style="color: #ffa657;">String</span> tableName, <span style="color: #ffa657;">CanalEntry.EventType</span> eventType, <span style="color: #ff7b72;">long</span> timestamp) {
        <span style="color: #ff7b72;">if</span>(<span style="color: #a5d6ff;">"user_info"</span>.<span style="color: #d2a8ff;">equals</span>(tableName))  {
            <span style="color: #ff7b72;">if</span> (<span style="color: #ffa657;">CanalEntry.EventType.INSERT</span>.<span style="color: #d2a8ff;">equals</span>(eventType)) {
              <span style="color: #8b949e;">//todo for insert</span>
            } <span style="color: #ff7b72;">else if</span> (<span style="color: #ffa657;">CanalEntry.EventType.UPDATE</span>.<span style="color: #d2a8ff;">equals</span>(eventType)) {
              <span style="color: #8b949e;">//todo for update</span>
            } <span style="color: #ff7b72;">else if</span> (<span style="color: #ffa657;">CanalEntry.EventType.DELETE</span>.<span style="color: #d2a8ff;">equals</span>(eventType)) {
             <span style="color: #8b949e;">//todo for delete</span>
            }
        }
    }
}</code></pre>
</div>


After reading it, you'll notice a design problem: right now, we're only handling the `user_info` table. But what if later iterations require listening to the `order` table too — do we just keep stacking logic on top? What if, three years later, there are another ten or twenty tables being maintained here? You'd end up with a situation where there are tons of `if`s, `if`s inside `if`s, and business logic buried inside those `if`s. By then, the code would definitely be "profound and great" — as the saying goes, "the 'if' gives birth to one, one gives birth to two, two gives birth to three, and three gives birth to all things," and those "all things" are basically a crematorium.

So, instead of piling everything into the same place, it's better to encapsulate a factory. The factory can give us a table-specific handler based on the table name. Inside the handler, we have methods for handling `update`, `insert`, and `delete` events. Then the Listener can be written like this (optimized pseudocode):

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
<pre style="margin: 0;"><code><span style="color: #8b949e;">/**
 * Data change listener logic
 * @param beforeColumns The data before the update
 * @param afterColumns The data after the update
 * @param tableName The name of the table
 * @param eventType The operation type [insert, update, delete]
 * @param timestamp The timestamp when the data change occurred
 */</span>
<span style="color: #ff7b72;">public class</span> <span style="color: #ffa657;">CanalListener</span> {
    <span style="color: #ff7b72;">public void</span> <span style="color: #d2a8ff;">listen</span>(<span style="color: #ffa657;">List</span>&lt;<span style="color: #ffa657;">CanalEntry.Column</span>&gt; beforeColumns, <span style="color: #ffa657;">List</span>&lt;<span style="color: #ffa657;">CanalEntry.Column</span>&gt; afterColumns, <span style="color: #ffa657;">String</span> tableName, <span style="color: #ffa657;">CanalEntry.EventType</span> eventType, <span style="color: #ff7b72;">long</span> timestamp) {
        <span style="color: #8b949e;">//获取处理器</span>
        <span style="color: #ffa657;">TableHandler</span> handler = <span style="color: #ffa657;">TableHandlerFactory</span>.<span style="color: #d2a8ff;">getHandler</span>(tableName);
        <span style="color: #ff7b72;">if</span> (<span style="color: #ffa657;">ObjectUtil</span>.<span style="color: #d2a8ff;">isEmpty</span>(handler)) <span style="color: #ff7b72;">return</span>;
        <span style="color: #8b949e;">//执行各自的业务</span>
        <span style="color: #ffa657;">ListenEntity</span> listenEntity = <span style="color: #ff7b72;">new</span> <span style="color: #ffa657;">ListenEntity</span>(afterColumns, beforeColumns, timestamp); <span style="color: #8b949e;">//监听到的数据对象</span>
        <span style="color: #ff7b72;">if</span> (<span style="color: #ffa657;">CanalEntry.EventType.INSERT</span>.<span style="color: #d2a8ff;">equals</span>(eventType)) {
            handler.<span style="color: #d2a8ff;">insert</span>(listenEntity);
        } <span style="color: #ff7b72;">else if</span> (<span style="color: #ffa657;">CanalEntry.EventType.UPDATE</span>.<span style="color: #d2a8ff;">equals</span>(eventType)) {
            handler.<span style="color: #d2a8ff;">update</span>(listenEntity);
        } <span style="color: #ff7b72;">else if</span> (<span style="color: #ffa657;">CanalEntry.EventType.DELETE</span>.<span style="color: #d2a8ff;">equals</span>(eventType)) {
            handler.<span style="color: #d2a8ff;">delete</span>(listenEntity);
        }
    }
}</code></pre>
</div>


The core listener code is this small — short and clean. If you want to add a new table to listen to, you just implement the `insert`, `update`, and `delete` template methods based on `TableHandler`, and register it with the factory.

Here's the implementation approach for the factory and handlers. First, the factory maintains a `HashMap`. When you get a handler from the factory by table name, it's actually pulling it from the `HashMap`. Registering a handler is just a `put` operation on the `HashMap`.

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
<pre style="margin: 0;"><code><span style="color: #8b949e;">// 工厂</span>
<span style="color: #ff7b72;">public class</span> <span style="color: #ffa657;">TableHandlerFactory</span> {
    <span style="color: #ff7b72;">public static</span> <span style="color: #ffa657;">Map</span>&lt;<span style="color: #ffa657;">String</span>, <span style="color: #ffa657;">TableHandler</span>&gt; tableNameHandlerMap = <span style="color: #ff7b72;">new</span> <span style="color: #ffa657;">HashMap</span>();
    <span style="color: #8b949e;">// 把表handler注册到TableHandlerFactory中</span>
    <span style="color: #ff7b72;">public static void</span> <span style="color: #d2a8ff;">register</span>(<span style="color: #ffa657;">String</span> tableName, <span style="color: #ffa657;">TableHandler</span> tableHandler) {
        <span style="color: #ff7b72;">if</span> (<span style="color: #ffa657;">StrUtil</span>.<span style="color: #d2a8ff;">isEmpty</span>(tableName)) {
            <span style="color: #ff7b72;">throw new</span> <span style="color: #ffa657;">IllegalArgumentException</span>(<span style="color: #a5d6ff;">"Registered tableName is empty"</span>);
        } <span style="color: #ff7b72;">else if</span> (<span style="color: #ffa657;">ObjectUtil</span>.<span style="color: #d2a8ff;">isEmpty</span>(tableHandler)) {
            <span style="color: #ff7b72;">throw new</span> <span style="color: #ffa657;">IllegalArgumentException</span>(<span style="color: #a5d6ff;">"Registered tableHandler is empty"</span>);
        }
        tableNameHandlerMap.<span style="color: #d2a8ff;">put</span>(tableName, tableHandler);
    }
    <span style="color: #8b949e;">// 根据表名获取处理器</span>
    <span style="color: #ff7b72;">public static</span> <span style="color: #ffa657;">TableHandler</span> <span style="color: #d2a8ff;">getHandler</span>(<span style="color: #ffa657;">String</span> tableName) {
        <span style="color: #ff7b72;">if</span> (<span style="color: #ffa657;">StrUtil</span>.<span style="color: #d2a8ff;">isEmpty</span>(tableName)) {
            <span style="color: #ff7b72;">throw new</span> <span style="color: #ffa657;">IllegalArgumentException</span>(<span style="color: #a5d6ff;">"illegal input for get tableHandler,tableName is empty"</span>);
        }
        <span style="color: #ff7b72;">return</span> tableNameHandlerMap.<span style="color: #d2a8ff;">get</span>(tableName);
    }
}</code></pre>
</div>

Why did I use an abstract class for `TableHandler` instead of an interface? The benefit is that an abstract class can provide default implementations. If a handler doesn't implement a certain method from the abstract class — say `delete` — but `delete` gets called in the business logic, it will throw an `UnsupportedOperationException` by default. Alternatively, we could also throw a `RuntimeException("This method has not been implemented")`;

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
<pre style="margin: 0;"><code><span style="color: #ff7b72;">public abstract class</span> <span style="color: #ffa657;">TableHandler</span> <span style="color: #ff7b72;">implements</span> <span style="color: #ffa657;">InitializingBean</span> {
    <span style="color: #ff7b72;">public void</span> <span style="color: #d2a8ff;">insert</span>(<span style="color: #ffa657;">ListenEntity</span> listenEntity) {
        <span style="color: #ff7b72;">throw new</span> <span style="color: #ffa657;">UnsupportedOperationException</span>();
    }
    <span style="color: #ff7b72;">public void</span> <span style="color: #d2a8ff;">update</span>(<span style="color: #ffa657;">ListenEntity</span> listenEntity) {
        <span style="color: #ff7b72;">throw new</span> <span style="color: #ffa657;">UnsupportedOperationException</span>();
    }
    <span style="color: #ff7b72;">public void</span> <span style="color: #d2a8ff;">delete</span>(<span style="color: #ffa657;">ListenEntity</span> listenEntity) {
        <span style="color: #ff7b72;">throw new</span> <span style="color: #ffa657;">UnsupportedOperationException</span>();
    }
}</code></pre>
</div>

Each table's business logic is encapsulated in its handler.

<div style="font-family: 'SF Mono', Consolas, 'Courier New', monospace; max-width: 720px; margin: 0 auto; padding: 20px; background: #0d1117; border-radius: 16px; color: #e6edf3; font-size: 13px; line-height: 1.7; box-shadow: 0 2px 12px rgba(0,0,0,0.4); overflow-x: auto;">
<pre style="margin: 0;"><code><span style="color: #79c0ff;">@Component</span>
<span style="color: #ff7b72;">public class</span> <span style="color: #ffa657;">OrderTableHandler</span> <span style="color: #ff7b72;">extends</span> <span style="color: #ffa657;">TableHandler</span> {
    <span style="color: #ff7b72;">private</span> <span style="color: #ffa657;">String</span> tableName = <span style="color: #a5d6ff;">"t_order"</span>;
    <span style="color: #79c0ff;">@Override</span>
    <span style="color: #ff7b72;">public void</span> <span style="color: #d2a8ff;">insert</span>(<span style="color: #ffa657;">ListenEntity</span> listenEntity) {
        <span style="color: #8b949e;">//todo</span>
    }
    <span style="color: #79c0ff;">@Override</span>
    <span style="color: #ff7b72;">public void</span> <span style="color: #d2a8ff;">update</span>(<span style="color: #ffa657;">ListenEntity</span> listenEntity) {
        <span style="color: #8b949e;">//todo</span>
    }
    <span style="color: #79c0ff;">@Override</span>
    <span style="color: #ff7b72;">public void</span> <span style="color: #d2a8ff;">delete</span>(<span style="color: #ffa657;">ListenEntity</span> listenEntity) {
        <span style="color: #8b949e;">//todo</span>
    }
    <span style="color: #8b949e;">// Spring启动时注册自己到Factory工厂</span>
    <span style="color: #79c0ff;">@Override</span>
    <span style="color: #ff7b72;">public void</span> <span style="color: #d2a8ff;">afterPropertiesSet</span>() <span style="color: #ff7b72;">throws</span> <span style="color: #ffa657;">Exception</span> {
        <span style="color: #ffa657;">TableHandlerFactory</span>.<span style="color: #d2a8ff;">register</span>(tableName, <span style="color: #ff7b72;">this</span>);
    }
}</code></pre>
</div>

Surprise! that's it.

Honestly, I can't even clearly name which design patterns I used here. The way I see it, design patterns are about using the features of your language well — interfaces, abstract classes, regular classes, any kind of component — and combining them in a way that makes the business foundation work like magic. That's my personal take on design. And design is flexible — the same thing can be designed in so many different ways, and everyone has their own style, so the implementation I showed is just one of many options.

Anyway, no brute force. 
