---
title: 什麼是 Bing 自訂搜尋？
titlesuffix: Azure Cognitive Services
description: 提供 Bing 自訂搜尋的高階概觀。
services: cognitive-services
author: brapel
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: overview
ms.date: 09/29/2017
ms.author: v-brapel
ms.openlocfilehash: f2946918f0c1a7a516788989042825e8f49d7b0b
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49318719"
---
# <a name="what-is-bing-custom-search"></a>什麼是 Bing 自訂搜尋？

Bing 自訂搜尋可讓您針對感興趣的主題，建立量身訂做的搜尋經驗。 例如，如果您的網站提供搜尋體驗，可以指定 Bing 搜尋的網域、網站及網頁。 您的使用者會看到針對他們感興趣的內容量身訂做的搜尋結果，而不需要逐頁查看包含不相關內容的搜尋結果。

若要建立自訂的網頁檢視，請使用 Bing 自訂搜尋[入口網站](https://customsearch.ai)。 入口網站可讓您建立自訂搜尋執行個體，指定要 Bing 搜尋的網域、網站和網頁，以及不想要搜尋的網站。 除了指定您知道內容的 URL 之外，也可以使用入口網站尋找您要加入的相關內容。

如果使用者輸入特定的搜尋字詞，入口網站也可讓您將特定網頁釘選到搜尋結果的頂端。 

定義您的執行個體之後，可以呼叫自訂搜尋 API，將自訂的搜尋整合到網站、傳統型應用程式或行動裝置應用程式。 如果您有網頁型的網站或應用程式，則可以讓託管的 UI 來呈現搜尋介面。

下圖顯示自訂搜尋整合的簡易性。

![圖片替代文字](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/cognitive-services/Bing-Custom-Search/media/BCS-Overview.png "Bing 自訂搜尋如何運作。")

## <a name="adding-custom-search-box-suggestions"></a>新增自訂搜尋方塊建議

您可以運用自訂搜尋方塊建議提供更豐富的自訂搜尋體驗。 此功能可讓您提供與搜尋體驗有關的自訂搜尋建議。 當使用者在搜尋方塊中輸入時，下拉式清單中會根據使用者輸入的部分查詢字串包含建議的查詢字串。 您可以指定只傳回您自訂的建議，或者也包含 Bing 的建議。 [閱讀更多](define-custom-suggestions.md)。

## <a name="adding-custom-image-search-experience"></a>新增自訂影像搜尋體驗

您可以運用影像提供更豐富的自訂搜尋體驗。 與 Web 結果類似，自訂搜尋支援搜尋執行個體之網站清單中的影像。 [閱讀更多](get-images-from-instance.md)。

## <a name="adding-custom-video-search-experience"></a>新增自訂影片搜尋體驗

您可以運用影片提供更豐富的自訂搜尋體驗。 與 Web 結果類似，自訂搜尋支援搜尋執行個體之網站清單中的影片。 [閱讀更多](get-videos-from-instance.md)。

## <a name="sharing-your-custom-search-instance-with-others"></a>與他人共用自訂搜尋執行個體

將執行個體分享給小組成員，輕鬆就能共同編輯和測試。 [閱讀更多](share-your-custom-search.md)。

## <a name="next-steps"></a>後續步驟

若要快速開始，請參閱[建立您的第一個 Bing 自訂搜尋執行個體](quick-start.md)。

如需有關自訂搜尋執行個體的詳細資，請參閱[定義自訂搜尋執行個體](define-your-custom-view.md)。

讓自己熟悉每個自訂搜尋端點的內容。 此參考包含端點、標頭，以及您用來要求搜尋結果的查詢參數。 它也包含回應物件的定義。

- [自訂搜尋 API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-search-api-v7-reference)
- [自訂影像 API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-images-api-v7-reference)
- [自訂影片 API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-videos-api-v7-reference)
- [自訂自動建議 API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-autosuggest-api-v7-reference)


務必閱讀 [Bing 使用和顯示需求](./use-and-display-requirements.md)，您才不會違反任何有關使用搜尋結果的規則。
