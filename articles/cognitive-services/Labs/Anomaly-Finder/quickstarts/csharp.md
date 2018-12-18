---
title: 如何搭配使用異常搜尋工具 API 與 C# - Microsoft 認知服務 | Microsoft Docs
description: 取得資訊和程式碼範例，以協助您快速開始使用 C# 和認知服務中的異常搜尋工具 API。
services: cognitive-services
author: chliang
manager: bix
ms.service: cognitive-services
ms.technology: anomaly-detection
ms.topic: article
ms.date: 05/01/2018
ms.author: chliang
ms.openlocfilehash: 3ae3d85e8e37fbf896405948f9bc7042bcdaca7b
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48246443"
---
# <a name="use-the-anomaly-finder-api-with-c"></a>搭配使用異常搜尋工具 API 與 C#

[!INCLUDE [PrivatePreviewNote](../../../../../includes/cognitive-services-anomaly-finder-private-preview-note.md)]

本文提供資訊和程式碼範例，協助您快速開始搭配使用異常搜尋工具 API 與 C#，以完成取得時間序列資料異常結果的工作。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [GetSubscriptionKey](../includes/get-subscription-key.md)]

## <a name="getting-anomaly-points-with-anomaly-finder-api-using-c"></a>使用 C# 與異常搜尋工具 API 取得異常點

[!INCLUDE [DataContract](../includes/datacontract.md)]

### <a name="example-of-time-series-data-points"></a>時間序列資料點範例

時間序列資料點的範例如下所示。
[!INCLUDE [Request](../includes/request.md)]

### <a name="analyze-data-and-get-anomaly-points-c-example"></a>分析資料並取得異常點的 C# 範例

使用此範例的步驟如下所示。

1. 在 Visual Studio 中建立新的主控台解決方案。
2. 將 Program.cs 取代為下列程式碼，並新增 System.Net.Http 的參考。
3. 將 `[YOUR_SUBSCRIPTION_KEY]` 值取代為您的有效訂用帳戶金鑰。
4. 將 `[REPLACE_WITH_THE_EXAMPLE_OR_YOUR_OWN_DATA_POINTS]` 取代為您的資料點。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Text.RegularExpressions;
using System.Threading.Tasks;

namespace Console
{
    class Program
    {
        // **********************************************
        // *** Update or verify the following values. ***
        // **********************************************

        // Replace the subscriptionKey string value with your valid subscription key.
        const string subscriptionKey = "[YOUR_SUBSCRIPTION_KEY]";

        const string endpoint = "https://api.labs.cognitive.microsoft.com/anomalyfinder/v1.0/anomalydetection";

        // Replace the request data with your real data.
        const string requestData = "[REPLACE_WITH_THE_EXAMPLE_OR_YOUR_OWN_DATA_POINTS]";
        static void Main(string[] args)
        {
            var match = Regex.Match(endpoint, "(?<BaseAddress>https://[^/]+)(?<Url>/*.+)");
            if (match.Success)
            {
                var res = Request(
                    match.Groups["BaseAddress"].Value,
                    match.Groups["Url"].Value,
                    subscriptionKey,
                    requestData).Result;
                System.Console.Write(res);
            }
            else
            {
                System.Console.Write("Incorrect endpoint.");
            }
        }

        /// <summary>
        /// Call API to detect the anomaly points
        /// </summary>
        /// <param name="baseAddress">Base address of the API endpoint.</param>
        /// <param name="endpoint">The endpoint of the API</param>
        /// <param name="subscriptionKey">The subscription key applied  </param>
        /// <param name="requestData">The JSON string for requet data points</param>
        /// <returns>The JSON string for anomaly points and expected values.</returns>
        static async Task<string> Request(string baseAddress, string endpoint, string subscriptionKey, string requestData)
        {
            using (HttpClient client = new HttpClient { BaseAddress = new Uri(baseAddress) })
            {
                client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
                client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);

                var content = new StringContent(requestData, Encoding.UTF8, "application/json");
                var res = await client.PostAsync(endpoint, content);
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
    }
}

```

### <a name="example-response"></a>範例回應

成功的回應會以 JSON 的形式傳回。 範例回應如下。
[!INCLUDE [Response](../includes/response.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [C# 應用程式](../tutorials/csharp-tutorial.md)
