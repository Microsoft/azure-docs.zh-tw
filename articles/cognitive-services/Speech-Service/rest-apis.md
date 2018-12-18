---
title: 語音服務 REST API
description: 語音服務 REST API 的參考。
services: cognitive-services
author: v-jerkin
ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 05/09/2018
ms.author: v-jerkin
ms.openlocfilehash: 6758cd658daf75beeea93bf9c719508cd271c8be
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47032422"
---
# <a name="speech-service-rest-apis"></a>語音服務 REST API

Azure 認知服務統一語音服務的 REST API 類似於 [Bing 語音 API](https://docs.microsoft.com/azure/cognitive-services/Speech) 所提供的 API。 而端點不同於 Bing 語音服務所使用的端點。 可以使用區域性端點，但您必須使用對應至您所用端點的訂用帳戶金鑰。

## <a name="speech-to-text"></a>語音轉換文字

下表顯示語音轉換文字 REST API 的端點。 請使用與訂用帳戶區域相符的端點。

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-speech-to-text.md)]

> [!NOTE]
> 如果您自訂原音模型或語言模型或是發音，請改用自訂端點。

此 API 只支援簡短語句。 要求可能包含多達 10 秒的音訊，整體最多可長達 14 秒。 REST API 只會傳回最終結果，而不是部分或中間的結果。 語音服務也有可以轉譯較長音訊的[批次轉譯](batch-transcription.md) API。

### <a name="query-parameters"></a>查詢參數

REST 要求的查詢字串中可能包含下列參數。

