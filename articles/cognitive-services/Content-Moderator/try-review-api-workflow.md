---
title: 從 API 主控台中使用內容審核工作流程 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 了解如何從 API 主控台中使用內容審核工作流程。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 02/05/2018
ms.author: sajagtap
ms.openlocfilehash: 4ef8951b30fa7aede08a1af3c834192b5ed18649
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223845"
---
# <a name="workflows-from-the-api-console"></a>API 主控台中的工作流程

在 Azure Content Moderator 中使用[工作流程作業](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/5813b46b3f9b0711b43c4c59)，以使用檢閱 API 來建立或更新工作流程，或取得工作流程詳細資料。 您可以使用此 API 為工作流程定義簡單、複雜甚至是巢狀的運算式。 工作流程會出現在審核工具中，以供您的小組使用。 工作流程也會由檢閱 API 的作業使用。

## <a name="prerequisites"></a>必要條件

1. 移至[審核工具](https://contentmoderator.cognitive.microsoft.com/)。 如果您尚未註冊，請先註冊。 
2. 在審核工具的 [設定] 下，選取 [工作流程] 索引標籤，如審核工具的[工作流程教學課程](Review-Tool-User-Guide/Workflows.md)中所述。

### <a name="browse-to-the-workflows-screen"></a>瀏覽至 [工作流程] 畫面

在 [Content Moderator] 儀表板上選取 [檢閱] > [設定] > [工作流程]。 您會看到預設工作流程。

  ![預設工作流程](images/default-workflow-listed.PNG)

### <a name="get-the-json-definition-of-the-default-workflow"></a>取得預設工作流程的 JSON 定義

針對您的工作流程選取 [編輯] 選項，然後選取 [JSON] 索引標籤。您會看到下列 JSON 運算式：

    {
        "Type": "Logic",
        "If": {
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

## <a name="get-workflow-details"></a>取得工作流程詳細資料

使用 [工作流程 - 取得] 作業，以取得現有預設工作流程的詳細資料。

在審核工具中，移至[認證](Review-Tool-User-Guide/credentials.md#the-review-tool)區段。

### <a name="browse-to-the-api-reference"></a>瀏覽至 API 參考

1. 在 [認證] 檢視中，選取[API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/5813b46b3f9b0711b43c4c59)。 
2. 當 [工作流程 - 建立或更新] 頁面開啟時，移至[工作流程 - 取得](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/5813b44b3f9b0711b43c4c58)參考。

### <a name="select-your-region"></a>選取您的區域

針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。

  ![[工作流程 - 取得] 的區域選取項目](images/test-drive-region.png)

  [工作流程 - 取得] API 主控台隨即開啟。

### <a name="enter-parameters"></a>輸入參數

針對 **team**、**workflowname** 和 **Ocp-Apim-Subscription-Key** (您的訂用帳戶識別碼) 輸入值：

- **team**：您設定[審核工具帳戶](https://contentmoderator.cognitive.microsoft.com/)時所建立的小組識別碼。 
- **workflowname**：您工作流程的名稱。 使用 `default`。
- **Ocp-Apim-Subscription-Key**：位於 [設定] 索引標籤上。如需詳細資訊，請參閱[概觀](overview.md)。

  ![取得查詢參數和標頭](images/workflow-get-default.PNG)

### <a name="submit-your-request"></a>提交您的要求
  
選取 [傳送]。 如果作業成功，[Response status] \(回應狀態\) 就會是 `200 OK`，而 [Response content] \(回應內容\) 方塊則會顯示下列 JSON 工作流程：

    {
        "Name": "default",
        "Description": "Default",
        "Type": "Image",
        "Expression": {
        "If": {
            "ConnectorName": "moderator",
            "OutputName": "isadult",
            "Operator": "eq",
            "Value": "true",
            "AlternateInput": null,
            "Type": "Condition"
            },
        "Then": {
            "Perform": [{
                "Name": "createreview",
                "Subteam": null,
                "CallbackEndpoint": null,
                "Tags": []
            }],
            "Type": "Actions"
            },
            "Else": null,
            "Type": "Logic"
            }
    }


## <a name="create-a-workflow"></a>建立工作流程

在審核工具中，移至[認證](Review-Tool-User-Guide/credentials.md#the-review-tool)區段。

### <a name="browse-to-the-api-reference"></a>瀏覽至 API 參考

在 [認證] 檢視中，選取[API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/5813b46b3f9b0711b43c4c59)。 [工作流程 - 建立或更新] 頁面隨即開啟。

### <a name="select-your-region"></a>選取您的區域

針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。

  ![[工作流程 - 建立或更新] 頁面的區域選取項目](images/test-drive-region.png)

  [工作流程 - 建立或更新] API 主控台隨即開啟。

### <a name="enter-parameters"></a>輸入參數

針對 **team**、**workflowname** 和 **Ocp-Apim-Subscription-Key** (您的訂用帳戶識別碼) 輸入值：

- **team**：您設定[審核工具帳戶](https://contentmoderator.cognitive.microsoft.com/)時所建立的小組識別碼。 
- **workflowname**：新工作流程的名稱。
- **Ocp-Apim-Subscription-Key**：位於 [設定] 索引標籤上。如需詳細資訊，請參閱[概觀](overview.md)。

  ![[工作流程 - 建立或更新] 主控台查詢參數和標頭](images/workflow-console-parameters.PNG)

### <a name="enter-the-workflow-definition"></a>輸入工作流程定義

1. 編輯 [要求本文] 方塊，輸入 JSON 要求的 [描述] 和 [類型] (影像或文字) 詳細資料。 
2. 針對 [運算式]，複製上一節中的預設工作流程運算式，如下所示：

        {
            "Description": "Default workflow from API console",
            "Type": "Image",
            "Expression": 
                // Copy the default workflow expression from the preceding section
        }

    要求本文看起來會像下列 JSON 要求：

        {
            "Description": "Default workflow from API console",
            "Type": "Image",
            "Expression": {
                "Type": "Logic",
                "If": {
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
                    "Tags": [ ]
                }
                ],
                "Type": "Actions"
                }
            }
        }
 
### <a name="submit-your-request"></a>提交您的要求
  
選取 [傳送]。 如果作業成功，[Response status] \(回應狀態\) 就會是 `200 OK`，而 [Response content] \(回應內容\) 方塊則會顯示 `true`。

### <a name="check-out-the-new-workflow"></a>檢查新工作流程

在審核工具中，選取 [檢閱] > [設定] > [工作流程]。 新的工作流程會出現，並已可供使用。

  ![工作流程的審核工具清單](images/workflow-console-new-workflow.PNG)
  
### <a name="review-your-new-workflow-details"></a>檢視新工作流程的詳細資料

1. 針對您的工作流程選取 [編輯] 選項，然後選取 [設計工具] 和 [JSON] 索引標籤。

   ![所選工作流程的 [設計工具] 索引標籤](images/workflow-console-new-workflow-designer.PNG)

2. 若要查看工作流程的 JSON 定義，請選取 [JSON] 索引標籤。

## <a name="next-steps"></a>後續步驟

* 如需更複雜的工作流程範例，請參閱[工作流程概觀](workflow-api.md)。
* 了解如何搭配[內容審核作業](try-review-api-job.md)使用工作流程。
