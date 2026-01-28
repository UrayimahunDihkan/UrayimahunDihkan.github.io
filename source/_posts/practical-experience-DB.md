---
title: A journey to DB optimization
date: 2026-01-28 10:41:01
tags: tech
---

A business originally starts with a smaller code project, usually with monolithic architecture. Through the ongoing maintenance over a time, SQLs uprises little by little, eventually，the main direction of queries are likely to incline a certain pattern or trend. for much better understanding, let me explain with a story.

___

<br>

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

1st SQL seemingly is easy to optimize, give it an index can be enhanced a lot (the value of \`customer_ref\` sufficiently diverse, of high cardinality). But, this is only solution of 1st SQL, we also have to account for 2nd SQL. Over the business, a composite index would be more appropriate for this scenario. users only can query theirown transactions, org and country usually of course somehow can get if you have the customer ref, or it's usually available in login user's context, it's all depends on business design, okay then let's say that exactly is how our business logic works,  then the index should be composited by three fields : \`idx_country_org_cust_ref\`. Let both query use 2nd SQL, abosolutely can be sure to they both follow the leftmost prefix rule, execution time reduced to 50ms.

>Question: Why don't add two indexes \`idx_country_org_cust_ref\`, \`idx_customer_ref\`?
>
>We usually avoid this approach bcz it increases the overhead of update and insert , we should avoid maintaining multiple indexes for a single update or insert , as possible.

---

<center>↓</center>		  

[ Nov 2026 ]

The date is now Nov 2026.
