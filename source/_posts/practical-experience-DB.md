---
title: practical experiences - DB
date: 2026-01-28 10:41:01
tags: tech
---

A business originally starts with a smaller code project, usually with monolithic architecture. Through ongoing maintenance over time, query SQLs uprises little by little, eventually，the main direction of queries are likely to incline a certain pattern or trend. for much better understanding, let me explain with a story.

<br>

[ Jan 2026 ] 

First release of the project went online. You received positive feedback and users are increasing.

<center>↓</center>		  

[ Aug 2026 ] 

A good news is, daily active users surpassed 300, users' data in DB is growing exponentially.  The bed one is,  DB's performance is declining. As the SQL monitor, following SQLs are the most frequently called:

<div style="
    background: linear-gradient(135deg, #0f0f23 0%, #1a1a2e 100%);
    padding: 30px;
    border-radius: 12px;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
    margin: 0 auto;
    width: fit-content;
">
<table style="
    border: 1px solid rgba(255, 255, 255, 0.3);
    border-collapse: collapse;
    background-color: rgba(255, 255, 255, 0.05);
    backdrop-filter: blur(10px);
    border-radius: 6px;
    overflow: hidden;
    min-width: 500px;
">
  <!-- 表头行 -->
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
          <div style="margin-left: 40px;">and <span style="color: #79c0ff;">deleted</span>=0;</div>
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
            21%
        </div>
    </td>
  </tr>
</table>
</div>


Writing...
