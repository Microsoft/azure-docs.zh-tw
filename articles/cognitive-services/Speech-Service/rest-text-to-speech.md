---
title: 文字轉換語音 API 參考 (REST) -語音服務
titleSuffix: Azure Cognitive Services
description: 瞭解如何使用文字轉換語音 REST API。 在本文中，您會了解到授權選項、查詢選項，以及如何建構要求與接收回應。
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/08/2021
ms.author: trbye
ms.custom: references_regions
ms.openlocfilehash: d858474eca34243a007d0d0ac1e023a4a0fab8ec
ms.sourcegitcommit: 65cef6e5d7c2827cf1194451c8f26a3458bc310a
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2021
ms.locfileid: "98572335"
---
# <a name="text-to-speech-rest-api"></a>文字轉換語音 REST API

語音服務可讓您 [將文字轉換為合成語音](#convert-text-to-speech) ，並使用一組 REST api [取得區域支援](#get-a-list-of-voices) 的語音清單。 每個可用的端點都會與區域相關聯。 您打算使用之端點/區域的訂用帳戶金鑰是必要項。

文字轉語音 API 支援類神經和標準文字轉語音，且各支援依地區設定所識別的特定語言和方言。

* 如語音的完整清單，請參閱[ 語言支援](language-support.md#text-to-speech)。
* 如需區域可用性的詳細資訊，請參閱[區域](regions.md#text-to-speech)。

> [!IMPORTANT]
> 標準、自訂和神經語音的成本各不相同。 如需詳細資訊，請參閱[定價](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/)。

在使用此 API 之前，請先瞭解：

* 文字轉語音 REST API 需要授權標頭。 這表示需要完成權杖交換，才能存取服務。 如需詳細資訊，請參閱[驗證](#authentication)。

> [!TIP]
> 請參閱 [這篇文章](sovereign-clouds.md) ，以瞭解 Azure Government 和 Azure 中國端點。

[!INCLUDE [](../../../includes/cognitive-services-speech-service-rest-auth.md)]

## <a name="get-a-list-of-voices"></a>取得語音清單

此 `voices/list` 端點可讓您取得特定區域/端點的完整語音清單。

### <a name="regions-and-endpoints"></a>區域與端點

| 區域 | 端點 |
|--------|----------|
| 澳大利亞東部 | `https://australiaeast.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 巴西南部 | `https://brazilsouth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 加拿大中部 | `https://canadacentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 美國中部 | `https://centralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 東亞 | `https://eastasia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 美國東部 | `https://eastus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 美國東部 2 | `https://eastus2.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 法國中部 | `https://francecentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 印度中部 | `https://centralindia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 日本東部 | `https://japaneast.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 南韓中部 | `https://koreacentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 美國中北部 | `https://northcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 北歐 | `https://northeurope.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 南非北部 | `https://southafricanorth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 美國中南部 | `https://southcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 東南亞 | `https://southeastasia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 英國南部 | `https://uksouth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 美國中西部 | `https://westcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 西歐 | `https://westeurope.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 美國西部 | `https://westus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| 美國西部 2 | `https://westus2.tts.speech.microsoft.com/cognitiveservices/voices/list` |

> [!TIP]
> [預覽版](language-support.md#neural-voices-in-preview) 中的語音僅適用于下列3個區域：美國東部、西歐和東南亞。

### <a name="request-headers"></a>要求標頭

下表列出文字轉換語音要求的必要和選擇性標頭。

| 標頭 | 描述 | 必要/選用 |
|--------|-------------|---------------------|
| `Ocp-Apim-Subscription-Key` | 您的語音服務訂用帳戶金鑰。 | 必須有此標頭或 `Authorization`。 |
| `Authorization` | 前面加入 `Bearer` 這個字的授權權杖。 如需詳細資訊，請參閱[驗證](#authentication)。 | 必須有此標頭或 `Ocp-Apim-Subscription-Key`。 |



### <a name="request-body"></a>要求本文

`GET`對此端點的要求不需要主體。

### <a name="sample-request"></a>範例要求

此要求只需要授權標頭。

```http
GET /cognitiveservices/voices/list HTTP/1.1

Host: westus.tts.speech.microsoft.com
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
```

### <a name="sample-response"></a>範例回應

此回應已截斷，以說明回應的結構。

> [!NOTE]
> 語音可用性會因區域/端點而異。

```json
[
  {
    "Name": "Microsoft Server Speech Text to Speech Voice (ar-EG, Hoda)",
    "DisplayName": "Hoda",
    "LocalName": "هدى",
    "ShortName": "ar-EG-Hoda",
    "Gender": "Female",
    "Locale": "ar-EG",
    "SampleRateHertz": "16000",
    "VoiceType": "Standard",
    "Status": "GA"
  },

...
      
    {
    "Name": "Microsoft Server Speech Text to Speech Voice (en-US, AriaNeural)",
    "DisplayName": "Aria",
    "LocalName": "Aria",
    "ShortName": "en-US-AriaNeural",
    "Gender": "Female",
    "Locale": "en-US",
    "StyleList": [
      "chat",
      "customerservice",
      "newscast-casual",
      "newscast-formal",
      "cheerful",
      "empathetic"
    ],
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "Status": "GA"
  },
  
  ...
    
     {
    "Name": "Microsoft Server Speech Text to Speech Voice (ga-IE, OrlaNeural)",
    "DisplayName": "Orla",
    "LocalName": "Orla",
    "ShortName": "ga-IE-OrlaNeural",
    "Gender": "Female",
    "Locale": "ga-IE",
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "Status": "Preview"
  },
  
  ...
    
   {
    "Name": "Microsoft Server Speech Text to Speech Voice (zh-CN, YunxiNeural)",
    "DisplayName": "Yunxi",
    "LocalName": "云希",
    "ShortName": "zh-CN-YunxiNeural",
    "Gender": "Male",
    "Locale": "zh-CN",
    "StyleList": [
      "Calm",
      "Fearful",
      "Cheerful",
      "Disgruntled",
      "Serious",
      "Angry",
      "Sad",
      "Depressed",
      "Embarrassed"
    ],
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "Status": "Preview"
  },

    ...
]
```

### <a name="http-status-codes"></a>HTTP 狀態碼

每個回應的 HTTP 狀態碼會指出成功或常見的錯誤。

| HTTP 狀態碼 | 描述 | 可能的原因 |
|------------------|-------------|-----------------|
| 200 | [確定] | 要求成功。 |
| 400 | 不正確的要求 | 必要的參數遺失、為空白或 Null。 或者，傳遞至必要或選用參數的值無效。 常見的問題是標頭太長。 |
| 401 | 未經授權 | 要求未經授權。 請檢查以確定您的訂用帳戶金鑰或權杖有效，並且位於正確的區域。 |
| 429 | 太多要求 | 您已超出訂用帳戶允許的配額或要求率。 |
| 502 | 錯誤的閘道    | 網路或伺服器端問題。 也可能表示標頭無效。 |


## <a name="convert-text-to-speech"></a>將文字轉換成語音

此 `v1` 端點可讓您使用 [語音合成標記語言 (SSML) ](speech-synthesis-markup.md)將文字轉換成語音。

### <a name="regions-and-endpoints"></a>區域與端點

支援使用 REST API 對以下區域進行文字轉語音。 請確定選取的是符合您訂用帳戶區域的端點。

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-text-to-speech.md)]

### <a name="request-headers"></a>要求標頭

下表列出文字轉換語音要求的必要和選擇性標頭。

| 標頭 | 描述 | 必要/選用 |
|--------|-------------|---------------------|
| `Authorization` | 前面加入 `Bearer` 這個字的授權權杖。 如需詳細資訊，請參閱[驗證](#authentication)。 | 必要 |
| `Content-Type` | 指定所提供文字的內容類型。 接受的值為 `application/ssml+xml`。 | 必要 |
| `X-Microsoft-OutputFormat` | 指定音訊輸出格式。 如需接受值的完整清單，請參閱[音訊輸出](#audio-outputs)。 | 必要 |
| `User-Agent` | 應用程式名稱。 提供的值必須小於255個字元。 | 必要 |

### <a name="audio-outputs"></a>音訊輸出

此清單列出了每個要求中系統做為 `X-Microsoft-OutputFormat` 標頭的傳送的支援音訊格式。 每個格式皆包含位元速率和編碼類型。 語音服務支援 24 kHz、16 kHz 和 8 kHz 音訊輸出。

```output
raw-16khz-16bit-mono-pcm            raw-8khz-8bit-mono-mulaw
riff-8khz-8bit-mono-alaw            riff-8khz-8bit-mono-mulaw
riff-16khz-16bit-mono-pcm           audio-16khz-128kbitrate-mono-mp3
audio-16khz-64kbitrate-mono-mp3     audio-16khz-32kbitrate-mono-mp3
raw-24khz-16bit-mono-pcm            riff-24khz-16bit-mono-pcm
audio-24khz-160kbitrate-mono-mp3    audio-24khz-96kbitrate-mono-mp3
audio-24khz-48kbitrate-mono-mp3     ogg-24khz-16bit-mono-opus
```

> [!NOTE]
> 如果您選取的語音和輸出格式具有不同的位元速率，則會視需要重新進行音訊取樣。 ogg-24khz-16bit-mono-opus 可以使用[opus 編解碼器](https://opus-codec.org/downloads/)解碼

### <a name="request-body"></a>要求本文

每個 `POST` 要求的本文都會以[語音合成標記語言 (SSML)](speech-synthesis-markup.md) 形式傳送。 SSML 可讓您選擇文字轉換語音服務所傳回合成語音的語音和語言。 如需支援的完整語音清單，請參閱[語言支援](language-support.md#text-to-speech)。

> [!NOTE]
> 如果使用自訂語音，可以純文字 (ASCII 或 UTF-8) 形式傳送要求本文。

### <a name="sample-request"></a>範例要求

此 HTTP 要求使用 SSML 指定語音與語言。 如果本文長度很長，且產生的音訊超過10分鐘，則會截斷為10分鐘。 換句話說，音訊長度不能超過10分鐘。

```http
POST /cognitiveservices/v1 HTTP/1.1

X-Microsoft-OutputFormat: raw-24khz-16bit-mono-pcm
Content-Type: application/ssml+xml
Host: westus.tts.speech.microsoft.com
Content-Length: 225
Authorization: Bearer [Base64 access_token]

<speak version='1.0' xml:lang='en-US'><voice xml:lang='en-US' xml:gender='Female'
    name='en-US-AriaNeural'>
        Microsoft Speech Service Text-to-Speech API
</voice></speak>
```

### <a name="http-status-codes"></a>HTTP 狀態碼

每個回應的 HTTP 狀態碼會指出成功或常見的錯誤。

| HTTP 狀態碼 | 描述 | 可能的原因 |
|------------------|-------------|-----------------|
| 200 | [確定] | 要求成功；回應主體是音訊檔案。 |
| 400 | 不正確的要求 | 必要的參數遺失、為空白或 Null。 或者，傳遞至必要或選用參數的值無效。 常見的問題是標頭太長。 |
| 401 | 未經授權 | 要求未經授權。 請檢查以確定您的訂用帳戶金鑰或權杖有效，並且位於正確的區域。 |
| 415 | 不支援的媒體類型 | 提供錯誤的可能原因 `Content-Type` 。 `Content-Type` 應設定為 `application/ssml+xml` 。 |
| 429 | 太多要求 | 您已超出訂用帳戶允許的配額或要求率。 |
| 502 | 錯誤的閘道    | 網路或伺服器端問題。 也可能表示標頭無效。 |

如果 HTTP 狀態為 `200 OK`，則回應主體會包含所要求格式的音訊檔案。 會在將此檔案傳輸、儲存到緩衝區或儲存到檔案時播放。

## <a name="next-steps"></a>後續步驟

- [建立 Azure 免費帳戶](https://azure.microsoft.com/free/cognitive-services/)
- [適用于長格式音訊的非同步合成](quickstarts/text-to-speech/async-synthesis-long-form-audio.md)
- [開始使用自訂語音](how-to-custom-voice.md)
