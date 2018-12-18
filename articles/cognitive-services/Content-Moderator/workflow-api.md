---
title: 仲裁工作流程 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 搭配內容仲裁使用工作流程。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 02/04/2018
ms.author: sajagtap
ms.openlocfilehash: 3de45c62eb208671cc2d1d4de5309d2f9d75adc9
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47226869"
---
# <a name="moderation-workflows"></a>仲裁工作流程

Content Moderator 包含用來管理工作流程的工具和 API。 您可以搭配[審核 API 的工作作業](review-api.md)使用工作流程，以根據內容原則和閾值，自動建立人在迴路 (human-in-the-loop) 審核項目。

「審核 API」提供下列方式來將人工監督包含在您的內容仲裁程序中：

1. 「工作」作業：用於將電腦輔助仲裁和人工審核項目的建立當作一個步驟來啟動。
1. 「審核」作業：用於在仲裁步驟之外，建立人工審核項目。
1. 「工作流程」作業：用於管理可搭配閾值自動執行掃描來建立審核項目的工作流程。

本文涵蓋「工作流程」作業。 請參閱[作業與審核](review-api.md)概觀，以了解仲裁作業與審核。

查看 **default** 工作流程是開始了解 Content Moderator 中工作流程的最佳方式。

## <a name="your-first-workflow"></a>您的第一個工作流程

您的第一個工作流程會與您的[審核工具小組](https://contentmoderator.cognitive.microsoft.com/)配套。 如果您尚未註冊，請註冊。

瀏覽至[審核工具的 [工作流程]](Review-Tool-User-Guide/Workflows.md) 畫面 (在 [設定] 索引標籤底下)。您會看到一個 **default** 工作流程，如下圖所示：

![Content Moderator 工作流程](Review-Tool-User-Guide/images/2-workflows-1.png)

### <a name="open-the-default-workflow"></a>開啟預設工作流程

使用 [編輯] 選項來開啟工作流程編輯頁面，如下圖所示：![Content Moderator 預設工作流程](images/default-workflow-listed.PNG)

### <a name="the-designer-view"></a>設計工具檢視

您會看到工作流程的 [設計工具] 索引標籤。 設計工具檢視會顯示下列步驟：

1. 要評估之工作流程的**條件**。 在此案例中，工作流程會呼叫 Content Moderator 的影像 API，然後檢查 `isAdult` 輸出是否等於 `true`。
1. 符合條件時，要執行的**動作**。 在此案例中，如果 `isAdult` 輸出為 `true`，工作流程就會在審核工具中建立審核項目。

![Content Moderator 預設工作流程 - 設計工具](images/default-workflow-designer.png)

### <a name="the-json-view"></a>JSON 檢視

選取 [JSON] 索引標籤，以查看工作流程的 JSON 定義。

    {
        "Type": "Logic",
        If": {
            "ConnectorName": "moderator",
            "OutputName": "isAdult",
            "Operator": "eq",
            "Value": "true",
            "Type": "Condition"
        },
        "Then": {
        "Perform": [
        {
            "Name": "createreview",
            "CallbackEndpoint": null,
            "Tags": []
        }
        ],
        "Type": "Actions"
        }
    }

### <a name="key-learning"></a>重點學習

Content Moderator 中的工作流程容易設定且具有彈性。 如果內建的設計工具不符合您的需求，請以 **JSON** 格式撰寫工作流程定義。 然後將該 JSON 定義與[工作流程 API](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/5813b46b3f9b0711b43c4c59) 搭配使用，以從您的應用程式建立和管理工作流程。

## <a name="define-a-custom-workflow"></a>定義自訂工作流程

Content Moderator 的工作流程功能可讓您定義及使用自訂工作流程。 請使用[審核工具工作流程使用說明](Review-Tool-User-Guide/Workflows.md)一文來定義自訂工作流程。 此工作流程會使用 Content Moderator 的 OCR 功能，從範例影像擷取文字。 接著，它會在審核工具中建立審核項目。

### <a name="the-sample-image"></a>範例影像

將[範例影像](https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png)儲存到本機磁碟機。 您需要此影像來進行練習。

### <a name="the-designer-view"></a>設計工具檢視

選取 [設計工具] 索引標籤和[工作流程建立教學課程](Review-Tool-User-Guide/Workflows.md)，以定義自訂工作流程。 下圖顯示設計工具的 [條件] 檢視。 請參考教學課程以了解剩餘的步驟。

![Content Moderator - 工作流程條件](Review-Tool-User-Guide/images/ocr-workflow-step-2-condition.PNG)

### <a name="the-json-view"></a>JSON 檢視

選取 [JSON] 索引標籤，以查看自訂工作流程的下列 JSON 定義。 請注意 JSON 定義中的**If-Then** 陳述式如何對應至您使用設計工具檢視定義的步驟。

    {
        "Type": "Logic",
        "If": {
            "ConnectorName": "moderator",
            "OutputName": "hasText",
            "Operator": "eq",
            "Value": "true",
            "Type": "Condition"
        },
        "Then": {
        "Perform": [
        {
            "Name": "createreview",
            "CallbackEndpoint": null,
            "Tags": [
            {
                "Tag": "a",
                "IfCondition": {
                    "ConnectorName": "moderator",
                    "OutputName": "hasText",
                    "Operator": "eq",
                    "Value": "true",
                    "Type": "Condition"
                }
            }
            ]
        }
        ],
        "Type": "Actions"
        }
    }

