---
title: Microsoft 語音服務的文字轉換語音 API | Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 使用文字轉換語音 API 以各種不同的語音和語言，提供即時文字到語音的轉換
services: cognitive-services
author: priyaravi20
manager: yanbo
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: priyar
ms.openlocfilehash: db69a9e3beb819600109603a8c0129547db57fa5
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49343022"
---
# <a name="bing-text-to-speech-api"></a>Bing 文字轉換語音 API

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

## <a name="Introduction"></a>簡介

您的應用程式可以利用 Bing 文字轉換語音 API，將 HTTP 要求傳送至雲端伺服器，其中文字會立即合成為真人發音的語音並傳回為音訊檔案。 此 API 可以用於許多不同的情況，以各種不同的語音和語言，提供即時文字到語音的轉換。

## <a name="VoiceSynReq"></a>語音合成要求

### <a name="Subscription"></a>授權權杖

每個語音合成要求都需要 JSON Web 權杖 (JWT) 存取權杖。 JWT 存取權杖是在語音要求標頭中傳遞。 權杖的到期時間為 10 分鐘。 如需有關訂閱並取得用於擷取有效 JWT 存取權杖之 API 金鑰的資訊，請參閱[認知服務訂閱](https://azure.microsoft.com/try/cognitive-services/)。

API 金鑰會傳遞至權杖服務。 例如︰

```HTTP
POST https://api.cognitive.microsoft.com/sts/v1.0/issueToken
Content-Length: 0
```

權杖存取所需的標頭資訊如下。

名稱| 格式 | 說明
----|----|----
Ocp-Apim-Subscription-Key | ASCII | 您的訂用帳戶金鑰

權杖服務會以 `text/plain` 形式傳回 JWT 存取權杖。 接著，JWT 會以 `Base64 access_token` 形式傳遞至語音端點，作為字首字串為 `Bearer` 的授權標頭。 例如︰

`Authorization: Bearer [Base64 access_token]`

用戶端必須使用下列端點來存取文字轉換語音服務：

`https://speech.platform.bing.com/synthesize`

>[!NOTE]
>如先前所述，在您使用訂用帳戶金鑰取得存取權杖之前，此連結會產生 `403 Forbidden` 回應錯誤。

### <a name="Http"></a>HTTP 標頭

下表顯示用於語音合成要求的 HTTP 標頭。

頁首 |值 |註解
----|----|----
Content-Type | application/ssml+xml | 輸入內容類型。
X-Microsoft-OutputFormat | **1.** ssml-16khz-16bit-mono-tts <br> **2.** raw-16khz-16bit-mono-pcm <br>**3.** audio-16khz-16kbps-mono-siren <br> **4.** riff-16khz-16kbps-mono-siren <br> **5.** riff-16khz-16bit-mono-pcm <br> **6.** audio-16khz-128kbitrate-mono-mp3 <br> **7.** audio-16khz-64kbitrate-mono-mp3 <br> **8.** audio-16khz-32kbitrate-mono-mp3 | 輸出音訊格式。
X-Search-AppId | GUID (僅限十六進位，無破折號) | 可唯一識別用戶端應用程式的識別碼。 這可以是應用程式的存放區識別碼。 如果沒有可用的 GUID，使用者可以針對應用程式產生識別碼。
X-Search-ClientID | GUID (僅限十六進位，無破折號) | 可唯一識別每個安裝之應用程式執行個體的識別碼。
User-Agent | 應用程式名稱 | 應用程式名稱是必要項目，而且必須少於 255 個字元。
Authorization | 授權權杖 |  請參閱<a href="#Subscription">授權權杖</a>一節。

### <a name="InputParam"></a>輸入參數

對 Bing 文字轉換語音 API 的要求是使用 HTTP POST 呼叫提出的。 標頭是在上一節指定的。 本文中會包含語音合成標記語言 (SSML) 輸入，此輸入代表要合成的文字。 如需用來控制語音各個層面 (例如說話者的語言和性別) 之標記的描述，請參閱 [SSML W3C 規格](http://www.w3.org/TR/speech-synthesis/) \(英文\)。

>[!NOTE]
>SSML 輸入所支援的大小上限為 1,024 個字元，包括所有標籤。

###  <a name="SampleVoiceOR"></a>範例：語音輸出要求

語音輸出要求的範例如下：

```HTTP
POST /synthesize
HTTP/1.1
Host: speech.platform.bing.com

X-Microsoft-OutputFormat: riff-8khz-8bit-mono-mulaw
Content-Type: application/ssml+xml
Host: speech.platform.bing.com
Content-Length: 197
Authorization: Bearer [Base64 access_token]

<speak version='1.0' xml:lang='en-US'><voice xml:lang='en-US' xml:gender='Female' name='Microsoft Server Speech Text to Speech Voice (en-US, ZiraRUS)'>Microsoft Bing Voice Output API</voice></speak>
```

## <a name="VoiceOutResponse"></a>語音輸出回應

Bing 文字轉換語音 API 會使用 HTTP POST，將音訊傳送回用戶端。 API 回應中包含音訊串流和轉碼器，而且它會比對要求的輸出格式。 針對給定要求傳回的音訊不得超過 15 秒。

### <a name="SuccessfulRecResponse"></a>範例：成功的合成回應

下列程式碼是成功語音合成要求之 JSON 回應的範例。 程式碼的註解和格式僅用於此範例的目的，因此會從實際的回應中省略。

```HTTP
HTTP/1.1 200 OK
Content-Length: XXX
Content-Type: audio/x-wav

Response audio payload
```

### <a name="RecFailure"></a>範例：合成失敗

下列範例程式碼顯示語音合成查詢失敗的 JSON 回應：

```HTTP
HTTP/1.1 400 XML parser error
Content-Type: text/xml
Content-Length: 0
```

### <a name="ErrorResponse"></a>錯誤回應

Error | 說明
----|----
HTTP/400 不正確的要求 | 必要參數遺漏、空白或 Null，或傳遞至必要或選用參數的值無效。 取得「無效」回應的其中一個原因是傳遞超過允許長度的字串值。 其中包含有問題的參數的簡短描述。
HTTP 401 未經授權 | 此要求未經授權。
HTTP/413 RequestEntityTooLarge  | SSML 輸入大於支援的限制。
HTTP/502 BadGateway | 發生網路相關問題或伺服器端問題。

錯誤回應的範例如下：

```HTTP
HTTP/1.0 400 Bad Request
Content-Length: XXX
Content-Type: text/plain; charset=UTF-8

Voice name not supported
```

## <a name="ChangeSSML"></a>透過 SSML 變更語音輸出

Microsoft 文字轉換語音 API 支援 SSML 1.0，如 W3C [語音合成標記語言 (SSML) 1.0 版](http://www.w3.org/TR/2009/REC-speech-synthesis-20090303/) \(英文\) 中所定義。 本節針對使用 SSML 標記所產生的語音輸出 (例如說話速度、發音等)，顯示變更其部分特性的範例。

1. 新增中斷

  ```
  <speak version='1.0' xmlns="http://www.w3.org/2001/10/synthesis" xml:lang='en-US'><voice  name='Microsoft Server Speech Text to Speech Voice (en-US, BenjaminRUS)'> Welcome to use Microsoft Cognitive Services <break time="100ms" /> Text-to-Speech API.</voice> </speak>
  ```

2. 改變說話速度

  ```
  <speak version='1.0' xmlns="http://www.w3.org/2001/10/synthesis" xml:lang='en-US'><voice  name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'><prosody rate="+30.00%">Welcome to use Microsoft Cognitive Services Text-to-Speech API.</prosody></voice> </speak>
  ```

3. 發音

  ```
  <speak version='1.0' xmlns="http://www.w3.org/2001/10/synthesis" xml:lang='en-US'><voice  name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'> <phoneme alphabet="ipa" ph="t&#x259;mei&#x325;&#x27E;ou&#x325;"> tomato </phoneme></voice> </speak>
  ```

4. 變更音量

  ```
  <speak version='1.0' xmlns="http://www.w3.org/2001/10/synthesis" xml:lang='en-US'><voice  name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'><prosody volume="+20.00%">Welcome to use Microsoft Cognitive Services Text-to-Speech API.</prosody></voice> </speak>
  ```

5. 變更音高

  ```
  <speak version='1.0' xmlns="http://www.w3.org/2001/10/synthesis" xml:lang='en-US'><voice  name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'>Welcome to use <prosody pitch="high">Microsoft Cognitive Services Text-to-Speech API.</prosody></voice> </speak>
  ```

6. 變更韻律結構

  ```
  <speak version='1.0' xmlns="http://www.w3.org/2001/10/synthesis" xml:lang='en-US'><voice  name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'><prosody contour="(80%,+20%) (90%,+30%)" >Good morning.</prosody></voice> </speak>
  ```

> [!NOTE]
> 請注意，音訊資料必須為下列格式的 8k 或 16k wav 檔案：**CRC 代碼** (CRC-32)：4 個位元組 (DWORD)，有效範圍為 0x00000000 ~ 0xFFFFFFFF；**音訊格式旗標**：4 個位元組 (DWORD)，有效範圍為 0x00000000 ~ 0xFFFFFFFF；**樣本計數**：4 個位元組 (DWORD)，有效範圍為 0x00000000 ~ 0x7FFFFFFF；**二進位內容的大小**：4 個位元組 (DWORD)，有效範圍為 0x00000000 ~ 0x7FFFFFFF；**二進位內容**：n 個位元組。

## <a name="SampleApp"></a>範例應用程式

如需實作詳細資料，請參閱 [Visual C#.NET 文字轉換語音範例應用程式](https://github.com/Microsoft/Cognitive-Speech-TTS/blob/master/Samples-Http/CSharp/TTSProgram.cs) \(英文\)。

## <a name="SupLocales"></a>支援的地區設定和語音字型

下表列出部分支援的地區設定和相關的語音字型。

地區設定 | 性別 | 服務名稱對應
---------|--------|------------
ar-EG* | 女性 | "Microsoft Server Speech Text to Speech Voice (ar-EG, Hoda)"
ar-SA | 男性 | "Microsoft Server Speech Text to Speech Voice (ar-SA, Naayf)"
bg-BG | 男性 | "Microsoft Server Speech Text to Speech Voice (bg-BG, Ivan)"
ca-ES | 女性 | "Microsoft Server Speech Text to Speech Voice (ca-ES, HerenaRUS)"
cs-CZ | 男性 | "Microsoft Server Speech Text to Speech Voice (cs-CZ, Jakub)"
da-DK | 女性 | "Microsoft Server Speech Text to Speech Voice (da-DK, HelleRUS)"
de-AT | 男性 | "Microsoft Server Speech Text to Speech Voice (de-AT, Michael)"
de-CH | 男性 | "Microsoft Server Speech Text to Speech Voice (de-CH, Karsten)"
de-DE | 女性 | "Microsoft Server Speech Text to Speech Voice (de-DE, Hedda) "
de-DE | 女性 | "Microsoft Server Speech Text to Speech Voice (de-DE, HeddaRUS)"
de-DE | 男性 | "Microsoft Server Speech Text to Speech Voice (de-DE, Stefan, Apollo) "
el-GR | 男性 | "Microsoft Server Speech Text to Speech Voice (el-GR, Stefanos)"
en-AU | 女性 | "Microsoft Server Speech Text to Speech Voice (en-AU, Catherine) "
en-AU | 女性 | "Microsoft Server Speech Text to Speech Voice (en-AU, HayleyRUS)"
en-CA | 女性 | "Microsoft Server Speech Text to Speech Voice (en-CA, Linda)"
en-CA | 女性 | "Microsoft Server Speech Text to Speech Voice (en-CA, HeatherRUS)"
en-GB | 女性 | "Microsoft Server Speech Text to Speech Voice (en-GB, Susan, Apollo)"
en-GB | 女性 | "Microsoft Server Speech Text to Speech Voice (en-GB, HazelRUS)"
en-GB | 男性 | "Microsoft Server Speech Text to Speech Voice (en-GB, George, Apollo)"
en-IE | 男性 | "Microsoft Server Speech Text to Speech Voice (en-IE, Sean)"
en-IN | 女性 | "Microsoft Server Speech Text to Speech Voice (en-IN, Heera, Apollo)"
en-IN | 女性 | "Microsoft Server Speech Text to Speech Voice (en-IN, PriyaRUS)"
en-IN | 男性 | "Microsoft Server Speech Text to Speech Voice (en-IN, Ravi, Apollo)"
zh-TW | 女性 | "Microsoft Server Speech Text to Speech Voice (en-US, ZiraRUS)"
zh-TW | 女性 | "Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)"
zh-TW | 男性 | "Microsoft Server Speech Text to Speech Voice (en-US, BenjaminRUS)"
es-ES | 女性 | "Microsoft Server Speech Text to Speech Voice (es-ES, Laura, Apollo)"
es-ES | 女性 | "Microsoft Server Speech Text to Speech Voice (es-ES, HelenaRUS)"
es-ES | 男性 | "Microsoft Server Speech Text to Speech Voice (es-ES, Pablo, Apollo)"
es-MX | 女性 | "Microsoft Server Speech Text to Speech Voice (es-MX, HildaRUS)"
es-MX | 男性 | "Microsoft Server Speech Text to Speech Voice (es-MX, Raul, Apollo)"
fi-FI | 女性 | "Microsoft Server Speech Text to Speech Voice (fi-FI, HeidiRUS)"
fr-CA | 女性 | "Microsoft Server Speech Text to Speech Voice (fr-CA, Caroline)"
fr-CA | 女性 | "Microsoft Server Speech Text to Speech Voice (fr-CA, HarmonieRUS)"
fr-CH | 男性 | "Microsoft Server Speech Text to Speech Voice (fr-CH, Guillaume)"
fr-FR | 女性 | "Microsoft Server Speech Text to Speech Voice (fr-FR, Julie, Apollo)"
fr-FR | 女性 | "Microsoft Server Speech Text to Speech Voice (fr-FR, HortenseRUS)"
fr-FR | 男性 | "Microsoft Server Speech Text to Speech Voice (fr-FR, Paul, Apollo)"
he-IL| 男性| "Microsoft Server Speech Text to Speech Voice (he-IL, Asaf)"
hi-IN | 女性 | "Microsoft Server Speech Text to Speech Voice (hi-IN, Kalpana, Apollo)"
hi-IN | 女性 | "Microsoft Server Speech Text to Speech Voice (hi-IN, Kalpana)"
hi-IN | 男性 | "Microsoft Server Speech Text to Speech Voice (hi-IN, Hemant)"
hr-HR | 男性 | "Microsoft Server Speech Text to Speech Voice (hr-HR, Matej)"
hu-HU | 男性 | "Microsoft Server Speech Text to Speech Voice (hu-HU, Szabolcs)"
id-ID | 男性 | "Microsoft Server Speech Text to Speech Voice (id-ID, Andika)"
it-IT | 男性 | "Microsoft Server Speech Text to Speech Voice (it-IT, Cosimo, Apollo)"
it-IT | 女性 | "Microsoft Server Speech Text to Speech Voice (it-IT, LuciaRUS)"
ja-JP | 女性 | "Microsoft Server Speech Text to Speech Voice (ja-JP, Ayumi, Apollo)"
ja-JP | 男性 | "Microsoft Server Speech Text to Speech Voice (ja-JP, Ichiro, Apollo)"
ja-JP | 女性 | "Microsoft Server Speech Text to Speech Voice (ja-JP, HarukaRUS)"
ko-KR | 女性 | "Microsoft Server Speech Text to Speech Voice (ko-KR, HeamiRUS)"
ms-MY | 男性 | "Microsoft Server Speech Text to Speech Voice (ms-MY, Rizwan)"
nb-NO | 女性 | "Microsoft Server Speech Text to Speech Voice (nb-NO, HuldaRUS)"
nl-NL | 女性 | "Microsoft Server Speech Text to Speech Voice (nl-NL, HannaRUS)"
pl-PL | 女性 | "Microsoft Server Speech Text to Speech Voice (pl-PL, PaulinaRUS)"
pt-BR | 女性 | "Microsoft Server Speech Text to Speech Voice (pt-BR, HeloisaRUS)"
pt-BR | 男性 | "Microsoft Server Speech Text to Speech Voice (pt-BR, Daniel, Apollo)"
pt-PT | 女性 | "Microsoft Server Speech Text to Speech Voice (pt-PT, HeliaRUS)"
ro-RO | 男性 | "Microsoft Server Speech Text to Speech Voice (ro-RO, Andrei)"
ru-RU | 女性 | "Microsoft Server Speech Text to Speech Voice (ru-RU, Irina, Apollo)"
ru-RU | 男性 | "Microsoft Server Speech Text to Speech Voice (ru-RU, Pavel, Apollo)"
ru-RU | 女性 | "Microsoft Server Speech Text to Speech Voice (ru-RU, EkaterinaRUS)"
sk-SK | 男性 | "Microsoft Server Speech Text to Speech Voice (sk-SK, Filip)"
sl-SI | 男性 | "Microsoft Server Speech Text to Speech Voice (sl-SI, Lado)"
sv-SE | 女性 | "Microsoft Server Speech Text to Speech Voice (sv-SE, HedvigRUS)"
ta-IN | 男性 | "Microsoft Server Speech Text to Speech Voice (ta-IN, Valluvar)"
th-TH | 男性 | "Microsoft Server Speech Text to Speech Voice (th-TH, Pattara)"
tr-TR | 女性 | "Microsoft Server Speech Text to Speech Voice (tr-TR, SedaRUS)"
vi-VN | 男性 | "Microsoft Server Speech Text to Speech Voice (vi-VN, An)"
zh-CN | 女性 | "Microsoft Server Speech Text to Speech Voice (zh-CN, HuihuiRUS)"
zh-CN | 女性 | "Microsoft Server Speech Text to Speech Voice (zh-CN, Yaoyao, Apollo)"
zh-CN | 男性 | "Microsoft Server Speech Text to Speech Voice (zh-CN, Kangkang, Apollo)"
zh-HK | 女性 | "Microsoft Server Speech Text to Speech Voice (zh-HK, Tracy, Apollo)"
zh-HK | 女性 | "Microsoft Server Speech Text to Speech Voice (zh-HK, TracyRUS)"
zh-HK | 男性 | "Microsoft Server Speech Text to Speech Voice (zh-HK, Danny, Apollo)"
zh-TW | 女性 | "Microsoft Server Speech Text to Speech Voice (zh-TW, Yating, Apollo)"
zh-TW | 女性 | "Microsoft Server Speech Text to Speech Voice (zh-TW, HanHanRUS)"
zh-TW | 男性 | "Microsoft Server Speech Text to Speech Voice (zh-TW, Zhiwei, Apollo)"
 *ar-EG 支援現代標準阿拉伯文 (MSA)。

> [!NOTE]
> 請注意，先前的服務名稱 **Microsoft Server Speech Text to Speech Voice (cs-CZ, Vit)** 和 **Microsoft Server Speech Text to Speech Voice (en-IE, Shaun)** 將會在 2018 年 3 月 31 日之後被取代，以最佳化 Bing 語音 API 的功能。 請以更新的名稱更新您的程式碼。

## <a name="TrouNSupport"></a>疑難排解和支援

將所有問題和議題張貼到 [Bing 語音服務](https://social.msdn.microsoft.com/Forums/en-US/home?forum=SpeechService) MSDN 論壇。 包括完整的詳細資訊，例如：

* 完整要求字串的範例。
* 如果適用的話，失敗要求的完整輸出，包含記錄識別碼。
* 失敗的要求百分比。
