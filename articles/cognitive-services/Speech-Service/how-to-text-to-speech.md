---
title: 透過語音服務使用文字轉換語音
description: 了解如何使用語音服務中的文字轉換語音。
titleSuffix: Microsoft Cognitive Services
services: cognitive-services
author: erhopf
ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 09/08/2018
ms.author: erhopf
ms.openlocfilehash: 774425d19101e4d326f0d6435d56aff1b37a31f7
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49165040"
---
# <a name="use-text-to-speech-in-speech-service"></a>使用語音服務中的文字轉換語音

語音服務可透過簡單的 HTTP 要求提供文字轉換語音功能。 您可以將要說出的文字張貼 (`POST`) 到適當的端點，服務就會傳回包含合成語音的音訊檔案 (`.wav`)。 接著，您的應用程式可以視需要使用此音訊。

文字轉換語音的 POST 要求本文可以是純文字 (ASCII 或 UTF8) 或 [SSML](speech-synthesis-markup.md) 文件。 系統會使用預設語音說出純文字要求。 在大部分情況下，您想要使用 SSML 本文。 HTTP 要求必須包含[授權](https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-apis#authentication)權杖。

以下會顯示區域性文字轉換語音端點。 請使用適合您訂用帳戶的端點。

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-text-to-speech.md)]

## <a name="specify-a-voice"></a>指定語音

若要指定語音，請使用 `<voice>` [SSML](speech-synthesis-markup.md) 標記。 例如︰

```xml
<speak version='1.0' xmlns="http://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
  <voice  name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'>
    Hello, world!
  </voice>
</speak>
```

如需可用語音及其名稱的清單，請參閱[文字轉換語音](language-support.md#text-to-speech)。

## <a name="make-a-request"></a>發出要求

以 POST 模式發出文字轉換語音的 HTTP 要求，該要求的本文中包含要說出的文字。 HTTP 要求本文的長度上限是 1024 個字元。 要求必須具有下列標頭：

頁首|值|註解
-|-|-
|`Content-Type` | `application/ssml+xml` | 輸入文字格式。
|`X-Microsoft-OutputFormat`|     `raw-16khz-16bit-mono-pcm`<br>`riff-16khz-16bit-mono-pcm`<br>`raw-8khz-8bit-mono-mulaw`<br>`riff-8khz-8bit-mono-mulaw`<br>`audio-16khz-128kbitrate-mono-mp3`<br>`audio-16khz-64kbitrate-mono-mp3`<br>`audio-16khz-32kbitrate-mono-mp3`<br>`raw-24khz-16bit-mono-pcm`<br>`riff-24khz-16bit-mono-pcm`<br>`audio-24khz-160kbitrate-mono-mp3`<br>`audio-24khz-96kbitrate-mono-mp3`<br>`audio-24khz-48kbitrate-mono-mp3` | 輸出音訊格式。
|`User-Agent`   |應用程式名稱 | 應用程式名稱是必要項目，而且必須少於 255 個字元。
| `Authorization`   | 透過向權杖服務呈現您的訂用帳戶金鑰而取得的授權權杖。 每個權杖的有效時間為十分鐘。 請參閱 [REST API：驗證](rest-apis.md#authentication)。

> [!NOTE]
> 如果您選取的語音和輸出格式具有不同的位元速率，則會視需要重新進行音訊取樣。

下方顯示一個範例要求。

```xml
POST /cognitiveservices/v1
HTTP/1.1
Host: westus.tts.speech.microsoft.com
X-Microsoft-OutputFormat: riff-24khz-16bit-mono-pcm
Content-Type: application/ssml+xml
User-Agent: Test TTS application
Authorization: (authorization token)

<speak version='1.0' xmlns="http://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
<voice  name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'>
    Hello, world!
</voice> </speak>
```

狀態為 200 的要求本文包含指定輸出格式的音訊。

```
HTTP/1.1 200 OK
Content-Length: XXX
Content-Type: audio/x-wav

Response audio payload
```

如果發生錯誤，則會使用下方的狀態碼。 錯誤的回應本文也會包含問題的描述。

|代碼|說明|問題|
|-|-|-|
400 |不正確的要求 |必要的參數遺失、為空白或 Null。 或者，傳遞至必要或選用參數的值無效。 常見的問題是標頭太長。
401|未經授權 |要求未經授權。 請檢查以確定您的訂用帳戶金鑰或權杖有效。
413|要求實體太大|輸入 SSML 太大或者包含超過 3 個 `<voice>` 元素。
429|太多要求|您已超出訂用帳戶允許的配額或要求率。
|502|錯誤的閘道    | 網路或伺服器端問題。 也可能表示標頭無效。

如需文字轉換語音 REST API 的詳細資訊，請參閱 [REST API](rest-apis.md#text-to-speech)。

## <a name="next-steps"></a>後續步驟

- [試用認知服務](https://azure.microsoft.com/try/cognitive-services/)
- [以 C++ 辨識語音](quickstart-cpp-windows.md)
- [以 C# 辨識語音](quickstart-csharp-dotnet-windows.md)
- [以 Java 辨識語音](quickstart-java-android.md)
