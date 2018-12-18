---
title: 範例：使用文字分析 REST API 分析情感
titleSuffix: Azure Cognitive Services
description: 了解如何使用文字分析 REST API 偵測情感。
services: cognitive-services
author: HeidiSteen
manager: cgronlun
ms.service: cognitive-services
ms.component: text-analytics
ms.topic: sample
ms.date: 09/12/2018
ms.author: heidist
ms.openlocfilehash: 981e663b6a93abed1da9c2765a1b43063c70ad43
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45605890"
---
# <a name="example-how-to-detect-sentiment-in-text-analytics"></a>範例：如何在文字分析中偵測情感

[情感分析 API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c9) \(英文\) 會評估文字輸入，並傳回每份文件的情感分數，範圍從 0 (負面) 到 1 (正面)。

此功能在偵測社交媒體、客戶評價及討論論壇中的正面和負面情感上非常實用。 內容是由您提供，模型和定型資料則由服務提供。

情感分析目前支援英文、德文、西班牙文及法文。 其他語言則為預覽狀態。 如需詳細資訊，請參閱[支援的語言](../text-analytics-supported-languages.md)。

## <a name="concepts"></a>概念

文字分析會使用機器學習分類演算法來產生介於 0 和 1 之間的情感分數。 接近 1 的分數表示正面情感，而接近 0 的分數則表示負面情感。 此模型是使用大量具有情感關聯的文字主體進行預先定型。 目前，您並無法提供自己的定型資料。 此模型會在文字分析期間使用數種技術的組合，包括文字處理、詞類分析、單字放置和單字關聯。 如需有關演算法的詳細資訊，請參閱[文字分析簡介](https://blogs.technet.microsoft.com/machinelearning/2015/04/08/introducing-text-analytics-in-the-azure-ml-marketplace/) \(英文\)。

情感分析會針對整份文件執行，而不會擷取對於文字中特定實體的情感。 在實務上，當文件包含一或兩個句子而非大型文字區塊時，會較容易提升評分的準確度。 在客觀評估階段，模型會判斷整份文件是客觀的，還是包含情感的。 主要為客觀的文件不會進展到情感偵測階段，因此分數為 .50，而且不會進一步處理。 針對繼續進行流程的文件，根據在文件中偵測到的情感程度而定，下一個階段將會產生高於或低於 .50 的分數。

## <a name="preparation"></a>準備工作

情感分析會在您為它提供較小區塊的文字來處理時，產生較高品質的結果。 這與關鍵片語擷取相反，後者在較大型文字區塊上的表現會更好。 若要從這兩個作業中取得最佳結果，請考慮據以重建輸入。

您必須具有此格式的 JSON 文件：識別碼、文字、語言

文件大小必須少於 5,000 個字元，而且您最多可以針對每個集合擁有 1,000 個項目 (識別碼)。 集合會在要求主體中提交。 以下是您可能提交來進行情感分析的內容範例。

```
    {
        "documents": [
            {
                "language": "en",
                "id": "1",
                "text": "We love this trail and make the trip every year. The views are breathtaking and well worth the hike!"
            },
            {
                "language": "en",
                "id": "2",
                "text": "Poorly marked trails! I thought we were goners. Worst hike ever."
            },
            {
                "language": "en",
                "id": "3",
                "text": "Everyone in my family liked the trail but thought it was too challenging for the less athletic among us. Not necessarily recommended for small children."
            },
            {
                "language": "en",
                "id": "4",
                "text": "It was foggy so we missed the spectacular views, but the trail was ok. Worth checking out if you are in the area."
            },                
            {
                "language": "en",
                "id": "5",
                "text": "This is my favorite trail. It has beautiful views and many places to stop and rest"
            }
        ]
    }
```

## <a name="step-1-structure-the-request"></a>步驟 1：建立要求結構

關於要求定義的詳細資料可以在[如何呼叫文字分析 API](text-analytics-how-to-call-api.md) 中找到。 為了方便起見，我們將重申下列各點：

+ 建立一個 **POST** 要求。 檢閱適用於此要求的 API 文件：[情感分析 API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c9) \(英文\)

+ 設定適用於關鍵片語擷取的 HTTP 端點。 它必須包括 `/sentiment` 資源：`https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/sentiment`

+ 設定要求標頭以包含適用於文字分析作業的存取金鑰。 如需詳細資訊，請參閱[如何尋找端點和存取金鑰](text-analytics-how-to-access-key.md)。

+ 在要求主體中，提供您準備用於此分析的 JSON 文件集合。

> [!Tip]
> 使用 [Postman](text-analytics-how-to-call-api.md) 或開啟[文件](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c9) \(英文\) 中的 **API 測試主控台**來建立要求結構，並將它 POST 到服務。

## <a name="step-2-post-the-request"></a>步驟 2：張貼要求

分析會在接收要求時執行。 服務每分鐘最多可接受 100 個要求。 每個要求最多可以是 1 MB。

請記得，服務是無狀態的。 您的帳戶中並不會儲存任何資料。 結果會在回應中立即傳回。


## <a name="step-3-view-results"></a>步驟 3：檢視結果

情感分析師會將文字明確地分類為正面或負面，指派範圍從 0 到 1 的分數。 接近 0.5 的值是中立或不確定的。 0.5 的分數表示中立。 當無法針對情感分析字串，或字串沒有任何情感時，分數一律為 0.5。 例如，如果您使用英文語言代碼傳入西班牙字串，分數將會是 0.5。

輸出會立即傳回。 您可以將結果串流處理到可接受 JSON 的應用程式，或將輸出儲存到本機系統上的檔案，然後將它匯入能讓您排序、搜尋和操作資料的應用程式。

下列範例顯示此文章中文件集合的回應。

```
{
    "documents": [
        {
            "score": 0.9999237060546875,
            "id": "1"
        },
        {
            "score": 0.0000540316104888916,
            "id": "2"
        },
        {
            "score": 0.99990355968475342,
            "id": "3"
        },
        {
            "score": 0.980544924736023,
            "id": "4"
        },
        {
            "score": 0.99996328353881836,
            "id": "5"
        }
    ],
    "errors": []
}
```

## <a name="summary"></a>總結

在此文章中，您已了解使用認知服務中的文字分析進行情感分析的概念和工作流程。 摘要說明：

+ [情感分析 API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c9) \(英文\) 僅針對特定語言提供。
+ 要求主體中的 JSON 文件包含識別碼、文字和語言代碼。
+ 使用對您訂用帳戶有效的個人化[存取金鑰和端點](text-analytics-how-to-access-key.md)，將要求 POST 到 `/sentiment` 端點。
+ 回應輸出 (其包含針對每個文件識別碼的情感分數) 可以串流處理到任何可接受 JSON 的應用程式，包括 Excel 和 Power BI 等。

## <a name="see-also"></a>另請參閱 

 [文字分析概觀](../overview.md)  
 [常見問題集 (FAQ)](../text-analytics-resource-faq.md)</br>
 [文字分析產品頁面](//go.microsoft.com/fwlink/?LinkID=759712) 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [擷取關鍵片語](text-analytics-how-to-keyword-extraction.md)
