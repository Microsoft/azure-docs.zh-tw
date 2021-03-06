---
title: 教學課程：使用 Azure Functions 管理計算
description: 如何使用 Azure Functions 來管理專屬 SQL 集區的計算， (先前的 SQL DW) Azure Synapse Analytics。
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/27/2018
ms.author: jrasnick
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: f0731f0deaf46ec419cfe43037804e10f2b73fd4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/01/2020
ms.locfileid: "96448378"
---
# <a name="use-azure-functions-to-manage-compute-resources-for-your-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>使用 Azure Functions 來管理專屬 SQL 集區的計算資源 (先前的 SQL DW) Azure Synapse Analytics

本教學課程會使用 Azure Functions 來管理專屬 SQL 集區的計算資源， (先前的 SQL DW) Azure Synapse Analytics。

若要使用具有專用 SQL 集區的 Azure 函數應用程式 (先前的 SQL DW) ，您必須建立 [服務主體帳戶](../../active-directory/develop/howto-create-service-principal-portal.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。 服務主體帳戶需要與您專用的 SQL 集區位於相同訂用帳戶下的參與者存取權， (先前的 SQL DW) 實例。

## <a name="deploy-timer-based-scaling-with-an-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本部署以計時器為基礎的 Scaler

部署此範本，您需要下列資訊：

- 您專用的 SQL 集區 (先前的 SQL DW) 實例所在的資源組名
- 您專用的 SQL 集區 (先前的 SQL DW) 實例所在的伺服器名稱
- 您專用的 SQL 集區名稱 (先前為 SQL DW) 實例
- Azure Active Directory 的租用戶識別碼 (目錄識別碼)
- 訂用帳戶識別碼
- 服務主體的應用程式識別碼
- 服務主體祕密金鑰

一旦取得前述資訊，即可部署此範本：

[![顯示標示為「部署至 Azure」的按鈕影像。](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Fsql-data-warehouse-samples%2Fmaster%2Farm-templates%2FsqlDwTimerScaler%2Fazuredeploy.json)

部署此範本後，您應會發現三項新資源：免費的 Azure App Service 方案、以耗用量為基礎的函式應用程式方案，以及處理記錄和作業佇列的儲存體帳戶。 繼續閱讀其他各節，以了解如何修改已部署的函式來符合您的需求。

## <a name="change-the-compute-level"></a>變更計算層級

1. 瀏覽到函式應用程式服務。 如果您部署的範本採用預設值，此服務應該命名為 DWOperations。 您的函式應用程式開啟後，您應會注意到有五個函式部署到您的函式應用程式服務。

   ![使用範本部署的函式](./media/manage-compute-with-azure-functions/five-functions.png)

2. 選取 [ *DWScaleDownTrigger* ] 或 [ *DWScaleUpTrigger* ] 以擴大或縮小。 在下拉式功能表中，選取 [整合]。

   ![對函式選取整合](./media/manage-compute-with-azure-functions/select-integrate.png)

3. 目前顯示的值應該為 %ScaleDownTime% 或 %ScaleUpTime%。 這些值表示排程是以[應用程式設定](../../azure-functions/functions-how-to-use-azure-function-app-settings.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) 中定義的值為基礎。 您現在可以忽略此值，並將排程變更為以後續步驟為基礎的慣用時間。

4. 在 [排程] 區域中，新增您想要的 CRON 運算式，以反映您希望 Azure Synapse Analytics 向上擴充的頻率。

   ![變更函式排程](./media/manage-compute-with-azure-functions/change-schedule.png)

   `schedule` 的值是包含以下 6 個欄位的 [CRON 運算式](https://en.wikipedia.org/wiki/Cron#CRON_expression)︰

   ```json
   {second} {minute} {hour} {day} {month} {day-of-week}
   ```

   例如，*"0 30 9 * * 1-5"* 會反映在每個工作日上午 9:30 執行的觸發程序。 如需詳細資訊，請瀏覽 Azure Functions [排程範例](../../azure-functions/functions-bindings-timer.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#example)。

## <a name="change-the-time-of-the-scale-operation"></a>變更調整規模作業的時間

1. 瀏覽到函式應用程式服務。 如果您部署的範本採用預設值，此服務應該命名為 DWOperations。 您的函式應用程式開啟後，您應會注意到有五個函式部署到您的函式應用程式服務。

2. 請選取 [ *DWScaleDownTrigger* ] 或 [ *DWScaleUpTrigger* ]，以擴大或縮小計算值。 選取函式時，您的窗格應會顯示 index.js 檔案。

   ![變更函式觸發程序計算層級](././media/manage-compute-with-azure-functions/index-js.png)

3. 將 *ServiceLevelObjective* 的值變更為您想要的層級，然後選取 [儲存]。 *ServiceLevelObjective* 是您的資料倉儲實例將根據 [整合] 區段中定義的排程來調整規模的計算層級。

## <a name="use-pause-or-resume-instead-of-scale"></a>使用暫停或繼續，而不是調整

目前的函式預設為 DWScaleDownTrigger 和 DWScaleUpTrigger。 如果您想要改用暫停和繼續功能，您可以啟用 DWPauseTrigger 或 DWResumeTrigger。

1. 瀏覽到 [函式] 窗格。

   ![函式窗格](./media/manage-compute-with-azure-functions/functions-pane.png)

2. 針對您想要啟用的對應觸發程式，選取滑動切換。

3. 瀏覽至個別觸發程序的 [整合] 索引標籤，以變更其排程。

   > [!NOTE]
   > 自動調整觸發程序和暫停/繼續觸發程序之間的功能差異是傳送給佇列的訊息。 如需詳細資訊，請參閱[新增觸發程序函式](manage-compute-with-azure-functions.md#add-a-new-trigger-function)。

## <a name="add-a-new-trigger-function"></a>新增觸發程序函式

目前範本中只包含兩個調整函式。 使用這些函式，在一天當中，您只能相應減少一次和相應增加一次。 您必須新增另一個觸發程序，才能取得更細微的控制，例如每日相應減少多次或在週末有不同的調整行為。

1. 建立新的空白函式。 選取您的函式 *+* 位置附近的按鈕，以顯示 [函式範本] 窗格。

   ![顯示 [函數應用程式] 功能表的螢幕擷取畫面，其中已選取 [函式] 旁的 [加號] 圖示。](./media/manage-compute-with-azure-functions/create-new-function.png)

2. 從 [語言] 中，選取 [ *JavaScript*]，然後選取 [ *TimerTrigger*]。

   ![建立新的函式](./media/manage-compute-with-azure-functions/timertrigger-js.png)

3. 替函式命名並設定您的排程。 此影像顯示使用者如何在每個星期六午夜 (星期五深夜) 觸發其函式。

   ![週六相應減少](./media/manage-compute-with-azure-functions/scale-down-saturday.png)

4. 從其中一個其他觸發程序函式複製 index.js 的內容。

   ![複製 index.js](././media/manage-compute-with-azure-functions/index-js.png)

5. 將作業變數設定為所要的行為，如下所示：

   ```JavaScript
   // Resume the dedicated SQL pool (formerly SQL DW) instance
   var operation = {
       "operationType": "ResumeDw"
   }

   // Pause the dedicated SQL pool (formerly SQL DW) instance
   var operation = {
       "operationType": "PauseDw"
   }

   // Scale the dedicated SQL pool (formerly SQL DW)l instance to DW600c
   var operation = {
       "operationType": "ScaleDw",
       "ServiceLevelObjective": "DW600c"
   }
   ```

## <a name="complex-scheduling"></a>複雜排程

這一節簡短示範取得更複雜的暫停、繼續和調整功能排程所需的項目。

### <a name="example-1"></a>範例 1

每日擴大至上午8點至 DW600c，並將8pm 縮小至 DW200c。

| 函式  | 排程     | 作業                                |
| :-------- | :----------- | :--------------------------------------- |
| Function1 | 0 0 8 * * *  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW600c"}` |
| Function2 | 0 0 20 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW200c"}` |

### <a name="example-2"></a>範例 2

每日擴大至上午8點至 DW1000c、相應減少一次至 DW600 4pm，並將下午10點縮減為 DW200c。

| 函式  | 排程     | 作業                                |
| :-------- | :----------- | :--------------------------------------- |
| Function1 | 0 0 8 * * *  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW1000c"}` |
| Function2 | 0 0 16 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW600c"}` |
| Function3 | 0 0 22 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW200c"}` |

### <a name="example-3"></a>範例 3

將上午8點擴大至 DW1000c，在工作日的4pm 相應減少一次。 在星期五晚上 11 點暫停，在星期一上午 7 點繼續。

| 函式  | 排程       | 作業                                |
| :-------- | :------------- | :--------------------------------------- |
| Function1 | 0 0 8 * * 1-5  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW1000c"}` |
| Function2 | 0 0 16 * * 1-5 | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW600c"}` |
| Function3 | 0 0 23 * * 5   | `var operation = {"operationType": "PauseDw"}` |
| Function4 | 0 0 7 * * 1    | `var operation = {"operationType": "ResumeDw"}` |

## <a name="next-steps"></a>後續步驟

深入瞭解 [計時器觸發](../../azure-functions/functions-create-scheduled-function.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) 程式 Azure Functions。

請參閱先前 (SQL DW) 範例存放 [庫](https://github.com/Microsoft/sql-data-warehouse-samples)的專用 sql 集區。