|參數名稱|必要/選用|意義|
|-|-|-|
|`language`|必要|要辨識的語言識別碼。 請參閱[支援的語言](supported-languages.md#speech-to-text)。|
|`format`|選用<br>預設值：`simple`|結果格式：`simple` 或 `detailed`。 簡單的結果包含 `RecognitionStatus``DisplayText`、`Offset` 和持續時間。 詳細的結果包含多個具有信賴值的候選項目和四個不同的表示法。|
|`profanity`|選用<br>預設值：`masked`|如何處理辨識結果中的不雅內容。 可能的處理方式為 `masked` (以星號取代不雅內容)、`removed` (移除所有不雅內容)，或 `raw` (包括不雅內容)。

### <a name="request-headers"></a>要求標頭

以下是在 HTTP 要求標頭中傳送的欄位。

|頁首|意義|
|------|-------|
|`Ocp-Apim-Subscription-Key`|您的語音服務訂用帳戶金鑰。 必須提供此標頭或 `Authorization`。|
|`Authorization`|前面加入 `Bearer` 這個字的授權權杖。 必須提供此標頭或 `Ocp-Apim-Subscription-Key`。 請參閱[驗證](#authentication)。|
|`Content-type`|描述音訊資料的格式和轉碼器。 此值目前必須為 `audio/wav; codec=audio/pcm; samplerate=16000`。|
|`Transfer-Encoding`|選用。 若已提供，則必須是 `chunked` 才允許以多個小型區塊 (而不是單一檔案) 傳送音訊資料。|
|`Expect`|如果使用區塊傳輸，請傳送 `Expect: 100-continue`。 語音服務會確認初始要求並等候其他資料。|
|`Accept`|選用。 若已提供，則必須包含 `application/json`，因為語音服務會提供 JSON 格式的結果。 (如果您未指定預設值，則有些 Web 要求架構會提供不相容的預設值，因此一律包含 `Accept` 是很好的做法。)|

### <a name="audio-format"></a>音訊格式

音訊會在 HTTP `PUT` 要求的主體中傳送。 它應該是 16 位元 WAV 格式，採用 16 KHz 的 PCM 單一通道 (mono)。

### <a name="chunked-transfer"></a>區塊傳輸

區塊傳輸 (`Transfer-Encoding: chunked`) 可協助減少辨識延遲，因為它可讓語音服務在音訊檔案進行傳輸時開始處理。 REST API 不會提供部分或中間的結果。 此選項僅僅為了改善回應能力。

下列程式碼說明如何以區塊傳送音訊。 `request` 是連線到適當 REST 端點的 HTTPWebRequest 物件。 `audioFile` 是音訊檔案在磁碟上的路徑。

```csharp
using (fs = new FileStream(audioFile, FileMode.Open, FileAccess.Read))
{

    /*
    * Open a request stream and write 1024 byte chunks in the stream one at a time.
    */
    byte[] buffer = null;
    int bytesRead = 0;
    using (Stream requestStream = request.GetRequestStream())
    {
        /*
        * Read 1024 raw bytes from the input audio file.
        */
        buffer = new Byte[checked((uint)Math.Min(1024, (int)fs.Length))];
        while ((bytesRead = fs.Read(buffer, 0, buffer.Length)) != 0)
        {
            requestStream.Write(buffer, 0, bytesRead);
        }

        // Flush
        requestStream.Flush();
    }
}
```

### <a name="example-request"></a>範例要求

以下是典型的要求。

```HTTP
POST speech/recognition/conversation/cognitiveservices/v1?language=en-US&format=detailed HTTP/1.1
Accept: application/json;text/xml
Content-Type: audio/wav; codec=audio/pcm; samplerate=16000
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: westus.stt.speech.microsoft.com
Transfer-Encoding: chunked
Expect: 100-continue
```

### <a name="http-status"></a>HTTP 狀態

回應的 HTTP 狀態表示成功或常見的錯誤狀況。

HTTP 代碼|意義|可能的原因
-|-|-|
100|Continue|已接受初始要求。 繼續傳送其餘的資料。 (搭配區塊傳輸使用。)
200|OK|要求成功；回應主體是 JSON 物件。
400|不正確的要求|未提供語言代碼，或不是支援的語言；無效的音訊檔案。
401|未經授權|訂用帳戶金鑰或授權權杖在指定的區域中無效，或是無效的端點。
403|禁止|遺漏訂用帳戶金鑰或授權權杖。

### <a name="json-response"></a>JSON 回應

結果會以 JSON 格式傳回。 `simple` 格式只包含下列最上層欄位。

|欄位名稱|內容|
|-|-|
|`RecognitionStatus`|狀態，例如 `Success` 代表辨識成功。 請參閱下一個表格。|
|`DisplayText`|在大寫字、標點符號、反向文字正規化 (將語音文字轉換為較短的形式，例如 200 表示 "two hundred" 或將 "Dr.Smith" 表示 "doctor smith")，以及不雅內容遮罩之後辨識的文字。 只會在成功時呈現。|
|`Offset`|辨識的語音在音訊資料流中開始的時間 (以 100 奈秒為單位)。|
|`Duration`|辨識的語音在音訊資料流中的持續時間 (以 100 奈秒為單位)。|

`RecognitionStatus` 欄位可能包含下列值。

|狀態值|說明
|-|-|
| `Success` | 辨識成功則會出現 DisplayText 欄位。 |
| `NoMatch` | 音訊串流中偵測到語音，但目標語言中沒有符合的字組。 通常表示辨識語言與使用者的口語語言不同。 |
| `InitialSilenceTimeout` | 音訊串流的開頭沒有任何聲音，而且等候語音的服務已逾時。 |
| `BabbleTimeout` | 音訊串流的開頭只有雜音，而且等候語音的服務已逾時。 |
| `Error` | 辨識服務發生內部錯誤，無法繼續。 可能的話，再試一次。 |

> [!NOTE]
> 如果音訊只包含不雅內容，而且 `profanity` 查詢參數設為 `remove`，則服務不會傳回語音結果。 


`detailed` 格式包含與 `simple` 格式相同的欄位，以及 `NBest` 欄位。 `NBest` 欄位是相同語音的替代解譯清單 (從最有可能的解譯排到最不可能的解譯)。 第一個項目與主要辨識結果相同。 每個項目都包含下列欄位：

|欄位名稱|內容|
|-|-|
|`Confidence`|項目的信賴分數從 0.0 (不信賴) 到 1.0 (完全信賴)
|`Lexical`|已辨識文字的語彙形式：已辨識的實際文字。
|`ITN`|已辨識文字的反向文字正規化 (「標準」) 形式，包含電話號碼、數字、縮寫 ("doctor smith" 縮短為 "dr smith")，以及其他已套件的轉換。
|`MaskedITN`| 如果要求，已套用不雅內容遮罩的 ITN 形式。
|`Display`| 已辨識文字的顯示形式，已新增標點符號和大寫。 與最上層結果中的 `DisplayText` 相同。

### <a name="sample-responses"></a>回應範例

以下是 `simple` 辨識的典型回應。

```json
{
  "RecognitionStatus": "Success",
  "DisplayText": "Remind me to buy 5 pencils.",
  "Offset": "1236645672289",
  "Duration": "1236645672289"
}
```

以下是 `detailed` 辨識的典型回應。

```json
{
  "RecognitionStatus": "Success",
  "Offset": "1236645672289",
  "Duration": "1236645672289",
  "NBest": [
      {
        "Confidence" : "0.87",
        "Lexical" : "remind me to buy five pencils",
        "ITN" : "remind me to buy 5 pencils",
        "MaskedITN" : "remind me to buy 5 pencils",
        "Display" : "Remind me to buy 5 pencils.",
      },
      {
        "Confidence" : "0.54",
        "Lexical" : "rewind me to buy five pencils",
        "ITN" : "rewind me to buy 5 pencils",
        "MaskedITN" : "rewind me to buy 5 pencils",
        "Display" : "Rewind me to buy 5 pencils.",
      }
  ]
}
```

## <a name="text-to-speech"></a>文字轉語音

以下是語音服務之文字轉換語音 API 的 REST 端點。 請使用與訂用帳戶區域相符的端點。

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-text-to-speech.md)]

