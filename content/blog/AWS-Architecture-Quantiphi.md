+++
title = "AWS Architecture Quantiphi Real-Time Call Analytics"
date = 2022-11-12T12:26:12+08:00
cover = "https://cdn.quantiphi.com/2021/07/dc2fc96e-telecom-homepage.png"
description = "靠 AWS 就可以達成智慧客戶服務，來看看如何使用 Combo 技，簡單整合 AWS 的服務就可以提供 AI 服務"
categories = ["architecture"]
tags = ["architecture"]
+++

## 簡介
Quantiphi 是一家 AI 公司，提供各式解決方案，包括

- 金融服務
- 教育
- 健康
- 保險
- 智造
- 多媒體娛樂產業
- 能源
- 公共設施
- 零售與民生消費
- 網路

## 架構
![Quantiphi AWS Architecture](https://d1.awsstatic.com/partner-network/QuickStart/datasheets/quantiphi-real-time-analytics-architecture-diagram.ceeb134f5a044dd97d79a61c20763c8cb5a245b4.png)

- 運用 Amazon Chime Voice Connector 讓企業可以整合電話線路讓客戶服務人員可以使用軟體接聽來電
- 運用 Kinesis Video Stream 把來電進行串流
- 來電得開始與結束運用 EventBridge 來寄送通知到以下三個 SQS Queue
  - Transcription
    - 從 Kinesis Video Stream 讀取來電
    - 運用 Amazon Transcribe 把語音轉文字記錄下來並存到 DynamoDB
    - 將來電轉成 mp3 存到 S3
  - Keyword-extraction
    - 截取關鍵字並存到 DynamoDB
    - 關鍵字送回 Quantiphi Web Application
  - Metadata-extraction
    - 利用 Amazon Comprehend 擷取 Metadata
    - 存到 DynamoDB
    - 並傳回 Quantiphi Web Application
- 最後在 Quantiphi Web Application 就可以在服務人員接到電話前拿到上方所有資訊，讓客戶服務人員可以在接電話前就可以相對了解來電者的需求，加速整體服務，亦提高服務滿意度

## 架構之我建
![Quantiphi Stream Architecture](/img/2022/11/QuantiphiStreamArchitecture.drawio.svg)
Quantiphi 運用 API Gateway 串起整個服務，如果在沒有特別資安的考慮下面，其實可以使用 DynamoDB 的 Stream 服務，這樣 Web 服務就可以簡單的去更新該來電者的資訊給客戶服務人員

> DynamoDB Streams are charged at $0.02 per 100,000 read operations.

## Ref

[Quantiphi Real-Time Call Analytics on the AWS Cloud](https://aws.amazon.com/quickstart/architecture/quantiphi-real-time-call-analytics/)
