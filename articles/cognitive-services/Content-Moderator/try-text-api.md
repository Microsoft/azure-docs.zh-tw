---
title: 使用文字仲裁 API 來仲裁文字 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 使用線上主控台中的「文字仲裁 API」來試用文字仲裁。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 08/05/2017
ms.author: sajagtap
ms.openlocfilehash: 95fd5507287a9294f4fca6af9cc5f01f0ea9fe1c
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219782"
---
# <a name="moderate-text-from-the-api-console"></a>從 API 主控台仲裁文字

您可以使用 Azure Content Moderator 中的[文字仲裁 API](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f) 來掃描文字內容。 此作業會掃描您的內容中是否有粗話，並將內容與自訂和共用的黑名單做比較。


## <a name="get-your-api-key"></a>取得 API 金鑰
您需要有訂用帳戶金鑰，才能在線上主控台中試用 API。 這位於 [設定] 索引標籤的 [Ocp-Apim-Subscription-Key] 方塊中。 如需詳細資訊，請參閱[概觀](overview.md)。

## <a name="navigate-to-the-api-reference"></a>瀏覽至 API 參考
移至[文字仲裁 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66f)。 

  [Text - Screen] \(文字 - 過濾\) 頁面隨即開啟。

## <a name="open-the-api-console"></a>開啟 API 主控台
針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。 

  ![[Text - Screen] \(文字 - 過濾\) 頁面區域選取項目](images/test-drive-region.png)

  [Text - Screen] \(文字 - 過濾\) API 主控台隨即開啟。

## <a name="select-the-inputs"></a>選取輸入

### <a name="parameters"></a>參數
選取您想要在文字過濾中使用的查詢參數。 就此範例而言，針對 **language**，請使用預設值。 您也可以將其保留空白，因為此作業會在其執行過程中自動偵測可能的語言。

