---
title: 快速入門：使用適用於 Java 的 Bing 影像搜尋 SDK 來搜尋影像
description: Bing 影像搜尋 SDK 是 API 的包裝函式，而且功能相同，您可以透過此快速入門，完成初次使用此 SDK 進行的影像搜尋。 這個簡單的 Java 應用程式會傳送影像搜尋查詢、剖析 JSON 回應，以及顯示第一個傳回影像的 URL。
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: aahi
ms.openlocfilehash: 36f59e1c405ef9e5cf69a19e49d69a3adfdc4636
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46298178"
---
# <a name="quickstart-search-for-images-with-the-bing-image-search-sdk-and-java"></a>快速入門：使用 Bing 影像搜尋 SDK 和 Java 來搜尋影像

Bing 影像搜尋 SDK 是 API 的包裝函式，而且功能相同，您可以透過此快速入門，完成初次使用此 SDK 進行的影像搜尋。 這個簡單的 Java 應用程式會傳送影像搜尋查詢、剖析 JSON 回應，以及顯示第一個傳回影像的 URL。

此範例的原始程式碼可從 [GitHub](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingImageSearch/Quickstart) 取得，其中含有其他錯誤處理和註釋。

## <a name="prerequisites"></a>先決條件

最新版的 [Java Development Kit](http://www.oracle.com/technetwork/java/javase/downloads/index.html) (JDK)

使用 Maven、Gradle 或另一個相依性管理系統，來安裝 Bing 影像搜尋 SDK 相依性。 Maven POM 檔案需要下列宣告：

```xml
 <dependencies>
    <dependency>
      <groupId>com.microsoft.azure.cognitiveservices</groupId>
      <artifactId>azure-cognitiveservices-imagesearch</artifactId>
      <version>0.0.1-beta-SNAPSHOT</version>
    </dependency>
 </dependencies>
```

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>建立並初始化應用程式

1. 在您慣用的 IDE 或編輯器中建立新的 Java 專案，並將下列匯入加入至您的類別實作：

    ```java
    import com.microsoft.azure.cognitiveservices.search.imagesearch.BingImageSearchAPI;
    import com.microsoft.azure.cognitiveservices.search.imagesearch.BingImageSearchManager;
    import com.microsoft.azure.cognitiveservices.search.imagesearch.models.ImageObject;
    import com.microsoft.azure.cognitiveservices.search.imagesearch.models.ImagesModel;
    ```

2. 在您的主要方法中，針對訂用帳戶金鑰和搜尋字詞建立變數。 然後將 Bing 影像搜尋用戶端具現化。

    ```java
    final String subscriptionKey = "COPY_YOUR_KEY_HERE";
    String searchTerm = "canadian rockies";
    //Image search client
    BingImageSearchAPI client = BingImageSearchManager.authenticate(subscriptionKey);
    ```

## <a name="send-a-search-request-to-the-bing-image-search-api"></a>將搜尋要求傳送到 Bing 影像搜尋 API

1. 使用 `bingImages().search()`，傳送包含搜尋查詢的 HTTP 要求。 將回應儲存為 `ImagesModel`。
    ```java
    ImagesModel imageResults = client.bingImages().search()
                .withQuery(searchTerm)
                .withMarket("en-us")
                .execute();
    ```

## <a name="parse-and-view-the-result"></a>剖析並檢視結果

剖析回應中所傳回的影像結果。
如果回應包含搜尋結果，請儲存第一個結果並列印出其詳細資料，例如縮圖 URL、原始 URL 及傳回影像的總數。  

```java
if (imageResults != null && imageResults.value().size() > 0) {
    // Image results
    ImageObject firstImageResult = imageResults.value().get(0);

    System.out.println(String.format("Total number of images found: %d", imageResults.value().size()));
    System.out.println(String.format("First image thumbnail url: %s", firstImageResult.thumbnailUrl()));
    System.out.println(String.format("First image content url: %s", firstImageResult.contentUrl()));
}
else {
        System.out.println("Couldn't find image results!");
    }
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing 影像搜尋單頁應用程式教學課程](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/tutorial-bing-image-search-single-page-app) (英文)

## <a name="see-also"></a>另請參閱

* [什麼是 Bing 影像搜尋？](https://docs.microsoft.com/azure/cognitive-services/bing-image-search/overview)  
* [試用線上互動式示範](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)  
* [取得免費的認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-image-search-api)
* [Azure 認知服務 SDK 的 Java 範例](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples)
* [Azure 認知服務文件](https://docs.microsoft.com/azure/cognitive-services)
* [Bing 影像搜尋 API 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)
