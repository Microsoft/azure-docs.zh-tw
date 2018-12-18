---
title: 翻譯工具語音 API 參考
titleSuffix: Azure Cognitive Services
description: 翻譯工具語音 API 參考文件。
services: cognitive-services
author: Jann-Skotdal
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-speech
ms.topic: reference
ms.date: 05/18/2018
ms.author: v-jansko
ROBOTS: NOINDEX
ms.openlocfilehash: 46aeab52014a28d1a962195de802d0e000b62509
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978704"
---
# <a name="translator-speech-api"></a>Translator Speech API

此服務提供串流處理 API，將交談語音從某種語言謄寫為另一種語言的文字。 此 API 也會整合文字轉換語音功能，以說出翻譯的文字。 翻譯工具語音 API 會啟用 Skype Translator 中即時翻譯交談這類情節。

使用翻譯工具語音 API，用戶端應用程式會將語音音訊串流處理至服務，並接收以文字為基礎之結果的資料流，包括來源語言中的已辨識文字以及它在目標語言中的翻譯。 產生文字結果時，是透過將採用深度類神經網路技術的「自動語音辨識」(ASR) 套用至傳入的音訊資料流來產生。 原始 ASR 輸出使用稱為 TrueText 的新技術進一步改善，以更密切地反映使用者意圖。 例如，TrueText 會移除言語上的不流利 (嗯和咳嗽)，並還原適當的標點符號和大小寫。 也會包括遮罩或排除粗話的能力。 辨識和翻譯引擎皆特別經過訓練來處理交談語音。 「語音翻譯」服務使用無聲偵測來判斷語句的結束。 在語音活動暫停之後，服務會串流來傳回已完成語句的最終結果。 此服務也可以送回部分結果，以提供進行中語句的中繼辨識和翻譯。 在最終結果中，此服務提供以目標語言從口語文字合成語音 (文字轉換語音) 的能力。 會以用戶端指定的格式建立文字轉換語音音訊。 可用的格式包括 WAV 和 MP3。

翻譯工具語音 API 利用 WebSocket 通訊協定來提供用戶端與伺服器之間的全雙工通訊通道。 應用程式將需要這些步驟，才能使用服務：

## <a name="1-getting-started"></a>1.開始使用
若要存取翻譯工具語音 API，您需要[註冊 Microsoft Azure](translator-speech-how-to-signup.md)。

## <a name="2-authentication"></a>2.驗證

使用訂用帳戶金鑰驗證。 翻譯工具語音 API 支援兩種驗證模式：

* **使用存取權杖：** 在應用程式中，從權杖服務取得存取權杖。 使用翻譯工具語音 API 訂用帳戶金鑰，從 Azure 認知服務驗證服務取得存取權杖。 存取權杖的有效時間為 10 分鐘。 每隔 10 分鐘取得新的存取權杖一次，並在這 10 分鐘內針對重複的要求持續使用相同的存取權杖。

* **直接使用訂用帳戶金鑰：** 在應用程式中，將訂用帳戶金鑰傳遞為 `Ocp-Apim-Subscription-Key` 標頭中的值。

將您的訂用帳戶金鑰和存取權杖作為應該隱藏不予檢視的祕密。

## <a name="3-query-languages"></a>3.查詢語言
**查詢目前這組受支援語言的 Languages 資源。** [languages 資源](languages-reference.md)會公開可用於語音辨識、文字翻譯和文字轉換語音的這組語言和語音。 每種語言或語音都會獲指定一個識別碼，而翻譯工具語音 API 使用它來識別相同的語言或語音。

## <a name="4-stream-audio"></a>4.資料流音訊
**開啟連線，然後開始將音訊串流處理至服務。** 服務 URL 是 `wss://dev.microsofttranslator.com/speech/translate`。 在下面，透過 `/speech/translate` 作業描述服務所需要的屬性和音訊格式。 使用其中一個參數傳遞上述步驟 2 中的存取權杖。

## <a name="5-process-the-results"></a>5.處理結果
**處理從服務串流處理回的結果。** 下面的 `/speech/translate` 作業文件描述局部結果、最終結果和文字轉換語音音訊區段的格式。

