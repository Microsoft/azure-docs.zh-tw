---
title: 使用自訂字詞清單來審核文字 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 在 Content Moderator API 主控台中試用自訂字詞清單。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 08/05/2017
ms.author: sajagtap
ms.openlocfilehash: 99df9fda2cc56f169a61ec215a976de28fc13d27
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220273"
---
# <a name="moderate-with-custom-term-lists-in-the-api-console"></a>使用 API 主控台中的自訂字詞清單進行審核

Azure Content Moderator 中的預設全域字詞清單已可滿足大部分內容審核需求。 不過，您可能會需要審查屬於您組織的特定字詞。 例如，您可能要標記競爭對手名稱以供進一步檢閱。 

使用[清單管理 API](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f) 可建立自訂字詞清單，以搭配文字審核 API 使用。 此**文字 - 審查**作業會掃描您的文字中是否有粗話，並將文字與自訂和共用的黑名單做比較。

> [!NOTE]
> 上限是 **5 個字詞清單**，其中每個清單**不可超過 10,000 個字詞**。
>

您可以使用「清單管理 API」來執行下列工作：
- 建立清單。
- 將字詞新增至清單。
- 針對清單中的字詞審查字詞。
- 刪除清單中的字詞。
- 刪除清單。
- 編輯清單資訊。
- 重新整理索引，以便將清單變更包含在新的掃描中。

## <a name="use-the-api-console"></a>使用 API 主控台

您需要有訂用帳戶金鑰，才能在線上主控台中試用 API。 此金鑰位於 [設定] 索引標籤的 [Ocp-Apim-Subscription-Key] 方塊中。 如需詳細資訊，請參閱[概觀](overview.md)。

## <a name="refresh-search-index"></a>重新整理搜尋索引

在您對字詞清單進行變更之後，必須重新整理其索引，變更才會包含在未來的掃描中。 此步驟就像桌面搜尋引擎 (如果已啟用) 或 Web 搜尋引擎持續重新整理其索引以包含新檔案或頁面一樣。

1.  在[字詞清單管理 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f)中，從左側功能表中選取 [Term Lists] \(字詞清單\)，然後選取 [Refresh Search Index] \(重新整理搜尋索引\)。 

  [Term Lists - Refresh Search Index] \(字詞清單 - 重新整理搜尋索引\) 頁面隨即開啟。

2. 針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。 

  ![[Term Lists - Refresh Search Index] \(字詞清單 - 重新整理搜尋索引\) 頁面區域選取項目](images/test-drive-region.png)

  [Term Lists - Refresh Search Index] \(字詞清單 - 重新整理搜尋索引\) API 主控台隨即開啟。

3.  在 [listId] 方塊中，輸入清單識別碼。 輸入您的訂用帳戶金鑰，然後選取 [Send] \(傳送\)。

  ![[Term Lists API - Refresh Search Index] \(字詞清單 API - 重新整理搜尋索引\) 主控台的 [Response content] \(回應內容\) 方塊](images/try-terms-list-refresh-1.png)

## <a name="create-a-term-list"></a>建立字詞清單
1.  移至[字詞清單管理 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f)。 

  [Term Lists - Create] \(字詞清單 - 建立\) 頁面隨即開啟。

2.  針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。 

  ![[Term Lists - Create] \(字詞清單 - 建立\) 頁面區域選取項目](images/test-drive-region.png)

  [Term Lists - Create] \(字詞清單 - 建立\) API 主控台隨即開啟。
 
3.  在 [Ocp-Apim-Subscription-Key] 中，輸入您的訂用帳戶金鑰。

4.  在 [Request body] \(要求本文\) 方塊中，輸入 [Name] \(名稱\) (例如 MyList) 和 [Description] \(描述\) 的值。

  ![[Term Lists - Create] \(字詞清單 - 建立\) 主控台的要求本文名稱和描述](images/try-terms-list-create-1.png)

5.  使用機碼值組預留位置來為清單指派更多描述性中繼資料。

        {
           "Name": "MyExclusionList",
           "Description": "MyListDescription",
           "Metadata": 
           {
              "Category": "Competitors",
              "Type": "Exclude"
           }
        }

  請以機碼值組形式新增清單中繼資料，而不要新增實際的字詞。
 
