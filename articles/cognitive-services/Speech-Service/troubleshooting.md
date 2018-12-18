---
title: 認知服務語音 SDK 疑難排解
description: 疑難排解認知服務語音 SDK
titleSuffix: Microsoft Cognitive Services
services: cognitive-services
author: wolfma61
ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 05/07/2018
ms.author: wolfma
ms.openlocfilehash: ff8aba562cfd2d6d54c708ee7fdc4c6ca7185f29
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39284117"
---
# <a name="troubleshooting-speech-services-sdk"></a>疑難排解語音服務 SDK

本文提供的資訊可協助您解決在使用語音 SDK 時可能會遇到的問題。

## <a name="error-websocket-upgrade-failed-with-an-authentication-error-403"></a>錯誤 `WebSocket Upgrade failed with an authentication error (403).`

您的區域或服務可能有錯誤的端點。 請仔細檢查 URI 以確保其正確無誤。 另請參閱下一節，因為也可能是訂用帳戶金鑰或授權權杖的問題。

## <a name="error-http-403-forbidden-or-error-http-401-unauthorized"></a>錯誤 `HTTP 403 Forbidden` 或錯誤 `HTTP 401 Unauthorized`

此錯誤通常是由驗證問題引起的。 連線不是有效之 `Ocp-Apim-Subscription-Key` 或 `Authorization` 標頭的要求會遭到拒絕，狀態為 401 或 403。

* 如果您使用訂用帳戶金鑰進行驗證，原因可能是：

    - 訂用帳戶金鑰缺少或無效
    - 您已超過訂用帳戶的使用量配額

* 如果您使用授權權杖進行驗證，原因可能是：

    - 授權權杖無效
    - 授權權杖已過期

### <a name="validate-your-subscription-key"></a>驗證您的訂用帳戶金鑰

您可以藉由執行下列其中一項命令來進行驗證，以確保您具備有效的訂用帳戶金鑰。

> [!NOTE]
> 將 `YOUR_SUBSCRIPTION_KEY` 和 `YOUR_REGION` 分別取代為您自己的訂用帳戶金鑰和相關區域。

* PowerShell

    ```Powershell
    $FetchTokenHeader = @{
      'Content-type'='application/x-www-form-urlencoded'
      'Content-Length'= '0'
      'Ocp-Apim-Subscription-Key' = 'YOUR_SUBSCRIPTION_KEY'
    }
    $OAuthToken = Invoke-RestMethod -Method POST -Uri https://YOUR_REGION.api.cognitive.microsoft.com/sts/v1.0/issueToken -Headers $FetchTokenHeader
    $OAuthToken
    ```

* cURL

    ```
    curl -v -X POST "https://YOUR_REGION.api.cognitive.microsoft.com/sts/v1.0/issueToken" -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY" -H "Content-type: application/x-www-form-urlencoded" -H "Content-Length: 0"
    ```

### <a name="validate-an-authorization-token"></a>驗證授權權杖

如果您是使用授權權杖進行驗證，請執行下列其中一項命令，驗證授權權杖是否仍然有效。 權杖的有效時間為 10 分鐘。

> [!NOTE]
> 將 `YOUR_AUDIO_FILE` 取代為您預錄之音訊檔的路徑，將 `YOUR_ACCESS_TOKEN` 取代為上一步傳回的驗證權杖，並將 `YOUR_REGION` 取代為正確的區域。

* PowerShell

    ```Powershell
    $SpeechServiceURI =
    'https://YOUR_REGION.stt.speech.microsoft.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US'
    
    # $OAuthToken is the authorization token returned by the token service.
    $RecoRequestHeader = @{
      'Authorization' = 'Bearer '+ $OAuthToken
      'Transfer-Encoding' = 'chunked'
      'Content-type' = 'audio/wav; codec=audio/pcm; samplerate=16000'
    }
    
    # Read audio into byte array
    $audioBytes = [System.IO.File]::ReadAllBytes("YOUR_AUDIO_FILE")
    
    $RecoResponse = Invoke-RestMethod -Method POST -Uri $SpeechServiceURI -Headers $RecoRequestHeader -Body $audioBytes
    
    # Show the result
    $RecoResponse
    ```

* cURL

    ```
    curl -v -X POST "https://YOUR_REGION.stt.speech.microsoft.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US" -H "Authorization: Bearer YOUR_ACCESS_TOKEN" -H "Transfer-Encoding: chunked" -H "Content-type: audio/wav; codec=audio/pcm; samplerate=16000" --data-binary @YOUR_AUDIO_FILE
    ```

---

## <a name="error-http-400-bad-request"></a>錯誤 `HTTP 400 Bad Request`

此錯誤通常發生在要求本文包含無效的音訊資料時。 僅支援 `WAV` 格式。 此外，請檢查要求的標頭，以確保指定適當的 `Content-Type` 和 `Content-Length`。

## <a name="error-http-408-request-timeout"></a>錯誤 `HTTP 408 Request Timeout`

該錯誤很可能是因為沒有任何音訊資料傳送至服務。 此錯誤也可能是由網路問題造成的。

## <a name="the-recognitionstatus-in-the-response-is-initialsilencetimeout"></a>回應中的 `RecognitionStatus` 是 `InitialSilenceTimeout`

音訊資料通常是造成此問題的原因。 例如︰

* 音訊開頭有一段長時間的靜音。 服務將於幾秒後停止辨識，並傳回 `InitialSilenceTimeout`。
* 音訊使用不支援的轉碼器格式，其造成音訊視為無回應的音訊資料。

## <a name="next-steps"></a>後續步驟

* [版本資訊](releasenotes.md)

