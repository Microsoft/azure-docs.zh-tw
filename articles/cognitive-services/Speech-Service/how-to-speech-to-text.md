---
title: 使用語音轉換文字 | Microsoft Docs
description: 了解如何使用語音服務中的語音轉換文字
titleSuffix: Microsoft Cognitive Services
services: cognitive-services
author: v-jerkin
manager: noellelacharite
ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 07/16/2018
ms.author: v-jerkin
ms.openlocfilehash: 26cecedfc3ad2d472b9686e25054fe08253cee77
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39068517"
---
# <a name="use-speech-to-text-in-the-speech-service"></a>使用語音服務中的「語音轉換文字」

使用 [語音轉換文字] 應用程式的方式有兩種。

| 方法 | 說明 |
|-|-|
| [SDK](speech-sdk.md) | 適用於 C/C++、C# 和 Java 開發人員的最簡單方法 |
| [REST](rest-apis.md) | 使用 HTTP POST 要求辨識簡短的語句 | 

## <a name="using-the-sdk"></a>使用 SDK

[語音 SDK](speech-sdk.md) 提供最簡單的方式，在您的應用程式中使用完整功能的 [語音轉換文字]。

1. 建立語音處理站，並提供語音服務訂用帳戶金鑰和[區域](regions.md)或授權權杖。 您也可以在此時設定一些選項，例如辨識語言或自有語音辨識模型的自訂端點。

2. 從處理站取得辨識器。 有不同類型的辨識器可以使用。 每個辨識器類型都可以使用您裝置的預設麥克風、音訊串流或檔案中的音訊。

    辨識器 | 函式
    -|-
    語音辨識器|提供語音的文字轉譯
    意圖辨識器|在辨識之後透過 [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/) 衍生喇叭意圖\*
    翻譯辨識器|將轉譯的文字翻譯為另一種語言 (請參閱[語音翻譯](how-to-translate-speech.md))

    \* *對於意圖辨識，您必須在為意圖辨識器建立語音處理站時，使用個別的 LUIS 訂用帳戶金鑰。*
    
4. 繫結事件以便執行非同步作業 (如有需要)。 當辨識器具有過渡期和最終結果時，它會接著呼叫事件處理常式。 否則，您的應用程式會收到最終轉譯結果。

5. 開始辨識。
   針對一次性辨識 (例如命令或查詢辨識)，請使用 `RecognizeAsync()`，它會傳回所辨識的第一個語句。
   針對長時間執行的辨識 (例如謄寫)，請使用 `StartContinuousRecognitionAsync()`，並綁定非同步辨識結果的事件。

### <a name="sdk-samples"></a>SDK 範例

如需最新的範例集，請參閱[認知服務語音 SDK 範例 GitHub 存放庫](https://aka.ms/csspeech/samples) (英文)。

## <a name="using-the-rest-api"></a>使用 REST API

如果您不是使用 SDK 所支援的語言，則 REST API 是最簡單的語音辨識方式。 您會對服務端點所提出的 HTTP POST 要求，並且在要求主體中傳遞整個語句。 您會收到回應，其中包含已辨識的文字。

> [!NOTE]
> 使用 REST API 時，語句會受限於 15 秒或更短。

如需 [REST API] REST API 的詳細資訊，請參閱 [REST API](rest-apis.md#speech-to-text)。 若要查看運作中的 REST API，可從 GitHub 下載 [REST API 範例](https://github.com/Azure-Samples/SpeechToText-REST)。

## <a name="next-steps"></a>後續步驟

- [試用認知服務](https://azure.microsoft.com/try/cognitive-services/)
- [如何以 C++ 辨識語音](quickstart-cpp-windows.md)
- [如何以 C# 辨識語音](quickstart-csharp-dotnet-windows.md)
- [如何以 Java 辨識語音](quickstart-java-android.md)
