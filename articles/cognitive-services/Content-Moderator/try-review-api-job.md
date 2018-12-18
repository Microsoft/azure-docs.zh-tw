---
title: 使用 API 主控台執行內容仲裁作業 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 了解如何在 API 主控台中執行內容審核作業。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 08/03/2017
ms.author: sajagtap
ms.openlocfilehash: 240b26cd86a6985825e3145c5bc43ef31524d7b7
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227096"
---
# <a name="start-a-moderation-job-from-the-api-console"></a>從 API 主控台啟動審核作業

使用檢閱 API 的[作業操作](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c5)來對 Azure Content Moderator 中的影像或文字內容起始端對端內容審核。 

審核作業會使用 Content Moderator 影像審核 API 或文字審核 API 來掃描內容。 然後，審核作業會使用工作流程 (定義於檢閱工具中) 在檢閱工具中產生檢閱。 

在人力審核者審查自動指派的標記和預測資料並提交最終審核決策之後，審查 API 會將所有資訊提交至您的 API 端點。

## <a name="prerequisites"></a>必要條件

瀏覽至[檢閱工具](https://contentmoderator.cognitive.microsoft.com/)。 如果您尚未註冊，請註冊。 在檢閱工具內，[定義自訂工作流程](Review-Tool-User-Guide/Workflows.md)以在 `Job` 操作中使用。

## <a name="use-the-api-console"></a>使用 API 主控台
若要使用線上主控台來試用此 API，您需要在主控台中輸入幾個值：
    
- `teamName`：使用檢閱工具認證畫面中的 `Id` 欄位。 
- `ContentId`：這個字串會傳遞至 API 並透過回呼傳回。 **ContentId** 適合用於建立內部識別項或中繼資料與審核作業結果的關聯。- `Workflowname`：您在上一節[所建立工作流程](Review-Tool-User-Guide/Workflows.md)的名稱。
- `Ocp-Apim-Subscription-Key`：位於 [設定] 索引標籤。如需詳細資訊，請參閱[概觀](overview.md)。

[存取 API 主控台] 來自 [認證] 視窗。

### <a name="navigate-to-the-api-reference"></a>瀏覽至 API 參考
在 [認證] 視窗中，選取 [API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c5)。

  `Job.Create` 頁面隨即開啟。

### <a name="select-your-region"></a>選取您的區域
針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。
  ![[作業 - 建立] 頁面區域選取項目](images/test-drive-job-1.png)

  `Job.Create` API 主控台隨即開啟。 

### <a name="enter-parameters"></a>輸入參數

針對必要查詢參數和訂用帳戶金鑰輸入值。 在 [要求本文] 方塊中，指定所要掃描資訊的位置。 在此範例中，我們使用這個[影像範例](https://moderatorsampleimages.blob.core.windows.net/samples/sample6.png)。

  ![[作業 - 建立] 主控台查詢參數、標頭和要求本文方塊](images/job-api-console-inputs.PNG)

### <a name="submit-your-request"></a>提交您的要求
選取 [傳送]。 隨即建立作業識別碼。 複製此值以便用於後續步驟。

  `"JobId": "2018014caceddebfe9446fab29056fd8d31ffe"`

### <a name="open-the-get-job-details-page"></a>開啟 [取得作業詳細資料] 頁面
選取 [取得]，然後藉由選取符合您區域的按鈕來開啟 API。

  ![[作業 - 建立] 主控台的取得結果](images/test-drive-job-4.png)

### <a name="review-the-response"></a>檢閱回應

輸入 **teamName** 和 **JobID** 的值。 輸入您的訂用帳戶金鑰，然後選取 [Send] \(傳送\)。 下列回應會顯示作業狀態範例和詳細資料。

```
    {
        "Id": "2018014caceddebfe9446fab29056fd8d31ffe",
        "TeamName": "some team name",
        "Status": "InProgress",
        "WorkflowId": "OCR",
        "Type": "Image",
        "CallBackEndpoint": "",
        "ReviewId": "",
        "ResultMetaData": [],
        "JobExecutionReport": 
        [
            {
            "Ts": "2018-01-07T00:38:26.7714671",
                "Msg": "Successfully got hasText response from Moderator"
            },
            {
                "Ts": "2018-01-07T00:38:26.4181346",
                "Msg": "Getting hasText from Moderator"
            },
            {
                "Ts": "2018-01-07T00:38:25.5122828",
                "Msg": "Starting Execution - Try 1"
            }
        ]
    }
```

## <a name="navigate-to-the-review-tool"></a>瀏覽至檢閱工具
在 Content Moderator 儀表板上，選取 [檢閱] > [影像]。 您已掃描的影像隨即出現，準備進行人工檢閱。

  ![三名自行車選手的檢閱工具影像](images/ocr-sample-image.PNG)

## <a name="next-steps"></a>後續步驟

在您的程式碼中使用 REST API，或從[作業 .NET 快速入門](moderation-jobs-quickstart-dotnet.md)開始著手，以便與您的應用程式進行整合。