> [!NOTE]
> 如果您已建立自訂語音字型，請改為使用相關聯的自訂端點。

除了 Bing 語音所支援的 16-Khz 輸出，語音服務可支援 24-KHz 音訊輸出。 有四種 24-KHz 輸出格式可使用於 `X-Microsoft-OutputFormat` HTTP 標頭中，因此有兩種 24-KHz 語音 `Jessa24kRUS` 和 `Guy24kRUS`。

地區設定 | 語言   | 性別 | 服務名稱對應
-------|------------|--------|------------
zh-TW  | 美式英文 | 女性 | "Microsoft Server Speech Text to Speech Voice (en-US, Jessa24kRUS)" 
zh-TW  | 美式英文 | 男性   | "Microsoft Server Speech Text to Speech Voice (en-US, Guy24kRUS)"

在[支援的語言](supported-languages.md#text-to-speech)中可取得可用語音的完整清單。

### <a name="request-headers"></a>要求標頭

以下是在 HTTP 要求標頭中傳送的欄位。

|頁首|意義|
|------|-------|
|`Authorization`|前面加入 `Bearer` 這個字的授權權杖。 必要。 請參閱[驗證](#authentication)。|
|`Content-Type`|輸入內容類型：`application/ssml+xml`。|
|`X-Microsoft-OutputFormat`|輸出音訊格式。 請參閱下一個表格。|
|`User-Agent`|應用程式名稱。 必要；必須包含少於 255 個字元。|

可用的音訊輸出格式 (`X-Microsoft-OutputFormat`) 包含位元速率和編碼方式。

|||
|-|-|
`raw-16khz-16bit-mono-pcm`         | `raw-8khz-8bit-mono-mulaw`
`riff-8khz-8bit-mono-mulaw`     | `riff-16khz-16bit-mono-pcm`
`audio-16khz-128kbitrate-mono-mp3` | `audio-16khz-64kbitrate-mono-mp3`
`audio-16khz-32kbitrate-mono-mp3`  | `raw-24khz-16bit-mono-pcm`
`riff-24khz-16bit-mono-pcm`        | `audio-24khz-160kbitrate-mono-mp3`
`audio-24khz-96kbitrate-mono-mp3`  | `audio-24khz-48kbitrate-mono-mp3`

> [!NOTE]
> 如果您選取的語音和輸出格式具有不同的位元速率，則會視需要重新進行音訊取樣。 不過，24KHz 語音不支援 `audio-16khz-16kbps-mono-siren` 和 `riff-16khz-16kbps-mono-siren` 輸出格式。 

### <a name="request-body"></a>Request body

要轉換成語音的文字會以純文字 (ASCII 或 UTF-8) 或[語音合成標記語言](speech-synthesis-markup.md) (SSML) 格式 (UTF-8)，當作 HTTP `POST` 要求的主體傳送。 純文字要求會使用服務的預設語音和語言。 傳送 SSML 以使用不同的語音。

### <a name="sample-request"></a>範例要求

下列 HTTP 要求會使用 SSML 主體來選擇語音。 主體的長度不得超過 1,000 個字元。

```xml
POST /cognitiveservices/v1 HTTP/1.1

X-Microsoft-OutputFormat: raw-16khz-16bit-mono-pcm
Content-Type: application/ssml+xml
Host: westus.tts.speech.microsoft.com
Content-Length: 225
Authorization: Bearer [Base64 access_token]

<speak version='1.0' xml:lang='en-US'><voice xml:lang='en-US' xml:gender='Female' 
    name='Microsoft Server Speech Text to Speech Voice (en-US, ZiraRUS)'>
        Microsoft Speech Service Text-to-Speech API
</voice></speak>
```

### <a name="http-response"></a>HTTP 回應

回應的 HTTP 狀態表示成功或常見的錯誤狀況。

HTTP 代碼|意義|可能的原因
-|-|-|
200|OK|要求成功；回應主體是音訊檔案。
400 |不正確的要求 |必要的參數遺失、為空白或 Null。 或者，傳遞至必要或選用參數的值無效。 常見的問題是標頭太長。
401|未經授權 |要求未經授權。 請檢查以確定您的訂用帳戶金鑰或權杖有效，並且位於正確的區域。
413|要求實體太大|SSML 輸入的長度大於 1024 個字元。
|502|錯誤的閘道    | 網路或伺服器端問題。 也可能表示標頭無效。

如果 HTTP 狀態為 `200 OK`，則回應主體會包含所要求格式的音訊檔案。 此檔案可以在傳輸或儲存至緩衝區或檔案時播放，以供稍後播放或其他用途。

## <a name="authentication"></a>驗證

將要求傳送至語音服務的 REST API 時，需要訂用帳戶金鑰或存取權杖。 一般而言，直接傳送訂用帳戶金鑰最容易。 語音服務接著會為您取得存取權杖。 若要將回應時間降到最低，您可以改為使用存取權杖。

若要取得權杖，向區域性語音服務 `issueToken` 端點呈現您的訂用帳戶金鑰，如下表所示。 請使用與訂用帳戶區域相符的端點。

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-token-service.md)]

