+++
title = "AWS Account Management Volkswagen Financial Services"
date = "2022-12-18T10:07:25+08:00"
cover = "img/2022/12/AWS-Account-Management-Volkswagen-Financial-Services.png"
categories = ["AWS"]
tags = ["AWS"]
description = "Volkswagen Financial Services 運用 AWS 服務做不少事情，帳號管理從原本的單一 AWS 帳號擴增到 1,400 帳號來分別管理服務，來看看他們如何管理他們的 AWS 帳號"
+++

{{< youtube r3g1Nym-ebY >}}

## 簡介
Volkswagen 汽車集團，旗下有十個汽車品牌，包含 Volkswagen, Audi, LAMBORGHINI... 等等，Volkswagen Financial Services 的服務包含旗下集團使用的金融，租借，移動服務，以及保險等，服務全球 48 個目標市場

## 遇到的問題
Volkswagen Financial Services 開發團隊，從 2017 開始使用 AWS ，從一個單一帳號開始，裏面有包含各個服務的開發環境，整合環境，以及正式環境，開發人員如果沒有被限制，就有可能刪除掉別人的服務，甚至是線上的服務

## 解決流程
 - 分散 workload 到不同的 workload account
 - 導入 staging 的概念，讓每個 workload 擁有 4 個 workload account

## 管理問題
在超多個 AWS account 下，管理變成很大的問題，他們利用 AWS IAM 讓開發人員可以用 SAML 登入各個 workload，並且管理使用類似暫時性 Token 的方式讓開發人員可以進入線上服務，而且所有動作是被監控的，另外，他們叫他們自己家建立的 Red Hat SSO 為 Bifrost (在漫威裡面的彩虹橋)，讓不同的 AWS account 可以互相連接

## Compliance & Auditing
### Compliance
Volkswagen Financial Services 利用單一個 AWS 帳號使用 AWS Config 中心化追蹤所有帳號的資源設定變化，並在去年導入 AWS Inspector ，讓他們可以知道某些設定的更改到底會對哪些資源造成影響


### Auditing
Volkswagen Financial Services 運用單一個 AWS 帳號並運用 CloudTrail 來做 Auditing

## 自動化
為了管理大量的帳號，Volkswagen Financial Services 使用 AWS Organization 以及 CloudFormation 的服務，讓他們可以統一管理一些最基本的服務設定
