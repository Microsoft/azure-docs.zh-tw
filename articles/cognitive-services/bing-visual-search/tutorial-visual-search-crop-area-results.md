---
title: 教學課程：影像裁剪區域和結果 - Bing 圖像式搜尋
description: 如何使用 Bing 圖像式搜尋 SDK，以取得類似已上傳影像裁剪區域的影像 URL。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-visual-search
ms.topic: tutorial
ms.date: 06/20/2018
ms.author: rosh
ms.openlocfilehash: 66e17c00da898e575bb858dbe16a35d1c44a2780
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47226905"
---
# <a name="tutorial-bing-visual-search-sdk-image-crop-area-and-results"></a>教學課程：Bing 圖像式搜尋 SDK 影像裁剪區域和結果
圖像式搜尋 SDK 包含一個選項，可讓您選取影像的某個區域，並於線上尋找類似該較大影像之裁剪區域的影像。  此範例會從包含數個人員的影像中，指定一個顯示單一人員的裁剪區域。  程式碼會傳送裁剪區域及較大影像的 URL，並傳回包含 Bing 搜尋 URL 及於線上找到類似影像之 URL 的結果。

## <a name="prerequisites"></a>必要條件

您將需要 [Visual Studio 2017](https://www.visualstudio.com/downloads/) 以在 Windows 上執行此程式碼。 (可使用免費的 Community Edition)。

您必須有具備 Bing 搜尋 API 的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) \(英文\)。 [免費試用版](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)即足以供本快速入門使用。 您必須要有啟用免費試用版時所提供的存取金鑰，或者您可以從 Azure 儀表板使用付費訂用帳戶金鑰。

## <a name="application-dependencies"></a>應用程式相依性
若要使用 Bing Web 搜尋 SDK 來設定主控台應用程式，請在 Visual Studio 中瀏覽至 [方案總管] 的 [管理 NuGet 套件] 選項。 新增 Microsoft.Azure.CognitiveServices.Search.VisualSearch 套件。

安裝 NuGet Web 搜尋 SDK 套件也會安裝相依性，包括：

* Microsoft.Rest.ClientRuntime
* Microsoft.Rest.ClientRuntime.Azure
* Newtonsoft.Json

## <a name="image-and-crop-area"></a>影像和裁剪區域
下列影像顯示的是 Microsoft 資深領導團隊。  透過使用圖像式搜尋 SDK，我們上傳了該影像的裁剪區域，並找到包含位於較大影像選取區域中之實體的其他影像和網頁。  在此範例中，該實體是一個人。

![Microsoft 資深領導團隊](./media/MS_SrLeaders.jpg)

## <a name="specify-the-crop-area-as-imageinfo-in-visualsearchrequest"></a>在 VisualSearchRequest 中將裁剪區域指定為 ImageInfo
此範例會使用先前影像的裁剪區域，並以整個影像之百分比的方式指定左上角和右下角的座標。  下列程式碼會從裁剪區域建立 `ImageInfo` 物件，並將該 `ImageInfo` 物件載入至 `VisualSearchRequest`。  `ImageInfo` 物件也會包含該影像於線上的 URL。

```
CropArea CropArea = new CropArea(top: (float)0.01, bottom: (float)0.30, left: (float)0.01, right: (float)0.20);
string imageURL = "https://docs.microsoft.com/azure/cognitive-services/bing-visual-search/media/ms_srleaders.jpg;
ImageInfo imageInfo = new ImageInfo(cropArea: CropArea, url: imageURL);

VisualSearchRequest visualSearchRequest = new VisualSearchRequest(imageInfo: imageInfo);
```
## <a name="search-for-images-similar-to-crop-area"></a>搜尋類似裁剪區域的影像
`VisualSearchRequest` 包含影像的相關裁剪區域資訊及其 URL。  `VisualSearchMethodAsync` 方法會取得結果。
```
Console.WriteLine("\r\nSending visual search request with knowledgeRequest that contains URL and crop area");
var visualSearchResults = client.Images.VisualSearchMethodAsync(knowledgeRequest: visualSearchRequest).Result; 

```

## <a name="get-the-url-data-from-imagemoduleaction"></a>從 ImageModuleAction 取得 URL 資料
圖像式搜尋的結果為 `ImageTag` 物件。  每個標記都包含 `ImageAction` 物件清單。  每個 `ImageAction` 都包含 `Data` 欄位，這是會取決於動作類型的值清單：

您可以透過下列程式碼來取得各種類型：
```
Console.WriteLine("\r\n" + "ActionType: " + i.ActionType + " -> WebSearchUrl: " + i.WebSearchUrl);

```
完整應用程式會傳回：