每一個存取權杖的有效時間為 10 分鐘。 您可以隨時取得新權杖。 如有需要，可在每個語音 REST API 要求之前取得。 為了盡量降低網路流量和延遲，建議您使用相同的權杖九分鐘。

下列幾節將示範如何取得權杖，以及如何在要求中使用它。

### <a name="get-a-token-http"></a>取得權杖：HTTP

以下範例是用於取得權杖的範例 HTTP 要求。 以您的語音服務訂用帳戶金鑰取代 `YOUR_SUBSCRIPTION_KEY`。 如果您的訂用帳戶不在美國西部區域，請以您區域的主機名稱取代 `Host`。

```
POST /sts/v1.0/issueToken HTTP/1.1
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: westus.api.cognitive.microsoft.com
Content-type: application/x-www-form-urlencoded
Content-Length: 0
```

此要求的回應本文是使用 Java Web 權杖 (JWT) 格式的存取權杖。

### <a name="get-a-token-powershell"></a>取得權杖：PowerShell

下列 Windows PowerShell 指令碼說明如何取得存取權杖。 以您的語音服務訂用帳戶金鑰取代 `YOUR_SUBSCRIPTION_KEY`。 如果您的訂用帳戶不在美國西部區域，請據此變更指定 URI 的主機名稱。

```Powershell
$FetchTokenHeader = @{
  'Content-type'='application/x-www-form-urlencoded';
  'Content-Length'= '0';
  'Ocp-Apim-Subscription-Key' = 'YOUR_SUBSCRIPTION_KEY'
}

$OAuthToken = Invoke-RestMethod -Method POST -Uri https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken
 -Headers $FetchTokenHeader

# show the token received
$OAuthToken

```

### <a name="get-a-token-curl"></a>取得權杖：cURL

cURL 是 Linux (以及適用於 Linux 的 Windows 子系統) 中可用的命令列工具。 下列 cURL 命令說明如何取得存取權杖。 以您的語音服務訂用帳戶金鑰取代 `YOUR_SUBSCRIPTION_KEY`。 如果您的訂用帳戶不在美國西部區域，請據此變更指定 URI 的主機名稱。

> [!NOTE]
> 為了方便閱讀，命令會以多行顯示，但在殼層提示字元中輸入為一行。

```
curl -v -X POST 
 "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken" 
 -H "Content-type: application/x-www-form-urlencoded" 
 -H "Content-Length: 0" 
 -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY"
```

### <a name="get-a-token-c"></a>取得權杖：C#

下列 C# 類別說明如何取得存取權杖。 在您具現化類別時，請傳遞您的語音服務訂用帳戶金鑰。 如果您的訂用帳戶不在美國西部區域，請據此變更 `FetchTokenUri` 的主機名稱。

```cs
/*
    * This class demonstrates how to get a valid access token.
    */
public class Authentication
{
    public static readonly string FetchTokenUri =
        "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken";
    private string subscriptionKey;
    private string token;

    public Authentication(string subscriptionKey)
    {
        this.subscriptionKey = subscriptionKey;
        this.token = FetchTokenAsync(FetchTokenUri, subscriptionKey).Result;
    }

    public string GetAccessToken()
    {
        return this.token;
    }

    private async Task<string> FetchTokenAsync(string fetchUri, string subscriptionKey)
    {
        using (var client = new HttpClient())
        {
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
            UriBuilder uriBuilder = new UriBuilder(fetchUri);

            var result = await client.PostAsync(uriBuilder.Uri.AbsoluteUri, null);
            Console.WriteLine("Token Uri: {0}", uriBuilder.Uri.AbsoluteUri);
            return await result.Content.ReadAsStringAsync();
        }
    }
}
```

