+++
title = "AWS MindTickle Multi Region Architecture"
date = "2023-01-15T16:03:42+08:00"
cover = "/img/2023/01/MindTickleMultiRegionArchitecture.png"
tags = ["architecture"]
description = "來看看 MindTickle 跨地區解決方案，如何服務不同地區的使用者以及如何快速擴張到不同區域"
+++

{{< youtube PgeQufaQy7I >}}

## 簡介
MindTickle 主要業務是處理銷售管理，以及知識管理，較為有名的客戶有 Splunk, Wipro, Johnson & Johnson 等等，主要服務在亞洲以及美國

## 遇到的問題
如何在不同的地區都給予快取，並且在沒有快取的情況下，呼叫最近的 Region 伺服器，給予計算

## 架構
![](/img/2023/01/MindTickleMultiRegionArchitecture.png)

- 使用者向 CloudFront (CDN) 發送請求
- 如果有快取，就回應使用者
- 如果沒有，呼叫 Lambda Edge
- Lambda Edge 會呼叫真實的服務，利用 Route 53 Firewall 限制 Lambda Edge 呼叫的 Domain 來達到呼叫指定的 Regions
- 解析到最後的 IP 後，向該 EKS 的 ALB 發送請求

## 結論
利用 Doamin 查詢限制來做到同一份程式可以部署在不同快取地點，並用 infra 來控制該快取地點可以問到的服務，再再顯示區分程式以及架構的好處，在程式好寫好開發的前提下，此架構發揮了滿好的效益
