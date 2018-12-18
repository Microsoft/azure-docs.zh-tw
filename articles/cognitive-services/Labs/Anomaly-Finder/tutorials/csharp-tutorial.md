---
title: 教學課程：異常偵測 C#
titlesuffix: Azure Cognitive Services
description: 探索會使用異常偵測 API 的 C# 應用程式。 將原始資料點傳送到 API，並取得預期的值和異常點。
services: cognitive-services
author: chliang
manager: bix
ms.service: cognitive-services
ms.component: anomaly-detection
ms.topic: tutorial
ms.date: 05/01/2018
ms.author: chliang
ms.openlocfilehash: f99ce765c1d9417fd5ca88b49214eca8a3b0bf49
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48887644"
---
# <a name="tutorial-anomaly-detection-with-c-application"></a>教學課程：使用 C# 應用程式進行異常偵測

[!INCLUDE [PrivatePreviewNote](../../../../../includes/cognitive-services-anomaly-finder-private-preview-note.md)]

瀏覽使用異常偵測 API 來偵測輸入異常的基本 Windows 應用程式。 此範例會使用您的訂用帳戶金鑰來將時間序列資料提交到異常偵測 API，然後從該 API 針對每個資料點取得所有異常點和預期的值。

## <a name="prerequisites"></a>必要條件

### <a name="platform-requirements"></a>平台需求

此範例已使用 [Visual Studio 2017 Community Edition](https://www.visualstudio.com/products/visual-studio-community-vs) 針對.NET Framework 進行開發。 

### <a name="subscribe-to-anomaly-detection-and-get-a-subscription-key"></a>訂閱異常偵測並取得訂用帳戶金鑰 

[!INCLUDE [GetSubscriptionKey](../includes/get-subscription-key.md)]

## <a name="get-and-use-the-example"></a>取得和使用範例

您可以將異常偵測範例應用程式從 [Github](https://github.com/MicrosoftAnomalyDetection/csharp-sample.git) \(英文\) 複製到您的電腦。 
<a name="Step1"></a>
### <a name="install-the-example"></a>安裝範例

在您的 GitHub Desktop 上開啟 Sample\AnomalyDetectionSample.sln。

<a name="Step2"></a>
### <a name="build-the-example"></a>建置範例

按 Ctrl+Shift+B，或按一下功能區功能表上的 [建置]，然後選取 [建置方案]。

<a name="Step3"></a>
### <a name="run-the-example"></a>執行範例

1. 建置完成之後，按 **F5** 或按一下功能區功表上的 [啟動] 來執行範例。
2. 使用標示 "{your_subscription_key}" 的文字編輯方塊來找出異常偵測使用者介面視窗。
3. 以您自己的資料取代 request.json 檔案 (其中包含範例資料)，然後按一下 [傳送] 按鈕。 Microsoft 會收到您上傳的資料，並使用它們來偵測它們之間的任何異常點。 您載入的資料將不會保存於 Microsoft 的伺服器中。 若要再次偵測異常點，您需要再上傳資料一次。
4. 如果資料正確，您將會在 [回應] 欄位中找到異常偵測結果。 如果發生任何錯誤，錯誤資訊也將顯示於 [回應] 欄位中。

<a name="Review"></a>
### <a name="read-the-result"></a>讀取結果

[!INCLUDE [diagrams](../includes/diagrams.md)]

<a name="Review"></a>
### <a name="review-and-learn"></a>檢閱和學習

既然您已有一個能執行的應用程式，現在讓我們檢閱此範例應用程式如何與認知服務技術整合。 這個步驟將可讓您使用 Microsoft 異常偵測，更輕鬆地繼續建置此應用程式或開發您自己的應用程式。

此範例應用程式會使用異常偵測 Restful API 端點。

讓我們看一下來自 **AnomalyDetectionClient.cs** 的程式碼片段，以檢閱如何在範例應用程式中使用 Restful API。 此檔案包含指出 “KEY SAMPLE CODE STARTS HERE” 和 “KEY SAMPLE CODE ENDS HERE” 的程式碼註解，可幫助您找出以下重新產生的程式碼片段。

```csharp
            // ----------------------------------------------------------------------
            // KEY SAMPLE CODE STARTS HERE
            // Set http request header
            // ---------------------------------------------------------------------- 
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
            // ----------------------------------------------------------------------
            // KEY SAMPLE CODE ENDS HERE 
            // ----------------------------------------------------------------------

```
### <a name="request"></a>**要求**
以下程式碼片段示範如何使用 HttpClient，將訂用帳戶金鑰和資料點提交到異常偵測 API 的端點。

```csharp
    public async Task<string> Request(string baseAddress, string endpoint, string subscriptionKey, string requestData)
    {
        using (HttpClient client = new HttpClient { BaseAddress = new Uri(baseAddress) })
        {
            // ----------------------------------------------------------------------
            // KEY SAMPLE CODE STARTS HERE
            // Set http request header
            // ---------------------------------------------------------------------- 
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
            // ----------------------------------------------------------------------
            // KEY SAMPLE CODE ENDS HERE 
            // ----------------------------------------------------------------------

            // ----------------------------------------------------------------------
            // KEY SAMPLE CODE STARTS HERE
            // Build the request content
            // ---------------------------------------------------------------------- 
            var content = new StringContent(requestData, Encoding.UTF8, "application/json");
            // ----------------------------------------------------------------------
            // KEY SAMPLE CODE ENDS HERE 
            // ----------------------------------------------------------------------

            // ----------------------------------------------------------------------
            // KEY SAMPLE CODE STARTS HERE
            // Send the request content with POST method.
            // ---------------------------------------------------------------------- 
            var res = await client.PostAsync(endpoint, content);
            // ----------------------------------------------------------------------
            // KEY SAMPLE CODE ENDS HERE 
            // ----------------------------------------------------------------------
            if (res.IsSuccessStatusCode)
            {
                return await res.Content.ReadAsStringAsync();
            }
            else
            {
                return $"ErrorCode: {res.StatusCode}";
            }
        }
    }
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [REST API 參考資料](https://dev.labs.cognitive.microsoft.com/docs/services/anomaly-detection/operations/post-anomalydetection)
