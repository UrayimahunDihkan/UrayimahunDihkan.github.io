---
title: A dramatic story of tech tinkering with DB performance
date: 2026-01-28 10:41:01
tags: tech
---

A business originally starts with a smaller code project, usually with monolithic architecture. Through the ongoing maintenance over a time, SQLs uprises little by little, eventually，the main direction of queries are likely to incline a certain pattern or trend. for much better understanding, let me explain with a story.

___



[ Jan 2026 ] 

First release of the project went online, the product received positive feedback and users are increasing.

<center>↓</center>		  

[ Aug 2026 ] 

A good news is, daily active users surpassed 300, users' data in DB is growing exponentially.  The bed one is,  DB's performance is declining. As the SQL monitor, following SQLs are the most frequently called:

<table style="
    border: 1px solid rgba(255, 255, 255, 0.3);
    border-collapse: collapse;
    background-color: rgba(255, 255, 255, 0.05);
    backdrop-filter: blur(10px);
    border-radius: 6px;
    overflow: hidden;
    min-width: 700px;
">
  <tr>
    <td style="
        border-right: 1px solid rgba(255, 255, 255, 0.3);
        border-bottom: 1px solid rgba(255, 255, 255, 0.3);
        padding: 12px 20px;
        color: #79c0ff;
        font-weight: 600;
        font-size: 14px;
        font-family: -apple-system, BlinkMacSystemFont, sans-serif;
    ">
        SQL
    </td>
    <td style="
        border-right: 1px solid rgba(255, 255, 255, 0.3);
        border-bottom: 1px solid rgba(255, 255, 255, 0.3);
        padding: 12px 20px;
        color: #3fb950;
        font-weight: 600;
        font-size: 14px;
        font-family: -apple-system, BlinkMacSystemFont, sans-serif;
        text-align: center;
    ">
        Time spend (ms)
    </td>
    <td style="
        border-bottom: 1px solid rgba(255, 255, 255, 0.3);
        padding: 12px 20px;
        color: #ff7b72;
        font-weight: 600;
        font-size: 14px;
        font-family: -apple-system, BlinkMacSystemFont, sans-serif;
        text-align: center;
    ">
        query rate per day
    </td>
  </tr>
  <tr>
      <td style="
          border-right: 1px solid rgba(255, 255, 255, 0.3);
          padding: 20px;
          color: #7ee787;
          font-family: 'SF Mono', Monaco, monospace;
          font-size: 13px;
          line-height: 1.5;
          vertical-align: top;
      ">
          <div style="color: #79c0ff;">select *</div>
          <div style="margin-left: 20px;">from <span style="color: #ff7b72;">transaction</span></div>
          <div style="margin-left: 20px;">where <span style="color: #79c0ff;">customer_ref</span> = #{custRef}</div>
          <div style="margin-left: 40px;">and <span style="color: #79c0ff;">deleted</span> = 0;</div>
      </td>
      <td style="
          border-right: 1px solid rgba(255, 255, 255, 0.3);
          padding: 20px;
          color: #e6edf3;
          vertical-align: middle;
          text-align: center;
      ">
          <div style="
              font-size: 28px; 
              color: #3fb950; 
              font-weight: bold;
          ">
              980ms
          </div>
      </td>
      <td style="
          padding: 20px;
          color: #e6edf3;
          vertical-align: middle;
          text-align: center;
      ">
          <div style="
              font-size: 28px; 
              color: #ff7b72; 
              font-weight: bold;
          ">
              33%
          </div>
      </td>
  </tr>
  <tr>
      <td style="
          border-right: 1px solid rgba(255, 255, 255, 0.3);
          padding: 20px;
          color: #7ee787;
          font-family: 'SF Mono', Monaco, monospace;
          font-size: 13px;
          line-height: 1.5;
          vertical-align: top;
      ">
        <div style="color: #79c0ff;">select *</div>
        <div style="margin-left: 20px;">from <span style="color: #ff7b72;">transaction</span></div>
        <div style="margin-left: 20px;">where <span style="color: #79c0ff;">customer_ref</span> = #{custRef}</div>
        <div style="margin-left: 40px;">and <span style="color: #79c0ff;">org</span> = #{org}</div>
        <div style="margin-left: 40px;">and <span style="color: #79c0ff;">country </span>= #{country}</div>
        <div style="margin-left: 40px;">and <span style="color: #79c0ff;">deleted</span> = 0;</div>
      </td>
      <td style="
          border-right: 1px solid rgba(255, 255, 255, 0.3);
          padding: 20px;
          color: #e6edf3;
          vertical-align: middle;
          text-align: center;
      ">
          <div style="
              font-size: 28px; 
              color: #3fb950; 
              font-weight: bold;
          ">
              1100ms
          </div>
      </td>
      <td style="
          padding: 20px;
          color: #e6edf3;
          vertical-align: middle;
          text-align: center;
      ">
          <div style="
              font-size: 28px; 
              color: #ff7b72; 
              font-weight: bold;
          ">
              18%
          </div>
      </td>
  </tr>
