---
title: 教學課程：ImageInsightsToken - Bing 圖像式搜尋
titlesuffix: Azure Cognitive Services
description: 如何使用 Bing 圖像式搜尋 SDK 來取得 ImageInsightsToken 所指定影像的 URL。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-visual-search
ms.topic: tutorial
ms.date: 06/21/2018
ms.author: rosh
ms.openlocfilehash: 06d6bc8e53276b5542210c2843d7221d6fd79c09
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49386429"
---
# <a name="tutorial-bing-visual-search-sdk-imageinsightstoken-and-results"></a>教學課程：Bing 圖像式搜尋 SDK ImageInsightsToken 和結果
圖像式搜尋 SDK 有選項可供從先前傳回 `ImageInsightsToken` 的搜尋中，線上尋找影像。  此範例可取得 `ImageInsightsToken`，並在後續搜尋中使用權杖。  程式碼會將 `ImageInsightsToken` 傳送到 Bing 並傳回結果，結果中包含 Bing 搜尋 URL 和線上所找到類似影像的 URL。

## <a name="prerequisites"></a>必要條件
Visual Studio 2017。 如有需要，您可從下列位置下載免費的社群版本： https://www.visualstudio.com/vs/community/。
必須要有認知服務 API 金鑰才能驗證 SDK 呼叫。 申請免費試用金鑰。 試用金鑰可讓您以每秒 1 個呼叫的頻率使用 7 天。 針對生產案例，請購買存取金鑰。 另請參閱定價資訊。
要能夠執行 .NET Core SDK，也就是 .NET Core 1.1 應用程式。 您可以從下列位置取得 CORE、架構和執行階段： https://www.microsoft.com/net/download/。

## <a name="application-dependencies"></a>應用程式相依性
若要使用 Bing Web 搜尋 SDK 來設定主控台應用程式，請在 Visual Studio 中瀏覽至 [方案總管] 的 [管理 NuGet 套件] 選項。 新增：
* Microsoft.Azure.CognitiveServices.Search.VisualSearch
* Microsoft.Azure.CognitiveServices.Search.ImageSearchpackage 套件。

安裝 NuGet Web 搜尋 SDK 套件也會安裝相依性，包括：

* Microsoft.Rest.ClientRuntime
* Microsoft.Rest.ClientRuntime.Azure
* Newtonsoft.Json

## <a name="get-the-imageinsightstoken-from-image-search"></a>從影像搜尋取得 ImageInsightsToken

