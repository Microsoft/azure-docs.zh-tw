---
title: 影像仲裁 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 使用影像仲裁來審核不適當的影像
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 01/20/2018
ms.author: sajagtap
ms.openlocfilehash: 6c5fed78c67f974a2af11efd133e9a79ec52124b
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219646"
---
# <a name="image-moderation"></a>影像仲裁

使用內容仲裁的電腦輔助影像仲裁以及[人工檢閱工具](Review-Tool-User-Guide/human-in-the-loop.md)來審核成人和猥褻內容的影像。 掃描影像的文字內容並擷取該文字，然後偵測臉部。 您可以根據自訂清單比對影像，並採取進一步的動作。

## <a name="evaluating-for-adult-and-racy-content"></a>評估成人和猥褻內容

**評估**作業會傳回介於 0 與 1 之間的信賴分數。 它也會傳回等於 true 或 false 的布林值資料。 這些值會預測影像是否包含潛在的成人和猥褻內容。 當您以影像 (檔案或 URL) 呼叫 API 時，傳回的回應包含下列資訊：

    "ImageModeration": {
      .............
      "adultClassificationScore": 0.019196987152099609,
      "isImageAdultClassified": false,
      "racyClassificationScore": 0.032390203326940536,
      "isImageRacyClassified": false,
      ............
      ],

> [!NOTE]

> - `isImageAdultClassified` 代表可能有在特定情況下被視為明顯色情或成人內容的影像存在。
> - `isImageRacyClassified` 代表可能有在特定情況下被視為具性暗示或成人內容的影像存在。
> - 分數介於 0 與 1 之間。 分數越高，模型預測為適用該類別的可能性就越高。 此預覽版功能倚賴統計模型，而不是手動編碼的結果。 建議您使用自己的內容進行測試，以判斷每個類別如何符合您的需求。
> - 布林值會是 true 或 false，視內部分數閾值而定。 客戶應該評估是要使用此值，還是根據其內容原則決定自訂閾值。
>

## <a name="detecting-text-with-optical-character-recognition-ocr"></a>使用光學字元辨識 (OCR) 偵測文字

**光學字元辨識 (OCR)** 作業會預測影像中是否存在文字內容，並擷取文字內容以進行文字仲裁及其他用途。 您可以指定語言。 如果您未指定語言，偵測會預設為英文。

回應包含下列資訊：
- 原始文字。
- 偵測到的文字項目及其信賴分數。

摘錄範例：

    "TextDetection": {
      "status": {
        "code": 3000.0,
        "description": "OK",
        "exception": null
      },
      .........
      "language": "eng",
      "text": "IF WE DID \r\nALL \r\nTHE THINGS \r\nWE ARE \r\nCAPABLE \r\nOF DOING, \r\nWE WOULD \r\nLITERALLY \r\nASTOUND \r\nOURSELVE \r\n",
      "candidates": []
    },


## <a name="detecting-faces"></a>偵測臉部

偵測臉部有助於偵測影像中的個人識別資訊 (PII)，例如臉部。 您可以偵測每個影像中的潛在臉部和潛在臉部數量。

回應包含下列資訊：

- 臉部計數
- 偵測到之臉部的位置清單

摘錄範例：


    "FaceDetection": {
       ......
      "result": true,
      "count": 2,
      "advancedInfo": [
      .....
      ],
      "faces": [
        {
          "bottom": 598,
          "left": 44,
          "right": 268,
          "top": 374
        },
        {
          "bottom": 620,
          "left": 308,
          "right": 532,
          "top": 396
        }
      ]
    }

## <a name="creating-and-managing-custom-lists"></a>建立和管理自訂清單

在許多線上社群中，當使用者上傳影像或其他類型的內容之後，具冒犯性的項目可能會在接下來的幾天、幾週和幾個月內被多次分享。 從多個位置重複掃描並篩選出相同的影像或稍微修改過的影像，其成本可能太過昂貴，而且容易產生錯誤。

您可以將具冒犯性的影像新增至自訂的封鎖內容清單，而不是審核相同的影像多次。 這樣一來，您的內容仲裁系統就會根據自訂清單比較傳入的影像，並停止任何的進一步處理。

> [!NOTE]
> 上限是 **5 個影像清單**，其中每個清單**不可超過 10,000 個影像**。
>

內容仲裁提供一個完整的[影像清單管理 API](try-image-list-api.md)，包含用來管理自訂影像清單的作業。 請從[影像清單 API 主控台](try-image-list-api.md)開始著手，然後使用 REST API 程式碼範例。 此外，如果您已熟悉 Visual Studio 和 C#，請一併參閱[影像清單 .NET 快速入門](image-lists-quickstart-dotnet.md)。

## <a name="matching-against-your-custom-lists"></a>根據您的自訂清單比對

比對作業可讓您根據使用清單作業建立和管理的自訂清單，進行傳入影像的模糊比對。

如果找到相符項目，該作業會傳回相符影像的識別碼和仲裁標籤。 回應包含下列資訊：

- 相符分數 (介於 0 和 1 之間)
- 相符影像
- 影像標籤 (Tag) (在上一個仲裁期間指派)
- 影像標籤 (Label)

摘錄範例：

    {
    ..............,
    "IsMatch": true,
    "Matches": [
        {
            "Score": 1.0,
            "MatchId": 169490,
            "Source": "169642",
            "Tags": [],
            "Label": "Sports"
        }
    ],
    ....
    }

## <a name="human-review-tool"></a>人工檢閱工具

如需詳細資訊，請使用內容仲裁的[檢閱工具](Review-Tool-User-Guide/human-in-the-loop.md)及其 API，以便為人工仲裁在檢閱中呈現仲裁結果和內容。 其會檢閱電腦指派的標籤，並確認其最終決策。

![給人工仲裁的影像檢閱](images/moderation-reviews-quickstart-dotnet.PNG)

## <a name="next-steps"></a>後續步驟

試用[影像仲裁 API 主控台](try-image-api.md)，並使用 REST API 程式碼範例。 此外，如果您已熟悉 Visual Studio 和 C#，請一併參閱[影像仲裁 .NET 快速入門](image-moderation-quickstart-dotnet.md)。
