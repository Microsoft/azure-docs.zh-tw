---
title: 針對 Android 以 Java 開始使用 Microsoft 語音辨識 API | Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 使用 Microsoft Speech API 來開發將語音轉換成文字的 Android 應用程式。
services: cognitive-services
author: zhouwangzw
manager: wolfma
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: zhouwang
ms.openlocfilehash: 4c5243ec14a4494222168bb33b3e840b96f8465e
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49345249"
---
[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

# <a name="quickstart-use-the-bing-speech-recognition-api-in-java-on-android"></a>快速入門：在 Android 上於 Java 中使用 Bing 語音辨識 API

使用「Bing 語音辨識 API」時，您可以開發使用雲端式「Bing 語音服務」將語音轉換成文字的 Android 應用程式。 此 API 支援即時串流，因此您的應用程式能夠在將音訊傳送給服務的同時，以同時且非同步方式接收部分辨識結果。

本文使用範例應用程式來示範如何使用適用於 Android 的語音用戶端程式庫，以 Java 為 Android 裝置開發語音轉換文字應用程式。

## <a name="prerequisites"></a>必要條件

### <a name="platform-requirements"></a>平台需求

此範例是 [Android Studio](http://developer.android.com/sdk/index.html) 以 Java 針對 Windows 開發的範例。

### <a name="get-the-client-library-and-sample-application"></a>取得用戶端程式庫和範例應用程式

您可以在[適用於 Android 的語音用戶端 SDK](https://github.com/microsoft/cognitive-speech-stt-android)，取得適用於 Android 的語音用戶端程式庫和範例。 您可以在 samples/SpeechRecoExample 目錄底下找到可建置的範例。 您可以在 SpeechSDK/libs 的 [armeabi] 和 [x86] 資料夾底下，找到需要在您自己應用程式中使用的兩個程式庫。 libandroid_platform.so 檔案的大小是 22 MB，但在部署階段會縮減成 4 MB。

#### <a name="subscribe-to-the-speech-api-and-get-a-free-trial-subscription-key"></a>訂閱 Speech API 並取得免費試用訂用帳戶金鑰

Speech API 是「認知服務」(先前稱為 Project Oxford) 的組件。 您可以從[認知服務訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)頁面，取得免費試用訂用帳戶金鑰。 選取 Speech API 之後，請選取 [取得 API 金鑰] 以取得金鑰。 這會傳回主要和次要金鑰。 兩個金鑰都繫結至同一個配額，因此你可以使用任一金鑰。

如果您想要使用「含有意圖的辨識」，就必須也註冊 [Language Understanding Intelligent Service (LUIS)](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/)。

> [!IMPORTANT]
>* 取得訂用帳戶金鑰。 您必須有[訂用帳戶金鑰](https://azure.microsoft.com/try/cognitive-services/)，才能使用語音用戶端程式庫。
>
>* 使用您的訂用帳戶金鑰。 在使用所提供 Android 範例應用程式的情況下，以您的訂用帳戶金鑰更新 samples/SpeechRecoExample/res/values/strings.xml 檔案。 如需詳細資訊，請參閱[建置及執行範例](#build-and-run-samples)。

## <a name="use-the-speech-client-library"></a>使用語音用戶端程式庫

若要在您的應用程式中使用此用戶端程式庫，請依照[指示](https://github.com/microsoft/cognitive-speech-stt-android#the-client-library) \(英文\) 進行操作。

您可以在 [適用於 Android 的語音用戶端 SDK](https://github.com/microsoft/cognitive-speech-stt-android) 的 [docs] 資料夾中，找到適用於 Android 的用戶端參考。

## <a name="build-and-run-samples"></a>建置及執行範例

若要了解如何建置及執行範例，請參閱這個[讀我檔案頁面](https://github.com/microsoft/cognitive-speech-stt-android#the-sample) \(英文\)。

## <a name="samples-explained"></a>範例說明

### <a name="create-recognition-clients"></a>建立辨識用戶端

以下範例中的程式碼示範如何根據使用者案例，建立辨識用戶端類別：

```java
void initializeRecoClient()
    {
        String language = "en-us";

        String subscriptionKey = this.getString(R.string.subscription_key);
        String luisAppID = this.getString(R.string.luisAppID);
        String luisSubscriptionID = this.getString(R.string.luisSubscriptionID);

        if (m_isMicrophoneReco && null == m_micClient) {
            if (!m_isIntent) {
                m_micClient = SpeechRecognitionServiceFactory.createMicrophoneClient(this,
                                                                                     m_recoMode,
                                                                                     language,
                                                                                     this,
                                                                                     subscriptionKey);
            }
            else {
                MicrophoneRecognitionClientWithIntent intentMicClient;
                intentMicClient = SpeechRecognitionServiceFactory.createMicrophoneClientWithIntent(this,
                                                                                                   language,
                                                                                                   this,
                                                                                                   subscriptionKey,
                                                                                                   luisAppID,
                                                                                                   luisSubscriptionID);
                m_micClient = intentMicClient;

            }
        }
        else if (!m_isMicrophoneReco && null == m_dataClient) {
            if (!m_isIntent) {
                m_dataClient = SpeechRecognitionServiceFactory.createDataClient(this,
                                                                                m_recoMode,
                                                                                language,
                                                                                this,
                                                                                subscriptionKey);
            }
            else {
                DataRecognitionClientWithIntent intentDataClient;
                intentDataClient = SpeechRecognitionServiceFactory.createDataClientWithIntent(this,
                                                                                              language,
                                                                                              this,
                                                                                              subscriptionKey,
                                                                                              luisAppID,
                                                                                              luisSubscriptionID);
                m_dataClient = intentDataClient;
            }
        }
    }

```

此用戶端類別針對語音辨識中的典型案例，提供預先實作的辨識用戶端類別：

* `DataRecognitionClient`：使用 PCM 資料 (例如來自檔案或音訊來源) 的語音辨識。 系統會將資料分解成緩衝區，然後將每個緩衝區傳送給「語音服務」。 由於不會對緩衝區進行任何修改，因此使用者可以視需要套用自己的無聲偵測。 如果資料是從 WAV 檔案提供的，您便可以將來自檔案的資料直接傳送給「語音服務」。 如果您有原始資料 (例如來自藍牙的音訊)，則需以格式標頭在前、資料在後的方式傳送給「語音服務」。
* `MicrophoneRecognitionClient`：音訊來自麥克風的語音辨識。 請確定已開啟麥克風，且系統會將來自麥克風的資料傳送給語音辨識服務。 在將麥克風資料傳送給辨識服務之前，系統會先在該資料上套用內建的「無聲偵測器」。
* `DataRecognitionClientWithIntent` 和 `MicrophoneRecognitionClientWithIntent`：除了辨識文字之外，這些用戶端還會傳回有關說話者意圖的結構化資訊，可供應用程式用來驅動進一步的動作。 若要使用「意圖」，您必須先使用 [LUIS](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/) 將模型定型。

### <a name="recognition-language"></a>辨識語言

使用 `SpeechRecognitionServiceFactory` 來建立用戶端時，您必須選取語言。 如需「語音服務」所支援的完整語言清單，請參閱[支援的語言](../API-Reference-REST/supportedlanguages.md)。

### `SpeechRecognitionMode`

使用 `SpeechRecognitionServiceFactory` 來建立用戶端時，您還需要指定 `SpeechRecognitionMode`：

* `ShortPhrase`：語句最長為 15 秒。 將資料傳送給服務時，用戶端會收到多個部分結果，以及一個含有多個前 N 名最佳選項的最終結果。
* `LongDictation`：語句最長為 2 分鐘。 將資料傳送給服務時，用戶端會根據服務識別句子暫停的位置，收到多個部分結果和多個最終結果。

### <a name="attach-event-handlers"></a>連結事件處理常式

您可以將各種事件處理常式連結至您所建立的用戶端：

* **部分結果事件**：每當「語音服務」預測您可能說的語句時，甚至是在您說完話 (如果使用 `MicrophoneRecognitionClient`) 或完成資料傳送 (如果使用 `DataRecognitionClient`) 之前，都會呼叫此事件。
* **錯誤事件**：當服務偵測到錯誤時，便會呼叫此事件。
* **意圖事件**：在最終辨識結果剖析成結構化 JSON 意圖之後，便會在 "WithIntent" 用戶端上呼叫此事件 (僅限在 `ShortPhrase` 模式下)。
* **結果事件**：
  * 在 `ShortPhrase` 模式下，會在您說完話時呼叫此事件，並傳回前 N 名最佳結果。
  * 在 `LongDictation` 模式下，會根據服務識別句子暫停的位置，多次呼叫此事件處理常式。
  * **針對前 N 名最佳選項**，會傳回信賴值和幾個不同形式的已辨識文字。 如需詳細資訊，請參閱[輸出格式](../Concepts.md#output-format)。

## <a name="related-topics"></a>相關主題

* [適用於 Android 的用戶端程式庫參考](https://github.com/Azure-Samples/Cognitive-Speech-STT-Android/tree/master/docs) \(英文\)
* [針對 Windows 以適用於 .NET 的 C# 開始使用 Microsoft Speech API](GetStartedCSharpDesktop.md)
* [針對 iOS 以 Objective-C 開始使用 Microsoft Speech API](Get-Started-ObjectiveC-iOS.md)
* [以 JavaScript 開始使用 Microsoft Speech API](GetStartedJSWebsockets.md)
* [透過 REST 開始使用 Microsoft Speech API](GetStartedREST.md)