> [!NOTE]
> 針對 **language** 參數，請指派 `eng` 或將其保留空白，以查看電腦輔助**分類** 回應 (預覽版功能)。 **此功能僅支援英文**。
>
> 針對**粗話字詞**偵測，請使用本文中所列支援語言的 [ISO 639-3 代碼](http://www-01.sil.org/iso639-3/codes.asp)或將其保留空白。

針對 **autocorrect** **PII**及 **classify (預覽版)**，請選取 **true**。 將 [ListId] 欄位保留空白。

  ![[Text - Screen] \(文字 - 過濾\) 主控台查詢參數](images/text-api-console-inputs.PNG)

### <a name="content-type"></a>內容類型
針對 **Content-Type**，選取您想要過濾的內容類型。 就此範例而言，請使用預設的 **text/plain** 內容類型。 在 [Ocp-Apim-Subscription-Key] 中，輸入您的訂用帳戶金鑰。

### <a name="sample-text-to-scan"></a>要掃描的範例文字
在 [Request body] \(要求本文\) 方塊中，輸入一些文字。 以下範例示範文字中刻意安排的錯字。

> [!NOTE]
> 以下範例文字中的無效社會安全號碼是刻意安排的。 目的是要傳達範例輸入和輸出格式。

```
    Is this a grabage or crap email abcdef@abcd.com, phone: 6657789887, IP: 255.255.255.255, 1 Microsoft Way, Redmond, WA 98052.
    These are all UK phone numbers, the last two being Microsoft UK support numbers: +44 870 608 4000 or 0344 800 2400 or 0800 820 3300.
    Also, 999-99-9999 looks like a social security number (SSN).
```

### <a name="text-classification-feature"></a>文字分類功能

在以下範例中，您會看到 Content Moderator 的電腦輔助文字分類回應。 它可協助偵測可能不想要的內容。 所標幟的內容可能是依據上下文而被視為不當的內容。 除了傳達每個類別的可能性之外，它也可能建議對內容進行人工審核。 此功能使用定型模型來識別可能的濫用、毀謗性或歧視性語言。 這包括要供審核的俚語、縮寫單字、冒犯性及刻意拼錯的單字。 

#### <a name="explanation"></a>說明

- `Category1` 代表可能有在特定情況下被視為明顯色情或成人內容的語言存在。
- `Category2` 代表可能有在特定情況下被視為具性暗示或成人內容的語言存在。
- `Category3` 代表可能有在特定情況下被視為具冒犯性的語言存在。
- `Score` 介於 0 到 1 之間。 分數越高，模型預測為適用該類別的可能性就越高。 此預覽版功能倚賴統計模型，而不是手動編碼的結果。 建議您使用自己的內容進行測試，以判斷每個類別如何符合您的需求。
- `ReviewRecommended` 會是 true 或 false，視內部分數閾值而定。 客戶應該評估是要使用此值，還是根據其內容原則決定自訂閾值。

### <a name="analyze-the-response"></a>分析回應
以下回應顯示來自 API 的各種深入解析。 它包含可能的粗話、PII、分類 (預覽版) 及自動校正後的版本。

> [!NOTE]
> 電腦輔助「分類」功能目前為預覽版且僅支援英文。

```
{
    "OriginalText": "Is this a grabage or crap email abcdef@abcd.com, phone: 6657789887, IP: 255.255.255.255, 1 Microsoft Way, Redmond, WA 98052.\r\nThese are all UK phone numbers, the last two being Microsoft UK support numbers: +44 870 608 4000 or 0344 800 2400 or 0800 820 3300.\r\nAlso, 544-56-7788 looks like a social security number (SSN).",
    "NormalizedText": "Is this a grabage or crap email abcdef@ abcd. com, phone: 6657789887, IP: 255. 255. 255. 255, 1 Microsoft Way, Redmond, WA 98052. \r\nThese are all UK phone numbers, the last two being Microsoft UK support numbers: +44 870 608 4000 or 0344 800 2400 or 0800 820 3300. \r\nAlso, 544- 56- 7788 looks like a social security number ( SSN) .",
"Misrepresentation": null,
    "PII": {
            "Email": [{
                "Detected": "abcdef@abcd.com",
                "SubType": "Regular",
                "Text": "abcdef@abcd.com",
                "Index": 32
                }],
            "IPA": [{
                "SubType": "IPV4",
                "Text": "255.255.255.255",
                "Index": 72
                }],
            "Phone": [{
                "CountryCode": "US",
                "Text": "6657789887",
                "Index": 56
                }, {
                "CountryCode": "US",
                "Text": "870 608 4000",
                "Index": 211
                }, {
                "CountryCode": "UK",
                "Text": "+44 870 608 4000",
                "Index": 207
                }, {
                "CountryCode": "UK",
                "Text": "0344 800 2400",
                "Index": 227
                }, {
                "CountryCode": "UK",
                "Text": "0800 820 3300",
                "Index": 244
            }],
         "Address": [{
                 "Text": "1 Microsoft Way, Redmond, WA 98052",
                "Index": 89
            }],
            "SSN": [{
                "Text": "999999999",
                "Index": 56
            }, {
                "Text": "999-99-9999",
                "Index": 266
            }]
        },
    "Classification": {
        "ReviewRecommended": true,
        "Category1": {
            "Score": 1.5113095059859916E-06
        },
        "Category2": {
            "Score": 0.12747249007225037
        },
        "Category3": {
            "Score": 0.98799997568130493
        }
        },
    "Language": "eng",
    "Terms": [{
            "Index": 21,
            "OriginalIndex": 21,
            "ListId": 0,
         "Term": "crap"
        }],
    "Status": {
            "Code": 3000,
            "Description": "OK",
            "Exception": null
        },
     "TrackingId": "2eaa012f-1604-4e36-a8d7-cc34b14ebcb4"
}
```

如需 JSON 回應中所有區段的詳細說明，請參考[文字仲裁 API 概觀](text-moderation-api.md)。

## <a name="next-steps"></a>後續步驟

在您的程式碼中使用 REST API，或從[文字仲裁 .NET 快速入門](text-moderation-quickstart-dotnet.md)開始著手，以便與您的應用程式進行整合。
