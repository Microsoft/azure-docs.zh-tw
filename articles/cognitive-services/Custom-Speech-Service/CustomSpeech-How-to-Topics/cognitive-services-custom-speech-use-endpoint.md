---
title: 搭配 Azure 上的自訂語音服務使用自訂語音端點 | Microsoft Docs
description: 了解如何搭配「認知服務」中的「自訂語音服務」使用自訂的語音轉換文字端點。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 02/08/2017
ms.author: panosper
ms.openlocfilehash: 9f1a21d6f099bc27c81e8c254f7df73d6c3e585d
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49341791"
---
# <a name="use-a-custom-speech-to-text-endpoint"></a>使用自訂的語音轉換文字端點

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

您可以將要求傳送給「Azure 自訂語音服務」語音轉換文字端點，就像傳送給預設的「認知服務」語音端點那樣。 這些端點在功能上與「語音 API」的端點完全相同。 因此，可透過用戶端程式庫或 REST API 供「語音 API」使用的功能，同樣也可供自訂端點使用。

您使用此服務來建立的端點可以處理不同數量的並行要求。 此數量取決於與您訂用帳戶相關的訂價層。 如果收到的要求太多，就會發生錯誤。 免費層有每月的要求數限制。

此服務會假設資料是即時傳輸的。 如果其傳送速度較快，系統便會將要求視為執行中，直到其即時音訊持續時間過去為止。

