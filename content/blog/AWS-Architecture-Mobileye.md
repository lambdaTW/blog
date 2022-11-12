+++
title = "AWS Architecture Study Mobileye"
date = 2022-11-04T21:39:03+08:00
categories = ["architecture"]
tags = ["architecture"]
cover = "/img/2022/11/MyMobileyeIoTArchitecture.png"
description = "Mobileye 是提供汽車自駕 (ADAS) 的晶片以及演算法的廠商，本篇文章會介紹他是如何處理每天千萬級的資料量"
+++
{{< youtube 5hjkSczrke4 >}}

## 簡介
Mobileye 是提供汽車自駕 (ADAS) 的晶片以及演算法的廠商，依照影片的說明已經有七千萬車輛使用他們家的解決方案
## AWS 的架構
![Mobileye Architecture](/img/2022/11/MobileyeAWSArchitecture.png)

- 車子透過 REST API 傳送資料到 AWS S3
- S3 透過更時會觸發 SQS
- EKS 依照 SQS 的長度去擴增 worker 做資料前處理
- EKS 處理完資料後分別送到
  - AWS Step Function
  - S3
  - Elasticsearch
  - CockroachDB
- 最後使用另一套 EKS 作為服務的介面
### 架構演進
- Service
  - Spark streaming, AWS EMR, KAFKA
  - Lambda functions, 為了拆分服務
  - EKS serve Lambda functions 為了 Scale
- DB
  - Postgres RDS
  - RIDB, 查了很久，找不到是啥，以下是猜的
    - Reserved Instance (EC2) for AWS RDS
    - 特規的商用資料庫
    - 可能字幕有問題
  - 為了查詢文件 (document) 使用 Elasticsearch
  - 發現 Elasticsearch 對於經常更新不太友善，使用 CockroachDB 並且將其部署到 EKS 上面，使他們可以 auto scale 還享有高可用性
### 未來展望
- 雲端圖像處理
- 降低成本
### Key Notes
- 為了節省成本，所以儘管有七千萬的車子使用該家產品，但是只有少部分的資料有打到雲端，每天約一千萬的資料量
- 由於 AWS Lambda 在同時執行 Lambda 是有其帳號上限的 (per account per region)，我猜是因為這樣才轉換到 EKS 上
- 架構是慢慢演化的
### 架構之我建
![MyMobileye IoT Architecture](/img/2022/11/MyMobileyeIoTArchitecture.png)

假設
  - 5G 網路速度夠快
  - 大家都是電動車

我們就可以讓行駛中的車輛將影像分別傳送到
- 正在充電的閒置車輛
- AWS 雲端

利用雲端可拓展以及先到先贏的方式，如果其他車輛已經將圖片處理完成，那我們就可以取消雲端上面的工作，用以減少成本，也可以利用基本圖像分割的方式，把大量的圖片資訊分別送給不同的閒置車輛，達到平行處理