### <a name="use-a-token"></a>使用權杖

若要在 REST API 要求中使用權杖，請在 `Authorization` 標頭中的 `Bearer` 這個字之後提供它。 以下是包含權杖的範例文字轉換語音 REST 要求。 將 `YOUR_ACCESS_TOKEN` 替換為您實際的權杖。 在 `Host` 標頭中使用正確的主機名稱。

```xml
POST /cognitiveservices/v1 HTTP/1.1
Authorization: Bearer YOUR_ACCESS_TOKEN
Host: westus.tts.speech.microsoft.com
Content-type: application/ssml+xml
Content-Length: 199
Connection: Keep-Alive

<speak version='1.0' xmlns="http://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
<voice name='Microsoft Server Speech Text to Speech Voice (en-US, Jessa24kRUS)'>
    Hello, world!
</voice></speak>
```

### <a name="renew-authorization"></a>更新授權

授權權杖會在 10 分鐘後到期。 請在權杖到期之前，藉由取得新的權杖來更新您的授權。 例如，您可以在九分鐘之後取得新權杖。

下列 C# 程式碼是之前所提及類別的簡易取代項目。 `Authentication` 類別會使用計時器，每隔九分鐘自動取得新的存取權杖。 這個方法可確保您的程式執行時，一律可以使用有效的權杖。

> [!NOTE]
> 您可以儲存取得最後一個權杖時的時間戳記，而不是使用計時器。 而後您只能在時間戳記即將過期時要求新的時間戳記。 這種方法可避免不必要地要求新的權杖，並且可能比較適合於不頻繁提出語音要求的程式。

如往常一般，請確定 `FetchTokenUri` 值符合您的訂用帳戶區域。 在您具現化類別時，請傳遞您的訂用帳戶金鑰。

```cs
/*
    * This class demonstrates how to maintain a valid access token.
    */
public class Authentication
{
    public static readonly string FetchTokenUri = 
        "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken";
    private string subscriptionKey;
    private string token;
    private Timer accessTokenRenewer;

    //Access token expires every 10 minutes. Renew it every 9 minutes.
    private const int RefreshTokenDuration = 9;

    public Authentication(string subscriptionKey)
    {
        this.subscriptionKey = subscriptionKey;
        this.token = FetchToken(FetchTokenUri, subscriptionKey).Result;

        // renew the token on set duration.
        accessTokenRenewer = new Timer(new TimerCallback(OnTokenExpiredCallback),
                                        this,
                                        TimeSpan.FromMinutes(RefreshTokenDuration),
                                        TimeSpan.FromMilliseconds(-1));
    }

    public string GetAccessToken()
    {
        return this.token;
    }

    private void RenewAccessToken()
    {
        this.token = FetchToken(FetchTokenUri, this.subscriptionKey).Result;
        Console.WriteLine("Renewed token.");
    }

    private void OnTokenExpiredCallback(object stateInfo)
    {
        try
        {
            RenewAccessToken();
        }
        catch (Exception ex)
        {
            Console.WriteLine(string.Format("Failed renewing access token. Details: {0}", ex.Message));
        }
        finally
        {
            try
            {
                accessTokenRenewer.Change(TimeSpan.FromMinutes(RefreshTokenDuration), TimeSpan.FromMilliseconds(-1));
            }
            catch (Exception ex)
            {
                Console.WriteLine(string.Format("Failed to reschedule the timer to renew access token. Details: {0}", ex.Message));
            }
        }
    }

    private async Task<string> FetchToken(string fetchUri, string subscriptionKey)
    {
        using (var client = new HttpClient())
        {
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
            UriBuilder uriBuilder = new UriBuilder(fetchUri);

            var result = await client.PostAsync(uriBuilder.Uri.AbsoluteUri, null);
            Console.WriteLine("Token Uri: {0}", uriBuilder.Uri.AbsoluteUri);
            return await result.Content.ReadAsStringAsync();
        }
    }
}
```

## <a name="next-steps"></a>後續步驟

- [試用認知服務](https://azure.microsoft.com/try/cognitive-services/)
- [自訂原音模型](how-to-customize-acoustic-models.md)
- [自訂語言模型](how-to-customize-language-model.md)

