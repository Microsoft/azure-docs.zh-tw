---
title: 範例：使用文字分析 REST API 進行語言偵測
titleSuffix: Azure Cognitive Services
description: 了解如何使用文字分析 REST API 偵測語言。
services: cognitive-services
author: HeidiSteen
manager: cgronlun
ms.service: cognitive-services
ms.component: text-analytics
ms.topic: sample
ms.date: 09/12/2018
ms.author: heidist
ms.openlocfilehash: fa71e4ce2e5cb5967bb583c7314072830de08051
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45604547"
---
# <a name="example-how-to-detect-language-in-text-analytics"></a>範例：如何在文字分析中偵測語言

[語言偵測 API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c7) \(英文\) 會針對每份文件評估文字輸入，並傳回語言識別項，其中含有指出分析強度的分數。 文字分析最多可辨識 120 種語言。

此功能對於收集任意文字且語言為未知的內容存放區而言非常實用。 您可以剖析此分析的結果，以判斷輸入文件中使用的是哪一種語言。 回應也會傳回分數來反映對模型的信賴度 (介於 0 和 1 之間的值)。

## <a name="preparation"></a>準備工作

您必須具有此格式的 JSON 文件：識別碼、文字

文件大小必須少於 5,000 個字元，而且您最多可以針對每個集合擁有 1,000 個項目 (識別碼)。 集合會在要求主體中提交。 以下是您可能提交來進行語言偵測的內容範例。

   ```
    {
        "documents": [
            {
                "id": "1",
                "text": "This document is in English."
            },
            {
                "id": "2",
                "text": "Este documento está en inglés."
            },
            {
                "id": "3",
                "text": "Ce document est en anglais."
            },
            {
                "id": "4",
                "text": "本文件为英文"
            },                
            {
                "id": "5",
                "text": "Этот документ находится на английском языке."
            }
        ]
    }
```

## <a name="step-1-structure-the-request"></a>步驟 1：建立要求結構

關於要求定義的詳細資料可以在[如何呼叫文字分析 API](text-analytics-how-to-call-api.md) 中找到。 為了方便起見，我們將重申下列各點：

+ 建立一個 **POST** 要求。 檢閱適用於此要求的 API 文件：[語言偵測 API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c7) \(英文\)

+ 設定適用於語言偵測的 HTTP 端點。 它必須包括 `/languages` 資源：`https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/languages`

+ 設定要求標頭以包含適用於文字分析作業的存取金鑰。 如需詳細資訊，請參閱[如何尋找端點和存取金鑰](text-analytics-how-to-access-key.md)。

+ 在要求本文中，提供您準備用於此分析的 JSON 文件集合

> [!Tip]
> 使用 [Postman](text-analytics-how-to-call-api.md) 或開啟[文件](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c7) \(英文\) 中的 **API 測試主控台**來建立要求結構，並將它 POST 到服務。

## <a name="step-2-post-the-request"></a>步驟 2：張貼要求

分析會在接收要求時執行。 服務每分鐘最多可接受 100 個要求。 每個要求最多可以是 1 MB。

請記得，服務是無狀態的。 您的帳戶中並不會儲存任何資料。 結果會在回應中立即傳回。


## <a name="step-3-view-results"></a>步驟 3：檢視結果

所有的 POST 要求都會傳回 JSON 格式的回應，具有識別碼和偵測到的屬性。

輸出會立即傳回。 您可以將結果串流處理到可接受 JSON 的應用程式，或將輸出儲存到本機系統上的檔案，然後將它匯入能讓您排序、搜尋和操作資料的應用程式。

範例要求的結果看起來應該類似下列 JSON。 請注意，它是一份具有多個項目的文件。 輸出的語言是英文。 語言識別項包括易記名稱和 [ISO 639-1](https://www.iso.org/standard/22109.html) \(英文\) 格式的語言代碼。

1.0 的正面分數表示最高的分析信賴等級。



```
{
    "documents": [
        {
            "id": "1",
            "detectedLanguages": [
                {
                    "name": "English",
                    "iso6391Name": "en",
                    "score": 1
                }
            ]
        },
        {
            "id": "2",
            "detectedLanguages": [
                {
                    "name": "Spanish",
                    "iso6391Name": "es",
                    "score": 1
                }
            ]
        },
        {
            "id": "3",
            "detectedLanguages": [
                {
                    "name": "French",
                    "iso6391Name": "fr",
                    "score": 1
                }
            ]
        },
        {
            "id": "4",
            "detectedLanguages": [
                {
                    "name": "Chinese_Simplified",
                    "iso6391Name": "zh_chs",
                    "score": 1
                }
            ]
        },
        {
            "id": "5",
            "detectedLanguages": [
                {
                    "name": "Russian",
                    "iso6391Name": "ru",
                    "score": 1
                }
            ]
        }
    ],
```

### <a name="ambiguous-content"></a>模稜兩可的內容

如果分析器無法剖析輸入 (例如，假設您提交了只包含阿拉伯數字的文字區塊)，則會傳回 `(Unknown)`。

```
    {
      "id": "5",
      "detectedLanguages": [
        {
          "name": "(Unknown)",
          "iso6391Name": "(Unknown)",
          "score": "NaN"
        }
      ]
```
### <a name="mixed-language-content"></a>混合式語言內容

相同文件中的混合式語言內容會傳回佔據最多內容的語言，但其正面評等將會較低，以反映該評量的臨界強度。 在下列範例中，輸入會混合英文、西班牙文和法文。 分析器會計算每個區段中的字元來判斷出主要的語言。

**輸入**

```
{
  "documents": [
    {
      "id": "1",
      "text": "Hello, I would like to take a class at your University. ¿Se ofrecen clases en español? Es mi primera lengua y más fácil para escribir. Que diriez-vous des cours en français?"
    }
  ]
}
```

**輸出**

結果輸出會顯示主要的語言，以及低於 1.0 的分數，表示較低的信賴等級。

```
{
  "documents": [
    {
      "id": "1",
      "detectedLanguages": [
        {
          "name": "Spanish",
          "iso6391Name": "es",
          "score": 0.9375
        }
      ]
    }
  ],
  "errors": []
}
```

## <a name="summary"></a>總結

在此文章中，您已了解使用認知服務中的文字分析進行語言偵測的概念和工作流程。 以下是先前所說明和示範之重點的快速提醒：

+ [語言偵測 API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c7) 可供 120 種語言使用。
+ 要求主體中的 JSON 文件包含識別碼和文字。
+ 使用對您訂用帳戶有效的個人化[存取金鑰和端點](text-analytics-how-to-access-key.md)，將要求 POST 到 `/languages` 端點。
+ 回應輸出 (其包含針對每個文件識別碼的語言識別項) 可以串流處理到任何可接受 JSON 的應用程式，包括 Excel 和 Power BI 等。

## <a name="see-also"></a>另請參閱 

 [文字分析概觀](../overview.md)  
 [常見問題集 (FAQ)](../text-analytics-resource-faq.md)</br>
 [文字分析產品頁面](//go.microsoft.com/fwlink/?LinkID=759712) 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [分析情感](text-analytics-how-to-sentiment-analysis.md)
