---
title: 透過 API 主控台使用自訂清單來仲裁影像 - Content Moderator
titlesuffix: Azure Content Moderator
description: 在 Content Moderator API 主控台中試用自訂影像清單。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 08/05/2017
ms.author: sajagtap
ms.openlocfilehash: 5e573262879b6a16e3cd530757900b86228ae299
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219952"
---
# <a name="moderate-with-custom-image-lists-in-the-api-console"></a>使用 API 主控台中的自訂影像清單進行仲裁

您可使用 Azure Content Moderator 中的[清單管理 API](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f672) 來建立自訂影像清單。 請搭配「影像仲裁 API」使用自訂影像清單。 影像仲裁作業會評估您的影像。 如果您建立自訂清單，此作業也會將它與您自訂清單中的影像做比較。 您可以使用自訂清單來封鎖或允許影像。

> [!NOTE]
> 上限是 **5 個影像清單**，其中每個清單**不可超過 10,000 個影像**。
>

您可以使用「清單管理 API」來執行下列工作：

- 建立清單。
- 將影像新增至清單。
- 依據清單中的影像過濾影像。
- 從清單中刪除影像。
- 刪除清單。
- 編輯清單資訊。
- 重新整理索引，以便將清單變更包含在新的掃描中。

## <a name="use-the-api-console"></a>使用 API 主控台
您需要有訂用帳戶金鑰，才能在線上主控台中試用 API。 這位於 [設定] 索引標籤的 [Ocp-Apim-Subscription-Key] 方塊中。 如需詳細資訊，請參閱[概觀](overview.md)。

## <a name="refresh-search-index"></a>重新整理搜尋索引

在您對影像清單進行變更之後，必須重新整理其索引，變更才會包含在未來的掃描中。 此步驟就像桌面搜尋引擎 (如果已啟用) 或 Web 搜尋引擎持續重新整理其索引以包含新檔案或頁面一樣。

1.  在[影像清單管理 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f672)中，從左側功能表中選取 [Image Lists] \(影像清單\)，然後選取 [Refresh Search Index] \(重新整理搜尋索引\)。

  [Image Lists - Refresh Search Index] \(影像清單 - 重新整理搜尋索引\) 頁面隨即開啟。

2. 針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。 
 
    ![[Image Lists - Refresh Search Index] \(影像清單 - 重新整理搜尋索引\) 頁面區域選取項目](images/test-drive-region.png)

    [Image Lists - Refresh Search Index] \(影像清單 - 重新整理搜尋索引\) API 主控台隨即開啟。

3.  在 [listId] 方塊中，輸入清單識別碼。 輸入您的訂用帳戶金鑰，然後選取 [Send] \(傳送\)。

  ![[Image Lists - Refresh Search Index] \(影像清單 - 重新整理搜尋索引\) 主控台的 [Response content] \(回應內容\) 方塊](images/try-image-list-refresh-1.png)


## <a name="create-an-image-list"></a>建立影像清單

1.  移至[影像清單管理 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f672)。

  [Image Lists - Create] \(影像清單 - 建立\) 頁面隨即開啟。 

3.  針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。

    ![[Image Lists - Create] \(影像清單 - 建立\) 頁面區域選取項目](images/test-drive-region.png)

    [Image Lists - Create] \(影像清單 - 建立\) API 主控台隨即開啟。
 
4.  在 [Ocp-Apim-Subscription-Key] 中，輸入您的訂用帳戶金鑰。

5.  在 [Request body] \(要求本文\) 方塊中，輸入 [Name] \(名稱\) (例如 MyList) 和 [Description] \(描述\) 的值。

  ![[Image Lists - Create] \(影像清單 - 建立\) 主控台的要求本文名稱和描述](images/try-terms-list-create-1.png)

6.  使用索引鍵/值組預留位置來為清單指派更多描述性中繼資料。

        {
           "Name": "MyExclusionList",
           "Description": "MyListDescription",
           "Metadata": 
           {
             "Category": "Competitors",
             "Type": "Exclude"
           }
        }

  請以索引鍵/值組形式新增清單中繼資料，而不要新增實際的影像。
 
7.  選取 [傳送]。 您的清單隨即建立。 請注意與新清單關聯的 [ID] \(識別碼\) 值。 您需要有此識別碼，才能進行其他影像清單管理功能。

  ![[Image Lists - Create] \(影像清單 - 建立\) 主控台的 [Response content] \(回應內容\) 方塊顯示清單識別碼](images/try-terms-list-create-2.png)
 
