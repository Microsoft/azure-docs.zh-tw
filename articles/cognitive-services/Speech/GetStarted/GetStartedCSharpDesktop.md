---
title: 使用 C# 傳統型程式庫來開始使用 Bing 語音辨識 API | Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 開發使用「Bing 語音辨識 API」將語音轉換成文字的基本 Windows 應用程式。
services: cognitive-services
author: zhouwangzw
manager: wolfma
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: zhouwang
ms.openlocfilehash: f79b148558e7881f852ccd57916b0b0f31a98219
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49342325"
---
# <a name="quickstart-use-the-bing-speech-recognition-api-in-c35-for-net-on-windows"></a>快速入門：針對 Windows 以適用於 .NET 的 C&#35; 使用 Bing 語音辨識 API

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

本頁說明如何開發一個使用「語音辨識 API」將語音轉換成文字的基本 Windows 應用程式。 使用用戶端程式庫可允許進行即時串流，這意謂著當您的用戶端應用程式將音訊傳送給服務時，會以同時且非同步方式接收部分辨識結果。

開發人員如果想要從在任何裝置上執行的應用程式使用「語音服務」，可以使用 C# 傳統型程式庫。 若要使用此程式庫，請安裝 [NuGet 套件 Microsoft.ProjectOxford.SpeechRecognition-x86](https://www.nuget.org/packages/Microsoft.ProjectOxford.SpeechRecognition-x86/) \(適用於 32 位元平台\) 和 [NuGet 套件 Microsoft.ProjectOxford.SpeechRecognition-x64](https://www.nuget.org/packages/Microsoft.ProjectOxford.SpeechRecognition-x64/) \(適用於 64 位元平台\)。 如需用戶端程式庫 API 參考，請參閱 [Microsoft Speech C# 傳統型程式庫](https://cdn.rawgit.com/Microsoft/Cognitive-Speech-STT-Windows/master/docs/SpeechSDK/index.html) \(英文\)。

下列各節說明如何使用 C# 傳統型程式庫來安裝、建置及執行 C# 範例應用程式。

## <a name="prerequisites"></a>必要條件

### <a name="platform-requirements"></a>平台需求

