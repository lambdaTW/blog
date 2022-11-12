+++
title = "Chick-Fil-A Architecture"
date = 2022-11-12T14:22:22+08:00
cover = "http://www.chick-fil-a.com/-/media/images/cfacom/default-images/chick-fil-a-logo-vector.ashx"
description = "本篇介紹美國連鎖速食業，從 Docker 到 Kubernetes，在各個千家商店使用的混合雲架構的演化"
+++

## 簡介

## 2017 架構
- 在這個階段已經有 MQTT 作為訊息傳遞
- 運用 Docker 去做大部分的事情

![Chick-Fil-A 2017 Architecture](https://res.infoq.com/presentations/chick-fil-a-k8-clusters/en/slides/sl5-1531966648307.jpg)

## 2018 架構
- 改用 K8s
- 因為 K8s 可以更簡單的用 Prometheus 做監控
- 用 Fleet 去做部署

![Chick-Fil-A 2018 Architecture](https://res.infoq.com/presentations/chick-fil-a-k8-clusters/en/slides/sl6-1531966652646.jpg)
## Equirements
> 3 NCU with Quadcore processor, 8 GB RAM, SSD

![Equirements](https://res.infoq.com/presentations/chick-fil-a-k8-clusters/en/slides/sl11-1531966645373.jpg)

## Goals
- 非工程人員也可以簡單安裝
- 可以遠端管理
- 可以自動發現已經安裝的裝置和裝置上的 K8s 叢集
- 可以自己恢復，可以做到 HA

## Bare Metal Cluster K8s at scale
![Bare Metal Cluster K8s at scale](https://res.infoq.com/presentations/chick-fil-a-k8-clusters/en/slides/sl14-1531966652381.jpg)
### Highlander
為了解決三台機器在內網，開機時要找到誰當 K8s cluster 的 Leader，Chick-Fil-A 自己做了這個工具去
- 確認誰是老大
- 執行 RKE
- 安裝一些基本的 Pods (ex: Istio...)

### Resetting Cluster State
當機器有任何問題時，透過 OverlayFS 讓遠端可以去清除機器 (回到 Golden image 的狀態)
  
### [Hooves Up](https://github.com/chick-fil-a/hoovesup)
Chick-Fil-A 自己做的 Ansible 工具，當機器啟動時，可以自動註冊到 AWS SSM

### Fleet
為了要部署到全部的店家中，Chick-Fil-A 遇到了幾個問題

- 如果有一千的叢集，部署中有 900 個部署成功，另外 100 個失敗，那要
  - 全部 rollback?
  - 手動解決並且處理掉那 100 個失敗?

最後他們建立了 Fleet 來做部署，這邊他們使用現有的 message broker (MQTT & Amazon SQS)，來通知地端的 Cluster 去做部署

> Fleet Ecosystem Components
- Fleet Client
- Fleet Server API
  - 產生部署需要的 k8s yaml
  - 管理 Cluster 的 Git
  - 部署狀態監控
- Atlas
  - 存放已經測試過的 k8s yaml
- Vessel
  - 放在店家的 Agent 用來做部署
- Dashboards

## Entire Flow

- 在工程機房
  - 使用 Golden image 去安裝系統，內建 Ansible
- 運送機器
- 店家
  - 安裝人員打開機器
  - Sherlock 運行，確認在店內網路，並且信任機器
  - Highlander，找到叢集需要的三台機器運行 RKE
  - 每台機器個別註冊 AWS SSM (運用 Hooves Up)
  - MQTT 通知 Fleet Vessel 自動從線上抓軟體並且部署

## Ref
[InfoQ](https://www.infoq.com/presentations/chick-fil-a-k8-clusters/)
