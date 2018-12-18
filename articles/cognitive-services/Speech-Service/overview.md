---
title: 什麼是語音服務？
description: 語音服務 (屬於 Microsoft 認知服務的一部分) 聯合了先前個別提供的數個 Azure 語音服務：Bing 語音 (包含語音辨識和文字轉換語音)、自訂語音和語音翻譯。
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: erhopf
ms.service: cognitive-services
ms.component: speech-service
ms.topic: overview
ms.date: 09/24/2018
ms.author: erhopf
ms.openlocfilehash: 7a61d85d96b993d49a536c38c2907a3d6ef55e65
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49365453"
---
# <a name="what-is-the-speech-service"></a>什麼是語音服務？


如同其他 Azure 服務，語音服務也是由 Cortana 和 Microsoft Office 等產品中使用的成熟語音技術所提供。

語音服務結合了先前透過 [Bing 語音 API](https://docs.microsoft.com/azure/cognitive-services/speech/home)、[Translator Speech](https://docs.microsoft.com/azure/cognitive-services/translator-speech/)、[自訂語音 (Custom Speech)](https://docs.microsoft.com/azure/cognitive-services/custom-speech-service/cognitive-services-custom-speech-home) 和[自訂語音 (Custom Voice)](http://customvoice.ai/) 服務提供的 Azure 語音功能。 現在，只要一個訂用帳戶即可存取前述所有功能。

## <a name="main-speech-service-functions"></a>主要語音服務功能

語音服務的主要功能是語音轉換文字 (也稱為語音辨識或轉譯)、文字轉換語音 (語音合成)，以及語音翻譯。

|函式|特性|
|-|-|
|[語音轉換文字](speech-to-text.md)| <li>連續的即時語音謄寫成文字。<li>可以分批謄寫錄音中的語音。 <li>支援的中繼結果、語音結束偵測、自動文字格式，和不雅內容遮罩。 <li>可呼叫 [Language Understanding](https://docs.microsoft.com/azure/cognitive-services/luis/) (LUIS)，從謄寫的語音衍生使用者意圖。\*|
|[文字轉換語音](text-to-speech.md)| <li>將文字轉換成自然發音語音。 <li>為多種支援的語言提供多個性別和/或方言。 <li>支援純文字輸入或語音合成標記語言 (SSML)。 |
|[語音翻譯](speech-translation.md)| <li>近乎即時地翻譯串流音訊。<li> 也可以處理錄製的語音。<li>提供文字或合成語音形式的結果。 |


## <a name="customize-speech-features"></a>自訂語音功能

您可以使用自己的資料來將模型定型，作為語音服務的語音轉換文字和文字轉換語音功能的基礎。

|功能|模型|目的|
|-|-|-|
|語音轉換文字|[原音模型](how-to-customize-acoustic-models.md)|有助於謄寫特定的說話者和環境，例如汽車或工廠。|
||[語言模型](how-to-customize-language-model.md)|有助於謄寫特定領域的詞彙和文法，例如醫療或 IT 專業術語。|
||[發音模型](how-to-customize-pronunciation.md)|有助於謄寫縮寫和縮略字，例如 "IOU" 代表 "I owe you"。 |
|文字轉語音|[聲音音調](how-to-customize-voice-font.md)|以人類語音的範例將模型定型，給予應用程式它自己的聲音。|

在應用程式的語音轉換文字或文字轉換語音功能中使用標準模型的地方，都可以使用您的自訂模型。

## <a name="use-the-speech-service"></a>使用語音服務

為了讓支援語音的應用程式更易於開發，Microsoft 提供了可與語音服務搭配使用的[語音 SDK](speech-sdk.md)。 語音 SDK 提供適用於 C#、C++ 和 Java 且具一致性的原生語音轉換文字和語音翻譯 API。 如果您要使用其中一種語言進行開發，語音 SDK 將可為您處理網路詳細資料，而讓開發更為容易。

語音服務也具有適用於任何程式設計語言的 [REST API](rest-apis.md)，可用來提出 HTTP 要求。 REST 介面並未提供 SDK 的即時串流功能。

|<br>方法|語音<br>轉換文字|文字轉換<br>語音|語音<br>翻譯|<br>說明|
|-|-|-|-|-|
|[語音 SDK](speech-sdk.md)|是|否|是|適用於 C#、C++ 和 Java 的原生 API，可簡化開發工作。|
|[REST](rest-apis.md)|是|yes|否|以 HTTP 為基礎的簡單 API，可讓您輕鬆地將語音新增至您的應用程式。|

### <a name="websockets"></a>WebSocket

語音服務也有用來串流語音轉換文字和語音翻譯的 Websocket 通訊協定。 語音 SDK 會使用這些通訊協定與語音服務進行通訊。 您應使用語音 SDK，而不是嘗試實作自己的 WebSocket 通訊來與語音服務通訊。

不過，如果您已有透過 Websocket 使用 Bing 語音或翻譯工具語音的程式碼，您可以更新它以使用語音服務。 WebSocket 通訊協定是相容的，只有端點不同。

### <a name="speech-devices-sdk"></a>語音裝置 SDK

[語音裝置 SDK](speech-devices-sdk.md) 是支援語音的裝置開發人員所適用的整合式硬體和軟體平台。 我們的硬體合作夥伴會提供參考設計和開發單元。 Microsoft 提供可充分運用硬體功能的裝置最佳化 SDK。


## <a name="speech-scenarios"></a>語音案例

語音服務的使用案例包括：

> [!div class="checklist"]
> * 建立語音觸發式應用程式
> * 謄寫話務中心錄音
> * 實作語音聊天機器人

### <a name="voice-user-interface"></a>語音使用者介面

語音輸入可讓應用程式富有彈性、能夠免持，且快速使用。 在支援語音的應用程式中，使用者可直接要求所需的資訊。

如果應用程式的適用對象是一般大眾，您可以使用預設的語音辨識模型。 它們可在一般環境下妥善辨識各種不同的說話者。

若您的應用程式是在特定環境 (例如醫療或 IT 環境) 中使用，您可以建立[語言模型](how-to-customize-language-model.md)。 您可以使用此模型來教導語音服務有關您應用程式所使用的特殊術語。

若您的應用程式在吵雜的環境 (例如工廠) 中使用，您可以建立自訂[原音模型](how-to-customize-acoustic-models.md)。 此模型可幫助語音服務分辨語音或噪音。

### <a name="call-center-transcription"></a>話務中心轉譯

我們通常只會在通話發生問題時，才會回頭聽取話務中心的錄音內容。 有了語音服務，將錄音謄寫為文字便很容易。 您可以輕鬆地為文字編製索引以進行[全文檢索搜尋](https://docs.microsoft.com/azure/search/search-what-is-azure-search)，或套用[文字分析](https://docs.microsoft.com/azure/cognitive-services/Text-Analytics/)來偵測情感、語言與關鍵片語。

如果話務中心錄音涉及專用術語 (例如，產品名稱或 IT 專業術語)，您可以建立[語言模型](how-to-customize-language-model.md)讓語音服務學習這些詞彙。 自訂[原音模型](how-to-customize-acoustic-models.md)可協助語音服務了解較差的電話連線。

如需此案例的詳細資訊，請深入了解語音服務的[批次謄寫](batch-transcription.md)。

### <a name="voice-bots"></a>語音聊天機器人

[聊天機器人](https://dev.botframework.com/)日漸成為讓使用者獲得其所需資訊，以及讓客戶接觸到所喜愛業務的熱門方式。 當您將對話式使用者介面新增到網站或應用程式時，可提升其功能的尋找和存取速度。 使用語音服務時，此對話會以同樣的方式回應口語查詢，而呈現出煥然一新的流暢度。

若要在具有語音功能的聊天機器人中新增獨特的風格，您可以為它添加自己的語音。 建立自訂語音的程序需要兩個步驟。 首先，[錄製](record-custom-voice-samples.md)您想要使用的語音。 接著，[提交那些錄音](how-to-customize-voice-font.md) (以及文字謄寫) 給語音服務的[語音自訂入口網站](https://cris.ai/Home/CustomVoice)，由它完成其餘作業。 建立您的自訂語音之後，在您的應用程式中使用它的步驟非常直覺。

## <a name="next-steps"></a>後續步驟

取得語音服務的訂用帳戶金鑰。

> [!div class="nextstepaction"]
> [免費試用語音服務](get-started.md)
