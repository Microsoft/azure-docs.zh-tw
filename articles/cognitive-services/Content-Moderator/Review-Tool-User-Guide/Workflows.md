---
title: 在 Azure Content Moderator 中定義和使用工作流程 | Microsoft Docs
description: 了解如何建立以內容原則為基礎的自訂工作流程。
services: cognitive-services
author: sanjeev3
manager: mikemcca
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: article
ms.date: 01/07/2018
ms.author: sajagtap
ms.openlocfilehash: dfe3ba8a2ef1bcbc69ef585b504a9367d9420bf0
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367986"
---
# <a name="define-test-and-use-workflows"></a>定義、測試和使用工作流程

您可以使用 Azure Content Moderator 工作流程設計工具和 API，來定義以內容原則為基礎的自訂工作流程和閾值。

工作流程會使用連接器「連接」到 Content Moderator API。 您可以使用其他有可用連接器的 API。 此處的範例使用預設隨附的 Content Moderator 連接器。

## <a name="browse-to-the-workflows-section"></a>瀏覽至 [工作流程] 區段

在 [設定] 索引標籤中，選取 [工作流程]。

  ![[工作流程] 設定](images/2-workflows-0.png)

## <a name="start-a-new-workflow"></a>開始新的工作流程

選取 [新增工作流程]。

  ![新增工作流程](images/2-workflows-1.png)

## <a name="assign-a-name-and-description"></a>指派名稱和描述

為工作流程命名、輸入描述，然後選擇該工作流程是要處理影像還是文字。

  ![工作流程名稱和描述](images/ocr-workflow-step-1.PNG)

## <a name="define-the-evaluation-criteria-condition"></a>定義評估準則 (「條件」)

在下列螢幕擷取畫面中，您會看到必須為工作流程定義的欄位和 If-Then-Else 選取項目。 選擇連接器。 這個範例會使用 **Content Moderator**。 根據您選擇的連接器，可用於輸出的選項會變更。

  ![定義工作流程的條件](images/ocr-workflow-step-2-condition.PNG)

選好所要的連接器和其輸出之後，請選取一個運算子和值以構成條件。

## <a name="define-the-action-to-take"></a>定義要採取的動作

選取要採取的動作和要符合的條件。 下列範例會建立影像檢閱、指派標記 `a`，然後針對所顯示的條件來加以醒目提示。 您也可以結合多個條件，以獲得您想要的結果。 或者，請新增替代 (Else) 路徑。

  ![定義工作流程的動作](images/ocr-workflow-step-3-action.PNG)

## <a name="save-your-workflow"></a>儲存工作流程

最後，請儲存工作流程，並記下工作流程名稱。 您需要該名稱才能使用檢閱 API 來開始審核。

## <a name="test-the-workflow"></a>測試工作流程

您已定義自訂工作流程，接著請以內容範例來加以測試。

選取對應的 [執行工作流程] 按鈕。

  ![工作流程測試](images/ocr-workflow-step-6-list.PNG)

### <a name="upload-a-file"></a>上傳檔案

將[範例影像](https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png)儲存到本機磁碟機。 若要測試工作流程，請選取 [選擇檔案] 並上傳影像。

  ![上傳影像檔案](images/ocr-workflow-step-7-upload.PNG)

### <a name="track-the-workflow"></a>追蹤工作流程

在工作流程執行時加以追蹤。

  ![追蹤工作流程的執行](images/ocr-workflow-step-4-test.PNG)

### <a name="review-any-images-flagged-for-human-moderation"></a>檢閱標示為要進行人工審核的影像

若要查看影像檢閱，請移至 [檢閱] 底下的 [影像] 索引標籤。

  ![檢閱映像](images/ocr-sample-image-workflow1.PNG)

## <a name="next-steps"></a>後續步驟 

若要從程式碼叫用工作流程，請使用自訂工作流程與 [`Job` API 主控台快速入門](../try-review-api-job.md)和 [.NET SDK 快速入門](../moderation-jobs-quickstart-dotnet.md)。
