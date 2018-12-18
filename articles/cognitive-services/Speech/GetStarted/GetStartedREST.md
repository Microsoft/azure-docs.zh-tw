---
title: 使用 REST 來開始使用 Bing 語音辨識 API | Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 使用 REST 來存取「Microsoft 認知服務」中的「語音辨識 API」，以將語音轉換成文字。
services: cognitive-services
author: zhouwangzw
manager: wolfma
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: zhouwang
ms.openlocfilehash: a9f74f4032a78ee51ea2a8f020cd1418bb3330ca
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49345351"
---
# <a name="quickstart-use-the-bing-speech-recognition-rest-api"></a>快速入門：使用 Bing 語音辨識 REST API

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

使用雲端式「Bing 語音服務」時，您可以使用 REST API 來開發應用程式，以將語音轉換成文字。

## <a name="prerequisites"></a>必要條件

### <a name="subscribe-to-the-speech-api-and-get-a-free-trial-subscription-key"></a>訂閱 Speech API 並取得免費試用訂用帳戶金鑰

Speech API 是「認知服務」(先前稱為 Project Oxford) 的組件。 您可以從[認知服務訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)頁面，取得免費試用訂用帳戶金鑰。 選取 Speech API 之後，請選取 [取得 API 金鑰] 以取得金鑰。 這會傳回主要和次要金鑰。 兩個金鑰都繫結至同一個配額，因此你可以使用任一金鑰。

