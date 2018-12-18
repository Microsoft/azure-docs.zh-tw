---
title: Bing 語音 WebSocket 通訊協定 | Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 以 WebSocket 為基礎的 Bing 語音通訊協定文件
services: cognitive-services
author: zhouwangzw
manager: wolfma
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: zhouwang
ms.openlocfilehash: 0bbc6b638d11335e6d46501fa651996f05957dd5
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49341815"
---
# <a name="bing-speech-websocket-protocol"></a>Bing 語音 WebSocket 通訊協定

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

Bing 語音是一個雲端架構的平台，具備適用於將語音轉換為文字的最先進演算法。 Bing 語音通訊協定會定義用戶端應用程式和服務之間的[連線設定](#connection-establishment)，以及在對應項目 ([用戶端產生的訊息](#client-originated-messages)和[服務產生的訊息](#service-originated-messages)) 之間交換的語音辨識訊息。 此外還會描述[遙測訊息](#telemetry-schema)和[錯誤處理](#error-handling)。

## <a name="connection-establishment"></a>連線建立

語音服務通訊協定會遵循 WebSocket 標準規格 [IETF RFC 6455](https://tools.ietf.org/html/rfc6455)。 WebSocket 連線會從包含 HTTP 標頭的 HTTP 要求開始，這些要求會指出用戶端想要升級與 WebSocket 的連線，而不是使用 HTTP 語意。 伺服器會傳回 HTTP `101 Switching Protocols` 回應，藉此表示參加 WebSocket 連線的意願。 交換此信號交換之後，用戶端與服務都會使通訊端保持在開啟狀態，並開始使用訊息架構的通訊協定傳送和接收資訊。

若要開始 WebSocket 信號交換，用戶端應用程式會將 HTTPS GET 要求傳送至服務。 此要求包含標準 WebSocket 升級標頭，以及專屬於語音的其他標頭。

```HTTP
GET /speech/recognition/interactive/cognitiveservices/v1 HTTP/1.1
Host: speech.platform.bing.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Authorization: t=EwCIAgALBAAUWkziSCJKS1VkhugDegv7L0eAAJqBYKKTzpPZOeGk7RfZmdBhYY28jl&p=
X-ConnectionId: A140CAF92F71469FA41C72C7B5849253
Origin: https://speech.platform.bing.com
```

服務會使用以下內容回應：

```HTTP
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: upgrade
Sec-WebSocket-Key: 2PTTXbeeBXlrrUNsY15n01d/Pcc=
Set-Cookie: SpeechServiceToken=AAAAABAAWTC8ncb8COL; expires=Wed, 17 Aug 2016 15:39:06 GMT; domain=bing.com; path="/"
Date: Wed, 17 Aug 2016 15:03:52 GMT
```

所有語音要求都需要經過 [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) 加密。 不支援使用未加密的語音要求。 支援下列 TLS 版本：

* TLS 1.2

### <a name="connection-identifier"></a>連線識別碼

語音服務會要求所有用戶端都包含唯一識別碼來識別連線。 啟動 WebSocket 信號交換時，用戶端*必須*包含 *X-ConnectionId* 標頭。 *X-ConnectionId* 標頭必須是[通用唯一識別碼](https://en.wikipedia.org/wiki/Universally_unique_identifier) (UUID) 值。 不包含 *X-ConnectionId*、不指定 *X-ConnectionId* 標頭的值，或不包含有效 UUID 值的 WebSocket 升級要求將會遭到含有 HTTP `400 Bad Request` 回應的服務拒絕。

### <a name="authorization"></a>Authorization

除了標準 WebSocket 信號交換標頭之外，語音要求還需要有*授權*標頭。 不含此標頭的連線要求會遭到含有 HTTP `403 Forbidden` 回應的服務拒絕。

*授權*標頭必須包含 JSON Web Token (JWT) 存取權杖。

如需有關如何訂閱並取得擷取有效 JWT 存取權杖所使用之 API 金鑰的資訊，請參閱[認知服務訂閱](https://azure.microsoft.com/try/cognitive-services/)頁面。

API 金鑰會傳遞至權杖服務。 例如︰

``` HTTP
POST https://api.cognitive.microsoft.com/sts/v1.0/issueToken
Content-Length: 0
```

權杖存取需要有下列標頭資訊。

| 名稱 | 格式 | 說明 |
|----|----|----|
| Ocp-Apim-Subscription-Key | ASCII | 您的訂用帳戶金鑰 |

權杖服務會以 `text/plain` 形式傳回 JWT 存取權杖。 接著，JWT 會當作 `Base64 access_token` 傳遞至信號交換，作為字首字串為 `Bearer` 的*授權*標頭。 例如︰

`Authorization: Bearer [Base64 access_token]`

### <a name="cookies"></a>Cookie

用戶端*必須*支援 HTTP Cookie，如 [RFC 6265](https://tools.ietf.org/html/rfc6265) 中所指定。

### <a name="http-redirection"></a>HTTP 重新導向

用戶端*必須*支援 [HTTP 通訊協定規格](http://www.w3.org/Protocols/rfc2616/rfc2616.html)所指定的標準重新導向機制。

### <a name="speech-endpoints"></a>語音端點

用戶端*必須*使用適當的語音服務端點。 此端點是以辨識模式和語言為基礎。 下表顯示一些範例。

| Mode | Path | 服務 URI |
| -----|-----|-----|
| 互動式 | /speech/recognition/interactive/cognitiveservices/v1 |https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=pt-BR |
| 交談 | /speech/recognition/conversation/cognitiveservices/v1 |https://speech.platform.bing.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US |
| 聽寫 | /speech/recognition/dictation/cognitiveservices/v1 |https://speech.platform.bing.com/speech/recognition/dictation/cognitiveservices/v1?language=fr-FR |

如需詳細資訊，請參閱[服務 URI](../GetStarted/GetStartedREST.md#service-uri) 頁面。

### <a name="report-connection-problems"></a>回報連線問題

用戶端應該立即回報建立連線時所遇到的所有問題。 回報連線失敗的訊息通訊協定詳述於[連線失敗遙測](#connection-failure-telemetry)中。

### <a name="connection-duration-limitations"></a>連線持續時間限制

相較於一般的 Web 服務 HTTP 連線，WebSocket 連線會持續一段*很長的*時間。 語音服務會限制 WebSocket 與服務連線的持續時間：

 * 任何作用中 WebSocket 連線的最大持續時間為 10 分鐘。 如果服務或用戶端透過該連線傳送 WebSocket 訊息，則表示連線作用中。 達到限制時，服務就會終止連線，而不會發出警告。 用戶端應該開發不需要連線在處於或接近最大連線存留期保持作用中的使用者案例。

 * 任何非作用中 WebSocket 連線的最大持續時間為 180 秒。 如果服務或用戶端都不透過該連線傳送 WebSocket 訊息，則表示連線非作用中。 達到最大非作用中存留期之後，服務就會終止非作用中的 WebSocket 連線。

## <a name="message-types"></a>訊息類型

用戶端與服務之間建立 WebSocket 連線之後，用戶端和服務就可以傳送訊息。 本節描述這些 WebSocket 訊息的格式。

[IETF RFC 6455](https://tools.ietf.org/html/rfc6455) 會指定 WebSocket 訊息可以使用文字或二進位編碼傳輸資料。 這兩個編碼使用不同的線上格式。 每一種格式都經過最佳化，以便有效率地編碼、傳輸和解碼訊息承載。

### <a name="text-websocket-messages"></a>WebSocket 簡訊

WebSocket 簡訊隨附文字資訊的承載，其中包含以熟悉的雙歸位字元換行對 (用於 HTTP 訊息) 分隔的標頭和內文。 此外，WebSocket 簡訊如同 HTTP 訊息，會以 *name: value* 格式指定標頭，並以單一歸位字元換行對分隔。 WebSocket 簡訊中包含的任何文字都*必須*使用 [UTF-8](https://tools.ietf.org/html/rfc3629) 編碼。

WebSocket 簡訊必須在標頭 *Path* 中指定訊息路徑。 此標頭的值必須是本文件稍後定義的其中一種語音通訊協定訊息類型。

### <a name="binary-websocket-messages"></a>二進位 WebSocket 訊息

二進位 WebSocket 訊息隨附二進位承載。 在語音服務通訊協定中，使用二進位 WebSocket 訊息可將音訊傳送至服務，以及從服務接收音訊。 其他所有訊息是 WebSocket 簡訊。 

二進位 WebSocket 訊息如同 WebSocket 簡訊，都包含標頭和本文區段。 二進位 WebSocket 訊息的前兩個位元組會以 [big-endian](https://en.wikipedia.org/wiki/Endianness) 順序，指定標頭區段的 16 位元整數大小。 最小標頭區段大小為 0 個位元組。 大小上限為 8,192 個位元組。 二進位 WebSocket 訊息標頭中的文字*必須*使用 [US-ASCII](https://tools.ietf.org/html/rfc20) 編碼。

二進位 WebSocket 訊息中的標頭會以相同的格式編碼，如同 WebSocket 簡訊。 *name:value* 格式會以單一歸位字元換行對分隔。 二進位 WebSocket 訊息必須在標頭 *Path* 中指定訊息路徑。 此標頭的值必須是本文件稍後定義的其中一種語音通訊協定訊息類型。

WebSocket 簡訊和二進位 WebSocket 訊息均用於語音服務通訊協定。 

## <a name="client-originated-messages"></a>用戶端產生的訊息

建立連接之後，用戶端和服務就可以開始傳送訊息。 本節描述用戶端應用程式傳送至語音服務之訊息的格式和承載。 [[服務產生的訊息]](#service-originated-messages) 區段會顯示源自於語音服務且將傳送至用戶端應用程式的訊息。

用戶端傳送至這些服務的主要訊息為 `speech.config`、`audio` 和 `telemetry` 訊息。 在我們詳細考慮每個訊息之前，會描述所有這些訊息所需的常見訊息標頭。

### <a name="required-message-headers"></a>所需的訊息標頭

所有用戶端產生的訊息都需要有下列標頭。

| 頁首 | 值 |
|----|----|
| Path | 本文件中所指定的訊息路徑 |
| X-RequestId | 「無破折號」格式的 UUID |
| X-Timestamp | ISO 8601 格式的用戶端 UTC 時鐘時間戳記 |

#### <a name="x-requestid-header"></a>X-RequestId 標頭

用戶端產生的要求會以 *X-RequestId* 訊息標頭唯一識別。 所有用戶端產生的訊息都需要有此標頭。 *X-RequestId* 標頭值必須是「無破折號」格式的 UUID，例如 *123e4567e89b12d3a456426655440000*。 它*無法*採用標準格式 *123e4567-e89b-12d3-a456-426655440000*。 如果要求不含 *X-RequestId* 標頭，或含使用 UUID 錯誤格式的標頭值，則會使服務終止 WebSocket 連線。

#### <a name="x-timestamp-header"></a>X-Timestamp 標頭

用戶端應用程式傳送至語音服務的每個訊息都*必須*包含 *X-Timestamp* 標頭。 此標頭的值是用戶端傳送訊息的時間。 如果要求不含 *X-Timestamp* 標頭，或含使用錯誤格式的標頭值，則會使服務終止 WebSocket 連線。

*X-Timestamp* 標頭值必須採用 'yyyy'-'MM'-'dd'T'HH':'mm':'ss'.'fffffffZ' 的格式，其中 'fffffff' 是秒數的小數。 例如，「12.5」表示「12 + 5/10 秒」，而「12.526」則表示「12 加上 526/1000 秒」。 此格式符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)，而且不同於標準 HTTP *Date* 標頭，可以提供毫秒解析。 用戶端應用程式可能會將時間戳記四捨五入到最接近的毫秒。 用戶端應用程式需要使用[網路時間通訊協定 (NTP) 伺服器](https://en.wikipedia.org/wiki/Network_Time_Protocol)，確保裝置時鐘正確地追蹤時間。

### <a name="message-speechconfig"></a>訊息 `speech.config`

語音服務必須知道您應用程式的特性，才能提供可能最佳的語音辨識。 所需的特性資料包括支援您應用程式的裝置和作業系統相關資訊。 您要在 `speech.config` 訊息中提供這項資訊。

用戶端*必須*在建立與語音服務的連線之後，以及在傳送任何 `audio` 訊息之前，立即傳送 `speech.config` 訊息。 每個連線僅需要傳送 `speech.config` 訊息一次。

| 欄位 | 說明 |
|----|----|
| WebSocket 訊息編碼 | 文字 |
| body | JSON 結構的承載 |

#### <a name="required-message-headers"></a>所需的訊息標頭

| 標頭名稱 | 值 |
|----|----|
| Path | `speech.config` |
| X-Timestamp | ISO 8601 格式的用戶端 UTC 時鐘時間戳記 |
| Content-Type | application/json; charset=utf-8 |

如同語音服務通訊協定中所有用戶端產生的訊息，`speech.config` 訊息*必須*包含 *X-Timestamp* 標頭，此標頭可在訊息傳送至服務時，記錄用戶端 UTC 時鐘的時間。 `speech.config` 訊息*不*需要有 *X-RequestId* 標頭，因為此訊息與特定的語音要求沒有關聯。

#### <a name="message-payload"></a>訊息承載
`speech.config` 訊息的承載是 JSON 結構，其中包含應用程式的相關資訊。 下列範例顯示這項資訊。 用戶端和裝置內容資訊都包含在 JSON 結構的 *context* 元素中。 

```JSON
{
  "context": {
    "system": {
      "version": "2.0.12341",
    },
    "os": {
      "platform": "Linux",
      "name": "Debian",
      "version": "2.14324324"
    },
    "device": {
      "manufacturer": "Contoso",
      "model": "Fabrikan",
      "version": "7.341"
      }
    },
  }
}
```

##### <a name="system-element"></a>系統元素

`speech.config` 訊息的 system.version 元素包含用戶端應用程式或裝置使用之語音 SDK 軟體的版本。 此值採用 *major.minor.build.branch* 的格式。 如果不適用，您可以省略*分支*元件。

##### <a name="os-element"></a>作業系統元素

| 欄位 | 說明 | 使用量 |
|-|-|-|
| os.platform | 裝載應用程式的作業系統平台，例如，Windows、Android、iOS 或 Linux |必要 |
| os.name | 作業系統產品名稱，例如，Debian 或 Windows 10 | 必要 |
| os.version | 採用 *major.minor.build.branch* 格式的作用系統版本 | 必要 |

##### <a name="device-element"></a>裝置元素

| 欄位 | 說明 | 使用量 |
|-|-|-|
| device.manufacturer | 裝置硬體製造商 | 必要 |
| device.model | 裝置機型 | 必要 |
| device.version | 裝置製造商提供的裝置軟體版本。 此值會指定可由製造商追蹤之裝置的版本。 | 必要 |

### <a name="message-audio"></a>訊息 `audio`

啟用語音的用戶端應用程式會將音訊串流轉換成一系列的音訊區塊，藉此將音訊傳送至語音服務。 音訊的每個區塊都隨附一段要由服務轉譯的語音。 單一音訊區塊的大小上限為 8,192 個位元組。 音訊串流訊息為*二進位 WebSocket 訊息*。

用戶端使用 `audio` 訊息，將音訊區塊傳送至服務。 用戶端會從麥克風讀取區塊中的音訊，並將這些區塊傳送至語音服務進行轉譯。 第一個 `audio` 訊息必須包含格式正確的標頭，此標頭可正確地指定音訊符合服務所支援的其中一個編碼格式。 其他 `audio` 訊息僅包含從麥克風讀取的二進位音訊串流資料。

用戶端可能會選擇性地傳送本文長度為零的 `audio` 訊息。 此訊息會告知服務，用戶端知道使用者已停止說話、表達方式已完成，而且麥克風已關閉。

語音服務會使用包含唯一要求識別碼的第一個 `audio` 訊息，對開始新的要求/回應週期或*回合*發出訊號。 在服務接收具有新要求識別碼的 `audio` 訊息之後，就會捨棄與先前回合相關聯的任何佇列或未傳送的訊息。

| 欄位 | 說明 |
|-------------|----------------|
| WebSocket 訊息編碼 | Binary |
| body | 音訊區塊的二進位資料。 大小上限為 8,192 個位元組。 |

#### <a name="required-message-headers"></a>所需的訊息標頭

所有 `audio` 訊息都需要有下列標頭。

| 頁首         |  值     |
| ------------- | ---------------- |
| Path | `audio` |
| X-RequestId | 「無破折號」格式的 UUID |
| X-Timestamp | ISO 8601 格式的用戶端 UTC 時鐘時間戳記 |
| Content-Type | 音訊內容類型。 類型必須是 *audio/x-wav* (PCM) 或 *audio/silk* (SILK)。 |

#### <a name="supported-audio-encodings"></a>支援的音訊編碼

本節描述語音服務支援的音訊轉碼器。

##### <a name="pcm"></a>PCM

語音服務會接受未壓縮的脈衝碼調制 (PCM) 音訊。 音訊會以 [WAV](https://en.wikipedia.org/wiki/WAV) 格式傳送至服務，因此第一個音訊區塊*必須*包含有效的[資源交換檔案格式](https://en.wikipedia.org/wiki/Resource_Interchange_File_Format) (RIFF) 標頭。 如果用戶端使用*不*包含無效 RIFF 標頭的音訊區塊起始回合，服務會拒絕要求，並終止 WebSocket 連線。

PCM 音訊*必須*以 16 kHz，使用每個範例和通道 16 位元 (*riff-16khz-16bit-mono-pcm*) 的大小取樣。 語音服務不支援立體聲音訊串流，而且會拒絕未使用指定的位元速率、取樣率或通道數目的音訊串流。

##### <a name="opus"></a>Opus

Opus 是一種開啟式、免權利金、用途廣泛的音訊轉碼器。 語音服務以限制的 `32000` 或 `16000` 位元速率支援 Opus。 目前僅支援 `audio/ogg` MIME 類型指定之 Opus 的 `OGG` 容器。

若要使用 Opus，請修改 [JavaScript 範例](https://github.com/Azure-Samples/SpeechToText-WebSockets-Javascript/blob/master/samples/browser/Sample.html#L101)並變更要傳回的 `RecognizerSetup` 方法。

```javascript
return SDK.CreateRecognizerWithCustomAudioSource(
            recognizerConfig,
            authentication,
            new SDK.MicAudioSource(
                     new SDK.OpusRecorder(
                     {
                         mimeType: "audio/ogg",
                         bitsPerSecond: 32000
                     }
              )
          ));
```

#### <a name="detect-end-of-speech"></a>偵測語音結束

當人類說完話時，不會明確地發出訊號。 接受語音作為輸入的任何應用程式在音訊串流中都有兩種處理語音結束的選擇：服務語音結束偵測以及用戶端語音結束偵測。 在這兩種選擇中，服務語音結束偵測通常提供更好的使用者體驗。

##### <a name="service-end-of-speech-detection"></a>服務語音結束偵測

若要打造理想的免持聽筒語音體驗，應用程式可讓服務偵測使用者什麼時候說完話。 用戶端會以*音訊*區塊的方式，從麥克風傳送音訊，直到服務偵測到無回應，並回應 `speech.endDetected` 訊息。

##### <a name="client-end-of-speech-detection"></a>用戶端語音結束偵測

允許使用者以特定方式對語音結束發出訊號的用戶端應用程式也可以為服務提供該訊號。 例如，用戶端應用程式可能會有使用者可以按下的 [停止] 或 [靜音] 按鈕。 若要對語音結束發出訊號，用戶端應用程式可傳送內文長度為零的*音訊*區塊訊息。 語音服務會將此訊息解譯為傳入音訊串流結束。

### <a name="message-telemetry"></a>訊息 `telemetry`

用戶端應用程式*必須*傳送語音服務回合的相關遙測，藉此確認每個回合結束。 回合結束確認可讓語音服務確保用戶端正確地收到完成要求及其回應所需的所有訊息。 回合結束確認也可讓語音服務確認用戶端應用程式如預期般運作。 如果您對疑難排解啟用語音的應用程式需要協助，這項資訊至關重要。

用戶端必須在收到 `turn.end` 訊息之後立即傳送 `telemetry` 訊息，藉此確認回合結束。 用戶端應該嘗試儘速確認 `turn.end`。 如果用戶端應用程式無法確認回合結束，語音服務可能會終止連線，並發生錯誤。 用戶端必須針對以 *X-RequestId* 值識別的每個要求和回應，僅傳送一個 `telemetry` 訊息。

| 欄位 | 說明 |
| ------------- | ---------------- |
| WebSocket 訊息編碼 | 文字 |
| Path | `telemetry` |
| X-Timestamp | ISO 8601 格式的用戶端 UTC 時鐘時間戳記 |
| Content-Type | `application/json` |
| body | JSON 結構，其中包含有關回合的用戶端資訊 |

`telemetry` 訊息主體的結構描述是在 [[遙測結構描述]](#telemetry-schema) 區段中定義的。

#### <a name="telemetry-for-interrupted-connections"></a>已中斷連線的遙測

如果網路連線在某個回合期間因為任何原因而失敗，且用戶端*沒有*從服務收到 `turn.end` 訊息，則用戶端會傳送 `telemetry` 訊息。 此訊息會在用戶端下次建立與服務的連線時，描述失敗的要求。 用戶端不需要立即嘗試連線，就可以傳送 `telemetry` 訊息。 此訊息可能會在用戶端上緩衝處理，並透過之後使用者要求的連線傳送。 失敗要求的 `telemetry` 訊息*必須*使用失敗要求中的 *X-RequestId* 值。 一旦建立連線之後，此訊息可能就會傳送至服務，而不必等待傳送或接收其他訊息。

## <a name="service-originated-messages"></a>服務產生的訊息

本節描述源自於語音服務，而且將傳送至用戶端的訊息。 語音服務會維護用戶端功能的登錄，並產生每個用戶端所需的訊息，因此並非所有用戶端都會收到此處所述的所有訊息。 為求簡單明瞭，*Path* 標頭的值會參考訊息。 例如，我們會參考具有 *Path* 值 `speech.hypothesis` 的 WebSocket 簡訊，作為 speech.hypothesis 訊息。

### <a name="message-speechstartdetected"></a>訊息 `speech.startDetected`

`speech.startDetected` 訊息會指出語音服務在音訊串流中偵測到語音。

| 欄位 | 說明 |
| ------------- | ---------------- |
| WebSocket 訊息編碼 | 文字 |
| Path | `speech.startDetected` |
| Content-Type | application/json; charset=utf-8 |
| body | JSON 結構，包含偵測到語音起點時，有關條件的資訊。 在音訊串流中偵測到語音時，此結構中的 [位移] 欄位會指定相對於串流起點的位移 (以 100 奈秒為單位)。 |

#### <a name="sample-message"></a>範例訊息

```HTML
Path: speech.startDetected
Content-Type: application/json; charset=utf-8
X-RequestId: 123e4567e89b12d3a456426655440000

{
  "Offset": 100000
}
```

### <a name="message-speechhypothesis"></a>訊息 `speech.hypothesis`

在語音辨識期間，語音服務會定期產生服務所能辨識單字的相關假設。 語音服務大約每隔 300 毫秒，就會將這些假設傳送至用戶端。 `speech.hypothesis` *僅*適用於提升使用者語音體驗。 您不得在這些訊息中採用任何內容相依性或文字精確度。

 `speech.hypothesis` 訊息適用的這些用戶端具備特定的文字轉譯功能，而且想要將進行中辨識接近即時的意見反應提供給正在說話的人員。

| 欄位 | 說明 |
| ------------- | ---------------- |
| WebSocket 訊息編碼 | 文字 |
| Path | `speech.hypothesis` |
| X-RequestId | 「無破折號」格式的 UUID |
| Content-Type | application/json |
| body | 語音假設 JSON 結構 |

#### <a name="sample-message"></a>範例訊息

```HTML
Path: speech.hypothesis
Content-Type: application/json; charset=utf-8
X-RequestId: 123e4567e89b12d3a456426655440000

{
  "Text": "this is a speech hypothesis",
  "Offset": 0,
  "Duration": 23600000
}
```

[位移] 元素會指定辨識片語後，相對於音訊串流起點的位移 (以 100 奈秒為單位)。

[持續時間] 元素會指定此語音片語的持續時間 (以 100 奈秒為單位)。

用戶端不得針對包含在語音假設中的頻率、時間或文字，或者任何兩個語音假設中文字的一致性，進行任何假設。 假設只是服務中，轉譯程序的快照集。 它們不代表穩定的轉譯累積。 例如，第一個語音假設可能包含 "fine fun" 字眼，而第二個假設可能包含 "find funny" 字眼。 語音服務不會對語音假設中的文字執行任何後續處理 (例如，大小寫、標點符號)。

### <a name="message-speechphrase"></a>訊息 `speech.phrase`

當語音服務判定它有足夠的資訊來產生將不會變更的辨識結果時，服務會產生 `speech.phrase` 訊息。 語音服務會在偵測到使用者已經完成一個句子或片語之後，產生這些結果。

| 欄位 | 說明 |
| ------------- | ---------------- |
| WebSocket 訊息編碼 | 文字 |
| Path | `speech.phrase` |
| Content-Type | application/json |
| body | 語音片語 JSON 結構 |

語音片語 JSON 結構描述包含下列欄位：`RecognitionStatus`、`DisplayText`、`Offset` 和 `Duration`。 如需有關這些欄位的詳細資訊，請參閱[轉譯回應](../concepts.md#transcription-responses)。

#### <a name="sample-message"></a>範例訊息

```HTML
Path: speech.phrase
Content-Type: application/json; charset=utf-8
X-RequestId: 123e4567e89b12d3a456426655440000

{
  "RecognitionStatus": "Success",
  "DisplayText": "Remind me to buy 5 pencils.",
  "Offset": 0,
  "Duration": 12300000
}
```

### <a name="message-speechenddetected"></a>訊息 `speech.endDetected`

`speech.endDetected` 訊息會指定用戶端應用程式應該停止將音訊串流處理至服務。

| 欄位 | 說明 |
| ------------- | ---------------- |
| WebSocket 訊息編碼 | 文字 |
| Path | `speech.endDetected` |
| body | 偵測到語音結尾時，包含位移的 JSON 結構。 位移會以 100 奈秒為單位的位移表示 (從用於辨識的音訊起點)。 |
| Content-Type | application/json; charset=utf-8 |

#### <a name="sample-message"></a>範例訊息

```HTML
Path: speech.endDetected
Content-Type: application/json; charset=utf-8
X-RequestId: 123e4567e89b12d3a456426655440000

{
  "Offset": 0
}
```

[位移] 元素會指定辨識片語後，相對於音訊串流起點的位移 (以 100 奈秒為單位)。

### <a name="message-turnstart"></a>訊息 `turn.start`

從服務的觀點來看，`turn.start` 會對回合的起點發出訊號。 `turn.start` 訊息永遠是您針對任何要求而收到的第一個回應訊息。 如果您沒有收到 `turn.start` 訊息，請假設服務連線的狀態無效。

| 欄位 | 說明 |
| ------------- | ---------------- |
| WebSocket 訊息編碼 | 文字 |
| Path | `turn.start` |
| Content-Type | application/json; charset=utf-8 |
| body | JSON 結構 |

#### <a name="sample-message"></a>範例訊息

```HTML
Path: turn.start
Content-Type: application/json; charset=utf-8
X-RequestId: 123e4567e89b12d3a456426655440000

{
  "context": {
    "serviceTag": "7B33613B91714B32817815DC89633AFA"
  }
}
```

`turn.start` 訊息的主體是 JSON 結構，其中包含回合開始的內容。 [內容] 元素包含 *serviceTag* 屬性。 此屬性會指定服務與回合相關聯的標記值。 如果您對疑難排解應用程式中的失敗需要協助，Microsoft 可以使用這個值。

### <a name="message-turnend"></a>訊息 `turn.end`

從服務的觀點來看，`turn.end` 會對回合的結尾發出訊號。 `turn.end` 訊息永遠是您針對任何要求而收到的最後一個回應訊息。 用戶端可以使用此訊息的回條作為清理活動和轉換為閒置狀態的訊號。 如果您沒有收到 `turn.end` 訊息，請假設服務連線的狀態無效。 在這些情況下，請關閉與服務的現有連線，然後重新連線。

| 欄位 | 說明 |
| ------------- | ---------------- |
| WebSocket 訊息編碼 | 文字 |
| Path | `turn.end` |
| body | None |

#### <a name="sample-message"></a>範例訊息

```HTML
Path: turn.end
X-RequestId: 123e4567e89b12d3a456426655440000
```

## <a name="telemetry-schema"></a>遙測結構描述

*遙測*訊息的主體是 JSON 結構，其中包含有關回合或嘗試連線的用戶端資訊。 此結構是由記錄用戶端事件發生時間的用戶端時間戳記所組成。 每次戳記都必須採用 ISO 8601 格式，如「X-Timestamp 標頭」一節中所述。 未在 JSON 結構中指定所有必要欄位，或未使用正確時間戳記格式的*遙測*訊息可能會使服務終止與用戶端的連線。 用戶端*必須*為所有必要欄位提供有效的值。 用戶端*應該*為選用欄位提供值 (如果適用的話)。 本節範例中顯示的值僅供參考。

遙測結構描述分為下列部分：收到的訊息時間戳記和計量。 下列各節中會指定每個部分的格式和使用方式。

### <a name="received-message-time-stamps"></a>收到的訊息時間戳記

用戶端必須包含成功連線至服務之後收到之所有訊息的收到時間值。 這些值必須記錄用戶端從網路*收到*每個訊息的時間。 此值不應記錄其他任何時間。 例如，用戶端不應記錄其對訊息*採取動作*的時間。 *name:value* 組的陣列中會指定收到的訊息時間戳記。 該組的名稱會指定訊息的 *Path* 值。 該組的值會指定用戶端收到訊息的時間。 或者，如果收到多個指定之名稱的訊息，該組的值則是表示收到這些訊息之時間的時間戳記陣列。

```JSON
  "ReceivedMessages": [
    { "speech.hypothesis": [ "2016-08-16T15:03:48.172Z", "2016-08-16T15:03:48.331Z", "2016-08-16T15:03:48.881Z" ] },
    { "speech.endDetected": "2016-08-16T15:03:49.721Z" },
    { "speech.phrase": "2016-08-16T15:03:50.001Z" },
    { "turn.end": "2016-08-16T15:03:51.021Z" }
  ]
```

用戶端*必須*在 JSON 主體中包含這些訊息的時間戳記，藉此確認收到服務所傳送的所有訊息。 如果用戶端無法確認收到訊息，服務可能會終止連線。

### <a name="metrics"></a>度量

用戶端必須包含要求存留期期間所發生事件的相關資訊。 支援下列計量：`Connection`、`Microphone` 和 `ListeningTrigger`。

### <a name="metric-connection"></a>計量 `Connection`

`Connection` 計量會指定用戶端嘗試連線的詳細資料。 計量必須包含開始和完成 WebSocket 連線時的時間戳記。 *只有連線的第一個回合*需要有 `Connection` 計量。 後續回合不需要包含這項資訊。 如果用戶端在建立連線之前進行多個連線嘗試，則應該包含*所有*連線嘗試的相關資訊。 如需詳細資訊，請參閱[連線失敗遙測](#connection-failure-telemetry)。

| 欄位 | 說明 | 使用量 |
| ----- | ----------- | ----- |
| 名稱 | `Connection` | 必要 |
| id | 此連線要求的 *X-ConnectionId* 標頭中所使用的連線識別碼值 | 必要 |
| Start | 用戶端傳送連線要求的時間 | 必要 |
| End | 用戶端收到成功建立連線通知的時間，如果發生錯誤，則是收到拒絕連線或連線失敗通知的時間 | 必要 |
| Error | 發生之錯誤的描述 (如果有的話)。 如果連線成功，用戶端應該省略此欄位。 此欄位的長度上限為 50 個字元。 | 如果發生錯誤，則必填，否則省略 |

錯誤描述應該最多 50 個字元，理想上應該是下表中所列的其中一個值。 如果錯誤情況與其中一個值不符，用戶端可以透過 [CamelCasing](https://en.wikipedia.org/wiki/Camel_case) (無空格) 的方式，使用簡潔的錯誤情況描述。 若要傳送*遙測*訊息，需要連線至服務，因此只能在*遙測*訊息中報告暫時性的錯誤情況。 使用戶端*永久*無法與服務連線的錯誤狀況會防止用戶端將任何訊息傳送至服務，包括*遙測*訊息。

| Error | 使用量 |
| ----- | ----- |
| DNSfailure | 用戶端因為網路堆疊中的 DNS 錯誤而無法連線至服務。 |
| NoNetwork | 用戶端嘗試連線，但網路堆疊回報沒有實體網路可用。 |
| NoAuthorization | 嘗試取得連線的授權權杖時，用戶端連接失敗。 |
| NoResources | 用戶端在嘗試建立連線時，用盡特定本機資源 (例如記憶體)。 |
| 禁止 | 用戶端無法連線至服務，因為服務在 WebSocket 升級要求上傳回 HTTP `403 Forbidden` 狀態碼。 |
| 未經授權 | 用戶端無法連線至服務，因為服務在 WebSocket 升級要求上傳回 HTTP `401 Unauthorized` 狀態碼。 |
| BadRequest | 用戶端無法連線至服務，因為服務在 WebSocket 升級要求上傳回 HTTP `400 Bad Request` 狀態碼。 |
| ServerUnavailable | 用戶端無法連線至服務，因為服務在 WebSocket 升級要求上傳回 HTTP `503 Server Unavailable` 狀態碼。 |
| ServerError | 用戶端無法連線至服務，因為服務在 WebSocket 升級要求上傳回 `HTTP 500` 內部錯誤狀態碼。 |
| 逾時 | 用戶端的連線要求逾時，未收到來自服務的回應。 [結束] 欄位包含用戶端逾時並停止等待連線的時間。 |
| ClientError | 用戶端因為特定的用戶端內部錯誤而終止連線。 | 

### <a name="metric-microphone"></a>計量 `Microphone`

所有語音回合都需要有 `Microphone` 計量。 此計量會測量用戶端正在對語音要求使用音訊輸入的持續時間。

使用下列範例作為在用戶端應用程式中記錄 `Microphone` 計量之 [開始] 時間值的指導方針：

* 用戶端應用程式要求使用者必須按下實體按鈕，才能啟動麥克風。 按下按鈕之後，用戶端應用程式會從麥克風讀取輸入，並將其傳送至語音服務。 `Microphone` 計量的 [開始] 值會記錄按下按鈕之後，麥克風初始化並準備好提供輸入的時間。 `Microphone` 計量的 [結束] 值會記錄用戶端應用程式從服務收到 `speech.endDetected` 訊息之後，停止將音訊串流處理至服務的時間。

* 用戶端應用程式會使用「一律」接聽的關鍵字偵查器。 只有在關鍵字偵查器偵測到說出觸發片語之後，用戶端應用程式才會從麥克風收集輸入，並將其傳送至語音服務。 `Microphone` 計量的 [開始] 值會記錄關鍵字偵察器通知用戶端開始使用麥克風中輸入的時間。 `Microphone` 計量的 [結束] 值會記錄用戶端應用程式從服務收到 `speech.endDetected` 訊息之後，停止將音訊串流處理至服務的時間。

* 用戶端應用程式可以存取持續的音訊串流，並在*語音偵測模組*中，針對該音訊串流執行無回應/語音偵測。 `Microphone` 計量的 [開始] 值會記錄*語音偵測模組*通知用戶端開始使用音訊串流中輸入的時間。 `Microphone` 計量的 [結束] 值會記錄用戶端應用程式從服務收到 `speech.endDetected` 訊息之後，停止將音訊串流處理至服務的時間。

* 用戶端應用程式正在處理多回合要求的第二個回合，而且正在透過服務回應訊息通知開啟麥克風，以收集第二個回合的輸入。 `Microphone` 計量的 [開始] 值會記錄用戶端應用程式啟用麥克風，並開始使用該音訊來源中輸入的時間。 `Microphone` 計量的 [結束] 值會記錄用戶端應用程式從服務收到 `speech.endDetected` 訊息之後，停止將音訊串流處理至服務的時間。

`Microphone` 計量的 [結束] 時間值會記錄用戶端應用程式停止串流處理音訊輸入的時間。 在大部分的情況下，此事件會在用戶端收到來自服務的 `speech.endDetected` 訊息後立即發生。 用戶端應用程式可以確認 `Microphone` 計量的 [結束] 時間值發生在收到 `speech.endDetected` 訊息的時間值之後，藉此驗證它們正確地符合通訊協定。 此外，通常在一個回合結束與另一個回合開始之間會有延遲，因此用戶端可以確認任何後續回合之 `Microphone` 計量的 [開始] 時間會正確地記錄用戶端*開始*使用麥克風將音訊輸入串流處理至服務，藉此驗證是否符合通訊協定。

| 欄位 | 說明 | 使用量 |
| ----- | ----------- | ----- |
| 名稱 | 麥克風 | 必要 |
| Start | 用戶端開始使用來自麥克風或其他音訊串流的音訊輸入，或從關鍵字偵察器收到觸發程序的時間 | 必要 |
| End | 用戶端停止使用麥克風或音訊串流的時間 | 必要 |
| Error | 發生之錯誤的描述 (如果有的話)。 如果麥克風操作成功，用戶端應該省略此欄位。 此欄位的長度上限為 50 個字元。 | 如果發生錯誤，則必填，否則省略 |

### <a name="metric-listeningtrigger"></a>計量 `ListeningTrigger`
`ListeningTrigger` 計量會測量使用者執行起始語音輸入動作的時間。 `ListeningTrigger` 計量是選用的，建議可以提供此計量的用戶端選用。

使用下列範例作為在用戶端應用程式中記錄 `ListeningTrigger` 計量之 [開始] 和 [結束] 時間值的指導方針。

* 用戶端應用程式要求使用者必須按下實體按鈕，才能啟動麥克風。 此計量的 [開始] 值會記錄按鈕按下的時間。 [結束] 值會記錄按鈕按下完成的時間。

* 用戶端應用程式會使用「一律」接聽的關鍵字偵查器。 在關鍵字偵查器偵測到說出觸發片語之後，用戶端應用程式會從麥克風讀取輸入，並將其傳送至語音服務。 此計量的 [開始] 值會記錄關鍵字偵察器收到音訊之後，再偵測為觸發片語的時間。 [結束] 值會記錄使用者說出觸發片語最後一個字的時間。

* 用戶端應用程式可以存取持續的音訊串流，並在*語音偵測模組*中，針對該音訊串流執行無回應/語音偵測。 此計量的 [開始] 值會記錄*語音偵測模組*收到音訊之後，再偵測為語音的時間。 [結束] 值會記錄*語音偵測模組*偵測到語音的時間。

* 用戶端應用程式正在處理多回合要求的第二個回合，而且正在透過服務回應訊息通知開啟麥克風，以收集第二個回合的輸入。 用戶端應用程式*不應*包含此回合的 `ListeningTrigger` 計量。

| 欄位 | 說明 | 使用量 |
| ----- | ----------- | ----- |
| 名稱 | ListeningTrigger | 選用 |
| Start | 啟動用戶端接聽觸發程序的時間 | 必要 |
| End | 完成用戶端接聽觸發程序的時間 | 必要 |
| Error | 發生之錯誤的描述 (如果有的話)。 如果觸發操作成功，用戶端應該省略此欄位。 此欄位的長度上限為 50 個字元。 | 如果發生錯誤，則必填，否則省略 |

#### <a name="sample-message"></a>範例訊息

下列範例顯示同時包含 ReceivedMessages 和 Metrics 部分的遙測訊息：

```HTML
Path: telemetry
Content-Type: application/json; charset=utf-8
X-RequestId: 123e4567e89b12d3a456426655440000
X-Timestamp: 2016-08-16T15:03:54.183Z

{
  "ReceivedMessages": [
    { "speech.hypothesis": [ "2016-08-16T15:03:48.171Z", "2016-08-16T15:03:48.331Z", "2016-08-16T15:03:48.881Z" ] },
    { "speech.endDetected": "2016-08-16T15:03:49.721Z" },
    { "speech.phrase": "2016-08-16T15:03:50.001Z" },
    { "turn.end": "2016-08-16T15:03:51.021Z" }
  ],
  "Metrics": [
    {
      "Name": "Connection",
      "Id": "A140CAF92F71469FA41C72C7B5849253",
      "Start": "2016-08-16T15:03:47.921Z",
      "End": "2016-08-16T15:03:48.000Z",
    },
    {
      "Name": "ListeningTrigger",
      "Start": "2016-08-16T15:03:48.776Z",
      "End": "2016-08-16T15:03:48.777Z",
    },
    {
      "Name": "Microphone",
      "Start": "2016-08-16T15:03:47.921Z",
      "End": "2016-08-16T15:03:51.921Z",
    },
  ],
}
```

## <a name="error-handling"></a>錯誤處理

本節描述錯誤訊息的種類，以及您應用程式必須處理的情況。

### <a name="http-status-codes"></a>HTTP 狀態碼

WebSocket 升級要求期間，語音服務可能會傳回任何標準 HTTP 狀態碼，例如 `400 Bad Request` 等等。您的應用程式必須正確處理這些錯誤情況。

#### <a name="authorization-errors"></a>授權錯誤

如果在 WebSocket 升級期間提供的授權不正確，語音服務會傳回 HTTP `403 Forbidden` 狀態碼。 可以觸發此錯誤代碼的情況包括：

* 缺少 *Authorization* 標頭

* 授權權杖無效

* 授權權杖過期

`403 Forbidden` 錯誤訊息並不表示語音服務發生問題。 此錯誤訊息表示用戶端應用程式發生問題。

### <a name="protocol-violation-errors"></a>通訊協定違規錯誤

如果語音服務偵測到來自用戶端的任何通訊協定違規，服務會在傳回終止的*狀態碼*和*原因*之後，終止 WebSocket 連線。 用戶端應用程式可以使用這項資訊進行疑難排解並修正違規。

#### <a name="incorrect-message-format"></a>訊息格式不正確

如果用戶端將簡訊或二進位訊息傳送至未使用此規格中指定之正確格式編碼的服務，服務便會結束連線，並出現 *1007 承載資料無效*狀態碼。 

此服務會因為多種原因而傳回此狀態碼，如下列範例所示：

* 「訊息格式不正確。 二進位訊息的標頭大小前置詞無效。」 用戶端傳送標頭大小前置詞無效的二進位訊息。

* 「訊息格式不正確。 二進位訊息的標頭大小無效。」 用戶端傳送指定無效標頭大小的二進位訊息。

* 「訊息格式不正確。 二進位訊息標頭解碼為 UTF-8 失敗。」 用戶端傳送含有未使用 UTF-8 正確編碼之標頭的二進位訊息。

* 「訊息格式不正確。 簡訊不包含任何資料。」 用戶端傳送不包含任何內文資料的簡訊。

* 「訊息格式不正確。 簡訊解碼為 UTF-8 失敗。」 用戶端傳送未使用 UTF-8 正確編碼的簡訊。

* 「訊息格式不正確。 簡訊不包含任何標頭分隔符號。」 用戶端傳送不包含標頭分隔符號，或使用錯誤標頭分隔符號的簡訊。

#### <a name="missing-or-empty-headers"></a>遺漏或空白標頭

如果用戶端傳送沒有所需標頭 *X-RequestId* 或 *Path* 的訊息，服務就會結束連線，並出現 *1002 通訊協定錯誤*狀態碼。 訊息是「遺漏/空白標頭。 {標頭名稱}。」

#### <a name="requestid-values"></a>RequestId 值

如果用戶端傳送使用不正確格式指定 *X-RequestId* 標頭的訊息，服務就會結束連線，並傳回 *1002 通訊協定錯誤*狀態。 訊息為「要求無效。 未以無破折號的 UUID 格式指定 X-RequestId 標頭值。」

#### <a name="audio-encoding-errors"></a>音訊編碼錯誤

如果用戶端傳送起始回合的音訊區塊，且音訊格式或編碼不符合所需的規格，服務就會結束連線，並傳回 *1007 承載資料無效*狀態碼。 訊息會指出格式編碼錯誤來源。

#### <a name="requestid-reuse"></a>RequestId 重複使用

在一個回合完成之後，如果用戶端傳送重複使用來自該回合之要求識別碼的訊息，服務就會結束連線，並傳回 *1002 通訊協定錯誤*狀態碼。 訊息為「要求無效。 不允許重複使用要求識別碼。」

## <a name="connection-failure-telemetry"></a>連線失敗遙測

為確保最佳的使用者經驗，用戶端必須使用*遙測*訊息，通知語音服務連線內重要檢查點的時間戳記。 用戶端通知服務已嘗試連線但失敗同樣很重要。

對於每個失敗的連線嘗試，用戶端都會以唯一的 *X-RequestId* 標頭值，建立一個*遙測*訊息。 用戶端無法建立連線，因此可以省略 JSON 主體中的 *ReceivedMessages* 欄位。 只有 [計量] 欄位中的 `Connection` 項目包含在內。 此項目包含開始和結束時間戳記，以及遇到的錯誤情況。

### <a name="connection-retries-in-telemetry"></a>遙測中的連接重試

用戶端應該區分*重試*以及觸發連線嘗試之事件的*多個連線嘗試*。 以程式設計方式執行不含使用者輸入的連線嘗試稱為重試。 為回應使用者輸入而執行的多個連線嘗試稱為多個連線嘗試。 用戶端會為每個使用者觸發的連線嘗試，提供一個唯一的 *X-RequestId* 和 *遙測*訊息。 用戶端會針對程式設計的重試，重複使用 *X-RequestId*。 如果對單一連線嘗試進行多個重試，每個重試嘗試就會當作 `Connection` 項目，包含在*遙測*訊息中。

例如，假設使用者說出關鍵字觸發程序啟動連線，而且第一次連線嘗試因為發生 DNS 錯誤而失敗。 不過，用戶端以程式設計方式發出的另一次嘗試會成功。 用戶端在不需要其他使用者輸入的情況下重試連線，因此用戶端會使用單一*遙測*訊息搭配多個`Connection`項目描述連線。

另一個範例是，假設使用者說出關鍵字觸發程序啟動連線，而且此連線嘗試會在三次重試之後失敗。 接著，用戶端會放棄、停止嘗試連線到服務，然後通知使用者發生錯誤。 之後使用者會再次說出關鍵字觸發程序。 這次，假設用戶端連線至服務。 連線之後，用戶端會立即將*遙測*訊息傳送至服務，此服務中包含三個描述連線失敗的 `Connection` 項目。 收到 `turn.end` 訊息之後，用戶端會傳送另一個描述成功連線的*遙測*訊息。

## <a name="error-message-reference"></a>錯誤訊息參照

### <a name="http-status-codes"></a>HTTP 狀態碼

| HTTP 狀態碼 | 說明 | 疑難排解 |
| - | - | - |
| 400 不正確的要求 | 用戶端傳送不正確的 WebSocket 連線要求。 | 請確認您提供所有必要的參數和 HTTP 標頭，而且這些值正確無誤。 |
| 401 未經授權 | 用戶端不包含所需的授權資訊。 | 請確認您將在 WebSocket 連線中傳送 *Authorization* 標頭。 |
| 403 禁止 | 用戶端傳送授權資訊，但該資訊無效。 | 請確認您沒有在 *Authorization* 標頭中傳送過期或無效的值。 |
| 404 找不到 | 用戶端嘗試存取不支援的 URL 路徑。 | 請確認您將為 WebSocket 連線使用正確的 URL。 |
| 500 伺服器錯誤 | 服務發生內部錯誤，無法繼續滿足要求。 | 在大部分的情況下，此錯誤是暫時性的。 重試要求。 |
| 503 服務無法使用 | 服務無法處理要求。 | 在大部分的情況下，此錯誤是暫時性的。 重試要求。 |

### <a name="websocket-error-codes"></a>WebSocket 錯誤碼

| WebSocketsStatus 程式碼 | 說明 | 疑難排解 |
| - | - | - |
| 1000 正常終止 | 服務結束 WebSocket 連線，而沒有產生錯誤。 | 如果是非預期的 WebSocket 終止，請重新閱讀文件以確定您了解服務可以終止 WebSocket 連線的方式和時機。 |
| 1002 通訊協定錯誤 | 用戶端無法遵守通訊協定需求。 | 請確定您了解通訊協定文件，而且清楚了解需求。 閱讀先前有關錯誤原因的文件，以了解您是否違反通訊協定需求。 |
| 1007 承載資料無效 | 用戶端在通訊協定訊息中傳送無效的承載。 | 請檢查您傳送至服務的最後一個訊息中是否有錯誤。 閱讀先前有關承載錯誤的文件。 |
| 1011 伺服器錯誤 | 服務發生內部錯誤，無法繼續滿足要求。 | 在大部分的情況下，此錯誤是暫時性的。 重試要求。 |

## <a name="related-topics"></a>相關主題

請參閱 [JavaScript SDK](https://github.com/Azure-Samples/SpeechToText-WebSockets-Javascript)，也就是 WebSocket 型語音服務通訊協定的實作。
