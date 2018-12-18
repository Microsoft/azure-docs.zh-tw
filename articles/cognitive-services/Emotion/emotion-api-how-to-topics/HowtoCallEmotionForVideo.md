---
title: 範例：呼叫影片的表情 API
titlesuffix: Azure Cognitive Services
description: 了解如何呼叫認知服務中適用於影片的表情 API。
services: cognitive-services
author: anrothMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: emotion-api
ms.topic: sample
ms.date: 02/06/2017
ms.author: anroth
ROBOTS: NOINDEX
ms.openlocfilehash: 2687145a89c11efb4a3bcb1494a39806e9aae551
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48238602"
---
# <a name="example-call-emotion-api-for-video"></a>範例：呼叫影片的表情 API

> [!IMPORTANT]
> 表情 API 將於 2019 年 2 月 15 日淘汰。 表情辨識功能現已公開推出，是[臉部 API](https://docs.microsoft.com/azure/cognitive-services/face/) 的一部分。 

本指南示範如何呼叫適用於影片的表情 API。 這些範例是使用適用於影片的表情 API 用戶端程式庫以 C# 撰寫。

### <a name="Prep">準備</a>
若要使用適用於影片的表情 API，您需要一段包含人員的影片，其中最好包含人員面對攝影機的片段。

### <a name="Step1">步驟 1：授權 API 呼叫</a>
每一個針對適用於影片的表情 API 的呼叫，都會需要訂用帳戶金鑰。 這個金鑰必須透過查詢字串參數傳遞，或是在要求標頭中指定。 若要透過查詢字串傳遞訂用帳戶密鑰，請以下列針對適用於影片的表情 API 的要求 URL 作為範例：

```
https://westus.api.cognitive.microsoft.com/emotion/v1.0/recognizeInVideo&subscription-key=<Your subscription key>
```

或者，也可以在 HTTP 要求標頭中指定訂用帳戶密鑰：

```
ocp-apim-subscription-key: <Your subscription key>
```

使用用戶端程式庫時，訂用帳戶金鑰會透過 VideoServiceClient 類別的建構函式傳遞。 例如︰

```
var emotionServiceClient = new emotionServiceClient("Your subscription key");
```
若要取得訂用帳戶金鑰，請參閱[訂閱] (https://azure.microsoft.com/try/cognitive-services/)。

### <a name="Step2">步驟 2：將影片上傳至服務，然後檢查其狀態</a>
針對適用於影片的表情 API，執行任何 呼叫的最基本方法便是直接上傳影片。 這是透過傳送 "POST" 要求並附上應用程式/八位元資料流內容類型，以及從影片檔案讀取的資料來完成。 影片的大小上限為 100MB。

藉由使用用戶端程式庫，會透過以資料流物件進行傳遞，來藉由上傳進行穩定處理。 請參閱下面的範例：

```
Operation videoOperation;
using (var fs = new FileStream(@"C:\Videos\Sample.mp4", FileMode.Open))
{
      videoOperation = await videoServiceClient.CreateOperationAsync(fs, OperationType.recognizeInVideo);
}
```

請注意，VideoServiceClient 的 CreateOperationAsync 方法是非同步的。 呼叫的方法也應該標記為非同步，才能使用 await 子句。
如果影片已上傳至網路上，而且有公用 URL，則可以提供該 URL 來存取適用於影片的表情 API。 在這個範例中，要求主體將會是包含該 URL 的 JSON 字串。

藉由使用用戶端程式庫，可透過使用另一個 CreateOperationAsync 方法的多載，來輕鬆地藉由 URL 進行穩定處理。


```
var videoUrl = "http://www.example.com/sample.mp4";
Operation videoOperation = await videoServiceClient.CreateOperationAsync(videoUrl, OperationType. recognizeInVideo);

```

針對所有適用於影片的表情 API 而言，這個上傳方法都是一樣的。

上傳影片之後，您接下來應該檢查其狀態。 因為影片檔案通常較大，且比其他檔案更具多樣性，使得這個步驟通常需要很長的時間才能處理完畢。 時間的長短取決於檔案的大小和長度。

藉由使用用戶端程式庫，您可以使用 GetOperationResultAsync 方法來擷取作業狀態和結果。


```
var operationResult = await videoServiceClient.GetOperationResultAsync(videoOperation);

```
一般而言，用戶端應該會定期擷取作業狀態，直到該狀態顯示為 [已成功] 或 [已失敗] 為止。

```
OperationResult operationResult;
while (true)
{
      operationResult = await videoServiceClient.GetOperationResultAsync(videoOperation);
      if (operationResult.Status == OperationStatus.Succeeded || operationResult.Status == OperationStatus.Failed)
      {
           break;
      }

      Task.Delay(30000).Wait();
}

```

當 VideoOperationResult 的狀態顯示為 [已成功] 時，您可以透過將 VideoOperationResult 轉換為 VideoOperationInfoResult<VideoAggregateRecognitionResult> 並存取 ProcessingResult 欄位來擷取結果。

```
var emotionRecognitionJsonString = ((VideoOperationInfoResult<VideoAggregateRecognitionResult>)operationResult).ProcessingResult;
```

### <a name="Step3">步驟 3：擷取並了解表情辨識及追蹤 JSON 輸出</a>

輸出結果會包含來自特定檔案內的臉部，並以 JSON 格式呈現的中繼資料。

如同步驟 2 所述，當狀態顯示為 [已成功] 時，該 JSON 輸出便可自 OperationResult 的 ProcessingResult 欄位取得。

臉部偵測和追蹤 JSON 包括下列屬性：

屬性 | 說明
-------------|-------------
版本 | 適用於影片的表情 API JSON 的版本。
時幅 | 影片每秒的「刻度」數目。
Offset  |時間戳記的時間位移。 在適用於影片的表情 API 1.0 版中，這一律為 0。 在未來支援的案例中，這個值可能會變更。
畫面播放速率 | 影片的每秒畫面格數。
片段   | 中繼資料會被分成稱為「片段」的較小個別部分。 每個片段皆包含開始、持續時間、間隔數字及事件。
Start   | 第一個事件的開始時間 (以刻度為單位)。
Duration |  片段的長度 (以刻度為單位)。
間隔 |  片段內每個事件的長度 (以刻度為單位)。
活動  | 事件的陣列。 外部陣列代表一個單位的時間間隔。 內部陣列包含在那個時間點所發生的 0 或多個事件。
windowFaceDistribution |    臉部於事件期間具有特殊表情的百分比。
windowMeanScores |  影像中每個臉部表情的平均分數。

以這種方式來格式化 JSON 的原因，是為了針對未來可能遇到的各種案例來設定 API，這些案例將會需要快速地擷取中繼資料，並管理大量的結果資料流。 此格式會同時運用片段技術 (可讓您將中繼資料分成以時間為基礎的較小單位，以便僅下載所需的部分)，以及分割技術 (可讓您在事件過大的情況下分割它們)。 某些簡單的計算可以協助您轉換資料。 例如，如果事件是從 6300 (刻度) 開始，時幅為 2997 (刻度/每秒)，而且畫面播放速率為 29.97 (畫面/每秒)，則：

*   開始/時幅 = 2.1 秒
*   秒數 x (畫面播放速率/時幅) = 63 個畫面

下列為針對臉部偵測和追蹤，將 JSON 擷取為每一畫面之格式的簡單範例：

```
var emotionRecognitionTrackingResultJsonString = operationResult.ProcessingResult;
var emotionRecognitionTracking = JsonConvert.DeserializeObject<EmotionRecognitionResult>(emotionRecognitionTrackingResultJsonString, settings);
```
因為表情會隨時間淡化，因此如果您準備建置視覺效果以將結果疊加至原始影片上方，請從提供的時間戳記減去 250 毫秒。

### <a name="Summary">總結</a>
在本指南中，您已經學會適用於影片的表情 API 的功能：如何上傳影片、如何檢查其狀態、如何擷取表情辨識中繼資料。

如需該 API 的詳細資訊，請參閱 API 參考指南：[適用於影片的表情 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/5639d931ca73072154c1ce89/operations/56f8d40e1984551ec0a0984e) \(英文\)。
