---
title: 疑難排解和診斷失敗 - Azure Logic Apps | Microsoft Docs
description: 了解如何針對 Azure Logic Apps 中的失敗進行疑難排解和診斷
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, jehollan, LADocs
ms.topic: article
ms.assetid: a6727ebd-39bd-4298-9e68-2ae98738576e
ms.date: 10/15/2017
ms.openlocfilehash: 994e7945a7107815029bd415f4cc0d45bb68e335
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123682"
---
# <a name="troubleshoot-and-diagnose-workflow-failures-in-azure-logic-apps"></a>針對 Azure Logic Apps 中的失敗進行疑難排解和診斷

您的邏輯應用程式會產生可協助您診斷和偵錯應用程式邏輯中問題的資訊。 您可以藉由在 Azure 入口網站檢閱工作流程中的每個步驟來診斷邏輯應用程式。 或者您可以新增一些步驟至工作流程，以進行執行階段偵錯。

## <a name="review-trigger-history"></a>檢閱觸發程序歷程記錄

每個邏輯應用程式會從觸發程序開始。 如果沒有引發觸發程序，請先檢查觸發程序歷程記錄。 這個歷程記錄會列出您的邏輯應用程式進行的所有觸發程序嘗試，以及有關每個觸發程序嘗試的輸入和輸出詳細資料。

1. 若要檢查是否引發觸發程序，請在邏輯應用程式功能表上選擇 [概觀]。 在 [觸發程序歷程記錄] 下，檢閱觸發程序的狀態。

   > [!TIP]
   > 如果您沒有看到邏輯應用程式功能表，請嘗試返回 Azure 儀表板，然後重新開啟邏輯應用程式。

   ![檢閱觸發程序歷程記錄](./media/logic-apps-diagnosing-failures/logic-app-trigger-history-overview.png)

   > [!TIP]
   > * 如果找不到您預期的資料，請嘗試在工具列上選取 [重新整理]。
   > * 如果此清單顯示許多觸發程序嘗試，而找不到您要的項目，請嘗試篩選清單。

   以下是觸發程序嘗試的可能狀態：

   | 狀態 | 說明 | 
   | ------ | ----------- | 
   | **已成功** | 觸發程序檢查了端點，並找到可用的資料。 通常，「已引發」狀態也會隨著這個狀態一起出現。 如果沒有，則觸發程序定義可能有條件或 `SplitOn` 命令不符合要求。 <p>此狀態可以套用至手動觸發程序、循環觸發程序或輪詢觸發程序。 觸發程序可以順利執行，但執行本身在動作產生未處理的錯誤時可能會失敗。 | 
   | **已略過** | 觸發程序檢查了端點，但不找到任何資料。 | 
   | **已失敗** | 發生錯誤。 若要檢閱失敗的觸發程序所產生的任何錯誤訊息，請選取該觸發程序嘗試並選擇 [輸出]。 例如，您可能會發現無效的輸入。 | 
   ||| 

   可能有多個觸發程序項目具有相同的日期和時間，當應用程式邏輯找到多個項目時會發生此情況。 
   每次引發觸發程序時，Logic Apps 引擎會建立邏輯應用程式執行個體，以執行您的工作流程。 根據預設，每個執行個體會並行執行，使得任何工作流程不需等待即可開始執行。

   > [!TIP]
   > 您可以重新檢查觸發程序，而不需等待下一個週期。 在概觀工具列上，選擇 [執行觸發程序]，然後選取觸發程序，這會強制進行檢查。 或者，在 Logic Apps 設計工具工具列上選取 [執行]。

3. 若要檢查詳細資料的觸發程序嘗試，請在 [觸發程序歷程記錄] 下選取該觸發程序嘗試。 

   ![選取觸發程序嘗試](./media/logic-apps-diagnosing-failures/logic-app-trigger-history.png)

