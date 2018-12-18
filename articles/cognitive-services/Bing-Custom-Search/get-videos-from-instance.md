---
title: 從您的自訂檢視取得影片 | Microsoft Docs
description: 使用 Bing 自訂搜尋從您的 Web 自訂檢視取得影片的概觀。
services: cognitive-services
author: swhite-msft
manager: ehansen
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: article
ms.date: 09/10/2018
ms.author: scottwhi
ms.openlocfilehash: 980081ae2f576aadd4ac31e29f0f77ddcb4e7f64
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978784"
---
# <a name="get-videos-from-your-custom-view"></a>從您的自訂檢視取得影片

Bing 自訂影片搜尋可讓您的自訂影片搜尋體驗更豐富。 與 Web 結果類似，自訂搜尋支援搜尋執行個體之網站清單中的影片。 您可以使用 Bing 的「自訂影片搜尋 API」，或透過「託管的 UI」功能來取得影片。 「託管的 UI」功能較容易使用且建議您使用，因為它只要需要一些時間就能讓您的搜尋體驗開始運作。 如需設定託管的 UI 以包含影片的相關資訊，請參閱[設定託管的 UI 體驗](hosted-ui.md)。

如果您想要進一步控制搜尋結果的顯示，您可以使用 Bing 的自訂影片搜尋 API。 因為呼叫該 API 就類似呼叫 Bing 影片搜尋 API，如需呼叫 API 的範例，請參閱 [Bing 影片搜尋](../Bing-Video-Search/search-the-web.md)。 但在您這樣做之前，建議您先了解[自訂影片搜尋 API 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-videos-api-v7-reference) \(英文\) 內容。 主要的差異在於支援的查詢參數 (您必須包含 customConfig 查詢參數) 以及接收您傳送之要求的端點。

<!--
## Next steps

[Call your custom view](search-your-custom-view.md)
-->