+++
title = "AWS Architecture CloudPlex Media"
date = 2022-11-16T21:15:50+08:00
cover = "/img/2022/11/CloudPlexMediaAWSArchitecture.svg"
description = "Serverless 很好用，但是整套服務能都全部都用 Serverless 兜出來的服務並不多，來看看韓國雲端影片處理服務商 CloudPlex Media 如何運用 AWS serverless 完成一整套影音平台"
categories = ["architecture"]
tags = ["architecture"]
+++

{{< youtube zqiNLMmEeSo >}}
## 簡介
CloudPlex Media 提供客戶 VoD (隨選視訊) 與 Streaming (影像串流) 的服務，不僅能夠處理 DRM (數位版權管理)，還有 AI 做字幕自動產生，以及各式影像轉碼服務

## 架構
該服務，簡單分成三大項

1. Web
2. VoD (隨選視訊)
3. Streaming (影像串流)

我們將分開講解

![Architecture](/img/2022/11/CloudPlexMediaAWSArchitecture.svg)

### Web
我們可以關注左上角藍色的線路

1. 首先全平台的網頁 (HTML, JS, CSS) 都是放在 S3，使用 S3 [靜態網頁託管](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)的功能提供給客戶
2. 當前端網頁載入後，所有功能包含上傳影片、轉碼、影片管理、分析等等，都是由 AWS API Gateway 去做代理，實際功能都以 AWS Lambda 去做實現

### VoD
我們可以關注上方綠色的線路

- 當使用者使用上傳影片後，Lambda 會讓使用者上傳影片到 S3
- 當 S3 上傳完成後，會啟用 Media Convert 去做影片的轉碼，或是丟到 Media Package 去做 DRM 的管理
   - 當影片轉碼完成後，會啟用 Step Function 去完成一連串的處理
   - 例如 (關注上方黃色線路)：
     - 使用 AWS Rekognition 去做裸露影像辨識
     - 用 Transcribe 做語音轉文字，用 Translate 翻譯轉換後的文字，自動產生字幕
   - 最後將資料放到該去的地方
- 其他人就可以透過 AWS S3 或是 CloudFront 去存取該影片

### Streaming
我們可以關注下方黑色的線路

- 當使用者開啟直播，前端會將影像用 RTMP 的方式串流到 Media Live
- 這時候，依照觀看流量，利用 Fargate 去自動擴展，來服務觀看者
