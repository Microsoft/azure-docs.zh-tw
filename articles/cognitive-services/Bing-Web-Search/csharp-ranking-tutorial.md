---
title: 使用順位顯示搜尋結果 | Microsoft Docs
description: 示範如何使用 Bing RankingResponse 答案，以順位的順序顯示搜尋結果。
services: cognitive-services
author: bradumbaugh
manager: bking
ms.assetid: 2575A80C-FC74-4631-AE5D-8101CF2591D3
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: article
ms.date: 05/08/2017
ms.author: brumbaug
ms.openlocfilehash: 0dd3a2057e73adda3224e7cebe7c492572f94105
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42093854"
---
# <a name="build-a-console-app-search-client-in-c"></a>使用 C# 建置主控台應用程式搜尋用戶端

本教學課程示範如何建置簡單的 .NET Core 主控台應用程式，讓使用者能夠查詢 Bing Web 搜尋 API 並顯示已設定順位的結果。

本教學課程說明如何：

- 對 Bing Web 搜尋 API 進行簡單查詢
- 以已設定順位的順序顯示查詢結果

## <a name="prerequisites"></a>必要條件

若要繼續進行本教學課程，您需要：

- Visual Studio。 如果您沒有此產品，請先[下載並安裝免費的 Visual Studio 2017 Community Edition](https://www.visualstudio.com/downloads/)。
- 適用於 Bing Web 搜尋 API 的訂用帳戶金鑰。 如果您沒有金鑰，請[註冊免費試用](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)。

## <a name="create-a-new-console-app-project"></a>建立新的主控台應用程式專案

在 Visual Studio 中，使用 `Ctrl`+`Shift`+`N` 建立專案。

在 [新增專案] 對話方塊中，按一下 [Visual C#] > [Windows 傳統桌面] > [主控台應用程式 (.NET Framework)]。

將應用程式命名為 **MyConsoleSearchApp**，然後按一下 [確定]。

## <a name="add-the-jsonnet-nuget-package-to-the-project"></a>將 JSON.net Nuget 套件新增到專案

JSON.net 讓您能夠使用 API 所傳回的 JSON 回應。 將它的 NuGet 套件新增到您的專案：

- 在 [方案總管] 中，以滑鼠右鍵按一下專案，然後選取 [管理 NuGet 套件]。 
- 在 [瀏覽] 索引標籤上，搜尋 `Newtonsoft.Json`。 選取最新版本，然後按一下 [安裝]。 
- 按一下 [檢閱變更] 視窗上的 [確定] 按鈕。
- 關閉標題為 **NuGet: MyConsoleSearchApp** 的 Visual Studio 索引標籤。

## <a name="add-a-reference-to-systemweb"></a>新增對 System.Web 的參考

本教學課程依賴 `System.Web` 組件。 將對這個組件的參考新增到您的專案：

- 在 [方案總管] 中，以滑鼠右鍵按一下 [參考]，然後選取 [新增參考]
- 選取 [組件] > [架構]，然後向下捲動並勾選 [System.Web]
- 選取 [確定]

## <a name="add-some-necessary-using-statements"></a>新增一些必要的 using 陳述式

本教學課程中的程式碼需要三個額外的 using 陳述式。 將這些陳述式新增到 **Program.cs** 頂端的現有 `using` 陳述式下方： 

```csharp
using System.Web;
using System.Net.Http;
```

## <a name="ask-the-user-for-a-query"></a>要求使用者查詢

在 [方案總管] 中開啟 **Program.cs**。 更新 `Main()` 方法：

```csharp
static void Main()
{
    // Get the user's query
    Console.Write("Enter Bing query: ");
    string userQuery = Console.ReadLine();
    Console.WriteLine();

    // Run the query and display the results
    RunQueryAndDisplayResults(userQuery);

    // Prevent the console window from closing immediately
    Console.WriteLine("\nHit ENTER to exit...");
    Console.ReadLine();
}
```

這個方法：

- 要求使用者查詢
- 呼叫 `RunQueryAndDisplayResults(userQuery)` 來執行查詢並顯示結果
- 等待使用者輸入，以避免立即關閉主控台視窗。

## <a name="search-for-query-results-using-the-bing-web-search-api"></a>使用 Bing Web 搜尋 API 來搜尋查詢結果

接著，新增方法來查詢 API 並顯示結果：

```csharp
static void RunQueryAndDisplayResults(string userQuery)
{
    try
    {
        // Create a query
        var client = new HttpClient();
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "<YOUR_SUBSCRIPTION_KEY_GOES_HERE>");
        var queryString = HttpUtility.ParseQueryString(string.Empty);
        queryString["q"] = userQuery;
        var query = "https://api.cognitive.microsoft.com/bing/v7.0/search?" + queryString;

        // Run the query
        HttpResponseMessage httpResponseMessage = client.GetAsync(query).Result;

        // Deserialize the response content
        var responseContentString = httpResponseMessage.Content.ReadAsStringAsync().Result;
        Newtonsoft.Json.Linq.JObject responseObjects = Newtonsoft.Json.Linq.JObject.Parse(responseContentString);

        // Handle success and error codes
        if (httpResponseMessage.IsSuccessStatusCode)
        {
            DisplayAllRankedResults(responseObjects);
        }
        else
        {
            Console.WriteLine($"HTTP error status code: {httpResponseMessage.StatusCode.ToString()}");
        }
    }
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
    }
}
```

這個方法：

- 建立 `HttpClient` 以查詢 Web 搜尋 API
- 設定 `Ocp-Apim-Subscription-Key` HTTP 標頭，Bing 會使用此標頭來驗證要求
- 執行要求，並使用 JSON.net 來將結果還原序列化
- 呼叫 `DisplayAllRankedResults(responseObjects)`，以已設定順位的順序顯示所有結果

確定會將 `Ocp-Apim-Subscription-Key` 的值設為您的訂用帳戶金鑰。

## <a name="display-ranked-results"></a>顯示已設定順位的結果

顯示如何以已設定順位的順序顯示結果之前，先查看範例 Web 搜尋回應： 

```json
{
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=70BE289346...",
        "totalEstimatedMatches" : 982000,
        "value" : [{
            "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0",
            "name" : "Contoso Sailing Club - Seattle",
            "url" : "https:\/\/www.bing.com\/cr?IG=70BE289346ED4594874FE...",
            "displayUrl" : "https:\/\/contososailingsea...",
            "snippet" : "Come sail with Contoso in Seattle...",
            "dateLastCrawled" : "2017-04-07T02:25:00"
        },
        {
            "id" : "https:\/\/api.cognitive.microsoft.com\/api\/7\/#WebPages.6",
            "name" : "Contoso Sailing Lessons - Official Site",
            "url" : "http:\/\/www.bing.com\/cr?IG=70BE289346ED4594874FE...",
            "displayUrl" : "https:\/\/www.constososailinglessonsseat...",
            "snippet" : "Contoso sailing lessons in Seattle...",
            "dateLastCrawled" : "2017-04-09T14:30:00"
        },

        ...
        
        ],
        "someResultsRemoved" : true
    },
    "relatedSearches" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/7\/#RelatedSearches",
        "value" : [{
            "text" : "sailing lessons",
            "displayText" : "sailing lessons",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=70BE289346E..."
        }

        ...
        
        ]
    },
    "rankingResponse" : {
        "mainline" : {
            "items" : [{
                "answerType" : "WebPages",
                "resultIndex" : 0,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 1,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.1"
                }
            }

            ...

            ]
        },
        "sidebar" : {
            "items" : [{
                "answerType" : "RelatedSearches",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#RelatedSearches"
                }
            }]
        }
    }
}
```

`rankingResponse` JSON 物件 ([文件](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#rankingresponse)) 會說明搜尋結果的適當顯示順序。 它會包含下列一或多個已設定優先順序的群組： 

- `pole`：獲得最明顯處理的搜尋結果 (例如，顯示在主線和資訊看板上方)。
- `mainline`：顯示在主線中的搜尋結果。
- `sidebar`：顯示在資訊看板中的搜尋結果。 如果沒有資訊看板，則會在主線下方顯示結果。

已設定順位的回應 JSON 可能包括一或多個群組。

在 **Program.cs** 中新增下列方法，以已適當設定順位的順序顯示結果：

```csharp
static void DisplayAllRankedResults(Newtonsoft.Json.Linq.JObject responseObjects)
{
    string[] rankingGroups = new string[] { "pole", "mainline", "sidebar" };

    // Loop through the ranking groups in priority order
    foreach (string rankingName in rankingGroups)
    {
        Newtonsoft.Json.Linq.JToken rankingResponseItems = responseObjects.SelectToken($"rankingResponse.{rankingName}.items");
        if (rankingResponseItems != null)
        {
            foreach (Newtonsoft.Json.Linq.JObject rankingResponseItem in rankingResponseItems)
            {
                Newtonsoft.Json.Linq.JToken resultIndex;
                rankingResponseItem.TryGetValue("resultIndex", out resultIndex);
                var answerType = rankingResponseItem.Value<string>("answerType");
                switch (answerType)
                {
                    case "WebPages":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("webPages.value"), "WebPage", "name", "url", "displayUrl", "snippet");
                        break;
                    case "News":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("news.value"), "News", "name", "url", "description");
                        break;
                    case "Images":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("images.value"), "Image", "thumbnailUrl");
                        break;
                    case "Videos":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("videos.value"), "Video", "embedHtml");
                        break;
                    case "RelatedSearches":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("relatedSearches.value"), "RelatedSearch", "displayText", "webSearchUrl");
                        break;
                }
            }
        }
    }
}
```

這個方法：

- 在回應包含的 `rankingResponse` 群組上進行迴圈
- 藉由呼叫 `DisplaySpecificResults(...)` 來顯示每個群組中的項目 

在 **Program.cs** 中，新增下列兩個方法：

```csharp
static void DisplaySpecificResults(Newtonsoft.Json.Linq.JToken resultIndex, Newtonsoft.Json.Linq.JToken items, string title, params string[] fields)
{
    if (resultIndex == null)
    {
        foreach (Newtonsoft.Json.Linq.JToken item in items)
        {
            DisplayItem(item, title, fields);
        }
    }
    else
    {
        DisplayItem(items.ElementAt((int)resultIndex), title, fields);
    }
}

static void DisplayItem(Newtonsoft.Json.Linq.JToken item, string title, string[] fields)
{
    Console.WriteLine($"{title}: ");
    foreach( string field in fields )
    {
        Console.WriteLine($"- {field}: {item[field]}");
    }
    Console.WriteLine();
}
```

這些方法會一起運作，以將搜尋結果輸出到主控台。

## <a name="run-the-application"></a>執行應用程式

執行應用程式。 輸出應如下所示：

```
Enter Bing query: sailing lessons seattle

WebPage:
- name: Contoso Sailing Club - Seattle
- url: https://www.bing.com/cr?IG=70BE289346ED4594874FE...
- displayUrl: https://contososailingsea....
- snippet: Come sail with Contoso in Seattle...

WebPage:
- name: Contoso Sailing Lessons Seattle - Official Site
- url: http://www.bing.com/cr?IG=70BE289346ED4594874FE...
- displayUrl: https://www.constososailinglessonsseat...
- snippet: Contoso sailing lessons in Seattle...

...
```

## <a name="next-steps"></a>後續步驟

深入了解如何[使用順位顯示結果](rank-results.md)。
