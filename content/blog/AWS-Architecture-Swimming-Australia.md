+++
title = "AWS Architecture Swimming Australia"
date = 2022-11-21T18:53:55+08:00
cover = "/img/2022/11/AWS-Architecture-Swimming-Australia.png"
description = "澳洲如和透過 AI 最佳化奧運游泳比賽比賽成績"
+++

{{< youtube 1VcpCVe3tLQ >}}

## 簡介
除了運動選手本身的能力以外，在接力比賽中，如何讓所有團隊都能拿到比較好的獎項，排隊伍就是個很關鍵的點

### 舉個例子
|  秒/國| 美國 A | 美國 B | 澳洲 A | 澳洲 B |
|  ---  |  ---   |   ---  |  ---   |   ---  |
| 選手1 |   2    |        |   1    |        |
| 選手2 |   3    |        |   5    |        |
| 選手3 |        |   2    |        |   3    |
| 選手4 |        |   1    |        |   1    |
| 合計秒|   5    |   3    |   6    |   4    |
> 如果 美國 A VS. 澳洲 A ，美國 B VS. 澳洲 B

依照上面的隊伍排列，澳洲全輸，但是如果這樣排
|  秒/國| 美國 A | 美國 B | 澳洲 A | 澳洲 B |
|  ---  |  ---   |   ---  |  ---   |   ---  |
| 選手1 |   2    |        |   3    |        |
| 選手2 |   3    |        |   5    |        |
| 選手3 |        |   2    |        |   1    |
| 選手4 |        |   1    |        |   1    |
| 合計秒|   5    |   3    |   8    |   2    |

就可以在 B 組拿到較好的成績

## 架構
![Architecture](/img/2022/11/AWS-Architecture-Swimming-Australia.png)

這裡分三個部分
  - Web
  - Data pipeline
  - Auto training

### Data pipeline
- 首先來看看上方黃線的部分
  - 首先來看看上方黃線的部分，資料會由各個不同的方式以及格式 (XML, JSON) ，放到 S3 的 Landing bucket
- 再來看到左半邊紫色的線路
  - 透過 AWS Glue 將資料整理成統一格式後放到 S3 的 Data Lake bucket
  - AWS Glue 將資料整理資料科學家想要的樣貌後放到 S3 的 Curated bucket
  - 資料科學家可以透過 SageMaker notebook 連接 Athena 使用 SQL 語法做資料操作以及訓練

### Web
關注左下角黑色線路

- 使用者透過手機可以從 API Gateway 呼叫 Lambda 取得想要的資料
- 其中包含使用 SageMaker 中已經訓練好的模型，Lambda 透過 Trigger SageMaker Endpoint，來獲得使用者想要的 Inference (AKA: 怎樣的隊伍排起來對這系列的比賽比較好)

### Auto training
由於比賽資料常常更新，但是資料科學家不會每一次都有時間來重新 Train AI model ，所以他們讓使用者可以控制，是否使用新的資料去重新 Train 一個新的 AI model (關注綠色的線路)

- 使用者 Trigger Lambda
- Lambda Trigger StepFunction
- StepFunction 將會完成一整套流程
  - 使用 Athena 獲取資料 (包含最新的資料)
  - 使用 Training 的程式碼去重新產生 AI model
  - 驗證產生出來的 AI model 沒有特別的問題 (可以產出資料，且該資料沒有超出設定的預期)
  - 將產生出來的 AI model 部署到 Sagemaker Endpoint
