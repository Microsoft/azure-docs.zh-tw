---
title: 語音轉換文字 API 參考 (REST) -語音服務
titleSuffix: Azure Cognitive Services
description: 瞭解如何使用語音轉換文字 REST API。 在本文中，您會了解到授權選項、查詢選項，以及如何建構要求與接收回應。
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/08/2021
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: 70c5593f29b5e83d5d3f318179d365a9235849ca
ms.sourcegitcommit: a055089dd6195fde2555b27a84ae052b668a18c7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98790608"
---
# <a name="speech-to-text-rest-api"></a>語音轉換文字 REST API

語音轉換文字有兩個不同的 REST Api。 每個 API 都提供其特殊用途，並使用不同的端點集合。

語音轉換文字 REST Api 如下：
- [語音轉換文字 REST API v3.0](#speech-to-text-rest-api-v30) 用於 [批次](batch-transcription.md) 轉譯和 [自訂語音](custom-speech-overview.md)。 v3.0 是 v2.0 [的後續版本](./migrate-v2-to-v3.md)。
- [適用于短音訊的語音轉換文字 REST API](#speech-to-text-rest-api-for-short-audio) 用於線上轉譯，作為 [語音 SDK](speech-sdk.md)的替代方案。 使用此 API 的要求只能針對每個要求傳輸最多60秒的音訊。 

## <a name="speech-to-text-rest-api-v30"></a>語音轉換文字 REST API v3。0

語音轉換文字 REST API v3.0 用於 [批次](batch-transcription.md) 轉譯和 [自訂語音](custom-speech-overview.md)。 如果您需要透過 REST 與線上轉譯通訊，請使用 [語音轉換文字 REST API 進行短音訊](#speech-to-text-rest-api-for-short-audio)。

使用 REST API v3.0：
- 如果您想要讓同事存取您所建立的模型，或您想要將模型部署到多個區域，請將模型複製到其他訂用帳戶
- 從容器轉譯資料 (大量轉譯) ，以及提供多個音訊檔案 Url
- 使用 SAS Uri 從 Azure 儲存體帳戶上傳資料
- 如果已針對該端點要求記錄，則取得每個端點的記錄
- 要求您所建立之模型的資訊清單，以設定內部部署容器

REST API v3.0 包含下列功能：
- **通知-webhook**-服務的所有正在執行的進程現在都支援 webhook 通知。 REST API v3.0 提供可讓您在傳送通知的情況下註冊 webhook 的呼叫。
- **更新端點後方的模型** 
- **使用多個資料集進行模型** 調整—使用聲場、語言和發音資料的多個資料集組合來調整模型
- **攜帶您自己的儲存體**-使用您自己的儲存體帳戶來儲存記錄、轉譯檔案和其他資料

請參閱 [這篇文章](batch-transcription.md)，以瞭解搭配使用 REST API V3.0 與批次轉譯的範例。

如果您使用語音轉換文字 REST API 2.0 版，請參閱 [本指南](./migrate-v2-to-v3.md)中的如何遷移至 v3.0。

請參閱 [這裡](https://centralus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0)的完整語音轉換文字 REST API v3.0 參考。

## <a name="speech-to-text-rest-api-for-short-audio"></a>適用于短音訊的語音轉換文字 REST API

語音服務可讓您使用 REST API 轉換語音轉換文字，作為 [語音 SDK](speech-sdk.md)的替代方案。 每個可存取的端點皆與區域相關聯。 您的應用程式需要您打算使用之端點的訂用帳戶金鑰。 簡短音訊的 REST API 非常有限，僅在 [語音 SDK](speech-sdk.md) 無法使用的情況下使用。

使用語音轉換文字 REST API 進行短音訊之前，請考慮下列事項：

* 使用短音訊的 REST API 以及直接傳輸音訊的要求，最多隻能包含60秒的音訊。
* 適用于短音訊的語音轉換文字 REST API 只會傳回最終結果。 不提供部分的結果。

如果您的應用程式需要傳送較長的音訊，請考慮使用 [語音 SDK](speech-sdk.md) 或 [語音轉換文字 REST API v3.0](#speech-to-text-rest-api-v30)。

> [!TIP]
> 請參閱 [這篇文章](sovereign-clouds.md) ，以瞭解 Azure Government 和 Azure 中國端點。

[!INCLUDE [](../../../includes/cognitive-services-speech-service-rest-auth.md)]

### <a name="regions-and-endpoints"></a>區域與端點

適用于 short 音訊的 REST API 端點的格式如下：

```
https://<REGION_IDENTIFIER>.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1
```

取代 `<REGION_IDENTIFIER>` 為與此資料表中的訂用帳戶區域相符的識別碼：

[!INCLUDE [](../../../includes/cognitive-services-speech-service-region-identifier.md)]

> [!NOTE]
> 必須將語言參數附加到 URL 後方，以避免收到 4xx HTTP 錯誤。 例如，使用美國西部端點設定為美式英文的語言是：`https://westus.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US`。

### <a name="query-parameters"></a>查詢參數

REST 要求的查詢字串中可能包括這些參數。

| 參數 | 描述 | 必要/選用 |
|-----------|-------------|---------------------|
| `language` | 識別正在辨識的口說語言。 請參閱 [支援的語言](language-support.md#speech-to-text)。 | 必要 |
| `format` | 指定結果格式。 接受的值為 `simple` 和 `detailed`。 簡單的結果包含 `RecognitionStatus`、`DisplayText`、`Offset` 和 `Duration`。 詳細的回應包括四種不同的顯示文字表示。 預設設定是 `simple`。 | 選擇性 |
| `profanity` | 指定如何處理辨識結果中的不雅內容。 接受的值是，它會將不雅 `masked` 內容取代為星號， `removed` 這會從結果中移除所有不雅內容，或 `raw` 在結果中包含不雅內容。 預設設定是 `masked`。 | 選擇性 |
| `cid` | 使用 [自訂語音入口網站](./custom-speech-overview.md)建立自訂模型時，您可以透過在 [**部署**] 頁面上找到的 **端點識別碼**，使用自訂模型。 使用 **端點識別碼** 作為 `cid` 查詢字串參數的引數。 | 選擇性 |

### <a name="request-headers"></a>要求標頭

下表列出語音轉換文字要求的必要和選擇性標頭。

|標頭| 描述 | 必要/選用 |
|------|-------------|---------------------|
| `Ocp-Apim-Subscription-Key` | 您的語音服務訂用帳戶金鑰。 | 必須有此標頭或 `Authorization`。 |
| `Authorization` | 前面加入 `Bearer` 這個字的授權權杖。 如需詳細資訊，請參閱[驗證](#authentication)。 | 必須有此標頭或 `Ocp-Apim-Subscription-Key`。 |
| `Pronunciation-Assessment` | 指定在辨識結果中顯示發音分數的參數，其會評估語音輸入的發音品質，以及精確度、流暢度、完整性等指標。此參數是包含多個詳細參數的 base64 編碼 json。 請參閱 [發音評定參數](#pronunciation-assessment-parameters) ，以瞭解如何建立此標頭。 | 選擇性 |
| `Content-type` | 描述所提供音訊資料的格式和轉碼器。 接受的值為 `audio/wav; codecs=audio/pcm; samplerate=16000` 和 `audio/ogg; codecs=opus`。 | 必要 |
| `Transfer-Encoding` | 指定正在傳送的音訊資料區塊，而不是單一檔案。 只有在以區塊處理音訊資料時，才能使用此標頭。 | 選擇性 |
| `Expect` | 如果使用區塊傳輸，請傳送 `Expect: 100-continue`。 語音服務會確認初始要求並等候其他資料。| 如果傳送的是音訊資料區塊，則為必要。 |
| `Accept` | 如果提供，則必須是 `application/json`。 語音服務會以 JSON 提供結果。 某些要求架構會提供不相容的預設值。 最好的作法是一律包含 `Accept` 。 | 此為選用步驟，但建議執行。 |

### <a name="audio-formats"></a>自動格式

音訊是在 HTTP `POST` 要求的主體中傳送。 它必須是此表格中的格式之一：

| 格式 | 轉碼器 | 位元速率 | 採樣速率  |
|--------|-------|----------|--------------|
| WAV    | PCM   | 256 kbps | 16 kHz，單聲道 |
| OGG    | OPUS  | 256 kpbs | 16 kHz，單聲道 |

>[!NOTE]
>使用語音服務中的簡短音訊和 WebSocket REST API 可支援上述格式。 [語音 SDK](speech-sdk.md)目前支援具有 PCM 編解碼器的 WAV 格式，以及[其他格式](how-to-use-codec-compressed-audio-input-streams.md)。

### <a name="pronunciation-assessment-parameters"></a>發音評定參數

下表列出發音評量的必要和選擇性參數。

| 參數 | 描述 | 必要？ |
|-----------|-------------|---------------------|
| ReferenceText | 用來評估發音的文字。 | 必要 |
| GradingSystem | 評分校正的點系統。 `FivePoint`系統會提供0-5 浮點分數，並 `HundredMark` 提供0-100 浮點分數。 預設：`FivePoint`。 | 選擇性 |
| 細微度 | 評估細微性。 接受的值為，顯示全文檢索文字和 `Phoneme` 音素層級的分數，其中顯示全文檢索和單字層級的分數，其中 `Word` 只會 `FullText` 顯示全文檢索層級的分數。 預設設定是 `Phoneme`。 | 選擇性 |
| 維度 | 定義輸出準則。 接受的值為 `Basic` （僅顯示精確度分數），會 `Comprehensive` 顯示更多維度的分數 (例如流暢度全文檢索層級的分數和完整性分數、word 層級的錯誤類型) 。 檢查 [回應參數](#response-parameters) ，以查看不同分數維度和文字錯誤類型的定義。 預設設定是 `Basic`。 | 選擇性 |
| EnableMiscue | 啟用 miscue 計算。 啟用此功能之後，會將發音的單字與參考文字進行比較，並根據比較來標示省略/插入。 接受的值為 `False` 和 `True`。 預設設定是 `False`。 | 選擇性 |
| ScenarioId | 表示自訂點系統的 GUID。 | 選擇性 |

以下是包含發音評定參數的 JSON 範例：

```json
{
  "ReferenceText": "Good morning.",
  "GradingSystem": "HundredMark",
  "Granularity": "FullText",
  "Dimension": "Comprehensive"
}
```

下列範例程式碼示範如何在標頭中建立發音評定參數 `Pronunciation-Assessment` ：

```csharp
var pronAssessmentParamsJson = $"{{\"ReferenceText\":\"Good morning.\",\"GradingSystem\":\"HundredMark\",\"Granularity\":\"FullText\",\"Dimension\":\"Comprehensive\"}}";
var pronAssessmentParamsBytes = Encoding.UTF8.GetBytes(pronAssessmentParamsJson);
var pronAssessmentHeader = Convert.ToBase64String(pronAssessmentParamsBytes);
```

我們強烈建議您在張貼音訊資料時，串流處理 (區塊) 上傳，這可大幅降低延遲。 請參閱 [不同程式設計語言的範例程式碼](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/PronunciationAssessment) ，以瞭解如何啟用串流。

>[!NOTE]
>「發音評定」功能目前僅供 `westus` `eastasia` 和區域使用 `centralindia` 。 這項功能目前僅適用于 `en-US` 語言。

### <a name="sample-request"></a>範例要求

下列範例包含主機名稱和必要標頭。 請務必注意，此服務也預期會有此範例中未包含的音訊資料。 如先前所述，建議以區塊處理，但非必要。

```HTTP
POST speech/recognition/conversation/cognitiveservices/v1?language=en-US&format=detailed HTTP/1.1
Accept: application/json;text/xml
Content-Type: audio/wav; codecs=audio/pcm; samplerate=16000
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: westus.stt.speech.microsoft.com
Transfer-Encoding: chunked
Expect: 100-continue
```

若要啟用發音評量，您可以新增下列標頭。 請參閱 [發音評定參數](#pronunciation-assessment-parameters) ，以瞭解如何建立此標頭。

```HTTP
Pronunciation-Assessment: eyJSZWZlcm...
```

### <a name="http-status-codes"></a>HTTP 狀態碼

每個回應的 HTTP 狀態碼會指出成功或常見的錯誤。

| HTTP 狀態碼 | 描述 | 可能的原因 |
|------------------|-------------|-----------------|
| `100` | 繼續 | 已接受初始要求。 繼續傳送其餘的資料。  (用於區區塊轉送)  |
| `200` | [確定] | 要求成功；回應主體是 JSON 物件。 |
| `400` | 不正確的要求 | 未提供語言代碼，不是支援的語言、不正確音訊檔案等等。 |
| `401` | 未經授權 | 訂用帳戶金鑰或授權權杖在指定的區域中無效，或是無效的端點。 |
| `403` | 禁止 | 遺漏訂用帳戶金鑰或授權權杖。 |

### <a name="chunked-transfer"></a>區塊傳輸

區塊傳輸 (`Transfer-Encoding: chunked`) 可協助減少辨識延遲。 它可讓語音服務在音訊檔案進行傳輸時開始處理。 Short 音訊的 REST API 不會提供部分或中間的結果。

此程式碼範例說明如何以區塊傳送音訊。 只有第一個區塊應該包含音訊檔案的標頭。 `request` 是 `HttpWebRequest` 連接到適當 REST 端點的物件。 `audioFile` 是音訊檔案在磁碟上的路徑。

```csharp
var request = (HttpWebRequest)HttpWebRequest.Create(requestUri);
request.SendChunked = true;
request.Accept = @"application/json;text/xml";
request.Method = "POST";
request.ProtocolVersion = HttpVersion.Version11;
request.Host = host;
request.ContentType = @"audio/wav; codecs=audio/pcm; samplerate=16000";
request.Headers["Ocp-Apim-Subscription-Key"] = "YOUR_SUBSCRIPTION_KEY";
request.AllowWriteStreamBuffering = false;

using (var fs = new FileStream(audioFile, FileMode.Open, FileAccess.Read))
{
    // Open a request stream and write 1024 byte chunks in the stream one at a time.
    byte[] buffer = null;
    int bytesRead = 0;
    using (var requestStream = request.GetRequestStream())
    {
        // Read 1024 raw bytes from the input audio file.
        buffer = new Byte[checked((uint)Math.Min(1024, (int)fs.Length))];
        while ((bytesRead = fs.Read(buffer, 0, buffer.Length)) != 0)
        {
            requestStream.Write(buffer, 0, bytesRead);
        }

        requestStream.Flush();
    }
}
```

### <a name="response-parameters"></a>回應參數

結果以 JSON 格式提供。 `simple` 格式包含以下的最上層欄位。

| 參數 | 描述  |
|-----------|--------------|
|`RecognitionStatus`|狀態，例如 `Success` 代表辨識成功。 請參閱下一個表格。|
|`DisplayText`|在大小寫、標點符號、反向文字正規化之後的已辨識文字 (將語音文字轉換成較短的表單，例如 "200" 的200或 "scripto smith" 的 "scripto smith" ) ，以及不雅內容遮罩。 只會在成功時呈現。|
|`Offset`|辨識的語音在音訊資料流中開始的時間 (以 100 奈秒為單位)。|
|`Duration`|辨識的語音在音訊資料流中的持續時間 (以 100 奈秒為單位)。|

`RecognitionStatus` 欄位可包含下列的值：

| 狀態 | 描述 |
|--------|-------------|
| `Success` | 辨識成功並顯示 `DisplayText` 欄位。 |
| `NoMatch` | 音訊串流中偵測到語音，但目標語言中沒有符合的字組。 通常表示辨識語言與使用者的口語語言不同。 |
| `InitialSilenceTimeout` | 音訊串流的開頭沒有任何聲音，而且等候語音的服務已逾時。 |
| `BabbleTimeout` | 音訊串流的開頭只有雜音，而且等候語音的服務已逾時。 |
| `Error` | 辨識服務發生內部錯誤，無法繼續。 可能的話，再試一次。 |

> [!NOTE]
> 如果音訊只包含不雅內容，而且 `profanity` 查詢參數設為 `remove`，則服務不會傳回語音結果。

`detailed`格式包含其他形式的已辨識結果。
使用 `detailed` 格式時，系統會提供 `DisplayText` 作為 `NBest` 清單中每個結果的 `Display`。

清單中的物件 `NBest` 可以包括：

| 參數 | 描述 |
|-----------|-------------|
| `Confidence` | 項目的信賴分數從 0.0 (不信賴) 到 1.0 (完全信賴) |
| `Lexical` | 已辨識文字的語彙形式：已辨識的實際文字。 |
| `ITN` | 已辨識文字的反向文字正規化 (「標準」) 形式，包含電話號碼、數字、縮寫 ("doctor smith" 縮短為 "dr smith")，以及其他已套件的轉換。 |
| `MaskedITN` | 如果要求，已套用不雅內容遮罩的 ITN 形式。 |
| `Display` | 已辨識文字的顯示形式，已新增標點符號和大寫。 此參數與當格式設定為 `simple` 時，所提供的 `DisplayText` 相同。 |
| `AccuracyScore` | 語音的發音精確度。 精確度表示音素與原生喇叭的發音相符程度。 文字和全文檢索層級精確度分數是從音素層級精確度分數匯總而來。 |
| `FluencyScore` | 指定語音的流暢度。 流暢度指出語音符合原生說話者在單字之間使用無訊息中斷的程度。 |
| `CompletenessScore` | 語音的完整性，藉由計算發音文字與參考文字輸入的比例來決定。 |
| `PronScore` | 表示指定語音之發音品質的整體分數。 這是從 `AccuracyScore` `FluencyScore` 和加權匯總而來 `CompletenessScore` 。 |
| `ErrorType` | 此值會指出文字是否省略、插入或不太明顯，相較于 `ReferenceText` 。 可能的值為 `None` (表示這個字組沒有錯誤) `Omission` 、 `Insertion` 和 `Mispronunciation` 。 |

### <a name="sample-responses"></a>回應範例

辨識的一般回應 `simple` ：

```json
{
  "RecognitionStatus": "Success",
  "DisplayText": "Remind me to buy 5 pencils.",
  "Offset": "1236645672289",
  "Duration": "1236645672289"
}
```

辨識的一般回應 `detailed` ：

```json
{
  "RecognitionStatus": "Success",
  "Offset": "1236645672289",
  "Duration": "1236645672289",
  "NBest": [
      {
        "Confidence" : "0.87",
        "Lexical" : "remind me to buy five pencils",
        "ITN" : "remind me to buy 5 pencils",
        "MaskedITN" : "remind me to buy 5 pencils",
        "Display" : "Remind me to buy 5 pencils.",
      }
  ]
}
```

使用發音評定進行辨識的一般回應：

```json
{
  "RecognitionStatus": "Success",
  "Offset": "400000",
  "Duration": "11000000",
  "NBest": [
      {
        "Confidence" : "0.87",
        "Lexical" : "good morning",
        "ITN" : "good morning",
        "MaskedITN" : "good morning",
        "Display" : "Good morning.",
        "PronScore" : 84.4,
        "AccuracyScore" : 100.0,
        "FluencyScore" : 74.0,
        "CompletenessScore" : 100.0,
        "Words": [
            {
              "Word" : "Good",
              "AccuracyScore" : 100.0,
              "ErrorType" : "None",
              "Offset" : 500000,
              "Duration" : 2700000
            },
            {
              "Word" : "morning",
              "AccuracyScore" : 100.0,
              "ErrorType" : "None",
              "Offset" : 5300000,
              "Duration" : 900000
            }
        ]
      }
  ]
}
```

## <a name="next-steps"></a>後續步驟

- [建立 Azure 免費帳戶](https://azure.microsoft.com/free/cognitive-services/)
- [自訂原音模型](./how-to-custom-speech-train-model.md)
- [自訂語言模型](./how-to-custom-speech-train-model.md)
- [熟悉批次轉譯](batch-transcription.md)