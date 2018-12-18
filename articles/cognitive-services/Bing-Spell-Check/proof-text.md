---
title: 什麼是 Bing 拼字檢查 API？
titlesuffix: Azure Cognitive Services
description: Bing 拼字檢查 API 會使用機器學習服務和統計機器翻譯進行內容的拼字檢查。
services: cognitive-services
author: noellelacharite
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-spell-check
ms.topic: overview
ms.date: 05/03/2018
ms.author: nolachar
ms.openlocfilehash: 4caa05ffa96dbc15922fed85edfdefdb68ead68b
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49361706"
---
# <a name="what-is-bing-spell-check-api"></a>什麼是 Bing 拼字檢查 API？

Bing 拼字檢查 API 可讓您執行內容的文法與拼字檢查。

一般的拼字檢查工具與 Bing 的第三代拼字檢查工具有何不同？ 現行拼字檢查工具的執行基礎，是根據以字典為基礎、且會定期更新和擴充的規則集，來驗證拼字和文法。 換句話說，此類拼字檢查工具的強度完全端賴於作為基礎的字典，以及負責維護字典的編輯人員。 您可以從 Microsoft Word 和其他文書處理程式了解這種類型的拼字檢查工具。

相對地，Bing 所開發的 Web 架構拼字檢查工具則運用了機器學習服務和統計機器翻譯，以動態方式定型持續進化且內容相關度高的演算法。 此拼字檢查工具以大規模的 Web 搜尋和文件主體為基礎。

此拼字檢查工具可處理任何文字處理案例：

- 辨識俚語與非正式用語
- 辨識內容中的一般名稱錯誤
- 利用單一旗標更正斷字問題
- 能夠更正內容中的同音異義詞，以及其他難以發現的錯誤
- 支援新興的品牌、數位娛樂和流行用語
- 發音相同、但意義與拼法不同的文字。例如 “see” 和 “sea”。

## <a name="spell-check-modes"></a>拼字檢查模式

此 API 支援兩種校訂模式：`Proof` 和 `Spell`。  請看看[此處](https://azure.microsoft.com/services/cognitive-services/spell-check/)的範例。
### <a name="proof---for-documents-scenario"></a>Proof - 適用於文件
預設模式為 `Proof`。 `Proof` 拼字模式可提供最全面的檢查，並可新增大小寫、基本標點符號及其他功能，以利建立文件。 但此模式僅適用於 en-US (美式英文)、es-ES (西班牙文)、pt-BR (葡萄牙文) 市場 (附註：西班牙文和葡萄牙文僅限於 Beta 版)。 對於其他所有的市場，請將 mode 查詢參數設定為 Spell。 
<br /><br/>**附註：** 查詢文字的長度若超過 4096，則會先截斷為 4096 個字元再進行處理。 
### <a name="spell----for-web-searchesqueries-scenario"></a>Spell - 適用於 Web 搜尋/查詢
`Spell` 會較積極地處理而傳回較佳的搜尋結果。 `Spell` 模式可找出大部分的拼字錯誤，但有可能找不到 `Proof` 可挑出的某些文法錯誤，例如大小寫和重複的文字。
<br /></br>**附註：** 支援的最大查詢長度如下所示。 如果查詢超出限制，結果將會顯示未變更的查詢。
<ul><li>en、de、es、fr、pl、pt、sv、ru、nl、nb、tr-tr、it、zh、ko 等語言代碼限為 130 個字元。 </li>
<li>其他語言限為 65 個字元</li></ul>

## <a name="market-setting"></a>市場設定
市場必須指定在要求 URL 的查詢參數中，否則拼字檢查將會根據 IP 位址來認定預設市場。


## <a name="post-vs-get"></a>POST 與GET

此 API 支援 HTTP POST 或 HTTP GET。 應使用何者，取決於您要校訂的文字長度。 如果字串一律少於 1,500 個字元，您應使用 GET。 但若您要支援高達 10,000 個字元的字串，則應使用 POST。 文字字串可包含任何有效的 UTF-8 字元。

下列範例顯示要對文字字串檢查拼字和文法的 POST 要求。 為求完整性，此範例加入了 [mode](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v7-reference#mode) 查詢參數 (原本會排除此參數，因為 `mode` 預設為 Proof)。 [text](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v7-reference#text) 查詢參數包含要校訂的字串。
  
```  
POST https://api.cognitive.microsoft.com/bing/v7.0/spellcheck?mode=proof&mkt=en-us HTTP/1.1  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 47  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
 
text=when+its+your+turn+turn,+john,+come+runing  
``` 

如果您使用 HTTP GET，您應在 URL 的查詢字串中加入 `text` 查詢參數
  
以下顯示前一個要求的回應。 回應包含 [SpellCheck](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v7-reference#spellcheck) 物件。 
  
```  
{  
    "_type" : "SpellCheck",  
    "flaggedTokens" : [{  
        "offset" : 5,  
        "token" : "its",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "it's",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 25,  
        "token" : "john",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "John",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 19,  
        "token" : "turn",  
        "type" : "RepeatedToken",  
        "suggestions" : [{  
            "suggestion" : "",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 35,  
        "token" : "runing",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "running",  
            "score" : 1  
        }]  
    }]  
}  
```  
  
[FlaggedTokens](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v7-reference#flaggedtokens) 欄位列出此 API 在[文字](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v7-reference#text)字串中找到的拼字和文法錯誤。 `token` 欄位包含要被取代的文字。 您可以在 `offset` 欄位中使用以零為起始的位移，尋找 `text` 字串中的權杖。 然後，您應將該位置的文字取代為 `suggestion` 欄位中的文字。 

如果 `type` 欄位為 RepeatedToken，您仍應將權杖取代為 `suggestion`，但您也可能需要移除尾端的空格。

## <a name="throttling-requests"></a>節流要求

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../includes/cognitive-services-bing-throttling-requests.md)]

## <a name="next-steps"></a>後續步驟

若要快速開始使用您的第一個要求，請參閱[建立您的第一個要求](quickstarts/csharp.md)。

熟悉 [Bing 拼字檢查 API 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v7-reference)。 此參考包含端點、標頭、您用來要求搜尋結果的查詢參數，以及回應物件定義的清單。 

請務必閱讀 [Bing 使用和顯示需求](./useanddisplayrequirements.md)，以免違反任何有關於使用結果的規則。
