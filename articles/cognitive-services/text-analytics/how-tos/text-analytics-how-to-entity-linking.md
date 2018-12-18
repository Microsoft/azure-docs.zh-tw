---
title: 如何在文字分析 REST API 中使用實體連結 (Azure 上的 Microsoft 認知服務) | Microsoft Docs
description: 在本逐步解說教學課程中，您將了解如何使用 Azure 上 Microsoft 認知服務中的文字分析 REST API 來識別和解析實體。
services: cognitive-services
author: ashmaka
manager: cgronlun
ms.service: cognitive-services
ms.technology: text-analytics
ms.topic: article
ms.date: 5/02/2018
ms.author: ashmaka
ms.openlocfilehash: 55bec1a0223b70749a97a30e2da92ef15128038c
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368730"
---
# <a name="how-to-identify-linked-entities-in-text-analytics-preview"></a>如何在文字分析 (預覽) 中識別連結實體

[實體連結 API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/5ac4251d5b4ccd1554da7634) 採用非結構化文字，並針對每個 JSON 文件傳回明確的實體清單，其中包含 Web (維基百科和 Bing) 上詳細資訊的連結。 

## <a name="entity-linking-vs-named-entity-recognition"></a>實體連結與具名實體辨識

在自然語言的處理中，實體連結和具名實體辨識 (NER) 的概念很容易讓人混淆。 在預覽版文字分析的 `entities` 端點中，僅支援連結實體。

連結實體可識別及區分文字中找到的實體身分識別 (例如，判斷 "Mars" 是用來指星體或指羅馬的戰神)。 此程序需具備可讓辨識項目與之連結的知識庫 - `entities` 端點文字分析即是使用維基百科作為知識庫。

### <a name="language-support"></a>語言支援

若以多種語言使用連結實體，則需使用每一種語言的對應知識庫。 對於文字分析中的實體連結，這表示 `entities` 端點支援的每一種語言會連結至該語言的對應維基百科語料庫。 由於語料庫的大小會因語言不同而有所差異，因此我們預期實體連結功能的召回也會有所不同。


## <a name="preparation"></a>準備工作

您必須具有此格式的 JSON 文件：識別碼、文字、語言

如需目前支援的語言，請參閱[這份清單](../text-analytics-supported-languages.md)。

文件大小必須少於 5,000 個字元，而且您最多可以針對每個集合擁有 1,000 個項目 (識別碼)。 集合會在要求本文中提交。 下列範例是您可能會提交至實體連結末端的內容說明。

```
{"documents": [{"id": "1",
                "language": "en",
                "text": "I really enjoy the new XBox One S. It has a clean look, it has 4K/HDR resolution and it is affordable."
                },
               {"id": "2",
                "language": "en",
                "text": "The Seattle Seahawks won the Super Bowl in 2014."
                }
               ]
}
```    
    
## <a name="step-1-structure-the-request"></a>步驟 1：建立要求結構

關於要求定義的詳細資料可以在[如何呼叫文字分析 API](text-analytics-how-to-call-api.md) 中找到。 為了方便起見，我們將重申下列各點：

+ 建立一個 **POST** 要求。 檢閱適用於此要求的 API 文件：[實體連結 API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/5ac4251d5b4ccd1554da7634) \(英文\)

+ 設定適用於關鍵片語擷取的 HTTP 端點。 它必須包括 `/entities` 資源：`https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/entities`

+ 設定要求標頭以包含適用於文字分析作業的存取金鑰。 如需詳細資訊，請參閱[如何尋找端點和存取金鑰](text-analytics-how-to-access-key.md)。

+ 在要求本文中，提供您準備用於此分析的 JSON 文件集合

> [!Tip]
> 使用 [Postman](text-analytics-how-to-call-api.md) 或開啟[文件](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/5ac4251d5b4ccd1554da7634) \(英文\) 中的 **API 測試主控台**來建立要求結構，並將它 POST 到服務。

## <a name="step-2-post-the-request"></a>步驟 2：張貼要求

分析會在接收要求時執行。 服務每分鐘最多可接受 100 個要求。 每個要求最多可以是 1 MB。

請記得，服務是無狀態的。 您的帳戶中並不會儲存任何資料。 結果會在回應中立即傳回。

## <a name="step-3-view-results"></a>步驟 3：檢視結果

所有的 POST 要求都會傳回 JSON 格式的回應，具有識別碼和偵測到的屬性。

輸出會立即傳回。 您可以將結果串流處理到可接受 JSON 的應用程式，或將輸出儲存到本機系統上的檔案，然後將它匯入能讓您排序、搜尋和操作資料的應用程式。

實體連結輸出的範例會隨即顯示：

```
{
    "documents": [
        {
            "id": "1",
            "entities": [
                {
                    "name": "Xbox One",
                    "matches": [
                        {
                            "text": "XBox One",
                            "offset": 23,
                            "length": 8
                        }
                    ],
                    "wikipediaLanguage": "en",
                    "wikipediaId": "Xbox One",
                    "wikipediaUrl": "https://en.wikipedia.org/wiki/Xbox_One",
                    "bingId": "446bb4df-4999-4243-84c0-74e0f6c60e75"
                },
                {
                    "name": "Ultra-high-definition television",
                    "matches": [
                        {
                            "text": "4K",
                            "offset": 63,
                            "length": 2
                        }
                    ],
                    "wikipediaLanguage": "en",
                    "wikipediaId": "Ultra-high-definition television",
                    "wikipediaUrl": "https://en.wikipedia.org/wiki/Ultra-high-definition_television",
                    "bingId": "7ee02026-b6ec-878b-f4de-f0bc7b0ab8c4"
                }
            ]
        },
        {
            "id": "2",
            "entities": [
                {
                    "name": "2013 Seattle Seahawks season",
                    "matches": [
                        {
                            "text": "Seattle Seahawks",
                            "offset": 4,
                            "length": 16
                        }
                    ],
                    "wikipediaLanguage": "en",
                    "wikipediaId": "2013 Seattle Seahawks season",
                    "wikipediaUrl": "https://en.wikipedia.org/wiki/2013_Seattle_Seahawks_season",
                    "bingId": "eb637865-4722-4eca-be9e-0ac0c376d361"
                }
            ]
        }
    ],
    "errors": []
}
```

一旦可以使用時，回應就會針對每個所偵測到的實體包括維基百科識別碼、維基百科 URL 和 Bing 識別碼。 透過這些項目，您就能使用連結實體的相關資訊來進一步加強應用程式。


## <a name="summary"></a>總結

在本文中，您已了解在認知服務中使用文字分析的實體連結概念和工作流程。 摘要說明：

+ [實體連結 API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/5ac4251d5b4ccd1554da7634) \(英文\) 僅針對特定語言提供。
+ 要求本文中的 JSON 文件包含識別碼、文字和語言代碼。
+ 使用對您訂用帳戶有效的個人化[存取金鑰和端點](text-analytics-how-to-access-key.md)，將要求 POST 到 `/entities` 端點。
+ 由連結實體 (包含每個文件識別碼的信賴分數、位移和網頁連結) 組成的回應輸出可用於任何應用程式

## <a name="see-also"></a>另請參閱 

 [文字分析概觀](../overview.md)  
 [常見問題集 (FAQ)](../text-analytics-resource-faq.md)</br>
 [文字分析產品頁面](//go.microsoft.com/fwlink/?LinkID=759712) 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [文字分析 API](//westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c6)
