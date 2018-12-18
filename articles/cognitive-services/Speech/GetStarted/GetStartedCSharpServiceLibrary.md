---
title: 使用 C# 服務程式庫來開始使用 Microsoft 語音辨識 API | Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 使用 Bing 語音辨識服務程式庫將口語轉換成文字。
services: cognitive-services
author: zhouwangzw
manager: wolfma
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: zhouwang
ms.openlocfilehash: e9b3d3207f5aca6cba3555ba2578b5c66b3bd193
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49343686"
---
# <a name="quickstart-use-the-bing-speech-recognition-service-library-in-c35-for-net-windows"></a>快速入門：以適用於 .NET Windows 的 C&#35; 開始使用 Bing 語音辨識服務程式庫

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

此服務程式庫適用於已有自己的雲端服務而想要從其服務呼叫「語音服務」的開發人員。 如果您想要從裝置繫結應用程式呼叫語音辨識服務，則請勿使用此 SDK。 (請使用其他用戶端程式庫或 REST API 來達到該目的)。

若要使用 C# 服務程式庫，請安裝 [NuGet 套件 Microsoft.Bing.Speech](https://www.nuget.org/packages/Microsoft.Bing.Speech/)。 如需程式庫 API 參考，請參閱 [Microsoft Speech C# 服務程式庫](https://cdn.rawgit.com/Microsoft/Cognitive-Speech-STT-ServiceLibrary/master/docs/index.html) \(英文\)。

下列各節說明如何使用 C# 服務程式庫來安裝、建置及執行 C# 範例應用程式。

## <a name="prerequisites"></a>必要條件

### <a name="platform-requirements"></a>平台需求

