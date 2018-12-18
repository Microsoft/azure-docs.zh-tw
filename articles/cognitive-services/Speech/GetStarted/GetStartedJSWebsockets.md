---
title: 開始在 JavaScript 中使用 Bing 語音辨識 API | Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 使用「認知服務」中的「Bing 語音辨識 API」開發將語音連續轉換成文字的應用程式。
services: cognitive-services
author: zhouwangzw
manager: wolfma
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: zhouwang
ms.openlocfilehash: 79f2aa9c3e73037aa25848b7d11c34a73bf35f64
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49342393"
---
# <a name="get-started-with-the-speech-recognition-api-in-javascript"></a>以 JavaScript 開始使用語音辨識 API

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

您可以使用「語音辨識 API」來開發將語音轉換成文字的應用程式。 JavaScript 用戶端程式庫使用[語音服務 WebSocket 通訊協定](../API-Reference-REST/websocketprotocol.md)，這可讓您在說話的同時接收轉譯的文字。 本文將協助您以 JavaScript 開始使用「語音辨識 API」。

## <a name="prerequisites"></a>必要條件

### <a name="subscribe-to-the-speech-recognition-api-and-get-a-free-trial-subscription-key"></a>訂閱語音辨識 API 並取得免費試用訂用帳戶金鑰

Speech API 是「認知服務」的組件。 您可以從[認知服務訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)頁面，取得免費試用訂用帳戶金鑰。 選取 Speech API 之後，請選取 [取得 API 金鑰] 以取得金鑰。 這會傳回主要和次要金鑰。 兩個金鑰都繫結至同一個配額，因此你可以使用任一金鑰。

> [!IMPORTANT]
> 取得訂用帳戶金鑰。 您必須有[訂用帳戶金鑰](https://azure.microsoft.com/try/cognitive-services/)，才能使用語音用戶端程式庫。

## <a name="get-started"></a>開始使用

在本節中，我們將逐步引導您完成載入範例 HTML 頁面的必要步驟。 此範例位於我們的 [GitHub 存放庫](https://github.com/Azure-Samples/SpeechToText-WebSockets-Javascript)中。 您可以從存放庫**直接開啟範例**，或是**從存放庫的本機複本開啟範例**。 

> [!NOTE]
> 有些瀏覽器會封鎖麥克風對不安全來源的存取。 因此，建議您將 'sample'/'your app' 裝載在 https 上，以便讓它能夠在所有支援的瀏覽器上運作。 

### <a name="open-the-sample-directly"></a>直接開啟範例

如以上所述，取得訂用帳戶金鑰。 然後開啟[範例連結](https://htmlpreview.github.io/? https://github.com/Azure-Samples/SpeechToText-WebSockets-Javascript/blob/preview/samples/browser/Sample.html)。 這會將頁面載入您的預設瀏覽器 (使用 [htmlPreview](https://github.com/htmlpreview/htmlpreview.github.com) 來轉譯)。

### <a name="open-the-sample-from-a-local-copy"></a>從本機複本開啟範例

若要在本機嘗試此範例，請複製以下存放庫：

```
git clone https://github.com/Azure-Samples/SpeechToText-WebSockets-Javascript
```

編譯 TypeScript 來源，然後將它們組合成單一 JavaScript 檔案 (必須在您的電腦上安裝 [npm](https://www.npmjs.com/))。 變更至所複製存放庫的根目錄，然後執行下列命令：

```
cd SpeechToText-WebSockets-Javascript && npm run bundle
```

在您慣用的瀏覽器中開啟 `samples\browser\Sample.html`。

## <a name="next-steps"></a>後續步驟

如需有關如何將 SDK 納入您自己網頁中的詳細資訊，請參閱[這裡](https://github.com/Azure-Samples/SpeechToText-WebSockets-Javascript) \(英文\)。

## <a name="remarks"></a>備註

- 「語音辨識 API」支援三種[辨識模式](../concepts.md#recognition-modes)。 您可以更新 Sample.html 檔案中的 **Setup()** 函式來切換模式。 此範例預設會將模式設定為 `Interactive`。 若要變更模式，請將 `SR.RecognitionMode.Interactive` 參數更新成另一個模式。 例如，將此參數變更成 `SR.RecognitionMode.Conversation`。
- 如需完整的支援語言清單，請參閱[支援的語言](../API-Reference-REST/supportedlanguages.md)。

## <a name="related-topics"></a>相關主題

- [JavaScript 語音辨識 API 範例存放庫](https://github.com/Azure-Samples/SpeechToText-WebSockets-Javascript)
- [開始使用 REST API](GetStartedREST.md)