</table>
1st SQL seemingly is easy to optimize, give it an index can be enhanced a lot (the value of \`customer_ref\` sufficiently diverse, of high cardinality). But, this is only solution of 1st SQL, we also have to account for 2nd SQL. Over the business, a composite index would be more appropriate for this scenario. users only can query theirown transactions, org and country usually of course somehow can get if you have the customer ref, or it's usually available in login user's context, it's all depends on business design, okay then let's say that exactly is how our business logic works,  then the index should be composited by three fields : `idx_country_org_cust_ref`. Let both query use 2nd SQL, abosolutely can be sure to they both follow the Leftmost Prefix rule , execution time reduced to 50ms. 

<div style="margin-top: 25px; padding: 15px; background: #2d3748; border-radius: 6px; border-left: 4px solid #4CAF50; font-size: 13px; color: #e2e8f0; text-align: left;">
    <div style="font-weight: bold; margin-bottom: 10px; color: #ffffff; font-size: 14px;">Question：</div>
    <div style="margin-bottom: 15px; padding: 10px; background: rgba(255,255,255,0.05); border-radius: 4px;">
        • Why don't add indexes `idx_country_org_cust_ref`, `idx_customer_ref` respectively ?
    </div>
    <div style="padding: 10px; background: rgba(76, 175, 80, 0.1); border-radius: 4px; border-left: 3px solid #4CAF50;">
        We usually avoid this approach bcz it increases the overhead of update and insert , we should avoid maintaining multiple indexes for a single update or insert , whenever possible.
    </div>
</div>

<br>The main idea is to handle over 80% queries via very less composited indexes , then use very less secondary indexes to cover the remaining non-typical queries as possible.  

---

<center>↓</center>		  

[ Nov 2026 ]

The date is now Nov 2026. slow SQLs mostly adjusted, we've done all we can do. Due to the exponentially growing user base and their visits , performance hit a bottleneck, again. Now SQL optimization is no longer a silver bullet as it used to be, then we consider **read / write splitting** to split the load.

<div style="text-align: center; font-family: 'Segoe UI', system-ui, sans-serif; padding: 30px; background: #1a1d29; border-radius: 12px; max-width: 500px; margin: 0 auto; border: 1px solid #2d3748;">
  <div style="margin-bottom: 15px;">
    <div style="
      display: inline-block;
      background: linear-gradient(145deg, #2d3748, #1a202c);
      border: 1.5px solid #48bb78;
      border-radius: 10px;
      padding: 8px 25px;
      box-shadow: 0 8px 25px rgba(72, 187, 120, 0.15);
    ">
      <div style="font-size: 32px; color: #48bb78; margin-bottom: 10px;">
      <svg width="24" height="24" viewBox="0 0 24 24" style="display: inline-block;">
            <rect x="3" y="3" width="18" height="18" rx="2" fill="#4299e1" opacity="0.2"/>
            <rect x="5" y="5" width="14" height="4" rx="1" fill="#4299e1"/>
            <rect x="5" y="11" width="14" height="4" rx="1" fill="#4299e1"/>
            <rect x="5" y="17" width="14" height="4" rx="1" fill="#4299e1"/>
          </svg>
      </div>
      <div style="font-weight: bold; color: #48bb78; font-size: 15px;">MASTER</div>
      <div style="font-size: 12px; color: #a0aec0; margin-top: 4px; letter-spacing: 0.5px;">Primary (Read/Write)</div>
    </div>
  </div>
  <div style="margin: 0 auto 20px; width: 1.5px; height: 40px; background: linear-gradient(to bottom, #48bb78, #4299e1, #4299e1); position: relative;">
    <div style="position: absolute; bottom: -8px; left: -4px; width: 10px; height: 10px; border-bottom: 2px solid #4299e1; border-right: 2px solid #4299e1; transform: rotate(45deg);"></div>
  </div>
  <div style="font-size: 11px; color: #718096; margin-bottom: 15px; letter-spacing: 1px; text-transform: uppercase;">Replication ↓</div>
  <div style="display: flex; justify-content: center; gap: 70px;">
    <div>
      <div style="
        background: linear-gradient(145deg, #2d3748, #1a202c);
        border: 1.5px solid #4299e1;
        border-radius: 8px;
        padding: 8px 20px;
        box-shadow: 0 6px 20px rgba(66, 153, 225, 0.12);
        transition: transform 0.2s;
      " onmouseover="this.style.transform='translateY(-3px)'" onmouseout="this.style.transform='translateY(0)'">
        <div style="font-size: 26px; color: #4299e1; margin-bottom: 8px;">
          <svg width="24" height="24" viewBox="0 0 24 24" style="display: inline-block;">
            <rect x="3" y="3" width="18" height="18" rx="2" fill="#4299e1" opacity="0.2"/>
            <rect x="5" y="5" width="14" height="4" rx="1" fill="#4299e1"/>
            <rect x="5" y="11" width="14" height="4" rx="1" fill="#4299e1"/>
            <rect x="5" y="17" width="14" height="4" rx="1" fill="#4299e1"/>
          </svg>
        </div>
        <div style="font-weight: bold; color: #4299e1; font-size: 14px;">SLAVE 01</div>
        <div style="font-size: 11px; color: #a0aec0; margin-top: 3px;">Read Replica</div>
      </div>
    </div>
    <div>
      <div style="
        background: linear-gradient(145deg, #2d3748, #1a202c);
        border: 1.5px solid #4299e1;
        border-radius: 8px;
        padding: 8px 20px;
        box-shadow: 0 6px 20px rgba(66, 153, 225, 0.12);
        transition: transform 0.2s;
      " onmouseover="this.style.transform='translateY(-3px)'" onmouseout="this.style.transform='translateY(0)'">
        <div style="font-size: 26px; color: #4299e1; margin-bottom: 8px;">
        <svg width="24" height="24" viewBox="0 0 24 24" style="display: inline-block;">
            <rect x="3" y="3" width="18" height="18" rx="2" fill="#4299e1" opacity="0.2"/>
            <rect x="5" y="5" width="14" height="4" rx="1" fill="#4299e1"/>
            <rect x="5" y="11" width="14" height="4" rx="1" fill="#4299e1"/>
            <rect x="5" y="17" width="14" height="4" rx="1" fill="#4299e1"/>
          </svg>
        </div>
        <div style="font-weight: bold; color: #4299e1; font-size: 14px;">SLAVE 02</div>
        <div style="font-size: 11px; color: #a0aec0; margin-top: 3px;">Read Replica</div>
      </div>
    </div>
  </div>
  <div style="margin-top: 35px; display: flex; justify-content: center; gap: 20px; font-size: 12px;">
    <div style="display: flex; align-items: center;">
      <div style="width: 10px; height: 10px; background: #48bb78; border-radius: 50%; margin-right: 8px;"></div>
      <span style="color: #cbd5e0;">Master Active</span>
    </div>
    <div style="display: flex; align-items: center;">
      <div style="width: 10px; height: 10px; background: #4299e1; border-radius: 50%; margin-right: 8px;"></div>
      <span style="color: #cbd5e0;">Slave Synced</span>
    </div>
  </div>
</div>
We decided master-slaves architecture comprising 1 master and 2 slaves. Route all read traffic to slaves , and all write traffic to the master.

<div style="margin-top: 25px; padding: 15px; background: #2d3748; border-radius: 6px; border-left: 4px solid #4CAF50; font-size: 13px; color: #e2e8f0; text-align: left;">
    <div style="font-weight: bold; margin-bottom: 10px; color: #ffffff; font-size: 14px;">Question：</div>
    <div style="margin-bottom: 15px; padding: 10px; background: rgba(255,255,255,0.05); border-radius: 4px;">
        • Why is there 1 master for writes and 2 slaves to reads?
    </div>
    <div style="padding: 10px; background: rgba(76, 175, 80, 0.1); border-radius: 4px; border-left: 3px solid #4CAF50;">
        Production workloads are often heavily skewed towards reads, there are far more read operations than write operations.
    </div>
    <br>
    <div style="margin-bottom: 15px; padding: 10px; background: rgba(255,255,255,0.05); border-radius: 4px;">
          • Any concerns about replication lag?
      </div>
      <div style="padding: 10px; background: rgba(76, 175, 80, 0.1); border-radius: 4px; border-left: 3px solid #4CAF50;">
        Indeed, during the synchronization delay, if there is a read 'a' operation performed on a slave node immediately after write 'a' operation on the master, it may retrieve a stale data, or data not found. We call this data consistency issues - dirty read. Teams often get around this with smart design at the application level.
      </div>
      <br>
      <div style="margin-bottom: 15px; padding: 10px; background: rgba(255,255,255,0.05); border-radius: 4px;">
            • Other things to keep in mind
        </div>
        <div style="padding: 10px; background: rgba(76, 175, 80, 0.1); border-radius: 4px; border-left: 3px solid #4CAF50;">
            1. Does not have high availablibility, if master downs , slave won't take over the master.
        </div>
</div>

As a result, the performance inhanced a lot , reads and writes got faster. That's why we say "三个臭皮匠胜过一个诸葛亮" (Three heads are better than one).

Cool.

---


<center>↓</center>		  

[ Jan 2027 ]

Data volume is continuously increasing, has now reached a massive scale. We have observed that some table like order table, already reached a skyrocketing count. We then adopted **database sharding**, sharding the table across multiple databases and tables.

<div style="max-width:700px;margin:0 auto;background:#1a1a2e;padding:20px;border-radius:10px">
    <div style="text-align:center;margin-bottom:20px">
        <h2 style="color:#e6e6e6">Database Sharding</h2>
        <p style="color:#a0a0a0">Splitting order table into multiple databases/tables</p>
    </div>
    <div style="display:flex;flex-direction:column;align-items:center;gap:20px">
        <div style="background:#ffd166;padding:15px;border-radius:8px;border-left:4px solid#ef476f;width:300px">
            <h4 style="margin:0 0 10px 0;color:#073b4c">Order Table</h4>
            <p style="margin:5px 0;color:#073b4c">• Single database<br>• Large monolithic table<br>• Performance bottleneck</p>
        </div>
        <div style="color:#a0a0a0;font-size:24px">↓ sharding </div>
        <div style="display:flex;gap:15px;flex-wrap:wrap;justify-content:center">
            <div style="background:#06d6a0;padding:15px;border-radius:8px;border-left:4px solid#118ab2;min-width:180px">
                <h4 style="margin:0 0 10px 0;color:#073b4c">Shard 0</h4>
                <p style="margin:5px 0;color:#073b4c">DB: cluster_alpha<br>Table: order_000<br>Table: order_001<br>Table: order_...</p>
            </div>
            <div style="background:#118ab2;padding:15px;border-radius:8px;border-left:4px solid#06d6a0;min-width:180px">
                <h4 style="margin:0 10px 0;color:#ffffff">Shard 1</h4>
                <p style="margin:5px 0;color:#ffffff">DB: cluster_beta<br>Table: order_000<br>Table: order_001<br>Table: order_...</p>
            </div>
            <div style="background:#ef476f;padding:15px;border-radius:8px;border-left:4px solid#ffd166;min-width:180px">
                <h4 style="margin:0 0 10px 0;color:#ffffff">Shard 2</h4>
                <p style="margin:5px 0;color:#ffffff">DB: cluster_gamma<br>Table: order_000<br>Table: order_001<br>Table: order_...</p>
            </div>
        </div>
    </div>
</div>

Tech-stack: I prefer to sharding-jdbc if the language is JAVA.  Using sharding-jdbc, the design of splitting algorithm can be implemented by ourselves. 

```java
// pseudocode
public class CoursePreciseDSShardingAlgorithm implements PreciseShardingAlgorithm<Long> {

    @Override
    public String doSharding(Collection<String> availableTables, PreciseShardingValue<BigInteger> snowflakeId) {
				
        // let's say there are 5 horizontal 
        // 		splitting tables in one sharding DB
        int tableNo = snowflakeId % 5; 
        if (availableTables.contains(tableNo)) {
            return tableNo;
        }
        throw new UnsupportedOperationException("route to shard"+ shardNo +"failed");
    }
}
```

Above pseudocode is only for table level sharding , I haven't implemented DB-level sharding before , honestly. but sharding-jdbc can do this. 
