---
title: 向 Bing 語音進行驗證 |Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 要求驗證以使用 Bing 語音 API
services: cognitive-services
author: zhouwangzw
manager: wolfma
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: zhouwang
ms.openlocfilehash: e3bfdcfaaf752321997636811b282ea920e6144a
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49344195"
---
# <a name="authenticate-to-the-speech-api"></a>向 Speech API 進行驗證

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

「Bing 語音」支援使用下列方式進行驗證：

- 訂用帳戶金鑰。
- 授權權杖。

## <a name="use-a-subscription-key"></a>使用訂用帳戶金鑰

若要使用「語音服務」，您必須先訂閱 Speech API，這是「認知服務」(先前稱為 Project Oxford) 的組件。 您可以從[認知服務訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)頁面，取得免費試用訂用帳戶金鑰。 選取 Speech API 之後，請選取 [取得 API 金鑰] 以取得金鑰。 這會傳回主要和次要金鑰。 兩個金鑰都繫結至同一個配額，因此你可以使用任一金鑰。

若要長期使用或增加配額，請註冊 [Azure 帳戶](https://azure.microsoft.com/free/)。

若要使用「語音 REST API」，您必須在要求標頭的 `Ocp-Apim-Subscription-Key` 欄位中傳遞訂用帳戶金鑰。

名稱| 格式| 說明
----|-------|------------
Ocp-Apim-Subscription-Key | ASCII | YOUR_SUBSCRIPTION_KEY

以下是一個要求標頭範例：

```HTTP
POST https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US&format=detailed HTTP/1.1
Accept: application/json;text/xml
Content-Type: audio/wav; codec=audio/pcm; samplerate=16000
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: speech.platform.bing.com
Transfer-Encoding: chunked
Expect: 100-continue
```

> [!IMPORTANT]
> 如果您在應用程式中使用[用戶端程式庫](../GetStarted/GetStartedClientLibraries.md)，請確認可以取得搭配訂用帳戶金鑰的授權權杖，如下一節所述。 用戶端程式庫會使用訂用帳戶金鑰來取得授權權杖，然後使用該權杖來進行驗證。

## <a name="use-an-authorization-token"></a>使用授權權杖

或者，您也可以使用授權權杖作為驗證/授權的證明，來進行驗證。 若要取得此權杖，您必須先從 Speech API 取得訂用帳戶金鑰，如[上一節](#use-a-subscription-key)所述。

### <a name="get-an-authorization-token"></a>取得授權權杖

在您擁有有效的訂用帳戶金鑰之後，請向「認知服務」的權杖服務傳送一個 POST 要求。 在回應中，您會收到「JSON Web 權杖」(JWT) 形式的授權權杖。

> [!NOTE]
> 此權杖的有效期為 10 分鐘。 若要更新權杖，請參閱下一節。

權杖服務 URI 位於此處：

```
https://api.cognitive.microsoft.com/sts/v1.0/issueToken
```

以下程式碼範例示範如何取得存取權杖。 請以您自己的訂用帳戶金鑰取代 `YOUR_SUBSCRIPTION_KEY`：

# <a name="powershelltabpowershell"></a>[PowerShell](#tab/Powershell)

```Powershell
$FetchTokenHeader = @{
  'Content-type'='application/x-www-form-urlencoded';
  'Content-Length'= '0';
  'Ocp-Apim-Subscription-Key' = 'YOUR_SUBSCRIPTION_KEY'
}

$OAuthToken = Invoke-RestMethod -Method POST -Uri https://api.cognitive.microsoft.com/sts/v1.0/issueToken -Headers $FetchTokenHeader

# show the token received
$OAuthToken

```

# <a name="curltabcurl"></a>[curl](#tab/curl)

此範例會在 Linux 上搭配 bash 使用 curl。 如果您的平台上沒有 curl，則可能需要安裝 curl。 此範例也適用於 Windows 上的 Cygwin、Git Bash、zsh 及其他殼層。

```
curl -v -X POST "https://api.cognitive.microsoft.com/sts/v1.0/issueToken" -H "Content-type: application/x-www-form-urlencoded" -H "Content-Length: 0" -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY"
```

# <a name="ctabcsharp"></a>[C#](#tab/CSharp)

```cs
    /*
     * This class demonstrates how to get a valid access token.
     */
    public class Authentication
    {
        public static readonly string FetchTokenUri = "https://api.cognitive.microsoft.com/sts/v1.0";
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
                uriBuilder.Path += "/issueToken";

                var result = await client.PostAsync(uriBuilder.Uri.AbsoluteUri, null);
                Console.WriteLine("Token Uri: {0}", uriBuilder.Uri.AbsoluteUri);
                return await result.Content.ReadAsStringAsync();
            }
        }
    }
```

---

以下是一個簡單的 POST 要求︰

```HTTP
POST https://api.cognitive.microsoft.com/sts/v1.0/issueToken HTTP/1.1
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: api.cognitive.microsoft.com
Content-type: application/x-www-form-urlencoded
Content-Length: 0
Connection: Keep-Alive
```

如果您無法從權杖服務取得授權權杖，請檢查您的訂用帳戶金鑰是否仍然有效。 如果您使用免費試用金鑰，請前往[認知服務訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)頁面，按一下 [登入] 以使用您用來申請免費試用金鑰的帳戶進行登入，然後檢查訂用帳戶金鑰是否已到期或超出配額。

### <a name="use-an-authorization-token-in-a-request"></a>在要求中使用授權權杖

每當您呼叫 Speech API 時，都必須在 `Authorization` 標頭中傳遞授權權杖。 `Authorization`標頭必須包含 JWT 存取權杖。

以下範例示範如何在呼叫「語音 REST API」時使用授權權杖。

> [!NOTE]
> 請以您的預錄音訊檔路徑取代 `YOUR_AUDIO_FILE`。 以您在上一個步驟[取得授權權杖](#get-an-authorization-token)中取得的授權權杖取代 `YOUR_ACCESS_TOKEN`。

# <a name="powershelltabpowershell"></a>[PowerShell](#tab/Powershell)

```Powershell

$SpeechServiceURI =
'https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-us&format=detailed'

# $OAuthToken is the authrization token returned by the token service.
$RecoRequestHeader = @{
  'Authorization' = 'Bearer '+ $OAuthToken;
  'Transfer-Encoding' = 'chunked'
  'Content-type' = 'audio/wav; codec=audio/pcm; samplerate=16000'
}

# Read audio into byte array
$audioBytes = [System.IO.File]::ReadAllBytes("YOUR_AUDIO_FILE")

$RecoResponse = Invoke-RestMethod -Method POST -Uri $SpeechServiceURI -Headers $RecoRequestHeader -Body $audioBytes

# Show the result
$RecoResponse

```

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -v -X POST "https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-us&format=detailed" -H "Transfer-Encoding: chunked" -H "Authorization: Bearer YOUR_ACCESS_TOKEN" -H "Content-type: audio/wav; codec=audio/pcm; samplerate=16000" --data-binary @YOUR_AUDIO_FILE
```

# <a name="ctabcsharp"></a>[C#](#tab/CSharp)

```cs
HttpWebRequest request = null;
request = (HttpWebRequest)HttpWebRequest.Create(requestUri);
request.SendChunked = true;
request.Accept = @"application/json;text/xml";
request.Method = "POST";
request.ProtocolVersion = HttpVersion.Version11;
request.Host = @"speech.platform.bing.com";
request.ContentType = @"audio/wav; codec=audio/pcm; samplerate=16000";
request.Headers["Authorization"] = "Bearer " + token;

// Send an audio file by 1024 byte chunks
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

---

### <a name="renew-an-authorization-token"></a>更新授權權杖

授權權杖會在特定的期間 (目前為 10 分鐘) 後到期。 您必須在授權權杖到期之前更新權杖。

以下程式碼是一個以 C# 撰寫的範例實作，說明如何更新授權權杖：

```cs
    /*
     * This class demonstrates how to get a valid O-auth token.
     */
    public class Authentication
    {
        public static readonly string FetchTokenUri = "https://api.cognitive.microsoft.com/sts/v1.0";
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
                uriBuilder.Path += "/issueToken";

                var result = await client.PostAsync(uriBuilder.Uri.AbsoluteUri, null);
                Console.WriteLine("Token Uri: {0}", uriBuilder.Uri.AbsoluteUri);
                return await result.Content.ReadAsStringAsync();
            }
        }
    }
```
