---
title: 透過 API 主控台來仲裁影像 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 在 Content Moderator API 主控台中試用影像審核。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 08/05/2017
ms.author: sajagtap
ms.openlocfilehash: a88eb1e0fc91fb47a95c8b1fea84cfac32674266
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224948"
---
# <a name="moderate-images-from-the-api-console"></a>從 API 主控台審核影像

使用 Azure Content Moderator 中的[影像審核 API](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c) 來起始影像內容的掃描與檢閱審核工作流程。 審核作業會掃描內容中是否有粗話，並將內容與自訂和共用的黑名單做比較。

## <a name="use-the-api-console"></a>使用 API 主控台
您需要有訂用帳戶金鑰，才能在線上主控台中試用 API。 這位於 [設定] 索引標籤的 [Ocp-Apim-Subscription-Key] 方塊中。 如需詳細資訊，請參閱[概觀](overview.md)。

1.  移至[影像審核 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c)。

  [影像 - 評估] 影像審核頁面隨即開啟。

2. 在 [開啟 API 測試主控台] 中，選取最能描述您所在位置的區域。 

  ![嘗試 [影像 - 評估] 頁面區域選取項目](images/test-drive-region.png)
  
  [影像 - 評估] API 主控台隨即開啟。

3. 在 [Ocp-Apim-Subscription-Key] 方塊中，輸入訂用帳戶金鑰。

  ![嘗試 [影像 - 評估] 主控台訂用帳戶金鑰](images/try-image-api-1.PNG)

4. 在 [要求本文] 方塊中，使用預設的影像範例，或指定要掃描的影像。 您可以提交二進位位元資料形式的影像本身，也可以指定影像的可公開存取 URL。 

  在此範例中，使用 [要求本文] 方塊中所提供的路徑，然後選取 [傳送]。 

   ![嘗試 [影像 - 評估] 主控台要求本文](images/try-image-api-2.PNG)

  這是位於該 URL 的影像：

  ![嘗試 [影像 - 評估] 主控台影像範例](images/sample-image.jpg) 

5. 選取 [傳送]。

6. API 會傳回每個分類的機率分數。 它也會傳回對於影像是否符合條件的判斷 (**true** 或 **false**)。 

  ![嘗試 [影像 - 評估] 主控台機率分數和條件判斷](images/try-image-api-3.PNG)

## <a name="face-detection"></a>臉部偵測

您可以使用影像審核 API 來找到影像中的臉部。 當您顧慮到隱私權，而想要防止平台上發佈特定臉部，就很適合使用此選項。 

1.  在[影像審核 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c)中，於左功能表的 [影像] 下，選取 [尋找臉部]。 

  [影像 - 尋找臉部] 頁面隨即開啟。

2.  在 [開啟 API 測試主控台] 中，選取最能描述您所在位置的區域。 

  ![嘗試 [影像 - 尋找臉部] 頁面區域選取項目](images/test-drive-region.png)

  [影像 - 尋找臉部] API 主控台隨即開啟。

3. 指定要掃描的影像。 您可以提交二進位位元資料形式的影像本身，也可以指定影像的可公開存取 URL。 這個範例連結至 CNN 文章中所使用的影像。

  ![嘗試 [影像 - 尋找臉部] 影像範例](images/try-image-api-face-image.jpg)

  ![嘗試 [影像 - 尋找臉部] 要求範例](images/try-image-api-face-request.png)

4. 選取 [傳送]。 在此範例中，API 會尋找兩個臉部，並傳回其在影像中的座標。

   ![嘗試 [影像 - 尋找臉部] 回應內容方塊範例](images/try-image-api-face-response.png)

## <a name="text-detection-via-ocr-capability"></a>透過 OCR 功能偵測文字

您可以使用 Content Moderator OCR 功能來偵測影像中的文字。

1. 在[影像審核 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c)中，於左功能表的 [影像] 下，選取 [OCR]。 

  [影像 - OCR] 頁面隨即開啟。

2. 在 [開啟 API 測試主控台] 中，選取最能描述您所在位置的區域。 

  ![[影像 - OCR] 頁面區域選取項目](images/test-drive-region.png)

  [影像 - OCR] API 主控台隨即開啟。

3. 在 [Ocp-Apim-Subscription-Key] 方塊中，輸入訂用帳戶金鑰。

4. 在 [要求本文] 方塊中，使用預設的影像範例。 此影像與上一節所使用的影像相同。

5. 選取 [傳送]。 所擷取的文字會以 JSON 顯示：

  ![[影像 - OCR] 回應內容方塊範例](images/try-image-api-ocr.PNG)

## <a name="next-steps"></a>後續步驟

在程式碼中使用 REST API，或從[影像審核 .NET 快速入門](image-moderation-quickstart-dotnet.md)開始著手，來與應用程式進行整合。
