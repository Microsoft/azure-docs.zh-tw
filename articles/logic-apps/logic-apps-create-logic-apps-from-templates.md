---
title: 從範本建立工作流程 - Azure Logic Apps | Microsoft Docs
description: 使用 Azure Logic Apps 中的邏輯應用程式範本加快建置工作流程的速度
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: kevinlam1
ms.author: klam
ms.reviewer: estfan, LADocs
ms.topic: article
ms.assetid: 3656acfb-eefd-4e75-b5d2-73da56c424c9
ms.date: 10/15/2017
ms.openlocfilehash: 10191a4fbab325dcd5134b082f050188c6798079
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43122115"
---
# <a name="create-logic-app-workflows-from-prebuilt-templates"></a>從預先建置的範本建立邏輯應用程式工作流程

Logic Apps 提供範本來協助您更快速開始建立工作流程，這些範本是依循常用模式的預先建置邏輯應用程式。 您可以直接使用這些提供的範本，或是加以編輯來符合您的案例需求。

以下是一些範本類別：

| 範本類型 | 說明 | 
| ------------- | ----------- | 
| 企業雲端範本 | 用來整合 Azure Blob、Dynamics CRM、Salesforce、Box，並且包含其他可滿足您企業雲端需求的連接器。 例如，您可以使用這些範本來組織業務潛在客戶，或備份公司檔案資料。 | 
| 個人生產力範本 | 藉由設定每日提醒、將重要的工作項目轉換成待辦事項清單，以及將冗長的工作自動縮減成單一的使用者核准步驟，提升個人生產力。 | 
| 取用者雲端範本 | 用來整合社交媒體服務，例如 Twitter、Slack 及電子郵件。 適用於加強社交媒體行銷計畫。 這些範本也包含雲端複製之類的工作，可藉由節省在傳統重複性工作上花費的時間來提升生產力。 | 
| 企業整合套件範本 | 用來設定 VETER (驗證、擷取、轉換、充實、路由傳送) 管線、透過 AS2 接收 X12 EDI 文件並轉換成 XML，以及處理 X12、EDIFACT 和 AS2 訊息。 | 
| 通訊協定模式範本 | 用來實作通訊協定模式，例如透過 HTTP 的要求回應及跨 FTP 和 SFTP 的整合。 您可以直接使用這些提供的範本，或以它們作為基礎來建置複雜的通訊協定模式。 | 
||| 

