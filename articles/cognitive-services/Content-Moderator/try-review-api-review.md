---
title: 透過 API 主控台使用人力審查來審核內容 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 了解如何在 Content Moderator API 主控台中建立人力審查。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 08/05/2017
ms.author: sajagtap
ms.openlocfilehash: bb95341a09f09ce8020f34476e720270fd401909
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219748"
---
# <a name="create-reviews-from-the-api-console"></a>從 API 主控台建立審查

使用審查 API 的[審查作業](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4)，建立影像或文字審核以便進行人力審核。 人力審核者會使用審查工具來審查內容。 根據您的審核後商務邏輯，使用這項作業。 請在您使用任何 Content Moderator 影像或文字 API，或其他認知服務 API 掃描您的內容之後，使用這項作業。 

在人力審核者審查自動指派的標記和預測資料並提交最終審核決策之後，審查 API 會將所有資訊提交至您的 API 端點。

## <a name="use-the-api-console"></a>使用 API 主控台
若要使用線上主控台來試用此 API，您需要在主控台中輸入幾個值：

- **teamName**：您在設定審查工具帳戶時所建立的小組名稱。 
- **ContentId**：這個字串會傳遞至 API 並透過回呼傳回。 ContentId 適合用於建立內部識別項或中繼資料與審核作業結果的關聯。
- **中繼資料**：在回呼期間傳回至 API 端點的自訂金鑰-值組。 如果金鑰是審查工具中定義的簡短代碼，則會顯示為標記。
- **Ocp-Apim-Subscription-Key**：位於 [設定] 索引標籤上。如需詳細資訊，請參閱[概觀](overview.md)。

存取測試主控台的最簡單方式就是經由 [認證] 視窗。

1.  在 [認證] 視窗中，選取[審查 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c4)。

  [審查 - 建立] 頁面隨即開啟。

2.  針對 [開啟 API 測試主控台] 中，選取最能描述您位置的區域。

  ![[審查 - 建立] 頁面區域選取項目](images/test-drive-region.png)

  [審查 - 建立] API 主控台隨即開啟。
  
3.  針對必要的查詢參數、內容類型和您的訂用帳戶金鑰，輸入一些值。 在 [要求本文] 方塊中，指定內容 (例如，影像位置)、中繼資料及其他與內容相關聯的資訊。

  ![[檢閱 - 建立] 主控台查詢參數、標頭和要求本文方塊](images/test-drive-review-1.PNG)
  
4.  選取 [傳送]。 隨即建立審查識別碼。 複製此識別碼以在下列步驟中使用。

  ![[審查 - 建立] 主控台的 [回應內容] 方塊會顯示審查識別碼](images/test-drive-review-2.PNG)
  
5.  選取 [取得]，然後藉由選取符合您區域的按鈕來開啟 API。 在結果頁面上，輸入 [teamName]、[ReviewID] 和 [訂閱金鑰] 的值。 選取頁面上的 [傳送] 按鈕。 

  ![審查 - 建立主控台的取得結果](images/test-drive-review-3.PNG)
  
6.  您將會看到掃描的結果。

  ![[審查 - 建立] 主控台的 [回應內容] 方塊](images/test-drive-review-4.PNG)
  
7.  在 Content Moderator 儀表板上，選取 [審查] > [影像]。 您已掃描的影像隨即出現，準備進行人力審查。

  ![足球的審查工具影像](images/test-drive-review-5.PNG)

## <a name="next-steps"></a>後續步驟

在您的程式碼中使用 REST API，或從[審查 .NET 快速入門](moderation-reviews-quickstart-dotnet.md)開始著手，以便與您的應用程式進行整合。
