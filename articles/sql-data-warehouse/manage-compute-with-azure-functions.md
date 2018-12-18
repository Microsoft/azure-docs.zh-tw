---
title: 教學課程：使用 Azure SQL 資料倉儲中的 Azure Functions 管理計算 | Microsoft Docs
description: 如何使用 Azure Functions 來管理資料倉儲的計算。
services: sql-data-warehouse
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: consume
ms.date: 04/27/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: 2f366718a11f715b0c91a556eb3b0f216424b82c
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43287896"
---
# <a name="use-azure-functions-to-manage-compute-resources-in-azure-sql-data-warehouse"></a>使用 Azure Functions 來管理 Azure SQL 資料倉儲中的計算資源

本教學課程使用 Azure Functions 來管理 Azure SQL 資料倉儲中的資料倉儲計算資源。

若要搭配使用 Azure 函式應用程式與 SQL 資料倉儲，您必須建立[服務主體帳戶](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal)，其在與您的資料倉儲執行個體相同的訂用帳戶之下，具有參與者存取權。 

## <a name="deploy-timer-based-scaling-with-an-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本部署以計時器為基礎的 Scaler

部署此範本，您需要下列資訊：

- 您的 SQL DW 執行個體所在的資源群組名稱
- 您的 SQL DW 執行個體所在的邏輯伺服器名稱
- 您的 SQL DW 執行個體名稱
- Azure Active Directory 的租用戶識別碼 (目錄識別碼)
- 訂用帳戶識別碼 
- 服務主體的應用程式識別碼
- 服務主體祕密金鑰

一旦取得前述資訊，即可部署此範本：

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Fsql-data-warehouse-samples%2Fmaster%2Farm-templates%2FsqlDwTimerScaler%2Fazuredeploy.json" target="_blank">
<img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png"/>
</a>

部署此範本後，您應會發現三項新資源：免費的 Azure App Service 方案、以耗用量為基礎的函式應用程式方案，以及處理記錄和作業佇列的儲存體帳戶。 繼續閱讀其他各節，以了解如何修改已部署的函式來符合您的需求。

## <a name="change-the-compute-level"></a>變更計算層級

1. 瀏覽到函式應用程式服務。 如果您部署的範本採用預設值，此服務應該命名為 DWOperations。 您的函式應用程式開啟後，您應會注意到有五個函式部署到您的函式應用程式服務。 

   ![使用範本部署的函式](media/manage-compute-with-azure-functions/five-functions.png)

2. 根據您是否要變更相應增加或相應減少時間，選取 [DWScaleDownTrigger] 或 [DWScaleUpTrigger]。 在下拉式功能表中，選取 [整合]。

   ![對函式選取整合](media/manage-compute-with-azure-functions/select-integrate.png)

3. 目前顯示的值應該為 %ScaleDownTime% 或 %ScaleUpTime%。 這些值表示排程是以[應用程式設定][Application Settings] 中定義的值為基礎。 您現在可以忽略此值，並將排程變更為以後續步驟為基礎的慣用時間。

