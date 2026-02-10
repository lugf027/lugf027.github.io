---
title: "外网反馈自动化提单与预分析"
date: 2023-06-30
draft: false
tags: ["LLM", "日志工具"]
categories: ["技术"]
---

> 本案例主要基于LLM相关能力，整合工单、日志等平台的能力，自动化完成用户反馈问题后的提单、捞日志、预流转、预分析，提高运营侧提单效率、提高研发侧解决问题的效率。
> 专利：[Data processing method, apparatus, device, readable storage medium, and program product](https://patents.google.com/patent/CN116975108A/en)

## 一、背景
以xx业务为例，最早时运营同学得到外网反馈的数据后，需要根据xx业务提单模板，手动填写各内容字段。然后单独由运营、测试或开发在日志平台下发捞日志，并人工轮询式地去查一下捞日志结果贴到工单。

本案例主要基于LLM相关能力，整合工单、日志等平台的能力，自动化完成用户反馈问题后的提单、捞日志、预流转、预分析，提高运营侧提单效率、提高研发侧解决问题的效率。

## 二、技术方案
### 1、架构图
如下图所示，用户只需提交反馈相关信息，LLM模型会协助提取各字段，并基于 embeddings 将问题描述与历史问题、commit 信息进行匹配，最终自动化完成建bug单、流转、捞日志、在bug单回填预分析内容和日志链接。
![图片](/images/3/Clipboard_Screenshot_1770701155.png)


除此之外，我们还规划实现利用LLM对日志内容做预分析，对日志中常见的crash给到推荐的解决办法；对日志中的特定id，如xx id或xx id，事先查询得到对应xx的详细信息给到开发者参考。
![图片](/images/3/db1051de-2d49-4521-8961-a211a279008d.png)

![图片](/images/3/4a3d1dd8-fab1-4653-bfc1-332d46607ad1.png)