4. 檢閱觸發程序產生的任何輸入和輸出。 觸發程序輸出會顯示來自觸發程序的資料。 這些輸出可協助您判斷所有屬性是否如預期般傳回。

   > [!NOTE]
   > 如果您看見任何不了解的內容，請了解 Azure Logic Apps 如何[處理不同的內容類型](../logic-apps/logic-apps-content-type.md)。

   ![觸發程序輸出](./media/logic-apps-diagnosing-failures/trigger-outputs.png)

## <a name="review-run-history"></a>檢閱執行歷程記錄

每個引發的觸發程序會開始一個工作流程執行。 您可以檢閱該執行期間發生什麼，包括工作流程中的每個步驟的狀態，加上每個步驟的輸入和輸出。

1. 在邏輯應用程式功能表上，選擇 [概觀]。 在 [執行歷程記錄] 下，檢閱所引發的觸發程序執行。

   > [!TIP]
   > 如果您沒有看到邏輯應用程式功能表，請嘗試返回 Azure 儀表板，然後重新開啟邏輯應用程式。

   ![檢閱執行歷程記錄](./media/logic-apps-diagnosing-failures/logic-app-runs-history-overview.png)

   > [!TIP]
   > * 如果找不到您預期的資料，請嘗試在工具列上選取 [重新整理]。
   > * 如果此清單顯示許多執行，而找不到您要的項目，請嘗試篩選清單。

   以下是執行的可能狀態：

   | 狀態 | 說明 | 
   | ------ | ----------- | 
   | **已成功** | 所有動作都已成功。 <p>如果特定動作發生任何失敗，工作流程中的後續動作已處理該失敗。 | 
   | **已失敗** | 至少一個動作失敗，並且工作流程中沒有設定後續動作來處理失敗。 | 
   | **已取消** | 工作流程正執行中，但收到了取消要求。 | 
   | **執行中** | 工作流程目前正在執行中。 <p>節流處理的流程可能會出現此狀態，或者因為目前的定價方案所致。 如需詳細資訊，請參閱[定價頁面上的動作限制](https://azure.microsoft.com/pricing/details/logic-apps/)。 如果您設定[診斷記錄](../logic-apps/logic-apps-monitor-your-logic-apps.md)，則也可以取得所發生任何節流事件的相關資訊。 | 
   ||| 

2. 檢閱特定執行中每個步驟的詳細資料。 在 [執行歷程記錄] 下，選取您想要檢查的執行。

   ![檢閱執行歷程記錄](./media/logic-apps-diagnosing-failures/logic-app-run-history.png)

   執行本身成功或失敗，[執行詳細資料] 檢視會顯示每個步驟以及它們是成功或失敗。

   ![檢視邏輯應用程式執行的詳細資料](./media/logic-apps-diagnosing-failures/logic-app-run-details.png)

3. 若要檢查特定步驟的輸入、輸出和任何錯誤訊息，請選擇該步驟，以便圖形展開並顯示詳細資料。 例如︰

   ![檢視步驟詳細資料](./media/logic-apps-diagnosing-failures/logic-app-run-details-expanded.png)

## <a name="perform-runtime-debugging"></a>執行執行階段偵錯

若要協助偵錯，您可以新增診斷步驟至工作流程，以及檢閱觸發程序和執行歷程記錄。 例如，您可以新增使用 [Webhook Tester](https://webhook.site/) 服務的步驟，使得您可以檢查 HTTP 要求，並判斷其確切的大小、圖形與格式。

1. 瀏覽 [Webhook Tester](https://webhook.site/) 並複製所建立的唯一 URL

2. 在您的邏輯應用程式中，以您想要測試的任何內文內容新增 HTTP POST 動作 (例如，運算式或另一個步驟輸出)。

3. 將 Webhook Tester 的 URL 貼上至 HTTP POST 動作。

4. 從 Logic Apps 引擎產生時，若要檢閱要求如何形成，請執行邏輯應用程式，並參閱 Webhook Tester 以取得詳細資料。

## <a name="next-steps"></a>後續步驟

[監視邏輯應用程式](../logic-apps/logic-apps-monitor-your-logic-apps.md)
