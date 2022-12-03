+++
title="AWS Architecture MyHeritage"
date=2022-12-03T14:11:06+08:00
description="Region resources quota 常常是絆住整體架構的瓶頸，MyHeritage 使用 Lambda 和 SQS 突破單一 region GPU instance 上限，提供更快速的服務給使用者"
cover="/img/2022/12/AWS-MyHeritage-Architecture.png"
+++

{{< youtube 0-db3wFRfSc>}}

## 簡介
MyHeritage 提供家族族譜分析以及家族 DNA 分析，最近推出新的服務 (Deep Nostalgia) 是讓你上傳一張祖先的照片，他們可以運用 AI 的方式令他活過來 (類似哈利波特牆上的畫像)

## 架構
![Architecture](/img/2022/12/AWS-MyHeritage-Architecture.png)

### Single region
就正常的架構 (關注咖啡色線路)，在單一 region 時，他們使用一個 SQS 來做工作分配，以及動態的增加 GPU instances 的依據

- 使用者上傳圖片到 S3
- 使用者呼叫 API 要求處理圖片
- 使用者等待工作處理完 (polling)
- Main queue 獲得該工作
- GPU instance 從 Main queue 獲得工作
  - 處理完成後通知 result queue
  - 將處理後的影像放到 S3
- State manager 從 Result queue 得知某工作做好了
  - 將 DB 中的狀態改變
- 使用者得知工作完成，並從 API 獲得 cloudfront 中影片的位置


### Multi-region
當使用者越來越多，單一 region 無法負荷其工作量， MyHeritage 多做了幾件事情

1. 增加一個 Lambda 分配工作
2. 在原本的 region 增加一個 SQS

整體流程變成：
- 使用者上傳圖片到 S3
- 使用者呼叫 API 要求處理圖片
- 使用者等待工作處理完 (polling)
- Main queue 獲得該工作
- Lambda 根據原 region SQS 的工作量來看要不要分配工作到各個 region 的 SQS (關注藍色線路)
  - 如果原本的 region SQS 工作量還可以負荷就放到原 region SQS 以減少跨 region 的資料傳輸
  - 將工作分配到各個 region 的 SQS
- GPU instance 從該 region SQS 獲得工作
  - 處理完成後通知 result queue
  - 將處理後的影像放到 S3
- State manager 從 Result queue 得知某工作做好了
  - 將 DB 中的狀態改變
- 使用者得知工作完成，並從 API 獲得 cloudfront 中影片的位置
