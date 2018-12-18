---
title: 快速入門：使用適用於 C# 的 Bing 影像搜尋 SDK 來搜尋影像
description: Bing 影像搜尋 SDK 是 API 的包裝函式，而且功能相同，您可以透過此快速入門，完成初次使用此 SDK 進行的影像搜尋。 這個簡單的 C# 應用程式會傳送影像搜尋查詢、剖析 JSON 回應，以及顯示第一個傳回影像的 URL。
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: aahi
ms.openlocfilehash: edebd1361e39a338672b4249dd159e5c1d4078ce
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46294147"
---
# <a name="quickstart-search-for-images-with-the-bing-image-search-sdk-and-c"></a>快速入門：使用 Bing 影像搜尋 SDK 和 C# 來搜尋影像

Bing 影像搜尋 SDK 是 API 的包裝函式，而且功能相同，您可以透過此快速入門，完成初次使用此 SDK 進行的影像搜尋。 這個簡單的 C# 應用程式會傳送影像搜尋查詢、剖析 JSON 回應，以及顯示第一個傳回影像的 URL。

此範例的原始程式碼可從 [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7/BingImageSearch) 取得，其中含有其他錯誤處理和註釋。

## <a name="prerequisites"></a>先決條件

* [Visual Studio 2017](https://visualstudio.microsoft.com/vs/whatsnew/) 的任意版本。
* [認知影像搜尋 NuGet 套件](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.ImageSearch/1.2.0)。

若要在 Visual Studio 中安裝 Bing 影像搜尋 SDK ，可在 Visual Studio 的 [方案總管] 中使用 `Manage NuGet Packages` 選項。

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]


## <a name="create-and-initialize-the-application"></a>建立並初始化應用程式

首先，在 Visual Studio 中建立新的 C# 主控台應用程式。 然後，將下列套件新增至您的專案。

```csharp
using System;
using System.Linq;
using Microsoft.Azure.CognitiveServices.Search.ImageSearch;
using Microsoft.Azure.CognitiveServices.Search.ImageSearch.Models;
```

在專案的主要方法中，針對有效訂用帳戶金鑰、要由 Bing 傳回的影像結果和搜尋字詞建立變數。 然後使用金鑰來將影像搜尋用戶端具現化。

```csharp
//IMPORTANT: replace this variable with your Cognitive Services subscription key
string subscriptionKey = "ENTER YOUR KEY HERE";
//stores the image results returned by Bing
Images imageResults = null;
// the image search term to be used in the query
string searchTerm = "canadian rockies";
//initialize the client
var client = new ImageSearchAPI(new ApiKeyServiceClientCredentials(subscriptionKey));
```

## <a name="send-a-search-query-using-the-client"></a>使用用戶端傳送搜尋查詢

使用用戶端以利用查詢文字搜尋：

```csharp
// make the search request to the Bing Image API, and get the results"
imageResults = client.Images.SearchAsync(query: searchTerm).Result; //search query
```

## <a name="parse-and-view-the-first-image-result"></a>剖析並檢視第一個影像結果

剖析回應中所傳回的影像結果。
如果回應包含搜尋結果，請儲存第一個結果並列印出其詳細資料，例如縮圖 URL、原始 URL 及傳回影像的總數。  

```csharp
if (imageResults != null)
{
    //display the details for the first image result.
    var firstImageResult = imageResults.Value.First();
    Console.WriteLine($"\nTotal number of returned images: {imageResults.Value.Count}\n");
    Console.WriteLine($"Copy the following URLs to view these images on your browser.\n");
    Console.WriteLine($"URL to the first image:\n\n {firstImageResult.ContentUrl}\n");
    Console.WriteLine($"Thumbnail URL for the first image:\n\n {firstImageResult.ThumbnailUrl}");
    Console.ReadKey();
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing 影像搜尋單頁應用程式教學課程](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/tutorial-bing-image-search-single-page-app) (英文)

## <a name="see-also"></a>另請參閱

* [什麼是 Bing 影像搜尋？](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/overview)  
* [試用線上互動式示範](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)  
* [取得免費的認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-image-search-api)  
* [Azure 認知服務 SDK 的 .NET 範例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7)
* [Azure 認知服務文件](https://docs.microsoft.com/azure/cognitive-services)
* [Bing 影像搜尋 API 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)