* ActionType: PagesIncluding WebSearchURL:
* ActionType: MoreSizes WebSearchURL:
* ActionType: VisualSearch WebSearchURL:
* ActionType: ImageById WebSearchURL: 
* ActionType: RelatedSearches  WebSearchURL:
* ActionType: Entity -> WebSearchUrl: https://www.bing.com/cr?IG=E40D0E1A13404994ACB073504BC937A4&CID=03DCF882D7386A442137F49BD6596BEF&rd=1&h=BvvDoRtmZ35Xc_UZE4lZx6_eg7FHgcCkigU1D98NHQo&v=1&r=https%3a%2f%2fwww.bing.com%2fsearch%3fq%3dSatya%2bNadella&p=DevEx,5380.1
* ActionType: TopicResults -> WebSearchUrl: https://www.bing.com/cr?IG=E40D0E1A13404994ACB073504BC937A4&CID=03DCF882D7386A442137F49BD6596BEF&rd=1&h=3QGtxPb3W9LemuHRxAlW4CW7XN4sPkUYCUynxAqI9zQ&v=1&r=https%3a%2f%2fwww.bing.com%2fdiscover%2fnadella%2bsatya&p=DevEx,5382.1
* ActionType: ImageResults -> WebSearchUrl: https://www.bing.com/cr?IG=E40D0E1A13404994ACB073504BC937A4&CID=03DCF882D7386A442137F49BD6596BEF&rd=1&h=l-WNHO89Kkw69AmIGe2MhlUp6MxR6YsJszgOuM5sVLs&v=1&r=https%3a%2f%2fwww.bing.com%2fimages%2fsearch%3fq%3dSatya%2bNadella&p=DevEx,5384.1

如先前的清單所示，`Entity` `ActionType` 所包含的 Bing 搜尋查詢，會傳回可辨識人員、地點或物體的相關資訊。  `TopicResults` 和 `ImageResults` 類型包含針對相關影像的查詢。 清單中的 URL 會連結至 Bing 搜尋結果。


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

## <a name="complete-code"></a>完整程式碼

下列程式碼會執行上述範例。 它會在張貼要求的主體中傳送影像二進位檔 (連同 cropArea 物件)，並列印出每個 ActionType 的 Bing 搜尋 URL。 若 ActionType 為 PagesIncluding，程式碼會取得 ImageObject Data 中的 ImageObject 項目。  Data 包含值清單，這些值是網頁上影像的 URL。  將 Visual Studio URL 複製並貼上至瀏覽器以顯示結果。 將 ContentUrl 項目複製並貼上至瀏覽器以顯示影像。

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

            VisualSearchImageBinaryWithCropArea(subscriptionKey);

            Console.WriteLine("Any key to quit...");
            Console.ReadKey();

        }

        public static void VisualSearchImageBinaryWithCropArea(string subscriptionKey)
        {
            var client = new VisualSearchAPI(new Microsoft.Azure.CognitiveServices.Search.VisualSearch.ApiKeyServiceClientCredentials(subscriptionKey));

            try
            {
                CropArea CropArea = new CropArea(top: (float)0.01, bottom: (float)0.30, left: (float)0.01, right: (float)0.20);
                
                // The ImageInfo struct specifies the crop area in the image and the URL of the larger image. 
                string imageURL = "https://docs.microsoft.com/azure/cognitive-services/bing-visual-search/media/ms_srleaders.jpg";
                ImageInfo imageInfo = new ImageInfo(cropArea: CropArea, url: imageURL);
                
                VisualSearchRequest visualSearchRequest = new VisualSearchRequest(imageInfo: imageInfo);

                var visualSearchResults = client.Images.VisualSearchMethodAsync(knowledgeRequest: visualSearchRequest).Result; 

                Console.WriteLine("\r\nVisual search request with image from URL and crop area combined in knowledgeRequest");

                if (visualSearchResults == null)
                {
                    Console.WriteLine("No visual search result data.");
                }
                else
                {
                    // List of tags
                    if (visualSearchResults.Tags.Count > 0)
                    {

                        foreach(ImageTag t in visualSearchResults.Tags)
                        {
                            foreach (ImageAction i in t.Actions)
                            { 
                                Console.WriteLine("\r\n" + "ActionType: " + i.ActionType + " -> WebSearchUrl: " + i.WebSearchUrl);

                                if (i.ActionType == "PagesIncluding")
                                {
                                    foreach(ImageObject o in (i as ImageModuleAction).Data.Value)
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