您可以從 [Microsoft Translator Github 網站](https://github.com/MicrosoftTranslator)取得示範翻譯工具語音 API 使用方式的程式碼範例。

## <a name="implementation-notes"></a>實作附註

GET /speech/translate 建立語音翻譯的工作階段

### <a name="connecting"></a>連接
連線至服務之前，請先檢閱本節稍後提供的參數清單。 範例要求如下：

`GET wss://dev.microsofttranslator.com/speech/translate?from=en-US&to=it-IT&features=texttospeech&voice=it-IT-Elsa&api-version=1.0`
`Ocp-Apim-Subscription-Key: {subscription key}`
`X-ClientTraceId: {GUID}`

此要求指定口語英文將會串流處理至服務，並翻譯為義大利文。 每個最終辨識結果將會產生女性聲音名為 Elsa 的文字轉換語音音訊回應。 請注意，要求會在 `Ocp-Apim-Subscription-Key header` 中包括認證。 在 `X-ClientTraceId` 標頭中設定全域唯一識別碼，要求也會遵循最佳做法。 用戶端應用程式應該記錄追蹤識別碼，使其可用來針對所發生的問題進行疑難排解。

### <a name="sending-audio"></a>傳送音訊
建立連線之後，用戶端會開始將音訊串流處理至服務。 用戶端會以區塊形式傳送音訊。 每個區塊都會使用二進位類型的 Websocket 訊息傳輸。

音訊輸入為 Waveform 音訊檔案格式 (WAVE，或因其副檔名通稱為 WAV)。 用戶端應用程式應該串流處理以 16 kHz 取樣的單一通道帶正負號 16 位元 PCM 音訊。 用戶端所串流處理的第一組位元組將包括 WAV 標頭。 以 16 kHz 取樣之單一通道帶正負號 16 位元 PCM 資料流的 44-位元組標頭如下：

|Offset|值|
|:---|:---|
|0 - 3|"RIFF"|
|4 - 7|0|
|8 - 11|"WAVE"|
|12 - 15|"fmt"|
|16 - 19|16|
|20 - 21|1|
|22 - 23|1|
|24 - 27|16000|
|28 - 31|32000|
|32 - 33|2|
|34 - 35|16|
|36 - 39|"data"|
|40 - 43|0|

請注意，檔案大小總計 (位元組 4-7) 和「資料」大小 (位元組 40-43) 設定為零。 這適用於不一定事先知道總大小的串流處理情節。

傳送 WAV (RIFF) 標頭之後，用戶端會傳送數個區塊的音訊資料。 用戶端通常會串流處理代表固定期間的固定大小區塊 (例如一次串流處理 100 毫秒的音訊)。

### <a name="final-result"></a>最終結果
在語句結尾會產生最終語音辨識結果。 結果會使用文字類型的 WebSocket 訊息從服務傳輸至用戶端。 訊息內容是具有下列屬性之物件的 JSON 序列化：

* `type`：識別結果類型的字串常數。 值是最終結果的最終值。
* `id`：指派給辨識結果的字串識別碼。
* `recognition`：來源語言中的已辨識文字。 若為 false 辨識，則文字可能是空字串。
* `translation`：以目標語言翻譯的已辨識文字。
* `audioTimeOffset`：辨識開始的時間位移 (刻度) (1 刻度 = 100 奈秒)。 此位移相對於串流處理的開始。
* `audioTimeSize`：辨識的期間 (刻度) (100 奈秒)。
* `audioStreamPosition`：辨識開始的位元組位移。 此位移相對於資料流的開始。
* `audioSizeBytes`：辨識的大小 (位元組)。

請注意，結果中預設不包含在音訊資料流中定位辨識。 用戶端必須選取 `TimingInfo` 功能 (請參閱 `features` 參數)。

範例最終結果如下所示：

```
{
  type: "final"
  id: "23",
  recognition: "what was said", 
  translation: "translation of what was said",
  audioStreamPosition: 319680,
  audioSizeBytes: 35840,
  audioTimeOffset: 2731600000,
  audioTimeSize: 21900000
}
```

### <a name="partial-result"></a>局部結果
局部或中繼語音辨識結果預設不會串流處理至用戶端。 用戶端可以使用 features 查詢參數來要求它們。

局部結果會使用文字類型的 WebSocket 訊息從服務傳輸至用戶端。 訊息內容是具有下列屬性之物件的 JSON 序列化：

* `type`：識別結果類型的字串常數。 值是局部結果的一部分。
* `id`：指派給辨識結果的字串識別碼。
* `recognition`：來源語言中的已辨識文字。
* `translation`：以目標語言翻譯的已辨識文字。
* `audioTimeOffset`：辨識開始的時間位移 (刻度) (1 刻度 = 100 奈秒)。 此位移相對於串流處理的開始。
* `audioTimeSize`：辨識的期間 (刻度) (100 奈秒)。
* `audioStreamPosition`：辨識開始的位元組位移。 此位移相對於資料流的開始。
* `audioSizeBytes`：辨識的大小 (位元組)。

請注意，結果中預設不包含在音訊資料流中定位辨識。 用戶端必須選取 TimingInfo 功能 (請參閱 features 參數)。

範例最終結果如下所示：

```
{
  type: "partial"
  id: "23.2",
  recognition: "what was", 
  translation: "translation of what was",
  audioStreamPosition: 319680,
  audioSizeBytes: 25840,
  audioTimeOffset: 2731600000,
  audioTimeSize: 11900000
}
```

### <a name="text-to-speech"></a>文字轉換語音
啟用文字轉換語音功能時 (請參閱下面的 `features` 參數)，最終結果後面接著口語翻譯文字的音訊。 音訊資料會分成區塊，並以二進位類型的一連串 Websocket 訊息從服務傳送至用戶端。 用戶端可以檢查每個訊息的 FIN 位元，以偵測資料流結尾。 最後一個二進位訊息會將其 FIN 位元設定為 1，表示資料流結尾。 資料流的格式取決於 `format` 參數的值。

### <a name="closing-the-connection"></a>關閉連線
用戶端應用程式完成串流處理音訊並收到最後一個最終結果時，應該會起始 WebSocket 關閉交握來關閉連線。 有些情況會造成伺服器終止連線。 用戶端可能會收到下列 WebSocket「已關閉」程式碼：

* `1003 - Invalid Message Type`：伺服器正在終止連線，因為它無法接受所接收的資料類型。 這通常發生於內送音訊的開頭不是正確標頭時。
* `1000 - Normal closure`：已在完成要求之後關閉連線。 伺服器將關閉連線：很長一段時間未收到來自用戶端的音訊時；很長一段時間串流處理無回應資料時；工作階段達到允許的最長期間 (大約 90 分鐘)。
* `1001 - Endpoint Unavailable`：表示伺服器會變成無法使用。 用戶端應用程式可能會在重試次數限制內嘗試重新連線。
* `1011 - Internal Server Error`：因為伺服器發生錯誤，所以伺服器將關閉連線。

### <a name="parameters"></a>參數

|參數|值|說明|參數類型|資料類型|
|:---|:---|:---|:---|:---|
|api-version|1.0|用戶端要求的 API 版本。 允許的值包括：`1.0`。|query   |字串|
|from|(空白)   |指定內送語音的語言。 此值是 Languages API 回應中來自 `speech` 範圍 的其中一個語言識別碼。|query|字串|
|to|(空白)|指定用來翻譯已轉譯文字的目標語言。 此值是 Languages API 回應中來自 `text` 範圍 的其中一個語言識別碼。|query|字串|
|特性|(空白)   |用戶端選取的逗號分隔功能集。 可用功能包括：<ul><li>`TextToSpeech`：指定服務必須傳回最終翻譯句子的翻譯音訊。</li><li>`Partial`：指定服務必須傳回中繼辨識結果，而音訊會串流處理至服務。</li><li>`TimingInfo`：指定服務必須傳回與每個辨識建立關聯的計時資訊。</li></ul>例如，用戶端指定 `features=partial,texttospeech` 收到局部結果和文字轉換語音，但沒有計時資訊。 請注意，最終結果一律會串流處理至用戶端。|query|字串|
|voice|(空白)|識別要用於翻譯文字之文字轉換語音轉譯的聲音。 此值是 Languages API 回應中來自 tts 範圍 的其中一個語音識別碼。 若未指定語音，則系統會在啟用文字轉換語音功能時自動選擇語音。|query|字串|
|format|(空白)|指定服務所傳回的文字轉換語音音訊資料流格式。 可用選項包括：<ul><li>`audio/wav`：Waveform 音訊資料流。 用戶端應該會使用 WAV 標頭，適當地解譯音訊格式。 文字轉換語音的 WAV 音訊是取樣率為 24kHz 或 16 kHz 的 16 位元單一通道 PCM。</li><li>`audio/mp3`：MP3 音訊資料流。</li></ul>預設值為 `audio/wav`。|query|字串|
|ProfanityAction    |(空白)    |指定服務應該如何處理語音中所辨識的粗話。 有效的動作包括︰<ul><li>`NoAction`：粗話會保持原狀。</li><li>`Marked`：粗話會取代為標記。 請參閱 `ProfanityMarker` 參數。</li><li>`Deleted`：刪除粗話。 例如，若將 `"jackass"` 這個字視為粗話，則片語 `"He is a jackass."` 會變成 `"He is a .".`</li></ul>預設值是 Marked。|query|字串|
|ProfanityMarker|(空白)    |指定 `ProfanityAction` 設定為 `Marked` 時如何處理偵測到的粗話。 有效的選項包括：<ul><li>`Asterisk`：粗話會取代為字串 `***`。 例如，若將 `"jackass"` 這個字視為粗話，則片語 `"He is a jackass."` 會變成 `"He is a ***.".`</li><li>`Tag`：粗話會括上 profanity XML 標籤。 例如，若將 `"jackass"` 這個字視為粗話，則片語 `"He is a jackass."` 會變成 `"He is a <profanity>jackass</profanity>."`。</li></ul>預設值為 `Asterisk`。|query|字串|
|Authorization|(空白)  |指定用戶端持有人權杖的值。 使用 `Bearer` 字首，後面接著驗證權杖服務所傳回 `access_token` 值的值。|頁首   |字串|
|Ocp-Apim-Subscription-Key|(空白)|若未指定 `Authorization` 標頭，則為必要項目。|頁首|字串|
|access_token|(空白)   |傳遞有效 OAuth 存取權杖的替代方式。 持有人權杖通常會與 `Authorization` 標頭一起提供。 部分 WebSocket 程式庫不允許用戶端程式碼設定標頭。 在這種情況下，用戶端可以使用 `access_token` 查詢參數來傳遞有效的權杖。 使用存取權杖驗證時，若未設定 `Authorization` 標頭，則必須設定 `access_token`。 若同時設定標頭和查詢參數，則會忽略查詢參數。 用戶端應該只使用一種方法來傳遞權杖。|query|字串|
|subscription-key|(空白)   |傳遞訂用帳戶金鑰的替代方式。 部分 WebSocket 程式庫不允許用戶端程式碼設定標頭。 在這種情況下，用戶端可以使用 `subscription-key` 查詢參數來傳遞有效的訂用帳戶金鑰。 使用訂用帳戶金鑰驗證時，若未設定 `Ocp-Apim-Subscription-Key` 標頭，則必須設定訂用帳戶金鑰。 若同時設定標頭和查詢參數，則會忽略查詢參數。 用戶端應該只使用一種方法來傳遞 `subscription key`。|query|字串|
|X-ClientTraceId    |(空白)    |用來追蹤要求之用戶端產生的 GUID。 如需針對問題進行適當地疑難排解，用戶端應該隨著每項要求提供新的值並予以記錄。<br/>此值可以使用 `X-ClientTraceId` 查詢參數傳遞，而不是使用標頭。 若同時設定標頭和查詢參數，則會忽略查詢參數。|頁首|字串|
|X-CorrelationId|(空白)    |用來相互關聯交談中多個通道之用戶端產生的識別碼。 可以建立多個語音翻譯工作階段，以啟用使用者之間的交談。 在這類情況下，所有語音翻譯工作階段都會使用相同的相互關聯識別碼，將通道繫結在一起。 這有助於追蹤和診斷。 識別碼應該符合：`^[a-zA-Z0-9-_.]{1,64}$`<br/>此值可以使用 `X-CorrelationId` 查詢參數傳遞，而不是使用標頭。 若同時設定標頭和查詢參數，則會忽略查詢參數。|頁首|字串|
|X-ClientVersion|(空白)    |識別用戶端應用程式的版本。 範例："2.1.0.123"。<br/>此值可以使用 `X-ClientVersion` 查詢參數傳遞，而不是使用標頭。 若同時設定標頭和查詢參數，則會忽略查詢參數。|頁首|字串|
|X-OsPlatform|(空白)   |識別用戶端應用程式在其上執行的作業系統名稱和版本。 範例："Android 5.0"、"iOs 8.1.3"、"Windows 8.1"。<br/>此值可以使用 `X-OsPlatform` 查詢參數傳遞，而不是使用標頭。 若同時設定標頭和查詢參數，則會忽略查詢參數。|頁首|字串|

### <a name="response-messages"></a>回應訊息

|HTTP 狀態碼|原因|回應模型|headers|
|:--|:--|:--|:--|
|101    |WebSocket 升級。|模型範例值 <br/> 物件 {}|X-RequestId<br/>值，識別進行疑難排解的要求。<br/>字串|
|400    |不正確的要求。 請檢查輸入參數，以確保參數有效。 回應物件包括錯誤的更詳細描述。|||
|401    |未經授權。 確定已設定認證、認證有效，而且您的 Azure Data Market 訂用帳戶具有可用餘額。|||
|500    |發生錯誤。 若錯誤仍然存在，請使用用戶端追蹤識別碼 (X-ClientTraceId) 或要求識別碼 (X-RequestId) 予以回報。|||
|503    |暫時無法使用伺服器。 請重試要求。 若錯誤仍然存在，請使用用戶端追蹤識別碼 (X-ClientTraceId) 或要求識別碼 (X-RequestId) 予以回報。|||

    


    





    
    




    




    




    

            




        









