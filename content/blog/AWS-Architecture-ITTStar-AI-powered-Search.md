+++
title = "AWS Architecture ITTStar AI Powered Search"
date = 2022-11-14T20:44:42+08:00
cover = "/img/2022/11/ITTStartAIpoweredSearchArchitecture.svg"
description = "ITTStar 是一間軟體服務商，提供軟體服務給來自全球的客戶，來看看他如何運用 AWS 架構提供精品商品的交易平台更便利的服務"
categories = ["architecture"]
tags = ["architecture"]
+++

{{< youtube Qi9soN5bpU4 >}}

## 簡介
[ITTStar](https://www.ittstar.com/) 主要是幫助企業做上雲的資訊顧問服務，提供老舊系統雲端遷移以產業上雲服務，網站客戶寫很多，但是我看得懂的只有 Whirlpool 和 AT&T

## 要解決什麼問題
AI powered search 主要是做商品資訊辨識，可以想像你上傳幾張圖片，他可以幫你辨識出這是哪個品牌、哪個型號，大概的市場價格，主要提供給精品產業品牌以及零售與批發的估價師

## 架構
![Architecture](/img/2022/11/ITTStartAIpoweredSearchArchitecture.svg)

- Amplify 的 App 使用 Cognito 做身份驗證
- 使用者上傳的影像會傳到 S3
- S3 的更新會通知 Lambda 去 Trigger Rekognition 去做物品辨識
- 如果影像中有大量文字，他們會用 Textract 去擷取，並且去更正 Rekognition 辨識出來的資訊
  - 可能可以知道品牌，型號，地點等等
- 最後把資料都丟到 DynamoDB
