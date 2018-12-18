---
title: 使用 Bing 影像搜尋 API 從 Web 取得影像 | Microsoft Docs
description: 使用 Bing 影像搜尋 API 來搜尋並取得 Web 中的相關影像。
services: cognitive-services
author: aahill
manager: cgronlun
ms.assetid: AB1B9898-C94A-4B59-91A1-8623C94BA3D4
ms.service: cognitive-services
ms.component: bing-image-search
ms.topic: conceptual
ms.date: 8/8/2018
ms.author: aahi
ms.openlocfilehash: c379cc2dfbe53f83e4d79500cd947879407c8d55
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "40082521"
---
# <a name="getting-images-from-the-web-with-the-bing-image-search-api"></a>使用 Bing 影像搜尋 API 從 Web 取得影像

使用 Bing 影像搜尋 REST API，即可藉由傳送下列 GET 要求，從 Web 中取得與使用者搜尋詞彙相關的影像：

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
X-MSEdge-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

> [!IMPORTANT]
> * 所有要求都必須是從伺服器進行，不能從用戶端進行。
> * 如果這是您第一次呼叫任何 Bing 搜尋 API，請勿包含用戶端識別碼標頭。 如果您先前已呼叫 Bing API，並且傳回了使用者和裝置組合的用戶端識別碼，才需包含用戶端識別碼。
> * 影像必須以回應中提供的順序來顯示。

## <a name="getting-images-from-a-specific-web-domain"></a>從特定 Web 網域中取得影像

若要從特定網域取得影像，請使用 [site:](http://msdn.microsoft.com/library/ff795613.aspx) 查詢運算子。

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies+site:contososailing.com&mkt=en-us HTTP/1.1
```

> [!NOTE]
> 不論是否有 [safeSearch](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#safesearch) 設定，使用 `site:` 運算子的查詢回應都可能包含成人內容。 請只在知曉網域上的內容時，才使用 `site:`。

下列範例示範如何從 Bing 在過去一週中探索的 ContosoSailing.com 取得小型影像。  

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies+site:contososailing.com&size=small&freshness=week&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```

## <a name="filtering-images"></a>篩選影像

 根據預設，影像搜尋 API 會傳回與查詢相關的所有影像。 如果您想要篩選 Bing 傳回的影像 (例如，只傳回具有透明背景的影像或特定大小的影像)，可使用下列查詢參數：

* [aspect](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#aspect) — 依照外觀比例篩選影像 (例如，標準或寬螢幕影像)
* [color](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#color) — 依照主色或黑白篩選影像
* [freshness](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#freshness) — 依照存留期篩選影響 (例如，Bing 在過去一週中探索的影像)
* [height](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#height)、[width](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#width) — 依照寬度和高度篩選影像
* [imageContent](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imagecontent) — 依照內容篩選影像 (例如，只顯示人臉的影像)
* [imageType](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#imagetype) — 依照類型篩選影像 (例如，美工圖案、動畫 GIF 或透明背景)
* [license](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#license) — 依照與網站相關聯的授權類型篩選影像
* [size](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#size) — 依照大小篩選影像，例如最大 200 x 200 像素的小型影像

若要從特定網域取得影像，請使用 [site:](http://msdn.microsoft.com/library/ff795613.aspx) 查詢運算子。

    > [!NOTE]
    > Responses to queries using the `site:` operator may include adult content regardless of the [safeSearch](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#safesearch) setting. Only use `site:` if you are aware of the content on the domain.

下列範例示範如何從 Bing 在過去一週中探索的 ContosoSailing.com 取得小型影像。  

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies+site:contososailing.com&size=small&freshness=week&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```

## <a name="bing-image-search-response-format"></a>Bing 影像搜尋的回應格式

從 Bing 傳回的回應訊息包含[影像](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#images)答案，其中包含 Azure 認知服務判斷為與查詢相關的影像清單。 清單中的每個[影像](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference#image)物件包含下列影像相關資訊：URL、影像大小、影像維度、影像編碼格式、影像縮圖的 URL 及縮圖的維度。

```json
{
    "name": "Rich Passage Sailing Dinghy",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=73118C8B4E3...",
    "thumbnailUrl": "https:\/\/tse1.mm.bing.net\/th?id=OIP.GNarK7m...",
    "datePublished": "2011-10-29T11:26:00",
    "contentUrl": "http:\/\/www.bing.com\/cr?IG=73118C8B4E3D4C3...",
    "hostPageUrl": "http:\/\/www.bing.com\/cr?IG=73118C8B4E3D4C3687...",
    "contentSize": "79239 B",
    "encodingFormat": "jpeg",
    "hostPageDisplayUrl": "en.contoso.org\/wiki\/File:Rich_Passage...",
    "width": 526,
    "height": 688,
    "thumbnail": {
        "width": 229,
        "height": 300
    },
    "imageInsightsToken": "ccid_GNarK7ma*mid_CCF85447ADA6...",
    "insightsSourcesSummary": {
        "shoppingSourcesCount": 0,
        "recipeSourcesCount": 0
    },
    "imageId": "CCF85447ADA6FFF9E96E7DF0B796F7A86E34593",
    "accentColor": "376094"
},
```

當您呼叫 Bing 影像搜尋 API 時，Bing 會傳回結果清單。 此清單是查詢相關結果總數的子集。 回應的 `totalEstimatedMatches` 欄位包含可以檢視的影像數目估計。 如需有關如何逐頁檢視其餘影像的詳細資訊，請參閱[逐頁檢視影像](../paging-images.md)。

## <a name="next-steps"></a>後續步驟

如果您從未試用過 Bing 影像搜尋 API，請透過[快速入門](../quickstarts/csharp.md)試用此功能。 如果您想要尋找更複雜的內容，請嘗試用於建立[單頁 Web 應用程式](../tutorial-bing-image-search-single-page-app.md)的教學課程。