6.  選取 [傳送]。 您的清單隨即建立。 請注意與新清單關聯的 [ID] \(識別碼\) 值。 您需要有此識別碼，才能進行其他字詞清單管理功能。

  ![[Term Lists - Create] \(字詞清單 - 建立\) 主控台的 [Response content] \(回應內容\) 方塊顯示清單識別碼](images/try-terms-list-create-2.png)
 
7.  將字詞新增至 MyList。 在左側功能表中的 [字詞] 下方，選取 [新增字詞]。 

  [字詞 - 新增字詞] 頁面隨即開啟。 

8.  針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。 

  ![[字詞 - 新增字詞] 頁面區域選取項目](images/test-drive-region.png)

  [字詞 - 新增字詞] API 主控台隨即開啟。
 
9.  在 [listId] 方塊中，輸入產生的清單識別碼，並選取一個**語言**值。 輸入您的訂用帳戶金鑰，然後選取 [Send] \(傳送\)。

  ![[字詞 - 新增字詞] 主控台查詢參數](images/try-terms-list-create-3.png)
 
10. 若要確認是否已將該字詞新增至清單，請從左側功能表中，選取 [Term] \(字詞\)，然後選取 [Get All Terms] \(取得所有字詞\)。 

  [字詞 - 取得所有字詞] API 主控台隨即開啟。

11. 在 [listId] 方塊中，輸入清單識別碼，然後輸入您的訂用帳戶金鑰。 選取 [傳送]。

12. 在 [回應內容] 方塊中，確認您輸入的文字。

  ![[字詞 - 取得所有字詞] 主控台的 [回應內容] 方塊會列出您輸入的字詞](images/try-terms-list-create-4.png)
 
13. 再多新增幾個字詞。 現在您已建立自訂的字詞清單，可試著使用自訂的字詞清單[掃描某些文字](try-text-api.md)。 

## <a name="delete-terms-and-lists"></a>刪除字詞和清單

刪除字詞或清單相當簡單。 您可以使用 API 來執行下列工作：

- 刪除字詞。 (**字詞 - 刪除**)
- 刪除清單中的所有字詞，但不刪除清單。 (**字詞 - 刪除所有字詞**)
- 刪除清單及其所有內容。 (**字詞清單 - 刪除**)

此範例會刪除單一字詞。

1.  在[字詞清單管理 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f)中，從左側功能表中選取 [字詞]，然後選取 [刪除]。 

  [字詞 - 刪除] 隨即開啟。

2. 針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。 

  ![[字詞 - 刪除] 頁面區域選取項目](images/test-drive-region.png)

  [字詞 - 刪除] API 主控台隨即開啟。
  
3.  在 [listId] 方塊中，輸入要從中刪除字詞的清單識別碼。 此識別碼是 MyList [字詞清單 - 取得詳細資料] 主控台中傳回的數字 (在我們的範例中是 **122**)。 輸入字詞並選取語言。
 
  ![[字詞 - 刪除] 主控台查詢參數](images/try-terms-list-delete-1.png)

4.  輸入您的訂用帳戶金鑰，然後選取 [Send] \(傳送\)。

5.  若要確認是否已刪除字詞，請使用 [字詞清單 - 取得所有項目] 主控台。

  ![[字詞清單 - 取得所有項目] 主控台的 [回應內容] 方塊會顯示刪除的字詞](images/try-terms-list-delete-2.png)
 
## <a name="change-list-information"></a>變更清單資訊

您可以編輯清單的名稱和描述，以及新增中繼資料項目。

1.  在[字詞清單管理 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f)中，從左側功能表中選取 [字詞清單]，然後選取 [更新詳細資料]。 

  [字詞清單 - 更新詳細資料] 頁面隨即開啟。

2. 針對 [Open API testing console] \(開啟 API 測試主控台\) 中，選取最能描述您位置的區域。 

  ![[字詞清單 - 更新詳細資料] 頁面區域選取項目](images/test-drive-region.png)

  [字詞清單 - 更新詳細資料] API 主控台隨即開啟。

3.  在 [listId] 方塊中，輸入清單識別碼，然後輸入您的訂用帳戶金鑰。

4.  在 [要求本文] 方塊中進行編輯，然後選取 [傳送] 按鈕。

  ![[字詞清單 - 更新詳細資料] 主控台的 [要求本文] 編輯](images/try-terms-list-change-1.png)
 

## <a name="next-steps"></a>後續步驟

在您的程式碼中使用 REST API，或從[字詞清單 .NET 快速入門](term-lists-quickstart-dotnet.md)開始著手，以便與您的應用程式進行整合。
