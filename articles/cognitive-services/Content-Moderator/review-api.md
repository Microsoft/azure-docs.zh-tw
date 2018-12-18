---
title: 審核作業和人機互動審核 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 對機器輔助審核套用人工監督制度以獲得最佳成果。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 1/21/2018
ms.author: sajagtap
ms.openlocfilehash: b4a2f62b1c9cefb716cb217baf7389c3e7c790b8
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223248"
---
# <a name="moderation-jobs-and-reviews"></a>審核作業和檢閱

使用 Azure Content Moderator 的[檢閱 API](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c5) 將機器輔助審核和人機互動功能相結合，以讓貴公司獲得最佳成果。

「檢閱 API」提供下列方式來將人工監督制度包含在內容審核程序中：

* `Job` 作業可透過單一步驟開始建立機器輔助審核和人工檢閱。
* `Review` 作業可用來在審核步驟之外建立人工檢閱程序。
* `Workflow` 作業可用來管理會搭配閾值自動執行掃描來建立檢閱程序的工作流程。

`Job` 和 `Review` 作業會接受回呼端點以接收狀態和結果。

本文涵蓋 `Job` 和 `Review` 作業。 請閱讀[工作流程概觀](workflow-api.md)，以了解如何建立、編輯和取得工作流程定義。

## <a name="job-operations"></a>工作的作業

### <a name="start-a-job"></a>啟動作業
使用 `Job.Create` 作業來啟動審核和人工檢閱的建立作業。 Content Moderator 會掃描內容，並評估指定的工作流程。 根據工作流程的結果，它會建立檢閱或略過此步驟。 它也會提交審核後和檢閱後標記給回呼端點。

輸入中包含下列資訊：

- 檢閱小組識別碼。
- 要審核的內容。
- 工作流程名稱。 (預設值是「default」工作流程)。
- 用於通知的 API 回呼點。
 
下列回應顯示已啟動作業的識別碼。 您可以使用作業識別碼來取得作業狀態和接收詳細資訊。

    {
        "JobId": "2018014caceddebfe9446fab29056fd8d31ffe"
    }

### <a name="get-job-status"></a>取得作業狀態

使用 `Job.Get` 作業和作業識別碼來取得執行中或已完成作業的詳細資料。 此作業會立即傳回，審核作業則以非同步方式執行。 結果會透過回呼端點傳回。

輸入中包含下列資訊：

- 檢閱小組識別碼：先前作業所傳回的作業識別碼

回應中包含下列資訊：

- 所建立檢閱的識別碼。 (使用此識別碼來取得最終的檢閱結果)。
- 作業的狀態 (已完成或進行中)：所指派的審核標記 (機碼值組)。
- 作業執行報告。
 
 
        {
            "Id": "2018014caceddebfe9446fab29056fd8d31ffe",
            "TeamName": "some team name",
            "Status": "Complete",
            "WorkflowId": "OCR",
            "Type": "Image",
            "CallBackEndpoint": "",
            "ReviewId": "201801i28fc0f7cbf424447846e509af853ea54",
            "ResultMetaData":[
            {
            "Key": "hasText",
            "Value": "True"
            },
            {
            "Key": "ocrText",
            "Value": "IF WE DID \r\nALL \r\nTHE THINGS \r\nWE ARE \r\nCAPABLE \r\nOF DOING, \r\nWE WOULD \r\nLITERALLY \r\nASTOUND \r\nOURSELVE \r\n"
            }
            ],
            "JobExecutionReport": [
            {
                "Ts": "2018-01-07T00:38:29.3238715",
                "Msg": "Posted results to the Callbackendpoint: https://requestb.in/vxke1mvx"
                },
                {
                "Ts": "2018-01-07T00:38:29.2928416",
                "Msg": "Job marked completed and job content has been removed"
                },
                {
                "Ts": "2018-01-07T00:38:29.0856472",
                "Msg": "Execution Complete"
                },
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
 
![給人工審核者的影像檢閱](images/ocr-sample-image.PNG)

## <a name="review-operations"></a>檢閱作業

### <a name="create-reviews"></a>建立檢閱

使用 `Review.Create` 作業來建立人工檢閱。 您可以在別處審核這些檢閱，也可以使用自訂邏輯來指派審核標記。

此作業的輸入中包含：

- 要檢閱的內容。
- 供人工審核者檢閱的所指派標記 (機碼值組)。

下列回應顯示檢閱識別碼：

    [
        "201712i46950138c61a4740b118a43cac33f434",
    ]


### <a name="get-review-status"></a>取得檢閱狀態
使用 `Review.Get` 作業來取得所審核影像的人工檢閱程序完成後的結果。 系統會透過您提供的回呼端點通知您。 

此作業會傳回兩組標記： 

* 審核服務所指派的標記
* 人工檢閱完成後的標記

輸入中至少會包含：

- 檢閱小組名稱
- 先前作業所傳回的檢閱識別碼

回應中包含下列資訊：

- 檢閱狀態
- 人工檢閱者所確認的標記 (機碼值組)
- 審核服務所指派的標記 (機碼值組)

您會在下列回應範例中同時看到檢閱者指派的標記 (**reviewerResultTags**) 和初始標記 (**metadata**)：

    {
        "reviewId": "201712i46950138c61a4740b118a43cac33f434",
        "subTeam": "public",
        "status": "Complete",
        "reviewerResultTags": [
        {
            "key": "a",
            "value": "False"
        },
        {
            "key": "r",
            "value": "True"
        },
        {
            "key": "sc",
            "value": "True"
        }
        ],
        "createdBy": "{teamname}",
        "metadata": [
        {
            "key": "sc",
            "value": "true"
        }
        ],
        "type": "Image",
        "content": "https://reviewcontentprod.blob.core.windows.net/{teamname}/IMG_201712i46950138c61a4740b118a43cac33f434",
        "contentId": "0",
        "callbackEndpoint": "{callbackUrl}"
    }

## <a name="next-steps"></a>後續步驟

* 試用[作業 API 主控台](try-review-api-job.md)，並使用 REST API 程式碼範例。 如果您已熟悉 Visual Studio 和 C#，也請參閱[作業 .NET 快速入門](moderation-jobs-quickstart-dotnet.md)。 
* 若為檢閱者，請開始使用[檢閱 API 主控台](try-review-api-review.md)，並使用 REST API 程式碼範例。 然後請參閱[檢閱 .NET 快速入門](moderation-reviews-quickstart-dotnet.md)。
* 若為影片檢閱者，請使用[影片檢閱快速入門](video-reviews-quickstart-dotnet.md)，並了解如何[在影片檢閱中新增文字記錄](video-transcript-reviews-quickstart-dotnet.md)。
