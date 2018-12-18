---
title: Microsoft Bing 語音服務 | Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 使用 Microsoft Speech API 將語音導向動作新增至您的應用程式，包括與使用者的即時互動。
services: cognitive-services
author: zhouwangzw
manager: wolfma
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: zhouwang
ms.openlocfilehash: 960d2d402f223b306aa6ff05b567d13525e3d525
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340353"
---
# <a name="what-is-bing-speech"></a>什麼是 Bing 語音？

[!INCLUDE [Deprecation note](../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

雲端式 Microsoft Bing 語音 API 可讓開發人員在其應用程式中輕鬆建立支援語音的強大功能，例如語音命令控制、使用自然語音轉換的使用者對話，以及語音文字記錄和聽寫。 Microsoft Speech API 同時支援「語音轉換文字」和「文字轉換語音」轉換。

- **語音轉換文字** API 會將人類語音轉換成可作為輸入或命令來控制應用程式的文字。
- **文字轉換語音** API 會將文字轉換成可向應用程式使用者播放的音訊資料流。

## <a name="speech-to-text-speech-recognition"></a>語音轉換文字 (語音辨識)

Microsoft 語音辨識 API 會將音訊資料流「轉譯」成文字，可供應用程式向使用者顯示，或作為命令輸入來執行動作。 它為開發人員提供兩種可將語音新增到應用程式的方式：REST API **或** Websocket 型用戶端程式庫。

- [REST API](GetStarted/GetStartedREST.md)：開發人員可從其應用程式對服務使用 HTTP 呼叫，來進行語音辨識。
- [用戶端程式庫](GetStarted/GetStartedClientLibraries.md)：如需進階功能，開發人員可以下載 Microsoft Speech 用戶端程式庫，並連結至他們的應用程式。  用戶端程式庫可用於使用不同語言 (C#、Java、JavaScript、ObjectiveC) 的各種平台 (Windows、Android、iOS)。 與 REST API 不同，用戶端程式庫會使用 Websocket 型通訊協定。

| 使用案例 | [REST APIs](GetStarted/GetStartedREST.md) | [用戶端程式庫](GetStarted/GetStartedClientLibraries.md) |
|-----|-----|-----|
| 轉換簡短的語音，例如命令 (音訊長度 < 15 秒)，但不提供中期結果 | 是 | 是 |
| 轉換長音訊 (> 15 秒) | 否 | 是 |
| 串流音訊並提供所需的中期結果 | 否 | 是 |
| 使用 LUIS 來理解從音訊轉換的文字 | 否 | 是 |

不論開發人員選擇哪一種方法 (REST API 或用戶端程式庫)，Microsoft 語音服務都支援下列各項：

- Cortana、「Office 聽寫」、「Office 翻譯工具」及其他 Microsoft 產品所使用的 Microsoft 進階語音辨識技術。
- 即時連續辨識。 語音辨識 API 可讓使用者將音訊即時轉譯成文字，並支援接收到目前為止已辨識單字的中繼結果。 語音服務也支援語音結束偵測。 此外，使用者也可以選擇額外的格式設定功能，例如轉換成大寫和標點符號、粗話遮罩，以及文字正規化。
- 針對「互動式」、「對話」及「聽寫」案例，支援最佳化語音辨識結果。 針對需要自訂的語言模型和原音模型的使用者案例，[自訂語音服務](../custom-speech-service/cognitive-services-custom-speech-home.md)可讓您建立為應用程式和使用者量身打造的語音模型。
- 支援多個方言的眾多口語。 如需每個辨識模式中所支援語言的完整清單，請參閱[辨識語言](api-reference-rest/supportedlanguages.md)。
- 與語言理解整合 除了將輸入音訊轉換成文字之外，「語音轉換文字」還為應用程式提供一項可理解文字意義的額外功能。 它會使用 [Language Understanding Intelligent Service (LUIS)](../LUIS/what-is-luis.md) 從已辨識的文字中擷取意圖和實體。

### <a name="next-steps"></a>後續步驟

- 使用 [REST API](GetStarted/GetStartedREST.md) 或[用戶端程式庫](GetStarted/GetStartedClientLibraries.md)來開始使用 Microsoft 語音辨識服務。
- 查看以您慣用程式設計語言撰寫的[範例應用程式](samples.md)。
- 移至＜參考＞一節來尋找 [Microsoft Speech 通訊協定](API-Reference-REST/websocketprotocol.md)詳細資料和 API 參考。

## <a name="text-to-speech-speech-synthesis"></a>文字轉換語音 (語音合成)

「文字轉換語音 API」會使用 REST 將結構化文字轉換成音訊資料流。 這些 API 提供各種語音和語言的快速文字轉換語音轉換。 此外，使用者還能夠變更音訊特性，例如發音、音量、音調等 (透過使用 SSML 標記來變更)。

### <a name="next-steps"></a>後續步驟

- 開始使用 Microsoft 文字轉換語音服務：[文字轉換語音 API 參考](api-reference-rest/bingvoiceoutput.md)。 如需「文字轉換語音」所支援語言和語音的完整清單，請參閱[支援的地區設定和音型](api-reference-rest/bingvoiceoutput.md#SupLocales)。