> [!NOTE]
> 不過，我們確實支援[新的 Web 通訊端](https://docs.microsoft.com/azure/cognitive-services/speech/api-reference-rest/websocketprotocol)。 如果您打算搭配自訂語音端點使用 Web 通訊端，請依照這裡的指示進行操作。
>
> 我們即將推出新的 [REST API](https://docs.microsoft.com/azure/cognitive-services/speech/getstarted/getstartedrest) 支援。 如果您打算透過 HTTP 呼叫自訂語音端點，請依照這裡的指示進行操作。
>

## <a name="send-requests-by-using-the-speech-client-library"></a>使用語音用戶端程式庫來傳送要求

若要使用語音用戶端程式庫將要求傳送給自訂端點，請啟動辨識用戶端。 請使用來自 [NuGet](http://nuget.org/) 的「用戶端語音 SDK」。 搜尋「語音辨識」，然後從 Microsoft 選取適用於您平台的語音辨識套件。 您可以在 [GitHub](https://github.com/Microsoft/Cognitive-Speech-STT-Windows) 上找到一些範例程式碼。 「用戶端語音 SDK」提供 **SpeechRecognitionServiceFactory** Factory 類別，此類別提供下列方法：

  *   ```CreateDataClient(...)```：資料辨識用戶端。
  *   ```CreateDataClientWithIntent(...)```：含有意圖的資料辨識用戶端。
  *   ```CreateMicrophoneClient(...)```：麥克風辨識用戶端。
  *   ```CreateMicrophoneClientWithIntent(...)```：含有意圖的麥克風辨識用戶端。

如需詳細的文件，請參閱 [Bing 語音 API](https://docs.microsoft.com/azure/cognitive-services/speech/home) \(英文\)。 「自訂語音服務」端點支援相同的 SDK。

資料辨識用戶端適用於從資料 (例如檔案或其他音訊來源) 進行語音辨識。 麥克風辨識用戶端適用於從麥克風進行語音辨識。 如果您已針對案例建置 [Language Understanding Intelligent Service](https://www.luis.ai/) (LUIS) 應用程式，則在這兩種用戶端中使用意圖都可從 LUIS 傳回結構化的意圖結果。

這四種類型的用戶端都可以透過兩種方式具現化。 第一種方式使用標準的「認知服務語音 API」。 第二種方式可讓您指定 URL，以對應至使用「自訂語音服務」所建立的自訂端點。

例如，您可以使用下列方法，建立一個將要求傳送給自訂端點的 **DataRecognitionClient**：

```csharp
public static DataRecognitionClient CreateDataClient(SpeeechRecognitionMode speechRecognitionMode, string language, string primaryOrSecondaryKey, **string url**);
```

**your_subscriptionId** 和 **endpointURL** 分別指 [部署資訊] 頁面上的訂用帳戶金鑰和 Web 通訊端 URL。

**AuthenticationUri** 是用來從驗證服務接收權杖。 此 URI 必須個別設定，如以下範例程式碼所示。

此範例程式碼示範如何使用用戶端 SDK：

```csharp
var dataClient = SpeechRecognitionServiceFactory.CreateDataClient(
  SpeechRecognitionMode.LongDictation,
  "en-us",
  "your_subscriptionId",
  "your_subscriptionId",
  "endpointURL");
// set the authorization Uri
dataClient.AuthenticationUri = "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken";
```

> [!NOTE]
> 當您使用 SDK 中的 **Create** 方法時，因為 **Create** 方法的多載緣故，因此必須提供兩次訂用帳戶 ID。
>

「自訂語音服務」針對短期和長期辨識使用兩個不同的 URL。 兩者都列在 [部署] 頁面上。 請針對您想要使用的特定形式，使用正確的端點 URL。

如需有關使用自訂端點來叫用各種辨識用戶端的詳細資訊，請參閱 [SpeechRecognitionServiceFactory](https://www.microsoft.com/cognitive-services/Speech-api/documentation/GetStarted/GetStartedCSharpDesktop) 類別。 此頁面上的文件係指原音模型調適，但它適用於使用「自訂語音服務」建立的所有端點。

## <a name="send-requests-by-using-the-speech-protocol"></a>使用語音通訊協定來傳送要求

針對[語音通訊協定](https://docs.microsoft.com/azure/cognitive-services/speech/api-reference-rest/websocketprotocol)顯示的端點是「開放原始碼 Web 通訊端語音通訊協定」的端點。

目前，唯一的官方用戶端實作是針對 [JavaScript](https://github.com/Azure-Samples/SpeechToText-WebSockets-Javascript)。 如果您想要從該處所提供的範例開始著手，請對程式碼進行下列變更，然後重新建置範例：

1. 在 _src\sdk\speech.browser\SpeechConnectionFactory.ts_中，將主機名稱 "wss://speech.platform.bing.com" 取代為您部署的詳細資料頁面上所顯示的主機名稱。 請勿在這裡插入完整的 URI，只要插入 *wss* 通訊協定配置和主機名稱即可。 例如︰

    ```JavaScript
    private get Host(): string {
        return Storage.Local.GetOrAdd("Host", "wss://<your_key_goes_here>.api.cris.ai");
    }
    ```

2. 根據您的需求設定 _samples\browser\Samples.html_ 中的 _recognitionMode_ 參數：
    * _RecognitionMode.Interactive_ 可支援長達 15 秒的要求。
    * _RecognitionMode.Conversation_ 和 _RecognitionMode.Dictation_ (在「自訂語音服務」中兩者相等) 可支援長達 10 分鐘的要求。

3. 使用範例之前，先使用 "gulp build" 來建置範例。

> [!NOTE]
> 請確定針對此通訊協定使用的 URI 正確。 必要的配置為 *wss* (不像在用戶端通訊協定中是使用 *http*)。 

如需詳細資訊，請參閱 [Bing 語音 API](https://docs.microsoft.com/azure/cognitive-services/speech/getstarted/getstartedclientlibraries) \(英文\) 文件。

## <a name="send-requests-by-using-http"></a>使用 HTTP 來傳送要求

使用 HTTP POST 將要求傳送給自訂端點，與透過 HTTP 將要求傳送給「認知服務 Bing 語音 API」類似。 請修改 URL 以反映您自訂部署的位址。

不論是對「認知服務語音」端點，還是對使用此服務來建立的自訂端點來說，在透過 HTTP 傳送的要求上都有一些限制。 HTTP 要求在辨識程序期間無法傳回部分結果。 此外，就音訊內容而言，要求的持續時間會限制為 10 秒，就整體而言，則限制為 14 秒。

若要建立 POST 要求，請依照您用於「認知服務語音 API」的相同程序進行操作。

1. 使用您的訂用帳戶 ID 來取得存取權杖。 必須有此權杖，才能存取辨識端點。 您可以重複使用此權杖 10 分鐘。

    ```
    curl -X POST --header "Ocp-Apim-Subscription-Key:<subscriptionId>" --data "" "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken"
    ```
      **subscriptionId** 應該設定為您用於此部署的訂用帳戶 ID。 回應會是下一個要求所需的一般權杖。

2. 再次使用 POST 來將音訊發佈至端點。

    ```
    curl -X POST --data-binary @example.wav -H "Authorization: Bearer <token>" -H "Content-Type: application/octet-stream" "<https_endpoint>"
    ```

    **token** 是您藉由上一個呼叫接收的存取權杖。 **https_endpoint** 是您自訂語音轉換文字端點的完整位址，顯示在 [部署資訊] 頁面上。

如需有關 HTTP POST 參數和回應格式的詳細資訊，請參閱 [Microsoft 認知服務 Bing 語音 HTTP API](https://www.microsoft.com/cognitive-services/speech-api/documentation/API-Reference-REST/BingVoiceRecognition#SampleImplementation)。

## <a name="send-requests-by-using-the-service-library"></a>使用服務程式庫來傳送要求
「服務程式庫」可讓您的服務利用 Microsoft 語音轉譯雲端將口語即時轉譯成文字，以便用戶端應用程式能夠以同時且非同步方式，傳送音訊及接收部分辨識結果。 如需「服務 SDK」的詳細資料，請參閱[這裡](https://docs.microsoft.com/azure/cognitive-services/speech/getstarted/getstartedcsharpservicelibrary)

> [!NOTE]
> 使用「服務程式庫」時，您必須將 **IAuthorizationProvider** 實作中的授權提供者 URI 變更為 https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken。

## <a name="next-steps"></a>後續步驟
* 使用您的[自訂原音模型](cognitive-services-custom-speech-create-acoustic-model.md)來提升準確性。
* 使用您的[自訂語言模型](cognitive-services-custom-speech-create-language-model.md)來提升準確性。