如果您沒有 Azure 訂用帳戶，請在開始前，先[註冊一個免費的 Azure 帳戶](https://azure.microsoft.com/free/)。 如需有關建置邏輯應用程式的詳細資訊，請參閱[建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="create-logic-apps-from-templates"></a>從範本建立邏輯應用程式

1. 如果您尚未登入 [Azure 入口網站](https://portal.azure.com "Azure 入口網站")，請先登入。

2. 從主要 Azure 功能表，選擇 [建立資源] > [企業整合] > [邏輯應用程式]。

   ![Azure 入口網站, 新增, 企業整合, 邏輯應用程式](./media/logic-apps-create-logic-apps-from-templates/azure-portal-create-logic-app.png)

3. 使用下圖底下表格中的設定來建立您的邏輯應用程式：

   ![提供邏輯應用程式詳細資料](./media/logic-apps-create-logic-apps-from-templates/logic-app-settings.png)

   | 設定 | 值 | 說明 | 
   | ------- | ----- | ----------- | 
   | **名稱** | *邏輯應用程式的名稱* | 提供唯一的邏輯應用程式名稱。 | 
   | **訂用帳戶** | 您的 Azure 訂用帳戶名稱 | 選取您要使用的 Azure 訂用帳戶。 | 
   | **資源群組** | 您的 Azure 資源群組名稱 | 為此邏輯應用程式建立或選取一個 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)，並用以組織與此應用程式關聯的所有資源。 | 
   | **位置** | 您的 Azure 資料中心區域 | 選取用於部署邏輯應用程式的資料中心區域，例如，美國西部。 | 
   | **Log Analytics** | **Off** (預設值) 或 **On** | 透過 [Azure Log Analytics](../log-analytics/log-analytics-overview.md) 開啟您邏輯應用程式的[診斷記錄](../logic-apps/logic-apps-monitor-your-logic-apps.md#turn-on-diagnostics-logging-for-your-logic-app)。 您必須已經有 Log Analytics 工作區。 | 
   |||| 

4. 當您準備就緒時，請選取 [釘選到儀表板]。 這樣一來，邏輯應用程式就會在部署之後自動出現在 Azure 儀表板上並開啟。 選擇 [建立] 。

   > [!NOTE]
   > 如果您不想要釘選邏輯應用程式，就必須在部署之後以手動方式尋找並開啟邏輯應用程式，才能繼續操作。

   在 Azure 部署您的邏輯應用程式之後，「Logic Apps 設計工具」會隨即開啟，並顯示含有簡介影片的分頁。 
   您可以在影片底下找到一般邏輯應用程式模式的範本。 

5. 捲動到 [範本] \(中間會經過簡介影片和一般觸發程序)。 選擇預先建置的範本。 例如︰

   ![選擇邏輯應用程式範本](./media/logic-apps-create-logic-apps-from-templates/choose-logic-app-template.png)

   > [!TIP]
   > 若要從頭開始建置您的邏輯應用程式，請選擇 [空白邏輯應用程式]。

   選取預先建置的範本時，您可以檢視該範本的相關詳細資訊。 
   例如︰

   ![選擇預先建置的範本](./media/logic-apps-create-logic-apps-from-templates/logic-app-choose-prebuilt-template.png)

6. 若要使用選取的範本繼續操作，請選擇 [使用此範本]。 

7. 根據範本中的連接器，將會提示您執行下列任一步驟：

   * 使用您的認證來登入範本所參考的系統或服務。

   * 針對範本所參考的任何服務或系統建立連線。 若要建立連線，請為連線提供名稱，並視需要選取您想要使用的資源。 

   * 如果您已經設定好這些連線，請選擇 [繼續]。

   例如︰

   ![建立連線](./media/logic-apps-create-logic-apps-from-templates/logic-app-create-connection.png)

   當您完成時，邏輯應用程式會在「Logic Apps 設計工具」中開啟並顯示。

   > [!TIP]
   > 若要返回範本檢視器，請選擇設計工具之工具列上的 [範本]。 此動作會捨棄所有未儲存的變更，因此會出現警告訊息來確認您的要求。

8. 繼續建置您的邏輯應用程式。

   > [!NOTE] 
   > 許多範本會包含可能已經預先填入必要屬性的連接器。 不過，有些範本可能還是需要您先提供值，您才能正確地部署邏輯應用程式。 如果您不完成遺漏的屬性欄位就嘗試部署，將會收到錯誤訊息。 

## <a name="update-logic-apps-with-templates"></a>使用範本來更新邏輯應用程式

1. 在 [Azure 入口網站](https://portal.azure.com "Azure 入口網站")的「邏輯應用程式設計工具」中，尋找並開啟您的邏輯應用程式。

2. 在設計工具的工具列上，選擇 [範本]。 此動作會捨棄所有未儲存的變更，因此會出現警告訊息，供您確認是否要繼續。 若要確認，請選擇 [確定]。 例如︰

   ![選擇 [範本]](./media/logic-apps-create-logic-apps-from-templates/logic-app-update-existing-with-template.png)

3. 捲動到 [範本] \(中間會經過簡介影片和一般觸發程序)。 選擇預先建置的範本。 例如︰

   ![選擇邏輯應用程式範本](./media/logic-apps-create-logic-apps-from-templates/choose-logic-app-template.png)

   選取預先建置的範本時，您可以檢視該範本的相關詳細資訊。 
   例如︰

   ![選擇預先建置的範本](./media/logic-apps-create-logic-apps-from-templates/logic-app-choose-prebuilt-template.png)

4. 若要使用選取的範本繼續操作，請選擇 [使用此範本]。 

5. 根據範本中的連接器，將會提示您執行下列任一步驟：

   * 使用您的認證來登入範本所參考的系統或服務。

   * 針對範本所參考的任何服務或系統建立連線。 若要建立連線，請為連線提供名稱，並視需要選取您想要使用的資源。 

   * 如果您已經設定好這些連線，請選擇 [繼續]。

   ![建立連線](./media/logic-apps-create-logic-apps-from-templates/logic-app-create-connection.png)

   您的邏輯應用程式現在會在「Logic Apps 設計工具」中開啟並顯示。

8. 繼續建置您的邏輯應用程式。 

   > [!TIP]
   > 如果您尚未儲存變更，則可以捨棄工作並返回先前的邏輯應用程式。 在設計工具的工具列上，選擇 [捨棄]。

> [!NOTE] 
> 許多範本會包含可能已經預先填入必要屬性的連接器。 不過，有些範本可能還是需要您先提供值，您才能正確地部署邏輯應用程式。 如果您不完成遺漏的屬性欄位就嘗試部署，將會收到錯誤訊息。

## <a name="deploy-logic-apps-built-from-templates"></a>從範本部署邏輯應用程式

對範本進行變更之後，您可以儲存所做的變更。 此動作也會自動發佈您的邏輯應用程式。

在設計工具的工具列上，選擇 [儲存]。

![儲存並發佈您的邏輯應用程式](./media/logic-apps-create-logic-apps-from-templates/logic-app-save.png)  

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

透過範例、情節、客戶案例及逐步解說，了解如何建置邏輯應用程式。

> [!div class="nextstepaction"]
> [檢閱邏輯應用程式範例、情節及逐步解說](../logic-apps/logic-apps-examples-and-scenarios.md)