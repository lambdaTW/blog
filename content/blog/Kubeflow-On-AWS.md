+++
title = "Kubeflow on AWS"
date = 2022-11-12T10:59:12+08:00
categories = ["AWS"]
tags = ["AWS"]
+++
{{< youtube EtEg6P-XLvg >}}

## 問題
![ML EC2 Architecture](/img/2022/11/MLEC2Architecture.png)
傳統使用 EC2 給各個 ML 工程師使用，會導致

 - 難以重現環境（如果工程師不是用 container）
 - 資源難以被完整分配
   - 例如：某人開了一個 GPU + CPU 的 EC2，但是它正在訓練的模型只吃 CPU
 - 難以進行大規模的訓練
   - 除非該 ML 工程師自己寫一堆 script 並且有辦法 scale EC2 ，否則相對困難
## Kubeflow
![EKS OpenSource Architecture](/img/2022/11/EKSOpenSourceArchitecture.png)
這邊演講者列出以下架構去服務 ML 工程師
1. Route53 + Load Balancer + AWS Certificate Manager 去提供 HTTPS 服務
2. 用 AWS Cognito 做身份驗證
3. 在 ELK 中部署 Kubeflow 去做訓練的服務
4. 用 Istio 去做服務的導流
5. 可以直接使用 AWS 靜態檔案服務或是資料庫服務，例如： AWS Deep Learning Containers, Amazon Elastic File System，Amazon FSx，AWS S3，RDS

![EKS with AWS Stateful](/img/2022/11/EKSwithAWSSStateful.png)
