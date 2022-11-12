+++
title = "AWS Architecture Study Halter"
date = 2022-11-06T20:30:18+08:00
categories = ["architecture"]
tags = ["architecture"]
cover = "/img/2022/11/HalterAWSArchitecture.png"
description = "Halter 提供遠端農場管理，來看看他是如何使用 IoT 和 AWS 服務客戶的吧"
+++

{{< youtube j-lPgPGBTwQ >}}

## 簡介
Halter 提供遠端管理農場的軟體，他們使用一個叫 `Collar` 的東西放在牛身上

 - 用太陽能為能源
 - 有 GPS
 - 可以發出聲響以及震動 (驅動牛去某個地方)
 - 觀測牛隻健康

有 APP 讓農夫建立虛擬柵欄

## AWS 的架構
![Halter AWS Architecture](/img/2022/11/HalterAWSArchitecture.png)
- `Collar` 使用 wifi, MQTT 以及 IoT Core 將資料送到雲端
- 除了 wifi 以外，他們使用了 LoRa 協定，將資料送到 station 再由其送至 AWS EC2
- 上面的資料都會用 binary 的方式送到 Kinesis (由於有兩個 source 所以可能會重複)
- 去除重複資料後，利用 Lambda 並解成 JSON 格式後送到另一個 Kinesis
- 利用 Kinesis 做 Data aggregation (by GEO index)
- 將上面整理好的資料存到 S3 (此處利用農場 ID 當作 partition key)
- 當使用者要求牛群資料時，會傳送需求到 ECS
- ECS 傳送運算需求給 Athena 做運算 (十秒可以處理兩千五百萬筆資料)
- 完成後，ECS 從 S3 獲取 Athena 運算完成的資料，整理後傳給使用者
### Key Notes
- LoRa 可以傳送好幾公里，其資料率可以從 27 Kbps 到 0.3 Kbps
- 很多服務使用者其實可以等待