下列範例是使用 [Visual Studio 2015 Community Edition](https://www.visualstudio.com/products/visual-studio-community-vs) 針對 Windows 8+ 和 .NET Framework 4.5+ 開發的。

### <a name="get-the-sample-application"></a>取得範例應用程式

從[語音 C# 傳統型程式庫範例](https://github.com/microsoft/cognitive-speech-stt-windows)存放庫中複製範例。

### <a name="subscribe-to-the-speech-recognition-api-and-get-a-free-trial-subscription-key"></a>訂閱語音辨識 API 並取得免費試用訂用帳戶金鑰

Speech API 是「認知服務」(先前稱為 Project Oxford) 的組件。 您可以從[認知服務訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)頁面，取得免費試用訂用帳戶金鑰。 選取 Speech API 之後，請選取 [取得 API 金鑰] 以取得金鑰。 這會傳回主要和次要金鑰。 兩個金鑰都繫結至同一個配額，因此你可以使用任一金鑰。

> [!IMPORTANT]
> * 取得訂用帳戶金鑰。 您必須有[訂用帳戶金鑰](https://azure.microsoft.com/try/cognitive-services/)，才能使用語音用戶端程式庫。
>
> * 使用您的訂用帳戶金鑰。 在使用所提供 C# 傳統型範例應用程式的情況下，於執行範例時，將您的訂用帳戶金鑰貼到文字方塊中。 如需詳細資訊，請參閱[執行範例應用程式](#step-3-run-the-sample-application)。

## <a name="step-1-install-the-sample-application"></a>步驟 1：安裝範例應用程式

1. 啟動 Visual Studio 2015，然後選取 [檔案] > [開啟] > [專案/方案]。

2. 瀏覽至儲存已下載之「語音辨識 API」檔案的資料夾。 選取 [語音] > [Windows]，然後選取 [Sample-WP] 資料夾。

3. 按兩下以開啟名為 SpeechToText-WPF-Samples.sln 的 Visual Studio 2015 解決方案 (.sln) 檔案。 此解決方案隨即在 Visual Studio 中開啟。

## <a name="step-2-build-the-sample-application"></a>步驟 2：建置範例應用程式

1. 如果您想要使用「含有意圖的辨識」，就必須先註冊 [Language Understanding Intelligent Service (LUIS)](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/)。 接著，使用您 LUIS 應用程式的端點 URL 來設定 samples/SpeechRecognitionServiceExample 資料夾之 app.config 檔案中 `LuisEndpointUrl` 索引鍵的值。 如需有關 LUIS 應用程式之端點 URL 的詳細資訊，請參閱[發佈您的應用程式](../../luis/luis-get-started-create-app.md#publish-your-app)。

   > [!TIP]
   > 以 `&amp;` 取代 LUIS 端點 URL 中的 `&` 字元，以確保 XML 剖析器會正確解譯該 URL。

2. 按 Ctrl+Shift+B，或選取功能區功能表上的 [建置]。 然後選取 [建置方案]。

## <a name="step-3-run-the-sample-application"></a>步驟 3：執行範例應用程式

1. 在建置完成之後，按 F5 或選取功能區功能表上的 [啟動] 來執行範例。

2. 移至 [Project Oxford 語音轉換文字範例] 視窗。 將您的訂用帳戶金鑰貼到 [請在這裡貼上您的訂用帳戶金鑰來開始] 文字方塊中，如下所示。 若要將您的訂用帳戶金鑰保存在您的電腦或膝上型電腦上，請選取 [儲存金鑰]。 若要從系統中刪除訂用帳戶金鑰，請選取 [刪除金鑰] 以將它從電腦或膝上型電腦中移除。

   ![語音辨識貼入金鑰](../Images/SpeechRecog_paste_key.PNG)

3. 在 [語音辨識來源] 底下，選擇六個來源其中之一，這分成兩個主要輸入分類：

   * 使用您電腦的麥克風或連結的麥克風來擷取語音。
   * 播放音訊檔案。

   每個分類有三種辨識模式：

    * **ShortPhrase 模式**：語句最長為 15 秒。 將資料傳送給伺服器時，用戶端會收到多個部分結果，以及一個含有多個前 N 名最佳選項的最終結果。
    * **LongDictation 模式**：語句最長為 2 分鐘。 將資料傳送給伺服器時，用戶端會根據伺服器指出句子暫停的位置，收到多個部分結果和多個最終結果。
    * **意圖偵測**伺服器會傳回與語音輸入相關的額外結構化資訊。 若要使用意圖偵測，您必須先使用 [LUIS](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/) 將模型定型。

請將範例音訊檔案與此範例應用程式搭配使用。 在您隨此範例下載之存放庫中的 samples/SpeechRecognitionServiceExample 資料夾底下，即可找到這些檔案。 當您選取 [將 wav 檔案用於 Shortphrase 模式] 或 [將 wav 檔案用於 Longdictation 模式] 作為語音輸入時，如果沒有選擇任何其他檔案，這些範例音訊檔案就會自動執行。 目前僅支援 WAV 音訊格式。

![語音轉換文字介面](../Images/HelloJones.PNG)

## <a name="samples-explained"></a>範例說明

### <a name="recognition-events"></a>辨識事件

* **部分結果事件**：每當「語音服務」預測您可能說的語句時，甚至是在您說完話 (如果使用 `MicrophoneRecognitionClient`) 或完成資料傳送 (如果使用 `DataRecognitionClient`) 之前，都會呼叫此事件。
* **錯誤事件**：當服務偵測到錯誤時，便會呼叫此事件。
* **意圖事件**：在最終辨識結果剖析成結構化 JSON 意圖之後，便會在 "WithIntent" 用戶端上呼叫此事件 (僅限在 ShortPhrase 模式下)。
* **結果事件**：
  * 在 `ShortPhrase` 模式下，會在您說完話時呼叫此事件，並傳回前 N 名最佳結果。
  * 在 `LongDictation` 模式下，會根據服務識別句子暫停的位置，多次呼叫此事件處理常式。
  * **針對前 N 名最佳選項**，會傳回信賴值和幾個不同形式的已辨識文字。 如需詳細資訊，請參閱[輸出格式](../Concepts.md#output-format)。

在程式碼中，已經以程式碼註解形式指出事件處理常式。

## <a name="related-topics"></a>相關主題

* [Microsoft Speech 傳統型程式庫參考](https://cdn.rawgit.com/Microsoft/Cognitive-Speech-STT-Windows/master/docs/SpeechSDK/index.html) \(英文\)
* [針對 Android 以 Java 開始使用 Microsoft 語音辨識 API](GetStartedJavaAndroid.md)
* [針對 iOS 以 Objective-C 開始使用 Microsoft 語音辨識 API](Get-Started-ObjectiveC-iOS.md)
* [以 JavaScript 開始使用 Microsoft 語音辨識 API](GetStartedJSWebsockets.md)
* [透過 REST 開始使用 Microsoft 語音辨識 API](GetStartedREST.md)