4. 在 [排程] 區域中，新增時間、您想要的 CRON 運算式，以反映 SQL 資料倉儲相應增加的頻率。 

  ![變更函式排程](media/manage-compute-with-azure-functions/change-schedule.png)

  `schedule` 的值是包含以下 6 個欄位的 [CRON 運算式](http://en.wikipedia.org/wiki/Cron#CRON_expression)︰ 
  ```json
  {second} {minute} {hour} {day} {month} {day-of-week}
  ```

  例如，*"0 30 9 * * 1-5"* 會反映在每個工作日上午 9:30 執行的觸發程序。 如需詳細資訊，請瀏覽 Azure Functions [排程範例][schedule examples]。


## <a name="change-the-time-of-the-scale-operation"></a>變更調整規模作業的時間

1. 瀏覽到函式應用程式服務。 如果您部署的範本採用預設值，此服務應該命名為 DWOperations。 您的函式應用程式開啟後，您應會注意到有五個函式部署到您的函式應用程式服務。 

2. 根據您是否要變更相應增加或相應減少計算值，選取 DWScaleDownTrigger 或 DWScaleUpTrigger。 選取函式時，您的窗格應會顯示 index.js 檔案。

   ![變更函式觸發程序計算層級](media/manage-compute-with-azure-functions/index-js.png)

3. 將 ServiceLevelObjective 的值變更為您想要的層級，然後按 [儲存]。 這個值是您的資料倉儲執行個體將根據 [整合] 區段中定義的排程而調整至的計算層級。

## <a name="use-pause-or-resume-instead-of-scale"></a>使用暫停或繼續，而不是調整 

目前的函式預設為 DWScaleDownTrigger 和 DWScaleUpTrigger。 如果您想要改用暫停和繼續功能，您可以啟用 DWPauseTrigger 或 DWResumeTrigger。

1. 瀏覽到 [函式] 窗格。

   ![函式窗格](media/manage-compute-with-azure-functions/functions-pane.png)



2. 針對您要啟用的對應觸發程序，按一下滑動切換開關。

3. 瀏覽至個別觸發程序的 [整合] 索引標籤，以變更其排程。

   > [!NOTE]
   > 自動調整觸發程序和暫停/繼續觸發程序之間的功能差異是傳送給佇列的訊息。 如需詳細資訊，請參閱[新增觸發程序函式][Add a new trigger function]。


## <a name="add-a-new-trigger-function"></a>新增觸發程序函式

目前範本中只包含兩個調整函式。 使用這些函式，在一天當中，您只能相應減少一次和相應增加一次。 您必須新增另一個觸發程序，才能取得更細微的控制，例如每日相應減少多次或在週末有不同的調整行為。

1. 建立新的空白函式。 選取您的函式附近的 *+* 按鈕，以顯示 [函式範本] 窗格。

   ![建立新的函式](media/manage-compute-with-azure-functions/create-new-function.png)

2. 從 [語言] 中選取 Javascript，然後選取 TimerTrigger。

   ![建立新的函式](media/manage-compute-with-azure-functions/timertrigger-js.png)

3. 替函式命名並設定您的排程。 此影像顯示使用者如何在每個星期六午夜 (星期五深夜) 觸發其函式。

   ![週六相應減少](media/manage-compute-with-azure-functions/scale-down-saturday.png)

4. 從其中一個其他觸發程序函式複製 index.js 的內容。

   ![複製 index.js](media/manage-compute-with-azure-functions/index-js.png)

5. 將作業變數設定為所要的行為，如下所示：

   ```javascript
   // Resume the data warehouse instance
   var operation = {
       "operationType": "ResumeDw"
   }

   // Pause the data warehouse instance
   var operation = {
       "operationType": "PauseDw"
   }

   // Scale the data warehouse instance to DW600
   var operation = {
       "operationType": "ScaleDw",
       "ServiceLevelObjective": "DW600"
   }
   ```


## <a name="complex-scheduling"></a>複雜排程

這一節簡短示範取得更複雜的暫停、繼續和調整功能排程所需的項目。

### <a name="example-1"></a>範例 1：

每天上午 8 點相應增加至 DW600，在下午 8 點相應減少至 DW200。

| 函式  | 排程     | 作業                                |
| :-------- | :----------- | :--------------------------------------- |
| Function1 | 0 0 8 * * *  | `var operation = {"operationType": "ScaleDw",  "ServiceLevelObjective": "DW600"}` |
| Function2 | 0 0 20 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW200"}` |

### <a name="example-2"></a>範例 2： 

每天上午 8 點相應增加至 DW1000、在下午 4 點一度相應減少至 DW600，並在下午 10 點相應減少至 DW200。

| 函式  | 排程     | 作業                                |
| :-------- | :----------- | :--------------------------------------- |
| Function1 | 0 0 8 * * *  | `var operation = {"operationType": "ScaleDw",  "ServiceLevelObjective": "DW1000"}` |
| Function2 | 0 0 16 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW600"}` |
| Function3 | 0 0 22 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW200"}` |

### <a name="example-3"></a>範例 3： 

在工作日的上午 8 點相應增加至 DW1000，在下午 4 點一度相應減少至 DW600。 在星期五晚上 11 點暫停，在星期一上午 7 點繼續。

| 函式  | 排程       | 作業                                |
| :-------- | :------------- | :--------------------------------------- |
| Function1 | 0 0 8 * * 1-5  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW1000"}` |
| Function2 | 0 0 16 * * 1-5 | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW600"}` |
| Function3 | 0 0 23 * * 5   | `var operation = {"operationType": "PauseDw"}` |
| Function4 | 0 0 7 * * 0    | `var operation = {"operationType": "ResumeDw"}` |



## <a name="next-steps"></a>後續步驟

深入了解[計時器觸發程序](../azure-functions/functions-create-scheduled-function.md) Azure Functions。

查看 SQL 資料倉儲[範例存放庫](https://github.com/Microsoft/sql-data-warehouse-samples)。



[schedule examples]: ../azure-functions/functions-bindings-timer.md#example

[Application Settings]: ../azure-functions/functions-how-to-use-azure-function-app-settings.md
[Add a new trigger function]: manage-compute-with-azure-functions.md#add-a-new-trigger-function