8.  接著，將影像新增至 MyList。 從左側功能表中，選取 [Image] \(影像\)，然後選取 [Add Image] \(新增影像\)。

  [Image - Add Image] \(影像 - 新增影像\) 頁面隨即開啟。 

9. 針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。

  ![[Image - Add Image] \(影像 - 新增影像\) 頁面區域選取項目](images/test-drive-region.png)

  [Image - Add Image] \(影像 - 新增影像\) API 主控台隨即開啟。
 
10. 在 [listId] 方塊中，輸入您產生的清單識別碼，然後輸入想要新增影像的 URL。 輸入您的訂用帳戶金鑰，然後選取 [Send] \(傳送\)。

11. 若要確認是否已將該影像新增至清單，請從左側功能表中，選取 [Image] \(影像\)，然後選取 [Get All Image Ids] \(取得所有影像識別碼\)。

  [Image - Get All Image Ids] \(影像 - 取得所有影像識別碼\) API 主控台隨即開啟。
  
12. 在 [listId] 方塊中，輸入清單識別碼，然後輸入您的訂用帳戶金鑰。 選取 [傳送]。

  ![[Image - Get All Image Ids] \(影像 - 取得所有影像識別碼\) 主控台的 [Response content] \(回應內容\) 方塊列出您輸入的影像](images/try-image-list-create-11.png)
 
10. 再多新增幾個影像。 既然您已建立自訂影像清單，現在即可嘗試使用自訂影像清單來[評估影像](try-image-api.md)。 

## <a name="delete-images-and-lists"></a>刪除影像和清單

刪除影像或清單相當簡單。 您可以使用 API 來執行下列工作：

- 删除映像。 ([Image - Delete] \(影像 - 刪除\))
- 刪除清單中的所有影像，但不刪除清單。 ([Image - Delete All Images] \(影像 - 刪除所有影像\))
- 刪除清單及其所有內容。 ([Image Lists - Delete] \(影像清單 - 刪除\))

此範例會刪除單一影像：

1. 在[影像清單管理 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f672)中，從左側功能表中選取 [Image] \(影像\)，然後選取 [Delete] \(刪除\)。 

  [Image - Delete] \(影像 - 刪除\) 頁面隨即開啟。

2. 針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。 

  ![[Image - Delete] \(影像 - 刪除\) 頁面區域選取項目](images/test-drive-region.png)
 
  [Image - Delete] \(影像 - 刪除\) API 主控台隨即開啟。
 
3.  在 [listId] 方塊中，輸入要作為影像刪除來源的清單識別碼。  這是 [Image - Get All Image Ids] \(影像 - 取得所有影像識別碼\) 主控台中針對 MyList 傳回的數字。 接著，輸入要刪除之影像的 [ImageId]。 

在我們的範例中，清單識別碼是 **58953**(**ContentSource** 的值)。 影像識別碼是 **59021** (**ContentIds**的值)。

4.  輸入您的訂用帳戶金鑰，然後選取 [Send] \(傳送\)。

5.  若要確認是否已將該影像刪除，請使用 [Image - Get All Image Ids] \(影像 - 取得所有影像識別碼\) 主控台。
 
## <a name="change-list-information"></a>變更清單資訊

您可以編輯清單的名稱和描述，以及新增中繼資料項目。

1.  在[影像清單管理 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f672)中，從左側功能表中選取 [Image Lists] \(影像清單\)，然後選取 [Update Details] \(更新詳細資料\)。 

  [Image Lists - Update Details] \(影像清單 - 更新詳細資料\) 頁面隨即開啟。

2. 針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。  

    ![[Image Lists - Update Details] \(影像清單 - 更新詳細資料\) 頁面區域選取項目](images/test-drive-region.png)

    [Image Lists - Update Details] \(影像清單 - 更新詳細資料\) API 主控台隨即開啟。
 
3.  在 [listId] 方塊中，輸入清單識別碼，然後輸入您的訂用帳戶金鑰。

4.  在 [Request body] \(要求本文\) 方塊中，進行編輯，然後選取頁面上的 [Send] \(傳送\) 按鈕。

  ![[Image Lists - Update Details] \(影像清單 - 更新詳細資料\) 主控台 [Request body] \(要求本文\) 編輯](images/try-terms-list-change-1.png)
 

## <a name="next-steps"></a>後續步驟

在您的程式碼中使用 REST API，或從[影像清單 .NET 快速入門](image-lists-quickstart-dotnet.md)開始著手，以便與您的應用程式進行整合。
