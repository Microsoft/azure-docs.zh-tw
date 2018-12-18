---
title: 針對 Bing 語音進行疑難排解 |Microsoft Docs
titlesuffix: Azure Cognitive Services
description: 如何解決使用「Bing 語音」時的問題。
services: cognitive-services
author: zhouwangzw
manager: wolfma
ms.service: cognitive-services
ms.component: bing-speech
ms.topic: article
ms.date: 09/18/2018
ms.author: zhouwang
ms.openlocfilehash: 80e39e845ab631ad5cc3ae8af9e2e42f09b521bf
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49342185"
---
# <a name="troubleshooting-bing-speech"></a>針對 Bing 語音進行疑難排解

[!INCLUDE [Deprecation note](../../../includes/cognitive-services-bing-speech-api-deprecation-note.md)]

## <a name="error-http-403-forbidden"></a>錯誤 `HTTP 403 Forbidden`

使用語音辨識 API 時，傳回 `HTTP 403 Forbidden` 錯誤。

### <a name="cause"></a>原因

此錯誤通常是由驗證問題引起的。 連線要求如果沒有有效的 `Ocp-Apim-Subscription-Key` 或 `Authorization` 標頭，就會被服務以 `HTTP 403 Forbidden` 回應拒絕。

如果您使用訂用帳戶金鑰來進行驗證，則原因可能是

- 遺漏訂用帳戶金鑰或金鑰無效
- 超出訂用帳戶金鑰的使用量配額
- 呼叫 REST API 時，要求標頭中未設定 `Ocp-Apim-Subscription-Key` 欄位

如果您使用授權權杖來進行驗證，則下列原因可能造成錯誤。

- 使用 REST 時，要求中遺漏 `Authorization` 標頭
- Authorization 標頭中指定的授權權杖無效
- 授權權杖已過期。 存取權杖的有效期為 10 分鐘

如需有關驗證的詳細資訊，請參閱[驗證](How-to/how-to-authentication.md)頁面。

### <a name="troubleshooting-steps"></a>疑難排解步驟

#### <a name="verify-that-your-subscription-key-is-valid"></a>確認您的訂用帳戶金鑰有效

您可以執行下列命令來進行驗證。 請注意，請以您自己的訂用帳戶金鑰取代 *YOUR_SUBSCRIPTION_KEY*。 如果您的訂用帳戶金鑰有效，就會在回應中收到「JSON Web 權杖」(JWT) 形式的授權權杖。 否則，您會在回應中收到錯誤。

> [!NOTE]
> 以您自己的訂用帳戶金鑰取代 `YOUR_SUBSCRIPTION_KEY`。

# <a name="powershelltabpowershell"></a>[Powershell](#tab/Powershell)

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

此範例會在 Linux 上搭配 bash 使用 curl。 如果您的平台上沒有 curl，則可能需要安裝 curl。 此範例應該也適用於 Windows 上的 Cygwin、Git Bash、zsh 及其他殼層。

```
curl -v -X POST "https://api.cognitive.microsoft.com/sts/v1.0/issueToken" -H "Content-type: application/x-www-form-urlencoded" -H "Content-Length: 0" -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY"
```
---

請確定您在應用程式或 REST 要求中使用的訂用帳戶金鑰與上面使用的相同。

#### <a name="verify-the-authorization-token"></a>確認授權權杖

只有當您使用授權權杖來進行驗證時，才需要執行此步驟。 請執行下列命令來確認授權權杖仍然有效。 此命令會向服務發出 POST 要求，並預期從服務收到回應訊息。 如果您仍然收到 HTTP `403 Forbidden` 錯誤，請仔細檢查存取權杖是否設定正確且未過期。

> [!IMPORTANT]
> 權杖的有效期為 10 分鐘。
> [!NOTE]
> 請以您預錄的音訊檔路徑取代 `YOUR_AUDIO_FILE`，以上一個步驟中傳回的授權權杖取代 `YOUR_ACCESS_TOKEN`。

# <a name="powershelltabpowershell"></a>[Powershell](#tab/Powershell)

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

---

## <a name="error-http-400-bad-request"></a>錯誤 `HTTP 400 Bad Request`

此原因通常是要求本文包含無效的音訊資料。 目前我們僅支援 WAV 檔案。

## <a name="error-http-408-request-timeout"></a>錯誤 `HTTP 408 Request Timeout`

此錯誤的最可能原因是未將任何音訊資料傳送給服務，因此服務在逾時後傳回此錯誤。 就 REST API 而言，音訊資料應該放在要求本文中。

## <a name="the-recognitionstatus-in-the-response-is-initialsilencetimeout"></a>回應中的 `RecognitionStatus` 是 `InitialSilenceTimeout`

音訊資料通常是造成此問題的原因。 例如，

- 音訊的開頭有很長的無聲時間。 服務會在幾秒後停止辨識，並傳回 `InitialSilenceTimeout`。
- 音訊使用不支援的轉碼器格式，導致音訊資料被視為無聲。
