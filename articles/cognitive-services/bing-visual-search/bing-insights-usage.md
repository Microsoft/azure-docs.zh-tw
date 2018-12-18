---
title: Bing 深入解析的範例 - Bing 圖像式搜尋
titleSuffix: Azure Cognitive Services
description: 顯示 Bing.com 顯示影像深入解析的範例。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-visual-search
ms.topic: conceptual
ms.date: 04/17/2018
ms.author: scottwhi
ms.openlocfilehash: 40b9685c56dd494e82c05bff9ed75f425a3006ff
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48886622"
---
# <a name="examples-of-bing-insights-usage"></a>Bing 深入解析使用方式的範例

本節包含 Bing 如何在 Bing.com 上顯示深入解析的範例。

## <a name="pagesincluding-insight-example"></a>PagesIncluding 深入解析範例

以下顯示 Bing 如何顯示包含影像的網頁。 範例會顯示第一個網頁的連結，而且可讓使用者展開和摺疊包含影像的其他網頁清單。

![展開的頁面包括](./media/pages-including.PNG)


## <a name="shoppingsources-insight-example"></a>ShoppingSources 深入解析範例

以下顯示 Bing 對於影像顯示的產品如何顯示購物來源。

![購物來源](./media/shopping-sources.PNG)


## <a name="visualsearch-insight-example"></a>VisualSearch 深入解析範例

以下顯示 Bing 如何顯示在視覺效果上類似的影像 (請參閱範例中的**相關影像**)。

![在視覺效果上類似的影像](./media/similar-images.PNG)

## <a name="recipes-insight-example"></a>Recipes 深入解析範例

以下顯示 Bing 對於影像顯示的食物如何顯示食譜。 範例可讓使用者知道有可用的食譜。

![食譜和頁面包括](./media/recipes-pages-including.PNG)

 而且，使用者展開清單時，會提供食譜的連結。

![展開的食譜頁面包括](./media/expanded-recipes-pages-including.PNG)


## <a name="relatedsearches-insight-example"></a>RelatedSearches 深入解析範例

以下顯示 Bing 如何顯示其他人對於影像進行的相關搜尋。 如果使用者按一下影像，使用者會進入該相關查詢的 Bing.com/images 搜尋結果頁面。

![影像的相關搜尋](./media/bordered-related-searches.PNG)


## <a name="entity-insight-example"></a>Entity 深入解析範例

以下顯示 Bing 如何顯示影像顯示的實體 (人、地、事) 相關的資訊。 如果使用者按一下實體連結，使用者會進入實體的 Bing.com 搜尋結果頁面。

![影像中顯示的實體](./media/entity.PNG)


## <a name="displaying-other-insights-that-the-user-might-explore"></a>顯示使用者可以探索的其他深入解析

以下顯示 Bing 如何顯示使用者可以探索的影像有關的其他資訊。

![探索影像的其他深入解析](./media/apple-pie-more-tags.PNG)


## <a name="bounding-boxes-and-hot-spots"></a>週框方塊和作用點

非預設標記包含識別標籤套用的影像之中關切區域的週框方塊。 如果週框方塊並未識別整個影像，可使用週框方塊在影像上建立作用點。 使用者可以按一下作用點取得在作用點 (或矩形) 之下找到的內容有關的資訊。 例如，如果影像是時尚影像，結果可能包含影像所示配件 (例如錢包、珠寶、圍巾等等) 的標記 (以及週框方塊)。下列範例顯示影像所示太陽眼鏡的作用點矩形。

![週框方塊和作用點](./media/click-to-search.PNG)



## <a name="next-steps"></a>後續步驟

若要查看這些範例背後的 JSON，請參閱[預設深入解析](default-insights-tag.md)和 [JSON 回應](overview.md#the-response)。

若要快速開始使用您的第一個要求，請參閱 [C#](quickstarts/csharp.md) | [JAVA](quickstarts/java.md) | [node.js](quickstarts/nodejs.md) | [Python](quickstarts/python.md)





