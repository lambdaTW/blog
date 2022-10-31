---
title: "AWS Architecture Study Buzzdial"
date: 2022-10-26T21:21:46+08:00
---
## 簡介
Buzzdial 是一家製作電視 Live 節目即時互動的公司，可以想像主持人問答，在家中看電視的你也可以使用 App 與其他相同收看電視的人們互相交流，目前已經找不到網站，看起來應該是已經收掉了，但是還是可以找到[相關資訊](https://nz.linkedin.com/company/buzzdial?trk=public_profile_experience-item_profile-section-card_image-click)
## 遇到的問題
當遇某一特殊事件，導致大量使用者在同一時間使用服務時，需要保證網路以及伺服器可以負荷，並且該特殊事件可能不會天天發生，如果使用傳統架構會需要大量的初始建構成本
## 導入 AWS 的架構
{{< figure src="https://d1.awsstatic.com/architecture-diagrams/customers/buzzdial-arch-diag.1cd13ca7855b730ce72977f106964923745b5ca2.png" title="Buzzdial AWS Architecture" width="100%" >}}
### 架構拆想
可以看到 Buzzdial 使用了兩個分開的 ASG (auto scale group)，一個是做 Web，另一個是做管理的部分
#### 第一次猜想
1. 可以想像，電視製作或是主持人可以透過 API 去發送事件廣播 (broadcast)，讓所有使用者去看到或是被通知有一個互動發生 (例如：新的題目)
2. Web App 中的 Deamon 可以透過 Logging DB 來通知使用者有新的事件

#### 第二次猜想
> We queue those records and store them in our databases to allow us to report back to clients on the success of their broadcast event, and also provide detailed business reports that we review internally. In addition, we monitor our application in real time as it scales up and down to check performance and error rates, and undertake extensive and aggressive load testing to predetermine its capabilities. If there any concerns, we rework the application and AWS infrastructure accordingly
1. 很明顯 Logging DB 主要是用來留存每次互動，所以想應該有少畫一條線，是 Broadcast Tool 使用 App DB，或是利用某些方式去 Trigger App 端的更新
2. 在調整架構時使用了以前的資料
#### 有趣的點
> the service provider can still scale in 10 to 15 minutes to support 10s of thousands of concurrent users.
- 很明顯架構是使用比較傳統的 EC2 才會導致增加流量要等那麼久
> Buzzdial would have also required six full-time equivalent administrators to manage 60 physical servers, costing about US$350,000, while the AWS infrastructure requires about 20 percent of a single full-time employee’s time to administer.
- IT 人力成本在國外真的很高
> Caching is undertaken at the Amazon EC2 level to prevent the database infrastructure from being overloaded during periods of high demand
- 使用 Local cache 而不是 redis 之類叢集快取
### 架構之我建
#### Amazon Kinesis
在這個年代，如果要重新製作類似的軟體服務，可以簡單使用 [Amazon Kinesis](https://aws.amazon.com/tw/solutions/implementations/aws-streaming-data-solution-for-amazon-kinesis/) 來達成，不僅在 streaming 的部分直接幫你做掉，還可以紀錄並分析使用狀況，最後在資料整合的部分也可以使用 Kinesis Data Analytics 來達成
{{< figure src="https://d1.awsstatic.com/Solutions/Solutions%20Category%20Template%20Draft/Solution%20Architecture%20Diagrams/aws-streaming-data-using-api-gateway-architecture.1b9d28f061fe84385cb871ec58ccad18c7265d22.png" >}}

#### Out of AWS
如果是比較 general 的方法，我想我會使用現在比較通用的 event sourcing 的方式，搭配讀寫分離來製作這一個軟體服務

![My Buzzdial Architecture](/img/2022/10/MyBuzzdialArchitecture.svg)
## Ref
[Source of AWS case study](https://aws.amazon.com/solutions/case-studies/buzzdial/)