> [!IMPORTANT]
>* 取得訂用帳戶金鑰。 您必須有[訂用帳戶金鑰](https://azure.microsoft.com/try/cognitive-services/)，才能存取 REST API。
>
>* 使用您的訂用帳戶金鑰。 在下列 REST 範例中，請以您自己的訂用帳戶金鑰取代 YOUR_SUBSCRIPTION_KEY。 
>
>* 如需了解如何取得訂用帳戶金鑰，請參考[驗證](../how-to/how-to-authentication.md)頁面。

### <a name="prerecorded-audio-file"></a>預錄的音訊檔案

在此範例中，我們會使用一個錄製的音訊檔案來說明如何使用 REST API。 請錄製一個您自己說出簡短語句的音訊檔案。 例如，說出「今天的天氣如何？」 或「找些有趣的電影來觀賞」。 語音辨識 API 也支援外部麥克風輸入。

> [!NOTE]
> 此範例會要求將音訊錄製成 **PCM 單聲道 16 KHz** WAV 檔案。

## <a name="build-a-recognition-request-and-send-it-to-the-speech-recognition-service"></a>建置辨識要求並將其傳送給語音辨識服務

語音辨識的下一個步驟是以適當的要求標頭和本文，將 POST 要求傳送給語音 HTTP 端點。

### <a name="service-uri"></a>服務 URI

語音辨識服務 URI 是根據[辨識模式](../concepts.md#recognition-modes)和[辨識語言](../concepts.md#recognition-languages)來定義的：

```HTTP
https://speech.platform.bing.com/speech/recognition/<RECOGNITION_MODE>/cognitiveservices/v1?language=<LANGUAGE_TAG>&format=<OUTPUT_FORMAT>
```

`<RECOGNITION_MODE>` 會指定辨識模式，且必須是下列其中一個值：`interactive`、`conversation` 或 `dictation`。 它是 URI 中的必要資源路徑。 如需詳細資訊，請參閱[辨識模式](../concepts.md#recognition-modes)。

`<LANGUAGE_TAG>` 是查詢字串中的必要參數。 它定義了音訊轉換的目標語言：例如 `en-US` 代表英文 (美國)。 如需詳細資訊，請參閱[辨識語言](../concepts.md#recognition-languages)。

`<OUTPUT_FORMAT>` 是查詢字串中的選擇性參數。 允許的值為 `simple` 和 `detailed`。 服務預設會以 `simple` 格式傳回結果。 如需詳細資訊，請參閱[輸出格式](../concepts.md#output-format)。

下表列出一些服務 URI 範例。

| 辨識模式  | 語言 | 輸出格式 | 服務 URI |
|---|---|---|---|
| `interactive` | pt-BR | 預設值 | https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=pt-BR |
| `conversation` | zh-TW | 詳細 |https://speech.platform.bing.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US&format=detailed |
| `dictation` | fr-FR | 簡單 | https://speech.platform.bing.com/speech/recognition/dictation/cognitiveservices/v1?language=fr-FR&format=simple |

> [!NOTE]
> 只有當您的應用程式使用 REST API 來呼叫語音辨識服務時，才需要服務 URI。 如果您使用其中一個[用戶端程式庫](GetStartedClientLibraries.md)，則通常不需要知道所使用的 URI。 用戶端程式庫可能使用不同的服務 URI，這些 URI 僅適用於特定的用戶端程式庫。 如需詳細資訊，請參閱您所選擇的用戶端程式庫。

### <a name="request-headers"></a>要求標頭

以下是必須在要求標頭中設定的欄位：

- `Ocp-Apim-Subscription-Key`：每當您呼叫服務時，都必須在 `Ocp-Apim-Subscription-Key` 標頭中傳遞您的訂用帳戶金鑰。 「語音服務」也支援傳遞授權權杖來取代傳遞訂用帳戶金鑰。 如需詳細資訊，請參閱[驗證](../How-to/how-to-authentication.md)。
- `Content-type`：`Content-type` 欄位會描述音訊資料流的格式和轉碼器。 目前僅支援 WAV 檔案和 PCM 單聲道 16000 編碼。 此格式的 Content-type 值是 `audio/wav; codec=audio/pcm; samplerate=16000`。

`Transfer-Encoding` 是選擇性欄位。 如果您將此欄位設定為 `chunked`，便可以將音訊切割成小區塊。 如需詳細資訊，請參閱[區塊傳輸](../How-to/how-to-chunked-transfer.md)。

以下是一個簡單的要求標頭︰

```HTTP
POST https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US&format=detailed HTTP/1.1
Accept: application/json;text/xml
Content-Type: audio/wav; codec=audio/pcm; samplerate=16000
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: speech.platform.bing.com
Transfer-Encoding: chunked
Expect: 100-continue
```

### <a name="send-a-request-to-the-service"></a>將要求傳送給服務

以下範例示範如何將語音辨識要求傳送給語音 REST 端點。 它使用 `interactive` 選擇辨識模式。

> [!NOTE]
> 請以您的預錄音訊檔路徑取代 `YOUR_AUDIO_FILE`。 以您自己的訂用帳戶金鑰取代 `YOUR_SUBSCRIPTION_KEY`。

# <a name="powershelltabpowershell"></a>[PowerShell](#tab/Powershell)

```Powershell

$SpeechServiceURI =
'https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-us&format=detailed'

# $OAuthToken is the authorization token returned by the token service.
$RecoRequestHeader = @{
  'Ocp-Apim-Subscription-Key' = 'YOUR_SUBSCRIPTION_KEY';
  'Transfer-Encoding' = 'chunked';
  'Content-type' = 'audio/wav; codec=audio/pcm; samplerate=16000'
}

# Read audio into byte array
$audioBytes = [System.IO.File]::ReadAllBytes("YOUR_AUDIO_FILE")

$RecoResponse = Invoke-RestMethod -Method POST -Uri $SpeechServiceURI -Headers $RecoRequestHeader -Body $audioBytes

# Show the result
$RecoResponse

```

# <a name="curltabcurl"></a>[curl](#tab/curl)

此範例會在 Linux 上搭配 bash 使用 curl。 如果您的平台上沒有 curl，則可能需要安裝 curl。 此範例也適用於 Windows 上的 Cygwin、Git Bash、zsh 及其他殼層。

> [!NOTE]
> 以您的預錄音訊檔路徑取代 `YOUR_AUDIO_FILE` 時，請保留音訊檔名稱前面的 `@`，因為它表示 `--data-binary` 的值是檔案名稱，而不是資料。

```
curl -v -X POST "https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-us&format=detailed" -H "Transfer-Encoding: chunked" -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY" -H "Content-type: audio/wav; codec=audio/pcm; samplerate=16000" --data-binary @YOUR_AUDIO_FILE
```

# <a name="ctabcsharp"></a>[C#](#tab/CSharp)

```cs
HttpWebRequest request = null;
request = (HttpWebRequest)HttpWebRequest.Create(requestUri);
request.SendChunked = true;
request.Accept = @"application/json;text/xml";
request.Method = "POST";
request.ProtocolVersion = HttpVersion.Version11;
request.ContentType = @"audio/wav; codec=audio/pcm; samplerate=16000";
request.Headers["Ocp-Apim-Subscription-Key"] = "YOUR_SUBSCRIPTION_KEY";

// Send an audio file by 1024 byte chunks
using (FileStream fs = new FileStream(YOUR_AUDIO_FILE, FileMode.Open, FileAccess.Read))
{

    /*
    * Open a request stream and write 1024 byte chunks in the stream one at a time.
    */
    byte[] buffer = null;
    int bytesRead = 0;
    using (Stream requestStream = request.GetRequestStream())
    {
        /*
        * Read 1024 raw bytes from the input audio file.
        */
        buffer = new Byte[checked((uint)Math.Min(1024, (int)fs.Length))];
        while ((bytesRead = fs.Read(buffer, 0, buffer.Length)) != 0)
        {
            requestStream.Write(buffer, 0, bytesRead);
        }

        // Flush
        requestStream.Flush();
    }
}
```

---

## <a name="process-the-speech-recognition-response"></a>處理語音辨識回應

處理要求之後，「語音服務」會在回應中以 JSON 格式傳回結果。

> [!NOTE]
> 如果先前的程式碼傳回錯誤，請參閱[疑難排解](../troubleshooting.md)來找出可能的原因。

以下程式碼片段顯示一個如何從資料流讀取回應的範例。

# <a name="powershelltabpowershell"></a>[PowerShell](#tab/Powershell)

```Powershell
# show the response in JSON format
ConvertTo-Json $RecoResponse
```

# <a name="curltabcurl"></a>[curl](#tab/curl)

在此範例中，curl 會直接以字串傳回回應訊息。 如果您想要以 JSON 格式顯示訊息，則可以使用額外的工具，例如 jq。

```
curl -X POST "https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-us&format=detailed" -H "Transfer-Encoding: chunked" -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY" -H "Content-type: audio/wav; codec=audio/pcm; samplerate=16000" --data-binary @YOUR_AUDIO_FILE | jq
```

# <a name="ctabcsharp"></a>[C#](#tab/CSharp)

```cs
/*
* Get the response from the service.
*/
Console.WriteLine("Response:");
using (WebResponse response = request.GetResponse())
{
    Console.WriteLine(((HttpWebResponse)response).StatusCode);

    using (StreamReader sr = new StreamReader(response.GetResponseStream()))
    {
        responseString = sr.ReadToEnd();
    }

    Console.WriteLine(responseString);
    Console.ReadLine();
}
```

---

以下範例是一個 JSON 回應：

```json
OK
{
  "RecognitionStatus": "Success",
  "Offset": 22500000,
  "Duration": 21000000,
  "NBest": [{
    "Confidence": 0.941552162,
    "Lexical": "find a funny movie to watch",
    "ITN": "find a funny movie to watch",
    "MaskedITN": "find a funny movie to watch",
    "Display": "Find a funny movie to watch."
  }]
}
```

## <a name="limitations"></a>限制

REST API 有一些限制：

- 它支援的音訊資料流最多只有 15 秒。
- 它不支援辨識期間的中繼結果。 使用者只會收到最終辨識結果。

若要移除限制，請使用語音[用戶端程式庫](GetStartedClientLibraries.md)。 或者，您也可以直接使用[語音 WebSocket 通訊協定](../API-Reference-REST/websocketprotocol.md)。

## <a name="whats-next"></a>後續步驟

- 若要了解如何使用以 C#、Java 等撰寫的 REST API，請參閱這些[範例應用程式](../samples.md)。
- 若要找出並修正錯誤，請參閱[疑難排解](../troubleshooting.md)。
- 若要使用更進階的功能，請使用語音[用戶端程式庫](GetStartedClientLibraries.md)來了解如何開始使用。

### <a name="license"></a>授權

所有「認知服務 SDK」和範例都是透過 MIT 授權來獲得授權的。 如需詳細資訊，請參閱[授權](https://github.com/Microsoft/Cognitive-Speech-STT-JavaScript/blob/master/LICENSE.md) \(英文\)。
