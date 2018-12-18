---
title: 連線到 Bing 搜尋 - Azure Logic Apps | Microsoft Docs
description: 使用 Bing 搜尋 REST API 和 Azure Logic Apps 尋找新聞
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.date: 05/21/2018
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: 8ac67f9df0e5baccc668c2aeb70f65d96e574df5
ms.sourcegitcommit: 0fa8b4622322b3d3003e760f364992f7f7e5d6a9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37021280"
---
# <a name="find-news-with-bing-search-and-azure-logic-apps"></a>使用 Bing 搜尋和 Azure Logic Apps 尋找新聞 

本文會示範如何從邏輯應用程式與 Bing 搜尋連接器，透過 Bing 搜尋尋找新聞、影片和其他項目。 這樣一來，您可以為處理搜尋結果建立可自動化工作和工作流程的邏輯應用程式，並讓這些項目可供其他動作使用。 

例如，您可以根據搜尋準則尋找新聞項目，並讓 Twitter 張貼這些項目作為 Twitter 摘要中的推文。

如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 如果您還不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)和[快速入門：建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。
如需連接器專屬的技術資訊，請參閱 <a href="https://docs.microsoft.com/connectors/bingsearch/" target="blank">Bing 搜尋連接器參考</a>。

## <a name="prerequisites"></a>先決條件

* [認知服務帳戶](../cognitive-services/cognitive-services-apis-create-account.md)

* [Bing 搜尋 API 金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-news-search-api)，可提供從邏輯應用程式存取 Bing 搜尋 API 的權限 

* 您要存取事件中樞的邏輯應用程式。 若要使用 Bing 搜尋觸發程序啟動邏輯應用程式，您需要[空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 

<a name="add-trigger"></a>

## <a name="add-a-bing-search-trigger"></a>新增 Bing 搜尋觸發程序

在 Azure Logic Apps 中，每個邏輯應用程式都必須使用[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)啟動，而該觸發程序會在特定事件發生或符合特定條件時引發。 每次引發觸發程序時，Logic Apps 引擎都會建立邏輯應用程式執行個體，並開始執行應用程式的工作流程。

1. 在 Azure 入口網站或 Visual Studio 中，建立空白的邏輯應用程式，以開啟邏輯應用程式設計工具。 這個範例會使用 Azure 入口網站。

2. 在搜尋方塊中，輸入 "Bing 搜尋" 作為篩選條件。 從觸發程序清單中，選取您想要的觸發程序。 

   這個範例使用此觸發程序：**Bing 搜尋 - 新的新聞文章**

   ![尋找 Bing 搜尋觸發程序](./media/connectors-create-api-bing-search/add-trigger.png)

3. 如果系統提示您輸入連線詳細資料，請[立即建立 Bing 搜尋連線](#create-connection)。 或者，如果連線已存在，請提供觸發程序的必要資訊。

   在此範例中，針對從 Bing 搜尋傳回相符的新聞文章提供準則。

   | 屬性 | 必要 | 值 | 說明 | 
   |----------|----------|-------|-------------| 
   | 搜尋查詢 | yes | <*search-words*> | 輸入您要使用的搜尋關鍵字。 |
   | 市場 | yes | <*locale*> | 搜尋地區設定。 預設為 "en-US"，但您可以選取其他值。 | 
   | 安全搜尋 | yes | <*search-level*> | 排除成人內容的篩選層級。 預設為「中度」，但您可以選取其他層級。 | 
   | Count | 否 | <*results-count*> | 傳回指定數目的結果。 預設為 20，但您可以指定其他值。 傳回結果的實際數目可能小於指定數目。 | 
   | Offset | 否 | <*skip-value*> | 傳回結果前要跳過的結果數目 | 
   ||||| 

   例如︰

   ![設定觸發程序](./media/connectors-create-api-bing-search/bing-search-trigger.png)

4. 選取要讓觸發程序檢查結果的間隔和頻率。

5. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。 

6. 現在，繼續針對您想要使用觸發程序結果來執行的工作，於邏輯應用程式中新增一或多個動作。

<a name="add-action"></a>

## <a name="add-a-bing-search-action"></a>新增 Bing 搜尋動作

在 Azure Logic Apps 中，[動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)是工作流程中跟隨在觸發程序或另一個動作之後的步驟。 在此範例中，邏輯應用程式會從 Bing 搜尋觸發程序開始，其會傳回符合指定準則的新聞文章。 

1. 在 Azure 入口網站或 Visual Studio 的邏輯應用程式設計工具中，開啟邏輯應用程式。 這個範例會使用 Azure 入口網站。

2. 在觸發程序或動作底下，選擇 [新增步驟] > [新增動作]。

   這個範例使用此觸發程序：**Bing 搜尋 - 新的新聞文章**

   ![新增動作](./media/connectors-create-api-bing-search/add-action.png)

   若要在現有步驟之間新增動作，請將滑鼠放在連接箭頭上。 
   選擇顯示的加號 (**+**)，然後選擇 [新增動作]。

3. 在搜尋方塊中，輸入 "Bing 搜尋" 作為篩選條件。
從 [動作] 清單中，選取您想要的動作。

   這個範例使用此動作：**Bing 搜尋 - 按查詢列出新聞**

   ![尋找 Bing 搜尋動作](./media/connectors-create-api-bing-search/bing-search-select-action.png)

4. 如果系統提示您輸入連線詳細資料，請[立即建立 Bing 搜尋連線](#create-connection)。 或者，如果連線已存在，請提供動作的必要資訊。 

   在此範例中，針對傳回觸發程序結果的子集提供準則。

   | 屬性 | 必要 | 值 | 說明 | 
   |----------|----------|-------|-------------| 
   | 搜尋查詢 | yes | <*search-expression*> | 輸入查詢觸發程序結果的運算式。 您可以從動態內容清單中的欄位選取，或使用運算式產生器建立運算式。 |
   | 市場 | yes | <*locale*> | 搜尋地區設定。 預設為 "en-US"，但您可以選取其他值。 | 
   | 安全搜尋 | yes | <*search-level*> | 排除成人內容的篩選層級。 預設為「中度」，但您可以選取其他層級。 | 
   | Count | 否 | <*results-count*> | 傳回指定數目的結果。 預設為 20，但您可以指定其他值。 傳回結果的實際數目可能小於指定數目。 | 
   | Offset | 否 | <*skip-value*> | 傳回結果前要跳過的結果數目 | 
   ||||| 

   例如，假設您想要的結果其類別名稱包括 "tech" 這個字。 
   
   1. 按一下 [搜尋查詢] 方塊，動態內容清單隨即出現。 
   從該清單中選擇 [運算式]，運算式產生器隨即出現。 

      ![Bing 搜尋觸發程序](./media/connectors-create-api-bing-search/bing-search-action.png)

      現在您可以開始建立運算式。

   2. 從函式清單中選取 **contains()** 函式，它會出現在運算式方塊中。 按一下 [動態內容]，欄位清單會再次出現，但請確認您的資料指標停留在括號內。

      ![選取函式](./media/connectors-create-api-bing-search/expression-select-function.png)

   3. 從欄位清單中選取 [類別]，它會轉換成參數。 
   在第一個參數後新增逗號，並在逗號後新增這個字：`'tech'` 

      ![選取欄位](./media/connectors-create-api-bing-search/expression-select-field.png)
   
   4. 完成時，選擇 [確定]。

      運算式現在會出現在 [搜尋查詢] 方塊中，格式如下：

      ![完成的運算式](./media/connectors-create-api-bing-search/resolved-expression.png)

      在程式碼檢視中，這個運算式會以下列格式出現：

      `"@{contains(triggerBody()?['category'],'tech')}"`

5. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。

<a name="create-connection"></a>

## <a name="connect-to-bing-search"></a>連線到 Bing 搜尋

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)] 

1. 當系統提示您需要連線資訊時，請提供以下詳細資料：

   | 屬性 | 必要 | 值 | 說明 | 
   |----------|----------|-------|-------------| 
   | 連線名稱 | yes | <*connection-name*> | 要為連線建立的名稱 |
   | API 版本 | yes | <*API-version*> | 依預設，Bing 搜尋 API 版本會設定為目前的版本。 您可以視需要選取較舊版本。 | 
   | API 金鑰 | yes | <*API-key*> | 您稍早取得的 Bing 搜尋 API 金鑰。 如果您沒有金鑰，請立即取得 [API 金鑰](https://azure.microsoft.com/try/cognitive-services/?api=bing-news-search-api)。 |  
   |||||  

   例如︰

   ![建立連線](./media/connectors-create-api-bing-search/bing-search-create-connection.png)

2. 完成之後，請選擇 [建立]。

## <a name="connector-reference"></a>連接器參考

如需連接器的 Swagger 檔案所敘述的技術詳細資料 (例如，觸發程序、動作和限制)，請參閱[連接器的參考頁面](/connectors/bingsearch/)。 

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
