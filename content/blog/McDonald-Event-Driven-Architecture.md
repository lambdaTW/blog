+++
title = "McDonald Event Driven Architecture"
date = 2022-11-08T21:16:20+08:00
categories = ["architecture"]
tags = ["architecture"]
cover = "https://miro.medium.com/max/1400/1*gCOnmHq4jXNjSX8Jp0NgOA.jpeg"
description = "本篇介紹 McDonald 如何運用 Event Driven 來服務全球等級的客戶"
+++
## 簡介
麥當勞，就是早餐滿福堡很好吃的那家，不服來辯

## Event Driven Architecture 用於哪些系統
> mobile-order progress tracking and sending marketing communications

很明顯都是很經典的應用情境，包含點餐進度追蹤和寄送一些商用訊息

## Components
![Components](https://miro.medium.com/max/1400/1*P0mtpk5Jk0rBQZuJl7zWIA.png)

比較特別的是他們有工程友善的 SDK 去處理訊息，包括 schema 檢查，訊息寄送，錯誤處理等等

## 架構
![Architecture](https://miro.medium.com/max/1400/1*gCOnmHq4jXNjSX8Jp0NgOA.jpeg)

- 外部訊息經過 Event Gateway 處理權限驗證等 (此處沒有說明怎樣辦到的)
- 經過驗證的外部訊息以及內部訊息都會打到 Producer 上面
- Producer SDK 經過 Schema validation 後送到 Primary Topic 上
  - 沒過送 Dead-letter
    - 可以自動化處理的錯誤的由 Lambda 重送
    - 其他人工處理後直接送到 Primary Topic
  - 送不出去存 DynamoDB
    - 由 Lambda 重送
- Consummers 接收後一樣經過 SDK 去處理

## Data governance
![DataGovernance](https://miro.medium.com/max/1400/1*LvV2J6pcNdSjRf0gSA4yAw.jpeg)

- 符合 Schema validation 就處理
- 不符合就丟到 Dead-letter Topic ，工人智慧處理
- 可以快速變更 Schema

## Cluster autoscaling
![ClusterAutoscaling](https://miro.medium.com/max/1248/1*0WQ4CpnzhlNthWt4uGLO3g.jpeg)

這裡比較好玩的是有用到 Step Function 去 Trigger Lambda 做 Partition re-assignment

## Domain-based sharding
![DomainSharding](https://miro.medium.com/max/1400/1*cCR1EaCLRhUKG8AruzJ43g.jpeg)

依照 Domain 去拆分，這邊感覺類似微服務，但又有點不太像，可以想像壞掉時的差異如下

- 微服務
  - 通知系統壞了但是首頁是好的
- Domain sharding
  - 點餐進度通知壞了，但是還是有收到廣告通知 (前提：某通知系統 Backend 是好的以及通知系統前端是好的)

## Refs
[Behind the scenes: McDonald’s event-driven architecture](https://medium.com/mcdonalds-technical-blog/behind-the-scenes-mcdonalds-event-driven-architecture-51a6542c0d86)

[McDonald’s event-driven architecture: The data journey and how it works](https://medium.com/mcdonalds-technical-blog/mcdonalds-event-driven-architecture-the-data-journey-and-how-it-works-4591d108821f)
