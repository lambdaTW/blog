+++
title = "AWS Architecture Accenture"
date = 2022-11-17T19:36:48+08:00
description = "本篇介紹石油產業如何運用 NLP 以及 Graph Database 做資料處理以及應用"
cover = "/img/2022/11/AccentureAWSArchitecture.svg"
+++
{{< youtube ekXdohpAy5U >}}

## 簡介
Accenture 是一家科技顧問公司，其服務包含管理諮詢，科技技術與業務流程外包等等，曾被稱作科技業界的麥肯錫

### 石油產業
在這產業，沒有精確的時間去購買原油，可能導致每年多出百萬美金的成本，如果有更加好的決策系統或是知識管理的系統就可以幫助他們省下大量的成本

### 關聯擷取
![Crude Relationship](/img/2022/11/CrudeRelationship.svg)
>他們稱原油：crude，假設某文件中提到 crude A 的 density (濃度) 和 sulfur level (含硫量)，另外的文件提到 crude B 的 density (濃度) 和 sulfur level (含硫量)，還有隱含其風險值，所以，當 crude B 的 density 和 sulfur level 很接近 crude A 時，那就表示 crude A 有很高的機會擁有相同的風險值

## 架構
![Architecture](/img/2022/11/AccentureAWSArchitecture.svg)

- 文件餵到 Lambda
  - Lambda 用 SageMaker 中訓練好的模型取得一些 Type 、Key Facts 等等
  - Lambda 用 Comprehend 獲取詞語分析以及 Token ，和一些 Fact
  - 將其整理後存到 Neptune，例如上方的關聯擷取圖
- 同時也會將原始資料用 ElasticSearch index 起來，方便後面的全文搜索
