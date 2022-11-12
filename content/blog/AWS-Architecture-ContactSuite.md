+++
title = "AWS Architecture ContactSuite"
date = 2022-11-12T11:19:38+08:00
categories = ["architecture"]
tags = ["architecture"]
+++
{{< youtube BPvr0qWpJlA >}}
## 簡介
ContactSuite 是一家做客戶服務系統的公司，讓客服可以在一個介面上完成所有客戶服務所需要的事情

- 可以馬上知道打過來的客戶的 email 或是 ID
- 整合到現有的 CRM 系統
- 整合現有知識庫

## AWS 架構
![ContactSuite AWS Architecture.png](/img/2022/11/ContactSuiteAWSArchitecture.png)

依照服務， ContactSuite 架構可以粗分為以下三個資料流
### 電話或文字訊息
 - 客戶打電話或是使用文字客服，該資訊會用 Amazon Connect 利用 IVR (互動式語音應答) 的方式，讓一些比較制式化的流程讓 lambda 處理
 - 處理後的資料透過 Amazon ECS 轉換成客服人員有用的資訊
 - 顯示在 ContactSuite 的頁面上
 
### Email
 - 透過 AWS SES 收件
 - Lambda 處理後送至 DyanamoDB
 - 顯示在 ContactSuite 的頁面上
### 報告
 - ContactSuite 的系統會記錄有用的資訊到 RDS
 - 利用 AWS SES 寄送 Report 給需要看到的人員