下列範例是使用 [Visual Studio 2015 Community Edition](https://www.visualstudio.com/products/visual-studio-community-vs) 針對 Windows 8+ 和 .NET 4.5+ Framework 開發的。

### <a name="get-the-sample-application"></a>取得範例應用程式

從[語音 C# 服務程式庫範例](https://github.com/Microsoft/Cognitive-Speech-STT-ServiceLibrary)存放庫中複製範例。

### <a name="subscribe-to-the-speech-recognition-api-and-get-a-free-trial-subscription-key"></a>訂閱語音辨識 API 並取得免費試用訂用帳戶金鑰

Speech API 是「認知服務」(先前稱為 Project Oxford) 的組件。 您可以從[認知服務訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)頁面，取得免費試用訂用帳戶金鑰。 選取 Speech API 之後，請選取 [取得 API 金鑰] 以取得金鑰。 這會傳回主要和次要金鑰。 兩個金鑰都繫結至同一個配額，因此你可以使用任一金鑰。

> [!IMPORTANT]
> * 取得訂用帳戶金鑰。 您必須有[訂用帳戶金鑰](https://azure.microsoft.com/try/cognitive-services/)，才能使用語音用戶端程式庫。
>
> * 使用您的訂用帳戶金鑰。 使用所提供的 C# 服務程式庫範例應用程式時，您必須提供提供訂用帳戶金鑰作為其中一個命令列參數。 如需詳細資訊，請參閱[執行範例應用程式](#step-3-run-the-sample-application)。

## <a name="step-1-install-the-sample-application"></a>步驟 1：安裝範例應用程式

1. 啟動 Visual Studio 2015，然後選取 [檔案] > [開啟] > [專案/方案]。

2. 按兩下以開啟名為 SpeechClient.sln 的 Visual Studio 2015 解決方案 (.sln) 檔案。 此解決方案隨即在 Visual Studio 中開啟。

## <a name="step-2-build-the-sample-application"></a>步驟 2：建置範例應用程式

按 Ctrl+Shift+B，或選取功能區功能表上的 [建置]。 然後選取 [建置方案]。

## <a name="step-3-run-the-sample-application"></a>步驟 3：執行範例應用程式

1. 在建置完成之後，按 F5 或選取功能區功能表上的 [啟動] 來執行範例。

2. 開啟範例的輸出目錄，例如 SpeechClientSample\bin\Debug。 按 Shift 和滑鼠右鍵，然後選取 [在此處開啟命令視窗]。

3. 搭配下列參數執行 `SpeechClientSample.exe`：

   * Arg[0]：指定輸入音訊 WAV 檔案。
   * Arg[1]：指定音訊地區設定。
   * Arg[2]：指定辨識模式：*Short* 代表 `ShortPhrase` 模式，而 *Long* 則代表 `LongDictation` 模式。
   * Arg[3]：指定用以存取語音辨識服務的訂用帳戶金鑰。

## <a name="samples-explained"></a>範例說明

### <a name="recognition-modes"></a>辨識模式

* `ShortPhrase` 模式：語句最長為 15 秒。 將資料傳送給伺服器時，用戶端會收到多個部分結果和一個最終的最佳結果。
* `LongDictation` 模式：語句最長為 10 分鐘。 將資料傳送給伺服器時，用戶端會根據伺服器指出句子暫停的位置，收到多個部分結果和多個最終結果。

### <a name="supported-audio-formats"></a>支援的音訊格式

Speech API 使用下列轉碼器來支援音訊/WAV：

* PCM 單聲道
* Siren
* SirenSR

### <a name="preferences"></a>喜好設定

若要建立 SpeechClient，您必須先建立 Preferences 物件。 Preferences 物件是一組參數，用來設定語音服務的行為。 它是由下列欄位所組成的：

* `SpeechLanguage`：傳送給語音服務之音訊的地區設定。
* `ServiceUri`：用來呼叫語音服務的端點。
* `AuthorizationProvider`：用來擷取權杖以存取語音服務的 IAuthorizationProvider 實作。 雖然此範例提供「認知服務」授權提供者，但強烈建議您建立自己的實作來處理權杖快取。
* `EnableAudioBuffering`：進階選項。 請參閱[連線管理](#connection-management)。

### <a name="speech-input"></a>語音輸入

SpeechInput 物件是由兩個欄位所組成的：

* **Audio**：您選擇的資料流實作，SDK 會從此資料流實作提取音訊。 它可以是任何支援讀取的[資料流](https://msdn.microsoft.com/library/system.io.stream(v=vs.110).aspx)。
   > [!NOTE]
   > SDK 會在資料流於讀取中傳回 **0** 時，偵測到資料流結尾。

* **RequestMetadata**：與語音要求相關的中繼資料。 如需詳細資訊，請參閱[參考](https://cdn.rawgit.com/Microsoft/Cognitive-Speech-STT-ServiceLibrary/master/docs/index.html) \(英文\)。

### <a name="speech-request"></a>語音要求

在您將 SpeechClient 和 SpeechInput 物件具現化之後，請使用 RecognizeAsync 來對「語音服務」發出要求。

```cs
    var task = speechClient.RecognizeAsync(speechInput);
```

在要求完成之後，RecognizeAsync 所傳回的工作也會完成。 最後一個 RecognitionResult 即為辨識的結束。 如果服務或 SDK 意外失敗，工作便可能失敗。

### <a name="speech-recognition-events"></a>語音辨識事件

#### <a name="partial-results-event"></a>部分結果事件

每當「語音服務」預測您可能說的語句時，甚至是在您說完話 (如果使用 `MicrophoneRecognitionClient`) 或完成資料傳送 (如果使用 `DataRecognitionClient`) 之前，都會呼叫此事件。 您可以使用 `SpeechClient.SubscribeToPartialResult()` 來訂閱此事件。 或是使用一般事件訂閱方法 `SpeechClient.SubscribeTo<RecognitionPartialResult>()`。

**傳回格式** | 說明 |
------|------
**LexicalForm** | 應用程式如果需要原始、未經處理的語言辨識結果，便最適合使用此形式。
**DisplayText** | 所辨識的片語會套用反向文字正規化、轉換成大寫、標點符號及粗話遮罩。 粗話在第一個字元之後會以星號遮罩，例如 "d***." 應用程式如果會向使用者顯示語音辨識結果，便最適合使用此形式。
**Confidence** | 所辨識的片語對相關音訊而言代表的信賴等級 (由語音辨識伺服器定義)。
**MediaTime** | 與音訊資料流開頭相對的目前時間 (時間單位為 100 奈秒).
**MediaDuration** | 與音訊區段相對的目前階段持續時間/長度 (時間單位為 100 奈秒).

#### <a name="result-event"></a>結果事件
當您說完話 (在 `ShortPhrase` 模式下) 時，便會呼叫此事件。 系統會提供您前 N 名最佳結果選項。 在 `LongDictation` 模式下，可以根據伺服器指出句子暫停的位置，多次呼叫此事件。 您可以使用 `SpeechClient.SubscribeToRecognitionResult()` 來訂閱此事件。 或是使用一般事件訂閱方法 `SpeechClient.SubscribeTo<RecognitionResult>()`。

**傳回格式** | 說明 |
------|------|
**RecognitionStatus** | 與辨識產生方式相關的狀態。 例如，是因辨識成功而產生，還是因取消連線而產生等等。
**Phrases** | 一組具有辨識信賴度的前 N 名最佳辨識片語。

如需有關辨識結果的詳細資訊，請參閱[輸出格式](../Concepts.md#output-format)。

### <a name="speech-recognition-response"></a>語音辨識回應

語音回應範例：
```
--- Partial result received by OnPartialResult  
---what  
--- Partial result received by OnPartialResult  
--what's  
--- Partial result received by OnPartialResult  
---what's the web  
--- Partial result received by OnPartialResult   
---what's the weather like  
---***** Phrase Recognition Status = [Success]   
***What's the weather like? (Confidence:High)  
What's the weather like? (Confidence:High) 
```

## <a name="connection-management"></a>連線管理

API 會針對每一要求使用一個 WebSocket 連線。 為了提供最佳使用者體驗，SDK 會嘗試重新連線至「語音服務」，然後從它收到的上一個 RecognitionResult 開始辨識。 例如，如果音訊要求的長度為兩分鐘，當 SDK 在一分鐘標記時收到 RecognitionEvent，然後網路在五秒後發生失敗時，SDK 便會從一分鐘標記處開始一個新連線。

>[!NOTE]
>SDK 不會往回搜尋該一分鐘標記，因為資料流可能不支援搜尋。 取而代之的是，SDK 會保留它用來緩衝處理音訊的內部緩衝區，然後在收到 RecognitionResult 事件時清除該緩衝區。

## <a name="buffering-behavior"></a>緩衝處理行為

SDK 預設會緩衝處理音訊，以便能夠在發生網路中斷時復原。 如果案例較適合捨棄網路中斷期間遺失的音訊並重新啟動連線，則最好是將 Preferences 物件中的 `EnableAudioBuffering` 設定為 `false` 來停用音訊緩衝處理。

## <a name="related-topics"></a>相關主題

[Microsoft Speech C# 服務程式庫參考](https://cdn.rawgit.com/Microsoft/Cognitive-Speech-STT-ServiceLibrary/master/docs/index.html) \(英文\)
