---
title: Azure Analysis Services 的診斷記錄 | Microsoft Docs
description: 深入了解設定 Azure Analysis Services 的診斷記錄。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: d19e45710aca3e1e18be6c4529da6474a97bc59f
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37448880"
---
# <a name="setup-diagnostic-logging"></a>設定診斷記錄

在任何 Analysis Services 解決方案中，監視您伺服器的執行方式皆是一大重點功能。 您可以透過 [Azure 資源診斷記錄](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)監視並傳送記錄至 [Azure 儲存體](https://azure.microsoft.com/services/storage/)，將記錄檔案串流至 [Azure 事件中樞](https://azure.microsoft.com/services/event-hubs/)，並將記錄檔案匯出至 [Log Analytics](https://azure.microsoft.com/services/log-analytics/) (這是 [Azure](https://www.microsoft.com/cloud-platform/operations-management-suite) 的服務)。 

![儲存體、事件中樞或 Log Analytics 的診斷記錄](./media/analysis-services-logging/aas-logging-overview.png)


## <a name="whats-logged"></a>記錄的內容？

您可以選取 [引擎]、[服務]，和 [計量] 類別。

### <a name="engine"></a>引擎

選取 [引擎] 記錄所有 [xEvents](https://docs.microsoft.com/sql/analysis-services/instances/monitor-analysis-services-with-sql-server-extended-events)。 您無法選取個別事件。 

|XEvent 類別 |事件名稱  |
|---------|---------|
|安全性稽核    |   稽核登入      |
|安全性稽核    |   稽核登出      |
|安全性稽核    |   稽核伺服器啟動與停止      |
|進度報告     |   進度報告開始      |
|進度報告     |   進度報告結束      |
|進度報告     |   進度報告目前狀態      |
|查詢     |  查詢開始       |
|查詢     |   查詢結束      |
|命令     |  命令開始       |
|命令     |  命令結束       |
|錯誤與警告     |   Error      |
|探索     |   探索結束      |
|通知     |    通知     |
|工作階段     |  工作階段初始化       |
|鎖定    |  鎖死       |
|查詢處理     |   VertiPaq SE 查詢開始      |
|查詢處理     |   VertiPaq SE 查詢結束      |
|查詢處理     |   VertiPaq SE 查詢快取比對      |
|查詢處理     |   直接查詢開始      |
|查詢處理     |  直接查詢結束       |

### <a name="service"></a>服務

|作業名稱  |發生條件  |
|---------|---------|
|ResumeServer     |    恢復伺服器     |
|SuspendServer    |   暫停伺服器      |
|DeleteServer     |    刪除伺服器     |
|RestartServer    |     使用者透過 SSMS 或 PowerShell 重新啟動伺服器    |
|GetServerLogFiles    |    使用者透過 PowerShell 匯出伺服器記錄     |
|ExportModel     |   使用者使用 [在 Visual Studio 中開啟]，在入口網站中匯出模型     |

### <a name="all-metrics"></a>所有計量

計量類別將「計量」中顯示的相同[伺服器計量](analysis-services-monitor.md#server-metrics)記錄起來。

## <a name="setup-diagnostics-logging"></a>設定診斷記錄

### <a name="azure-portal"></a>Azure 入口網站

1. 在 [Azure 入口網站](https://portal.azure.com) > 伺服器中，在左側瀏覽區中按一下 [診斷記錄]，然後按一下 [開啟診斷]。

    ![在 Azure 入口網站中開啟 Azure Cosmos DB 的診斷記錄](./media/analysis-services-logging/aas-logging-turn-on-diagnostics.png)

2. 在 [診斷設定] 中，指定下列選項： 

    * **名稱**。 輸入要建立之記錄的名稱。

    * **封存至儲存體帳戶**。 若要使用此選項，您需要可以連接的現有儲存體帳戶。 請參閱[建立儲存體帳戶](../storage/common/storage-create-storage-account.md)。 請依照指示建立資源管理員、一般用途的帳戶，然後在返回入口網站上的此頁面時，選取您的儲存體帳戶。 新建立的儲存體帳戶可能在數分鐘後才會出現在下拉式功能表中。
    * **串流處理至事件中樞**。 若要使用此選項，您需要可以連接的現有事件中樞命名空間和事件中樞。 若想深入了解，請參閱[使用 Azure 入口網站來建立事件中樞命名空間和事件中樞](../event-hubs/event-hubs-create.md)。 然後返回入口網站的此頁面選取事件中樞命名空間和原則名稱。
    * **傳送至 Log Analytics**。 若要使用此選項，請使用現有的工作區，或是在入口網站中依照[建立新的工作區](../log-analytics/log-analytics-quick-collect-azurevm.md#create-a-workspace)的步驟建立新的 Log Analytics 工作區。 如需有關如何檢視 Log Analytics 記錄的詳細資訊，請參閱[檢視 Log Analytics 中的記錄](#view-in-loganalytics)。

    * **引擎**。 選取此選項可記錄 xEvents。 如果您要封存至儲存體帳戶，您可以為診斷記錄選取保留期限。 保留期限過後，就會自動刪除記錄。
    * **服務**。 選取此選項可記錄服務層級事件。 如果您要封存至儲存體帳戶，可以為診斷記錄選取保留期限。 保留期限過後，就會自動刪除記錄。
    * **計量**。 選取此選項可儲存[計量](analysis-services-monitor.md#server-metrics)中的詳細資料。 如果您要封存至儲存體帳戶，可以為診斷記錄選取保留期限。 保留期限過後，就會自動刪除記錄。

3. 按一下 [檔案] 。

    如果您收到錯誤，指出「無法更新 \<工作區名稱> 的診斷。 訂用帳戶 \<訂用帳戶識別碼> 未註冊為使用 microsoft.insights」， 請遵循[針對 Azure 診斷進行疑難排解](https://docs.microsoft.com/azure/log-analytics/log-analytics-azure-storage)的指示註冊帳戶，然後重試此程序。

    未來如果您想變更診斷記錄的儲存方式，可以返回此頁面修改設定。

### <a name="powershell"></a>PowerShell

以下基本命令將引導您進行後續作業。 如果您需要逐步說明，了解如何使用 PowerShell 將記錄設定到儲存體帳戶，請參閱本文後段中的教學課程。

若要使用 Powershell 啟用計量和診斷記錄功能，請使用下列 Cmdlet：

- 若要啟用儲存體帳戶中診斷記錄的儲存體，請使用下列命令：

   ```powershell
   Set-AzureRmDiagnosticSetting -ResourceId [your resource id] -StorageAccountId [your storage account id] -Enabled $true
   ```

   儲存體帳戶識別碼是您要傳送記錄之目標儲存體帳戶的資源識別碼。

- 若要將診斷記錄串流至事件中樞，請使用下列命令：

   ```powershell
   Set-AzureRmDiagnosticSetting -ResourceId [your resource id] -ServiceBusRuleId [your service bus rule id] -Enabled $true
   ```

   Azure 服務匯流排規則識別碼是此格式的字串︰

   ```powershell
   {service bus resource ID}/authorizationrules/{key name}
   ``` 

- 若要將診斷記錄傳送到 Log Analytics 工作區，請使用下列命令：

   ```powershell
   Set-AzureRmDiagnosticSetting -ResourceId [your resource id] -WorkspaceId [resource id of the log analytics workspace] -Enabled $true
   ```

- 您可以使用下列命令取得 Log Analytics 工作區的資源識別碼：

   ```powershell
   (Get-AzureRmOperationalInsightsWorkspace).ResourceId
   ```

您可以結合這些參數讓多個輸出選項。

### <a name="rest-api"></a>REST API

了解如何[使用 Azure 監視器 REST API 變更診斷設定](https://msdn.microsoft.com/library/azure/dn931931.aspx)。 

### <a name="resource-manager-template"></a>Resource Manager 範本

了解如何[使用 Resource Manager 範本在建立資源時啟用診斷設定](../monitoring-and-diagnostics/monitoring-enable-diagnostic-logs-using-template.md)。 

## <a name="manage-your-logs"></a>管理您的記錄

記錄通常會在設定記錄時幾個小時內提供。 儲存體帳戶中的記錄由您全權管理：

* 請使用標準的 Azure 存取控制方法限制可存取記錄的人員，藉此來保護記錄。
* 刪除不想繼續保留在儲存體帳戶中的記錄。
* 請務必設定保留期限，以便從儲存體帳戶刪除舊的記錄。

## <a name="view-logs-in-log-analytics"></a>檢視 Log Analytics 中的記錄

計量和伺服器事件會與 Log Analytics 中的 xEvents 整合以進行並存分析。 也可以將 Log Analytics 設定為接收來自其他 Azure 服務的事件，以提供整體檢視，了解整個架構的診斷記錄資料。

若要在 Log Analytics 檢視診斷資料，請從左側功能表或 [管理] 區域開啟 [記錄搜尋] 頁面，如下所示。

![Azure 入口網站中的記錄搜尋選項](./media/analysis-services-logging/aas-logging-open-log-search.png)

現在您已啟用資料收集，請在 [記錄搜尋] 中，按一下 [所有收集的資料]。

在 [類型] 中，按一下 [AzureDiagnostics]，然後按一下 [套用]。 AzureDiagnostics 包括「引擎」和「服務」事件。 請注意，Log Analytics 查詢是即時建立的。 EventClass\_s 的欄位包含 xEvent 名稱，如果您曾經使用 xEvents 進行內部部署記錄，這些名稱就可能看起來似曾相似。

按一下 **EventClass\_s** 或其中一個事件名稱，Log Analytics 將會繼續建構查詢。 請務必儲存您的查詢，以供日後重複使用。

請務必查看 Log Analytics，在其所提供的一個網站中，增強了對收集資料的查詢、儀表板管理，以及警示功能。

### <a name="queries"></a>查詢

有數百個查詢可供使用。 以下事項可快速協助您準備入門。
若要深入了解有關使用新的記錄搜尋語言，請參閱[了解 Log Analytics 中的記錄搜尋](../log-analytics/log-analytics-log-search-new.md)。 

* 查詢提交到 Azure Analysis Services 的回傳查詢，且作業時間超過五分鐘以上 (300,000 毫秒)。

    ```
    search * | where ( Type == "AzureDiagnostics" ) | where ( EventClass_s == "QUERY_END" ) | where toint(Duration_s) > 300000
    ```

* 識別相應放大複本。

    ```
    search * | summarize count() by ServerName_s
    ```
    使用相應放大時，您可以識別唯讀複本，因為 ServerName\_s 欄位值具有附加至名稱的複本執行個體號碼。 資源欄位包含 Azure 資源名稱，且該名稱符合使用者看到的伺服器名稱。 IsQueryScaleoutReadonlyInstance_s 欄位等於複本。



> [!TIP]
> 您想共用很棒的 Log Analytics 查詢嗎？ 如果您有 GitHub 帳戶，您就可以將它新增至此文章。 只要按一下頁面右上側的 [編輯] 即可。


## <a name="tutorial---turn-on-logging-by-using-powershell"></a>教學課程 - 使用 PowerShell 開啟記錄
在本快速教學課程中，您可以在與 Analysis Service 伺服器相同的訂用帳戶和資源群組中，建立儲存體帳戶。 接著使用 Set-AzureRmDiagnosticSetting 開啟診斷記錄，將輸出傳送至新的儲存體帳戶。

### <a name="prerequisites"></a>先決條件
若要完成本教學課程，您必須具備下列資源：

* 現有的 Azure Analysis Services 伺服器。 如需建立伺服器資源的指示，請參閱 [在 Azure 入口網站中建立伺服器](analysis-services-create-server.md)，或[使用 PowerShell 建立 Azure Analysis Services 伺服器](analysis-services-create-powershell.md)。

### <a name="aconnect-to-your-subscriptions"></a></a>連線到您的訂閱

開始 Azure PowerShell 工作階段，並使用下列命令登入您的 Azure 帳戶：  

```powershell
Connect-AzureRmAccount
```

在快顯瀏覽器視窗中，輸入您的 Azure 帳戶使用者名稱與密碼。 Azure PowerShell 會取得與此帳戶相關聯的所有訂用帳戶，並依預設使用第一個訂用帳戶。

如果您有多個訂用帳戶，您可能必須指定用來建立 Azure 金鑰保存庫的那一個特定訂用帳戶。 輸入下列命令以查看您帳戶的訂用帳戶：

```powershell
Get-AzureRmSubscription
```

然後為了指定已與所要記錄 Azure Analysis Services 帳戶建立關聯的訂用帳戶，輸入：

```powershell
Set-AzureRmContext -SubscriptionId <subscription ID>
```

> [!NOTE]
> 如果您有多個與帳戶相關聯的訂用帳戶，請務必指定訂用帳戶。
>
>

### <a name="create-a-new-storage-account-for-your-logs"></a>為您的記錄建立新的儲存體帳戶

您可以針對您的記錄使用現有的儲存體帳戶，前提是該帳戶與您的伺服器位於相同的訂用帳戶中。 此教學課程中，您會建立專門用於 Analysis Services 記錄的新儲存體帳戶。 為了方便起見，您可將儲存體帳戶詳細資訊儲存到名為 **sa** 的變數。

您也可以使用與包含 Analysis Services 伺服器資源群組相同的資源群組。 請將 `awsales_resgroup`、`awsaleslogs` 和 `West Central US` 的值替代成您自己的值：

```powershell
$sa = New-AzureRmStorageAccount -ResourceGroupName awsales_resgroup `
-Name awsaleslogs -Type Standard_LRS -Location 'West Central US'
```

### <a name="identify-the-server-account-for-your-logs"></a>識別記錄的伺服器帳戶

將帳戶名稱設為名為 **account** 的變數，其中的 ResourceName 是帳戶的名稱。

```powershell
$account = Get-AzureRmResource -ResourceGroupName awsales_resgroup `
-ResourceName awsales -ResourceType "Microsoft.AnalysisServices/servers"
```

### <a name="enable-logging"></a>啟用記錄

若要啟用記錄，請使用 Set-AzureRmDiagnosticSetting Cmdlet，並搭配新儲存體帳戶、伺服器帳戶和類別的變數一起使用。 執行下列命令，將 **-Enabled** 旗標設為 **$true**：

```powershell
Set-AzureRmDiagnosticSetting  -ResourceId $account.ResourceId -StorageAccountId $sa.Id -Enabled $true -Categories Engine
```

輸出應該會看起來像這樣：

```powershell
StorageAccountId            : 
/subscriptions/a23279b5-xxxx-xxxx-xxxx-47b7c6d423ea/resourceGroups/awsales_resgroup/providers/Microsoft.Storage/storageAccounts/awsaleslogs
ServiceBusRuleId            :
EventHubAuthorizationRuleId :
Metrics                    
    TimeGrain       : PT1M
    Enabled         : False
    RetentionPolicy
    Enabled : False
    Days    : 0


Logs                       
    Category        : Engine
    Enabled         : True
    RetentionPolicy
    Enabled : False
    Days    : 0


    Category        : Service
    Enabled         : False
    RetentionPolicy
    Enabled : False
    Days    : 0


WorkspaceId                 :
Id                          : /subscriptions/a23279b5-xxxx-xxxx-xxxx-47b7c6d423ea/resourcegroups/awsales_resgroup/providers/microsoft.analysisservic
es/servers/awsales/providers/microsoft.insights/diagnosticSettings/service
Name                        : service
Type                        :
Location                    :
Tags                        :
```

這個結果可確認伺服器記錄現已啟用，系統會將資訊儲存到儲存體帳戶中。

您也可以設定記錄的保留原則，以便自動刪除較舊的記錄。 例如，使用 **-RetentionEnabled** 旗標將保留原則設為 **$true** 並將 **-RetentionInDays** 參數設為 **90**。 系統將自動刪除超過 90 天的舊記錄。

```powershell
Set-AzureRmDiagnosticSetting -ResourceId $account.ResourceId`
 -StorageAccountId $sa.Id -Enabled $true -Categories Engine`
  -RetentionEnabled $true -RetentionInDays 90
```

## <a name="next-steps"></a>後續步驟

深入了解 [Azure 資源診斷記錄](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)。

請參閱 PowerShell 說明中的 [Set-AzureRmDiagnosticSetting](https://docs.microsoft.com/powershell/module/azurerm.insights/Set-AzureRmDiagnosticSetting)。