此範例會使用下列方法所取得的 `ImageInsightsToken`。  如需此呼叫的詳細資訊，請參閱[影像搜尋 SDK C# 快速入門](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/image-search-sdk-quickstart)。

程式碼會搜尋 'Canadian Rockies' 的查詢結果，並取得 ImageInsightsToken。 它會列印第一個影像的見解權杖、縮圖 URL 和影像內容 URL。  方法會傳回 `ImageInsightsToken` 以供後續的圖像式搜尋要求使用。

```
        private static String ImageResults(String subKey)
        {
            String insightTok = "None";
            try
            {
                var client = new ImageSearchAPI(new Microsoft.Azure.CognitiveServices.Search.ImageSearch.ApiKeyServiceClientCredentials(subKey)); //
                var imageResults = client.Images.SearchAsync(query: "canadian rockies").Result;
                Console.WriteLine("Search images for query \"canadian rockies\"");

                if (imageResults == null)
                {
                    Console.WriteLine("No image result data.");
                }
                else
                {
                    // Image results
                    if (imageResults.Value.Count > 0)
                    {
                        var firstImageResult = imageResults.Value.First();

                        Console.WriteLine($"\r\nImage result count: {imageResults.Value.Count}");
                        insightTok = firstImageResult.ImageInsightsToken;
                        Console.WriteLine($"First image insights token: {firstImageResult.ImageInsightsToken}");  
                        Console.WriteLine($"First image thumbnail url: {firstImageResult.ThumbnailUrl}");
                        Console.WriteLine($"First image content url: {firstImageResult.ContentUrl}");
                    }
                    else
                    {
                        insightTok = "None found";
                        Console.WriteLine("Couldn't find image results!");
                    }
                }
            }

            catch (Exception ex)
            {
                Console.WriteLine("\r\nEncountered exception. " + ex.Message);
            }

            return insightTok;
        }
```

## <a name="specify-the-imageinsightstoken-for-visual-search-request"></a>指定圖像式搜尋要求的 ImageInsightsToken

此範例會使用上一個方法所傳回的見解權杖。 下列程式碼會從 `ImageInsightsToken` 建立 `ImageInfo` 物件，並將 ImageInfo 物件載入至 `VisualSearchRequest`。 在 `ImageInfo` 中指定 `VisualSearchRequest` 的 `ImageInsightsToken`

```
ImageInfo ImageInfo = new ImageInfo(imageInsightsToken: insightsTok);
```

## <a name="use-visual-search-to-find-images-from-an-imageinsightstoken"></a>使用圖像式搜尋來尋找 ImageInsightsToken 中的影像

`VisualSearchRequest` 包含 `ImageInfo` 物件中所要搜尋影像的相關資訊。  `VisualSearchMethodAsync` 方法會取得結果。
```
// An image binary is not necessary here, as the image is specified by insights token.
VisualSearchRequest VisualSearchRequest = new VisualSearchRequest(ImageInfo);

var visualSearchResults = client.Images.VisualSearchMethodAsync(knowledgeRequest: VisualSearchRequest).Result;
Console.WriteLine("\r\nVisual search request with knowledgeRequest");

```

## <a name="get-the-url-data-from-imagemoduleaction"></a>從 ImageModuleAction 取得 URL 資料
圖像式搜尋的結果為 `ImageTag` 物件。  每個標記都包含 `ImageAction` 物件清單。  每個 `ImageAction` 都包含 `Data` 欄位，這是會取決於動作類型的值清單：

您可以透過下列程式碼來取得各種類型：
```
Console.WriteLine("\r\n" + "ActionType: " + i.ActionType + " -> WebSearchUrl: " + i.WebSearchUrl);

```
完整應用程式會傳回：

* ActionType: MoreSizes -> WebSearchUrl:
* ActionType: VisualSearch -> WebSearchUrl:
* ActionType: ImageById -> WebSearchUrl:
* ActionType: RelatedSearches -> WebSearchUrl:
* ActionType: DocumentLevelSuggestions -> WebSearchUrl:
* ActionType: TopicResults -> WebSearchUrl: https://www.bing.com/cr?IG=3E32CC6CA5934FBBA14ABC3B2E4651F9&CID=1BA795A21EAF6A63175699B71FC36B7C&rd=1&h=BcQifmzdKFyyBusjLxxgO42kzq1Geh7RucVVqvH-900&v=1&r=https%3a%2f%2fwww.bing.com%2fdiscover%2fcanadian%2brocky&p=DevEx,5823.1
* ActionType: ImageResults -> WebSearchUrl: https://www.bing.com/cr?IG=3E32CC6CA5934FBBA14ABC3B2E4651F9&CID=1BA795A21EAF6A63175699B71FC36B7C&rd=1&h=PV9GzMFOI0AHZp2gKeWJ8DcveSDRE3fP2jHDKMpJSU8&v=1&r=https%3a%2f%2fwww.bing.com%2fimages%2fsearch%3fq%3doutdoor&p=DevEx,5831.1

如上述清單所示，`TopicResults` 和 `ImageResults` 類型包含針對相關影像的查詢。 清單中的 URL 會連結至 Bing 搜尋結果。


## <a name="pagesincluding-actiontype-urls-of-images-found-by-visual-search"></a>由圖像式搜尋所找到的影像 PagesIncluding ActionType URL

取得實際的影像 URL 需要進行轉換，以將 `ActionType` 讀取為 `ImageModuleAction`，其中包含具有值清單的 `Data` 元素。  每個值都是某個影像的 URL。  下列程式碼會將 `PagesIncluding` 動作類型轉換為 `ImageModuleAction`，並讀取那些值。
```
    if (i.ActionType == "PagesIncluding")
    {
        foreach(ImageObject o in (i as ImageModuleAction).Data.Value)
        {
            Console.WriteLine("ContentURL: " + o.ContentUrl);
        }
    }
```
如需這些資料類型的詳細資訊，請參閱[影像 - 圖像式搜尋](https://docs.microsoft.com/rest/api/cognitiveservices/bingvisualsearch/images/visualsearch)。

## <a name="complete-code"></a>完整程式碼

下列程式碼會執行上述範例。 它會在 POST 要求中傳送 `ImageInsightsToken`。 然後，它會列印每個 ActionType 的 Bing 搜尋 URL。 如果 ActionType 是 `PagesIncluding`，程式碼會取得 `Data` 中的 `ImageObject` 項目。  `Data` 包含值清單，這些值是網頁上影像的 URL。  將 Visual Studio URL 複製並貼上至瀏覽器以顯示結果。 將 ContentUrl 項目複製並貼上至瀏覽器以顯示影像。

```
using System;
using System.IO;
using System.Linq;
using Microsoft.Azure.CognitiveServices.Search.VisualSearch;
using Microsoft.Azure.CognitiveServices.Search.VisualSearch.Models;
using Microsoft.Azure.CognitiveServices.Search.ImageSearch;

namespace VisualSearchFeatures
{
    class Program
    {
        static void Main(string[] args)
        {
            String subscriptionKey = "YOUR-ACCESS-KEY";

            insightsToken = ImageResults(subscriptionKey);

            VisualSearchInsightsToken(subscriptionKey, insightsToken);

            Console.WriteLine("Any key to quit...");
            Console.ReadKey();

        }

        // Searches for results on query (Canadian Rockies) and gets an ImageInsightsToken.
        // Also prints first image insights token, thumbnail url, and image content url.
        private static String ImageResults(String subKey)
        {
            String insightTok = "None";
            try
            {
                var client = new ImageSearchAPI(new Microsoft.Azure.CognitiveServices.Search.ImageSearch.ApiKeyServiceClientCredentials(subKey)); //
                var imageResults = client.Images.SearchAsync(query: "canadian rockies").Result;
                Console.WriteLine("Search images for query \"canadian rockies\"");

                if (imageResults == null)
                {
                    Console.WriteLine("No image result data.");
                }
                else
                {
                    // Image results
                    if (imageResults.Value.Count > 0)
                    {
                        var firstImageResult = imageResults.Value.First();

                        Console.WriteLine($"\r\nImage result count: {imageResults.Value.Count}");
                        insightTok = firstImageResult.ImageInsightsToken;
                        Console.WriteLine($"First image insights token: {firstImageResult.ImageInsightsToken}");  
                        Console.WriteLine($"First image thumbnail url: {firstImageResult.ThumbnailUrl}");
                        Console.WriteLine($"First image content url: {firstImageResult.ContentUrl}");
                    }
                    else
                    {
                        insightTok = "None found";
                        Console.WriteLine("Couldn't find image results!");
                    }
                }
            }

            catch (Exception ex)
            {
                Console.WriteLine("\r\nEncountered exception. " + ex.Message);
            }

            return insightTok;
        }


        // This method will get Visual Search results from an imageInsightsToken obtained from the return value of the ImageResults method.
        // The method includes imageInsightsToken the in a knowledgeRequest parameter, along with a cropArea object. 
        // It prints out the imageInsightsToken uploaded in the request.
        // Finally the example prints URLs of images found using the imageInsightsToken.

        public static void VisualSearchInsightsToken(string subscriptionKey, string insightsTok)
        {
            var client = new VisualSearchAPI(new Microsoft.Azure.CognitiveServices.Search.VisualSearch.ApiKeyServiceClientCredentials(subscriptionKey));

            try
            {
                // The image can be specified via an insights token, in the ImageInfo object.
                ImageInfo ImageInfo = new ImageInfo(imageInsightsToken: insightsTok);

                // An image binary is not necessary here, as the image is specified by insights token.
                VisualSearchRequest VisualSearchRequest = new VisualSearchRequest(ImageInfo);

                var visualSearchResults = client.Images.VisualSearchMethodAsync(knowledgeRequest: VisualSearchRequest).Result;
                Console.WriteLine("\r\nVisual search request with imageInsightsToken and knowledgeRequest");

                if (visualSearchResults == null)
                {
                    Console.WriteLine("No visual search result data.");
                }
                else
                {
                    // Visual Search results
                    if (visualSearchResults.Image?.ImageInsightsToken != null)
                    {
                        Console.WriteLine($"Uploaded image insights token: {insightsTok}");
                    }
                    else
                    {
                        Console.WriteLine("Couldn't find image insights token!");
                    }

                    if (visualSearchResults.Tags.Count > 0)
                    {
                        // List of tags
                        foreach (ImageTag t in visualSearchResults.Tags)
                        {
                            foreach (ImageAction i in t.Actions)
                            {
                                Console.WriteLine("\r\n" + "ActionType: " + i.ActionType + " WebSearchURL: " + i.WebSearchUrl);

                                if (i.ActionType == "PagesIncluding")
                                {
                                    foreach (ImageObject o in (i as ImageModuleAction).Data.Value)
                                    {
                                        Console.WriteLine("ContentURL: " + o.ContentUrl);
                                    }
                                }
                            }
                        }
                    }

                    else
                    {
                        Console.WriteLine("Couldn't find image tags!");
                    }

                }
            }

            catch (Exception ex)
            {
                Console.WriteLine("Encountered exception. " + ex.Message);
            }
        }
    }
}

```

## <a name="next-steps"></a>後續步驟
[圖像式搜尋回應](https://docs.microsoft.com/azure/cognitive-services/bing-visual-search/overview#the-response) \(英文\)
