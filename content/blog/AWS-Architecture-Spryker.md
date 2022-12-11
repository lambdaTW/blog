+++
title = "AWS Architecture Spryker"
date = "2022-12-11T11:51:54+08:00"
cover = "/img/2022/12/Spryker-Application-Composition-Platform-Architecture.png"
tags = [""]
description = "做為軟體平台商，如何讓用戶以及軟體開發商做更好的銜接，是留住以及吸引開發商和用戶開發者的一大關鍵要素，來看看 Spryker 如何利用 Message 架構做到這件事情"
+++
{{< youtube DBmmQDlIXWM >}}

## 簡介

Spryker 專門做商業相關的軟體平台，無論 B2B, B2C, marketplace 等都有涉略

### PBC
Packaged business capabilities，是某些包好的軟體提供商業使用，有點像是商業版 AWS marketplace

### APC
是 Spryker 設計出來讓 PBC 和客戶 (使用 PBC 的用戶) 開發出來的軟體可以有相同的介面可以溝通

## 架構

分為兩條線，第一條綠色的是由 PBC 的軟體去通知或是呼叫客戶的軟體，另一條藍色的就反過來，是由客戶的軟體去呼叫 PBC ，概念都差不多，為了確保訊息的順序，大部分的 component 都是 FIFO

![Architecture](/img/2022/12/Spryker-Application-Composition-Platform-Architecture.png)

- PBC -> Tenant (綠色)
  - PBC 將訊息放到自己的 SQS FIFO
  - 將訊息打到 ACP 中給 PBC 的 API gateway
  - Lambda 依照 DynamoDB 中的 Role 去設定 Rule 發送到 SNS FIFO，以確保收到的是使用該 PBC 的用戶
  - SNS 將訊息送到 Tenant 的 APP

---

- Tenant -> PBC (藍色)
  - Tenat 將訊息放到自己的 SQS FIFO
  - 將訊息打到 ACP 中給 Tenat 的 API gateway
  - Lambda 將訊息發送到 SNS FIFO
  - SNS 將訊息送到 PBC 的 APP

## Recap
- 利用相同的方式讓兩邊軟體可以溝通，這樣可以降低開發成本
- SNS rule 可以是動態的，這樣做群發時可以利用資料庫來做 Filter，就不會讓訊息被其他不相干的人也接收到
