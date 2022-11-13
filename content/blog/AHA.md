+++
title = "Chick-Fil-A AHA"
date = 2022-11-13T09:59:48+08:00
cover = "/img/2022/11/Chick-Fil-A-AHA.drawio.png"
description = "來看看 Chick Fil A 如何運用自身企業資源讓 AI 應用硬著陸"
+++
## 簡介
Chick-fil-A 是一間總部位於美國喬治亞州 College Park 的美式連鎖速食店，以雞肉三明治為主 (圖片看起來其實是漢堡) ，目前有超過 2,200 間連鎖餐廳

## 遇到的問題
在速食產業 (QSR) 中，大部分食物都會有所謂的有效期，在食物放到備料區以後，如果超過該食物的有效期，食物就會被倒掉，以保證食物比較不會有食安上的問題，傳統上都是人工去點擊計時器，但是

> “Ain’t nobody got time for that!”

如果說人員忘記點擊開始，下一次想起來再來去點，該食物的計時是完全沒有效用的

## POC
為了解決這個問題， Chick-Fil-A 直接設計一個新的托盤，讓放食物的鍋子邊邊可以被 3D camera 掃描，並且直接在鍋子上給予 Bar code，這樣就可以知道是哪種食物被放上來，用以重設計時器

![Pan and Bracket](https://miro.medium.com/max/1400/1*cssV3Ug0FAmxx74YlmgTwg.png)

為了做到該功能，他們在托盤與鍋子上面架設 Camera 並且運用 NCU 進行 Bar code 偵測

![NCU install](https://miro.medium.com/max/1400/1*iH80Ibk3pN6HJWZxhmQmew.png)


## AI
何時啟用 Bar code 偵測是一個不小的問題，原本他們使用 3D 相機給予的深度資訊來做 pixel counting，但是，如果 Camera 被動到，或是安裝人員安裝時，把相機安裝比預設的高兩英吋，就有可能讓 Bar code 難以被偵測 (pixel counting)，為了解決該問題，他們打算捨棄傳統的 pixel counting 改用 AI 來判斷鍋子是否安裝到托盤上

### Training Data
 - 用 3D camera 擷取沒有鍋子的影像
   - 為了模擬真實店家情況，他們在影像中加入一些會在店內出現的東西，例如：三明治
 - 一樣方式擷取有鍋子的影像
 - 解析度: 1280 x 720
 
### Neural Network Implementation
很簡單的一層 dense 一層 dropout 的 CNN 就達到不錯的效果，以下為詳細訓練的參數

- grayscale
- 96 x 96 pixels
- 2 epochs of 600 batches
- 32 images per batch
- 30 validation batches

### Feature
運用鍋子的深度，以及計時器的資料，可以知道裡面有多少雞肉，那就可以知道多少肉在何時被煮起來，使用油品的紀錄也可以被知道

## 進化 AHA 2.0
完成了相關實驗後，他們要挑戰安裝到 2800 以上的店家，他們做了不少進化，例如

- 如何讓平板 (顯示器)，可以在廚房惡劣的環境下還能使用，而且成本要夠低到可以擴展到所有店家
- 重新設計 機器 - 相機 - 托盤，讓顯示器不會擋到相機
- 加強 AI 的準確度
- 壓力測試鍋子上雷射上去的 Bar code 經過刷洗後，耐用度如何
- 找到更好的 Bar code，就算工作人員的手微微擋住一部分條碼也可以辨識
- 加上一些傳統電腦視覺的算法，讓條碼更好被判斷
- 各式 Bar code 除錯，例如：地板上有條碼，不該被混淆
- 優化深度轉乘雞肉量的轉換

### 硬體改進
在部署 prototype 到一些店家後，開始進行 [Failure Mode and Effects Analysis](https://en.wikipedia.org/wiki/Failure_mode_and_effects_analysis) 為了要讓他們的解決方案可以支援到 24 常駐

- 一開始想辦法讓 Chrome 直接支援 Intel RealSense camera，這樣就不用 NCU 了，但是失敗
- 找到夠穩定的 Chrome 平板，最後找到一個支援 Linux 的平板
- 讓研究人員確定該平板跑 AI model 是穩定的

最後平板只有 USB, Power, Camera，他們認為已經有足夠少的 failure point ，壞掉機率也夠低，足以支撐他們的服務

## Ref
[AHA 2.0](https://medium.com/chick-fil-atech/aha-2-0-623a0ec1cacc)
[Edge AI in a Smarter Chick-fil-A](https://medium.com/chick-fil-atech/edge-ai-in-a-smarter-chick-fil-a-2e2112f5e5d8)