### <a name="workflow-result"></a>工作流程結果

從工作流程畫面測試工作流程之後，會建立下列審核項目。 請瀏覽至 [審核] 底下的 [影像] 索引標籤，以查看您的審核項目。
工作流程之所以建立審核項目，是因為文字存在性的主要條件測試結果為肯定。 此審核也在影像審核項目中醒目提示了 **`a`** 標籤。

![Content Moderator - 簡易工作流程輸出](images/ocr-sample-image-workflow1.PNG)


## <a name="advanced-workflow-with-combination"></a>含有組合的進階工作流程

### <a name="the-sample-image"></a>範例影像

使用與上一節中使用的相同[範例影像](https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png)。

不過，這次請將您的主要條件變更為兩個檢查的組合。 除了檢查是否有文字之外，還會檢查該文字是否有任何粗話。 如果找到文字**且**偵測到其中有粗話，工作流程就會建立審核項目。

### <a name="the-designer-view"></a>設計工具檢視

若要將 [條件] 變更為 [組合]，請修改工作流程。 下圖顯示您在設計工具中看到的新檢視。

![Content Moderator - 已修改的工作流程條件](images/ocr-workflow-2-designer.PNG)

### <a name="the-json-view"></a>JSON 檢視

選取 [JSON] 索引標籤，以查看已修改之自訂工作流程的下列 JSON 定義。 請注意 JSON 定義中的**If-Then** 陳述式如何對應至您新增到工作流程中的新步驟。

    {
        "Type": "Logic",
        "If": {
        "Left": {
            "ConnectorName": "moderator",
            "OutputName": "hasText",
            "Operator": "eq",
            "Value": "true",
            "Type": "Condition"
            },
        "Right": {
            "ConnectorName": "moderator",
            "OutputName": "text.HasProfanity",
            "Operator": "eq",
            "Value": "true",
            "Type": "Condition",
            "AlternateInput": "moderator.ocrText"
            },
        "Combine": "AND",
        "Type": "Combine"
        },
        "Then": {
        "Perform": [
        {
            "Name": "createreview",
            "CallbackEndpoint": null,
            "Tags": [
            {
                "Tag": "a",
                "IfCondition": {
                    "ConnectorName": "moderator",
                    "OutputName": "hasText",
                    "Operator": "eq",
                    "Value": "true",
                    "Type": "Condition"
                }
            }
            ]
        }
        ],
        "Type": "Actions"
        }
    }

    
### <a name="workflow-result"></a>工作流程結果

在您重新測試工作流程之後，會發現沒有建立任何審核項目。 若要確認是否沒有任何審核項目，請瀏覽至 [審核] 底下的 [影像] 索引標籤。
工作流程之所以沒有建立審核項目，是因為在所擷取的文字中偵測不到粗話。

![Content Moderator - 已修改的工作流程輸出](images/ocr-workflow-2-result.PNG)


## <a name="the-workflow-api"></a>工作流程 API

[工作流程作業](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/5813b46b3f9b0711b43c4c59)提供工作流程功能的程式設計介面。 您可以使用「工作流程 API」來建立工作流程、取得工作流程詳細資料，以及更新工作流程定義。

### <a name="get-all-workflow-details"></a>取得 [所有] 工作流程詳細資料

[Workflow - Get] \(工作流程 - 取得\) 作業可接受下列輸入：

- **team**：您設定[審核工具帳戶](https://contentmoderator.cognitive.microsoft.com/)時所建立的小組識別碼。 
- **workflowname**：您工作流程的名稱。 一開始請使用 `default`。
- **Ocp-Apim-Subscription-Key**：位於 [設定] 索引標籤上。如需詳細資訊，請參閱[概觀](overview.md)。

如果作業成功，[Response status] \(回應狀態\) 就會是 `200 OK`，而 [Response content] \(回應內容\) 方塊則會以 JSON 格式顯示工作流程定義。
若要深入了解，請參閱[工作流程 API 主控台快速入門](try-review-api-job.md)。

### <a name="create-or-update-workflow"></a>建立或更新工作流程

建立和更新作業可讓您從 API 建立工作流程。

[Workflow - Create or Update] \(工作流程 - 建立或更新\) 作業可接受下列輸入：

- **team**：您設定[審核工具帳戶](https://contentmoderator.cognitive.microsoft.com/)時所建立的小組識別碼。 
- **workflowname**：您工作流程的名稱。 一開始請使用 `default`。
- **Ocp-Apim-Subscription-Key**：位於 [設定] 索引標籤上。如需詳細資訊，請參閱[概觀](overview.md)。

如果作業成功，[Response status] \(回應狀態\) 就會是 `200 OK`，而 [Response content] \(回應內容\) 方塊則會顯示 `true`。 若要深入了解，請[試用 `Create` 作業](try-review-api-job.md)。

## <a name="next-steps"></a>後續步驟

若要了解如何建立自訂工作流程，請查看[審核工具的工作流程教學課程](Review-Tool-User-Guide/Workflows.md)。 

試用[工作流程 API 主控台](try-review-api-job.md)，並使用 REST API 程式碼範例。 

最後，將自訂工作流程與「工作」作業搭配使用，如[工作 API 主控台](try-review-api-job.md)和 [工作 .NET 快速入門](moderation-jobs-quickstart-dotnet.md)所示。
