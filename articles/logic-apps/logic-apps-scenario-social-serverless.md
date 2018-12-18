---
title: 無伺服器案例 - 使用 Azure 服務建立客戶深入解析儀表板 | Microsoft Docs
description: 使用 Azure Logic Apps 與 Azure Functions 來建置客戶儀表板，以管理客戶的意見反應、社交媒體資料和其他項目
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: jeffhollan
ms.author: jehollan
ms.reviewer: estfan, LADocs
ms.assetid: d565873c-6b1b-4057-9250-cf81a96180ae
ms.topic: article
ms.date: 03/15/2018
ms.openlocfilehash: 6ba274bb3ff3679b4a44950db168215c54f3ade6
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44299741"
---
# <a name="create-streaming-customer-insights-dashboard-with-azure-logic-apps-and-azure-functions"></a>使用 Azure Logic Apps 與 Azure Functions 來建立串流的客戶深入解析儀表板

Azure 提供的無伺服器工具，可協助您在雲端中快速建置及裝載應用程式，而不必考慮基礎結構。 在此教學課程中，您可以建立會對客戶的意見反應而觸發的儀表板、使用機器學習服務分析意見反應，並將深入解析發佈至來源，例如 Power BI 或 Azure Data Lake。

針對此解決方案，您將使用以下無伺服器應用程式的主要 Azure 元件：[Azure Functions](https://azure.microsoft.com/services/functions/) 和 [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/)。
Azure Logic Apps 在雲端中提供無伺服器工作流程引擎，讓您可以在無伺服器元件之間建立協調流程，並連線到 200 個以上的服務和 API。 Azure Functions 可在雲端中提供無伺服器計算。 此解決方案會根據預先定義的關鍵字，使用 Azure Functions 為客戶的推文加上旗標。

在此案例中，您建立的邏輯應用程式會在找到客戶意見反應時觸發。 可協助您回應客戶意見反應的一些連接器包括 Outlook.com、Office 365、Survey Monkey、Twitter 和[來自 Web 表單的 HTTP 要求](https://blogs.msdn.microsoft.com/logicapps/2017/01/30/calling-a-logic-app-from-an-html-form/)。 您所建立的工作流程會監視 Twitter 上的主題標籤。

您可以[在 Visual Studio 中建置整個解決方案](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)，並[使用 Azure Resource Manager 範本部署此解決方案](../logic-apps/logic-apps-create-deploy-template.md)。 如需如何建立此解決方案的逐步解說影片，請[觀看此 Channel 9 影片](http://aka.ms/logicappsdemo)。 

## <a name="trigger-on-customer-data"></a>根據客戶資料進行觸發

1. 在 Azure 入口網站或 Visual Studio 中，建立空白的邏輯應用程式。 

   如果您還不熟悉邏輯應用程式，請檢閱 [Azure 入口網站快速入門](../logic-apps/quickstart-create-first-logic-app-workflow.md)或 [Visual Studio 快速入門](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)。

2. 在邏輯應用程式設計工具中，尋找並新增具有以下動作的 Twitter 觸發程序：**有新推文張貼時**

3. 設定觸發程序以根據關鍵字或主題標籤來接聽推文。

   在 Twitter 觸發程序這類的輪詢式觸發程序上，循環屬性會決定邏輯應用程式檢查新項目的頻率。

   ![Twitter 觸發程序的範例][1]

此邏輯應用程式現在會根據所有新推文而引發。 然後，您可以使用推文資料並加以分析，進而更了解其中所表達的情感。 

## <a name="analyze-tweet-text"></a>分析推文中的文字

若要偵測某些文字後的情感，您可以使用 [Azure 認知服務](https://azure.microsoft.com/services/cognitive-services/)。

1. 在邏輯應用程式設計工具中的觸發程序之下，選擇 [新增步驟]。

2. 尋找 [文字分析] 連接器。

3. 選取 [偵測情感] 動作。

4. 如果出現提示，請提供適用於文字分析服務的有效認知服務金鑰。

5. 在 [要求本文] 下，選取 [推文文字] 欄位，以提供推文文字作為分析的輸入。

取得推文資料與推文的深入解析之後，您現在可以使用其他數個相關的連接器以及其動作：

* **Power BI - 將資料列新增至串流資料集**︰在 Power BI 儀表板上檢視傳入的推文。
* **Azure Data Lake - 附加檔案**︰將客戶資料新增至要納入分析作業的 Azure Data Lake 資料集。
* **SQL - 新增資料列**︰將資料儲存在資料庫中，以供日後擷取。
* **Slack - 傳送訊息**︰通知 Slack 通道有關可能需要有所行動的負面意見反應。

您也可以建立 Azure 函式，以便對資料執行自訂處理。 

## <a name="process-data-with-azure-functions"></a>使用 Azure Functions 處理資料

建立函式之前，請先在 Azure 訂用帳戶中建立函式應用程式。 此外，為了讓邏輯應用程式直接呼叫函式，函式必須具有 HTTP 觸發程序繫結，例如，可使用 **HttpTrigger** 範本。 了解[如何在 Azure 入口網站中建立您的第一個函式應用程式和涵式](../azure-functions/functions-create-first-azure-function-azure-portal.md)。

針對此案例，請使用推文文字作為 Azure 函式的要求本文。 在您的函式程式碼中，定義用來判斷推文文字是否包含關鍵字或片語的邏輯。 根據案例的需要，維持簡單或複雜的函式。
在函式結尾處，將包含某些資料的回應傳回邏輯應用程式，例如，簡單的布林值 (例如 `containsKeyword`) 或複雜的物件。

> [!TIP]
> 若要存取邏輯應用程式函式中的複雜回應，請使用**剖析 JSON** 動作。

完成後，請儲存函式，然後在您建置的邏輯應用程式中，將此函式新增為動作。

## <a name="add-azure-function-to-logic-app"></a>將 Azure 函式新增至邏輯應用程式

1. 在邏輯應用程式設計工具中的 [偵測情感] 動作下，選擇 [新增步驟]。

2. 尋找 **Azure Functions** 連接器，然後選取您建立的函式。

3. 在 [要求本文] 下，選取 [推文文字]。

![已設定的 Azure Function 步驟][2]

## <a name="run-and-monitor-your-logic-app"></a>執行邏輯應用程式並加以監視

若要檢閱目前或先前任何應用程式邏輯的執行，您可以在 Azure 入口網站或 Visual Studio 中，或是透過 Azure REST API 和 SDK，使用 Azure Logic Apps 提供的多種偵錯和監視功能。

若要輕鬆地測試邏輯應用程式，請在邏輯應用程式設計工具中選擇 [執行觸發程序]。 觸發程序會根據您指定的排程，對推文進行輪詢，直到找到符合準則的推文。 執行進行時，設計工具會顯示該執行的即時檢視。

若要在 Visual Studio 或 Azure 入口網站中檢視之前的執行記錄： 

* 開啟 Visual Studio Cloud Explorer。 尋找邏輯應用程式，開啟應用程式的捷徑功能表。 選取 [開啟執行歷程記錄]。

* 在 Azure 入口網站中，尋找您的邏輯應用程式。 在邏輯應用程式功能表上，選擇 [概觀]。 

## <a name="create-automated-deployment-templates"></a>建立自動部署範本

建立邏輯應用程式解決方案之後，您可以擷取您的應用程式並將其當作 [Azure Resource Manager 範本](../azure-resource-manager/resource-group-overview.md#template-deployment)，部署至世界上任何 Azure 區域。 您可以使用這項功能來修改參數，以建立應用程式的不同版本，以及將您的解決方案整合至 Azure Pipelines。 您也可以將 Azure Functions 納入部署範本中，以便將整個解決方案與所有相依性當作單一範本來管理。 了解[如何建立邏輯應用程式部署範本](../logic-apps/logic-apps-create-deploy-template.md)。

如需 Azure 函式的部署範本範例，請查看 [Azure 快速入門範本存放庫](https://github.com/Azure/azure-quickstart-templates/tree/master/101-function-app-create-dynamic)。

## <a name="next-steps"></a>後續步驟

* [尋找 Azure Logic Apps 的其他範例和案例](logic-apps-examples-and-scenarios.md)

<!-- Image References -->
[1]: ./media/logic-apps-scenario-social-serverless/twitter.png
[2]: ./media/logic-apps-scenario-social-serverless/function.png