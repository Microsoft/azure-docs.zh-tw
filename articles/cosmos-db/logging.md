---
title: Azure Cosmos DB 診斷記錄 | Microsoft Docs
description: 使用本教學課程來協助您開始使用 Azure Cosmos DB 記錄。
services: cosmos-db
author: SnehaGunda
manager: kfile
tags: azure-resource-manager
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 03/07/2018
ms.author: sngun
ms.openlocfilehash: 68eb567235897641d5d4027160f62c5aa6e7e4f9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46963384"
---
# <a name="azure-cosmos-db-diagnostic-logging"></a>Azure Cosmos DB 診斷記錄

在您開始使用一或多個 Azure Cosmos DB 資料庫後，建議您監視資料庫的存取情形和時間。 本文提供可在 Azure 平台中使用之記錄的概觀。 您將了解如何啟用供監視使用的診斷記錄，以傳送記錄至 [Azure 儲存體](https://azure.microsoft.com/services/storage/)，以及如何將記錄串流處理至 [Azure 事件中心](https://azure.microsoft.com/services/event-hubs/)，和如何將記錄匯出至 [Azure Log Analytics](https://azure.microsoft.com/services/log-analytics/)。

## <a name="logs-available-in-azure"></a>Azure 中可用的記錄

在討論如何監視您的 Azure Cosmos DB 帳戶之前，讓我們釐清關於記錄和監視的一些事項。 Azure 平台上有不同類型的記錄。 我們有 [Azure 活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)、[Azure 診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)、[Azure 計量](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics)、事件、活動訊號監視、作業記錄等。 記錄十分繁多。 您可以在 Azure 入口網站的 [Azure Log Analytics](https://azure.microsoft.com/services/log-analytics/) 中查看完整記錄清單。 

下圖顯示不同種類的可用 Azure 記錄：

![不同種類的 Azure 記錄](./media/logging/azurelogging.png)

在上圖中，**計算資源**代表您可以存取其 Microsoft 客體 OS 的 Azure 資源。 例如，Azure 虛擬機器、虛擬機器擴展集、Azure Container Service 等皆被視為計算資源。 計算資源會產生「活動記錄」、「診斷記錄」及「應用程式記錄」。 若要深入了解，請參閱 [Azure 中的監視資料來源](../monitoring/monitoring-data-sources.md#)一文。

**非計算資源**係指您無法存取基礎 OS 並直接使用資源的資源。 例如，網路安全性群組、Logic Apps 等。 Azure Cosmos DB 是非計算資源。 您可以在「活動記錄」中檢視非計算資源的記錄，或是在入口網站中啟用 [診斷記錄] 選項。 若要深入了解，請參閱 [Azure 監視器中的資源來源](../monitoring/monitoring-data-sources.md)一文。

「活動記錄」會記錄 Azure Cosmos DB 的訂用帳戶層級作業。 像是 ListKeys、Write DatabaseAccounts 等作業，都會記錄。 「診斷記錄」提供更細微的記錄功能，並可讓您記錄 DataPlaneRequests (Create、Read、Query 等) 和 MongoRequests。


在本文中，我們將著重於 Azure 活動記錄、Azure 診斷記錄和 Azure 計量。 這三種記錄有何差異？ 

### <a name="azure-activity-log"></a>Azure 活動記錄檔

「Azure 活動記錄」是訂用帳戶記錄，可讓您深入探索 Azure 中發生的訂用帳戶層級事件。 活動記錄會報告系統管理類別下您的訂用帳戶的控制平面事件。 您可以使用活動記錄來判斷對您訂用帳戶中的資源執行的任何寫入作業 (PUT、POST、DELETE) 的「內容、執行者和時間」。 您也可以了解作業的狀態和其他相關屬性。 

活動記錄不同於診斷記錄。 活動記錄會提供關於外部資源 (_控制平面_) 之作業的資料。 在 Azure Cosmos DB 內容中，控制平面作業包括建立容器、列出金鑰、刪除金鑰、列出資料庫等。 診斷記錄是由資源所發出，會提供該資源作業的相關資訊 (_資料平面_)。 舉例來說，Delete、Insert 和 ReadFeed 都是診斷記錄中的資料平面作業。

活動記錄 (控制平面作業) 在本質上可能更豐富，且可能會包含呼叫端的完整電子郵件地址、呼叫端 IP 位址、資源名稱、作業名稱和 TenantId 等等。 活動記錄包含數個資料[類別](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-activity-log-schema)。 如需這些類別結構描述的完整詳細資料，請參閱 [Azure 活動記錄事件結構描述](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-activity-log-schema)。 不過，診斷記錄在本質上有所限制，因為個人資料通常會從這些記錄中移除。 您可能會有呼叫端的 IP 位址，但最後一個 octent 會被移除。

### <a name="azure-metrics"></a>Azure 計量

[Azure 計量](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics)具有由大多數 Azure 資源發出的關鍵 Azure 遙測資料 (也稱為_效能計數器_)。 計量可讓您檢視關於輸送量、儲存體、一致性、可用性和 Azure Cosmos DB 資源延遲的資訊。 如需詳細資訊，請參閱[使用 Azure Cosmos DB 中的計量監控及偵錯](use-metrics.md)。

### <a name="azure-diagnostic-logs"></a>Azure 診斷記錄

Azure 診斷記錄是由資源所發出，提供關於該資源之作業的豐富、經常性資料。 這些記錄的內容會依資源類型而有所不同。 資源層級診斷記錄也與客體 OS 層級診斷記錄不同。 客體 OS 診斷記錄是由虛擬機器內執行的代理程式或其他支援的資源類型所收集的記錄。 資源層級的診斷記錄不需要代理程式，而會從 Azure 平台本身擷取特定資源資料。 客體 OS 層級的診斷記錄會從虛擬機器所執行的作業系統和應用程式擷取資料。

![儲存體、事件中樞或 Log Analytics 的診斷記錄](./media/logging/azure-cosmos-db-logging-overview.png)

### <a name="what-is-logged-by-azure-diagnostic-logs"></a>Azure 診斷記錄記錄哪些項目？

* 會跨所有 API 記錄所有已驗證的後端要求 (TCP/REST)，包括因為存取權限、系統錯誤或要求錯誤而發生的失敗要求。 目前不支援使用者起始的 Graph、Cassandra 和資料表 API 要求。
* 對資料庫本身的作業，包括對所有文件、容器和資料庫的 CRUD 作業。
* 對帳戶金鑰的作業，包括建立、修改或刪除金鑰。
* 產生 401 回應的未經驗證要求。 例如，沒有持有人權杖的要求，或格式不正確或已過期的要求，或具有無效權杖的要求。

<a id="#turn-on"></a>
## <a name="turn-on-logging-in-the-azure-portal"></a>在 Azure 入口網站中開啟記錄

若要啟用診斷記錄，您必須擁有下列資源：

* 現有的 Azure Cosmos DB 帳戶、資料庫和容器。 如需建立這些資源的指示，請參閱[使用 Azure 入口網站建立資料庫帳戶](create-sql-api-dotnet.md#create-a-database-account)、[Azure CLI 範例](cli-samples.md)或 [PowerShell 範例](powershell-samples.md)。

若要在 Azure 入口網站中啟用診斷記錄，請執行下列步驟：

1. 進入 [Azure 入口網站](https://portal.azure.com)，在您 Azure Cosmos DB 帳戶的左側導覽區中選取 [診斷記錄]，然後選取 [開啟診斷]。

    ![在 Azure 入口網站中開啟 Azure Cosmos DB 的診斷記錄](./media/logging/turn-on-portal-logging.png)

2. 在 [診斷設定] 頁面中，執行下列步驟： 

    * **名稱**：輸入要建立之記錄的名稱。

    * **封存至儲存體帳戶**：若要使用此選項，您需要可以連接的現有儲存體帳戶。 若要在入口網站中建立新的儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-create-storage-account.md)，依指示建立一個 Azure Resource Manager (一般用途帳戶)。 然後，返回入口網站的此頁面選取您的儲存體帳戶。 新建立的儲存體帳戶可能在數分鐘後才會出現於下拉式功能表中。
    * **串流至事件中樞**：若要使用此選項，您需要可以連接的現有事件中樞命名空間和事件中樞。 若要建立事件中樞命名空間，請參閱[使用 Azure 入口網站建立事件中樞命名空間和事件中樞](../event-hubs/event-hubs-create.md)。 然後，返回入口網站的此頁面，選取事件中樞命名空間和原則名稱。
    * **傳送至 Log Analytics**：若要使用此選項，請使用現有的工作區，或是在入口網站中依照[建立新的工作區](../log-analytics/log-analytics-quick-collect-azurevm.md#create-a-workspace)的步驟建立新的 Log Analytics 工作區。 如需關於如何檢視 Log Analytics 記錄的詳細資訊，請參閱[檢視 Log Analytics 中的記錄](#view-in-loganalytics)。
    * **記錄 DataPlaneRequests**：選取此選項，可從適用於 SQL、Graph、MongoDB、Cassandra 和資料表 API 帳戶的基礎 Azure Cosmos DB 分散式平台記錄後端要求。 如果您要封存至儲存體帳戶，您可以為診斷記錄選取保留期限。 保留期限過後，就會自動刪除記錄。
    * **記錄 MongoRequests**：選取此選項，可從為 MongoDB API 帳戶提供服務的 Azure Cosmos DB 前端記錄使用者起始的要求。 如果您要封存至儲存體帳戶，您可以為診斷記錄選取保留期限。 保留期限過後，就會自動刪除記錄。
    * **計量要求**：選取此選項可儲存 [Azure 計量](../monitoring-and-diagnostics/monitoring-supported-metrics.md)中的詳細資料。 如果您要封存至儲存體帳戶，您可以為診斷記錄選取保留期限。 保留期限過後，就會自動刪除記錄。

3. 選取 [ **儲存**]。

    如果您收到錯誤，指出「無法更新 \<工作區名稱> 的診斷。 訂用帳戶 \<訂用帳戶識別碼> 未註冊使用 microsoft.insights。請遵循[針對 Azure 診斷進行疑難排解](https://docs.microsoft.com/azure/log-analytics/log-analytics-azure-storage)的指示註冊帳戶，然後重試此程序。

    未來如果您想變更診斷記錄的儲存方式，請返回此頁面修改帳戶的診斷記錄設定。

## <a name="turn-on-logging-by-using-azure-cli"></a>使用 Azure CLI 開啟記錄

若要使用 Azure CLI 啟用計量和診斷記錄功能，請使用下列 Cmdlet：

- 若要在儲存體帳戶中啟用診斷記錄的儲存，請使用下列命令：

   ```azurecli-interactive
   azure insights diagnostic set --resourceId <resourceId> --storageId <storageAccountId> --enabled true
   ```

   `resourceId` 是 Azure Cosmos DB 帳戶的名稱。 `storageId` 是您要傳送記錄的目的地儲存體帳戶名稱。

- 若要啟用將診斷記錄檔串流至事件中樞，使用下列命令︰

   ```azurecli-interactive
   azure insights diagnostic set --resourceId <resourceId> --serviceBusRuleId <serviceBusRuleId> --enabled true
   ```

   `resourceId` 是 Azure Cosmos DB 帳戶的名稱。 `serviceBusRuleId` 將會是此格式的字串︰

   ```azurecli-interactive
   {service bus resource ID}/authorizationrules/{key name}
   ```

- 若要啟用將診斷記錄傳送至 Log Analytics 工作區的功能，請使用下列命令：

   ```azurecli-interactive
   azure insights diagnostic set --resourceId <resourceId> --workspaceId <resource id of the log analytics workspace> --enabled true
   ```

您可以結合這些參數讓多個輸出選項。

## <a name="turn-on-logging-by-using-powershell"></a>使用 PowerShell 開啟記錄

若要使用 PowerShell 開啟診斷記錄，您需要最低版本為 1.0.1 的 Azure Powershell。

若要安裝 Azure PowerShell，並將它與 Azure 訂用帳戶建立關聯，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

如果您尚未安裝 Azure PowerShell 且不知道版本，請從 PowerShell 主控台輸入 `(Get-Module azure -ListAvailable).Version`。  

### <a id="connect"></a>連線到您的訂用帳戶
開始 Azure PowerShell 工作階段，並使用下列命令登入您的 Azure 帳戶：  

```powershell
Connect-AzureRmAccount
```

在快顯瀏覽器視窗中，輸入您的 Azure 帳戶使用者名稱與密碼。 Azure PowerShell 會取得與此帳戶相關聯的所有訂用帳戶，並依預設使用第一個訂用帳戶。

如果您有多個訂用帳戶，您可能必須指定用來建立 Azure 金鑰保存庫的特定訂用帳戶。 若要查看您的帳戶的訂用帳戶，請輸入下列命令：

```powershell
Get-AzureRmSubscription
```

然後，若要指定與所要記錄之 Azure Cosmos DB 帳戶相關聯的訂用帳戶，請輸入下列命令：

```powershell
Set-AzureRmContext -SubscriptionId <subscription ID>
```

> [!NOTE]
> 如果您有多個與帳戶相關聯的訂用帳戶，請務必指定您要使用的訂用帳戶。
>
>

如需關於如何設定 Azure PowerShell 的詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

### <a id="storage"></a>建立新的儲存體帳戶來儲存記錄
雖然您可以使用現有的儲存體帳戶來儲存記錄，但在本教學課程中，我們將建立 Azure Cosmos DB 記錄專用的新儲存體帳戶。 為了方便起見，我們會將儲存體帳戶詳細資料儲存到名為 **sa** 的變數。

為了進一步簡化管理，在本教學課程中，我們會使用包含 Azure Cosmos DB 資料庫的相同資源群組。 視情況將 **ContosoResourceGroup**、**contosocosmosdblogs** 和 **North Central US** 等參數的值替換成您的值：

```powershell
$sa = New-AzureRmStorageAccount -ResourceGroupName ContosoResourceGroup `
-Name contosocosmosdblogs -Type Standard_LRS -Location 'North Central US'
```

> [!NOTE]
> 如果您決定使用現有儲存體帳戶，該帳戶必須使用與 Azure Cosmos DB 訂用帳戶相同的訂用帳戶。 該帳戶也必須使用 Resource Manager 部署模型，而非傳統部署模型。
>
>

### <a id="identify"></a>識別記錄的 Azure Cosmos DB 帳戶
將 Azure Cosmos DB 帳戶名稱設為名為 **account** 的變數，其中的 **ResourceName** 是 Azure Cosmos DB 帳戶的名稱。

```powershell
$account = Get-AzureRmResource -ResourceGroupName ContosoResourceGroup `
-ResourceName contosocosmosdb -ResourceType "Microsoft.DocumentDb/databaseAccounts"
```

### <a id="enable"></a>啟用記錄
若要啟用 Azure Cosmos DB 的記錄，請使用 `Set-AzureRmDiagnosticSetting` Cmdlet 搭配分別代表下列項目的變數：新的儲存體帳戶、Azure Cosmos DB 帳戶，以及要啟用記錄的類別。 執行下列命令，並將 **-Enabled** 旗標設為 **$true**：

```powershell
Set-AzureRmDiagnosticSetting  -ResourceId $account.ResourceId -StorageAccountId $sa.Id -Enabled $true -Categories DataPlaneRequests
```

命令的輸出應該會類似於下列範例：

```powershell
    StorageAccountId            : /subscriptions/<subscription-ID>/resourceGroups/ContosoResourceGroup/providers`
    /Microsoft.Storage/storageAccounts/contosocosmosdblogs
    ServiceBusRuleId            :
    EventHubAuthorizationRuleId :
    Metrics
        TimeGrain       : PT1M
        Enabled         : False
        RetentionPolicy
        Enabled : False
        Days    : 0
    
    Logs
        Category        : DataPlaneRequests
        Enabled         : True
        RetentionPolicy
        Enabled : False
        Days    : 0
    
    WorkspaceId                 :
    Id                          : /subscriptions/<subscription-ID>/resourcegroups/ContosoResourceGroup/providers`
    /microsoft.documentdb/databaseaccounts/contosocosmosdb/providers/microsoft.insights/diagnosticSettings/service
    Name                        : service
    Type                        :
    Location                    :
    Tags                        :
```

命令的輸出可確認您的資料庫現已啟用記錄，且資訊會儲存到您的儲存體帳戶中。

您也可以選擇性地設定記錄的保留原則，以便自動刪除較舊的記錄。 例如，您可以將 **-RetentionEnabled** 旗標設為 **$true**，以設定保留原則。 將 **-RetentionInDays** 參數設為 **90**，系統將自動刪除超過 90 天的舊記錄。

```powershell
Set-AzureRmDiagnosticSetting -ResourceId $account.ResourceId`
 -StorageAccountId $sa.Id -Enabled $true -Categories DataPlaneRequests`
  -RetentionEnabled $true -RetentionInDays 90
```

### <a id="access"></a>存取記錄
**DataPlaneRequests** 類別的 Azure Cosmos DB 記錄會儲存在您提供之儲存體帳戶的 **insights-logs-data-plane-requests** 容器中。 

首先，建立容器名稱的變數。 此變數會用於逐步解說的各個部分。

```powershell
    $container = 'insights-logs-dataplanerequests'
```

若要列出此容器中的所有 Blob，請輸入：

```powershell
Get-AzureStorageBlob -Container $container -Context $sa.Context
```

命令的輸出應該會類似於下列範例：

```powershell
ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob
BlobType          : BlockBlob
Length            : 10510193
ContentType       : application/octet-stream
LastModified      : 9/28/2017 7:49:04 PM +00:00
SnapshotTime      :
ContinuationToken:
Context           : Microsoft.WindowsAzure.Commands.Common.Storage.`
                    LazyAzureStorageContext
Name              : resourceId=/SUBSCRIPTIONS/<subscription-ID>/RESOURCEGROUPS/CONTOSORESOURCEGROUP/PROVIDERS`
/MICROSOFT.DOCUMENTDB/DATABASEACCOUNTS/CONTOSOCOSMOSDB/y=2017/m=09/d=28/h=19/m=00/PT1H.json
```

在此輸出中我們可以看到，blob 遵循以下命名慣例：`resourceId=/SUBSCRIPTIONS/<subscription-ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DOCUMENTDB/DATABASEACCOUNTS/<Database Account Name>/y=<year>/m=<month>/d=<day of month>/h=<hour>/m=<minute>/filename.json`

日期和時間值使用 UTC。

由於相同的儲存體帳戶可用來收集多個資源的記錄，因此您可以使用 Blob 名稱中的完整資源識別碼來存取和下載所需特定 Blob。 在這麼做之前，我們要先討論如何下載所有 Blob。

首先，建立資料夾來下載 blob。 例如︰

```powershell
New-Item -Path 'C:\Users\username\ContosoCosmosDBLogs'`
 -ItemType Directory -Force
```

然後，取得所有 Blob 的清單：  

```powershell
$blobs = Get-AzureStorageBlob -Container $container -Context $sa.Context
```

透過 `Get-AzureStorageBlobContent` 命令以管道傳送這份清單，將 Blob 下載到目的地資料夾中：

```powershell
$blobs | Get-AzureStorageBlobContent `
 -Destination 'C:\Users\username\ContosoCosmosDBLogs'
```

在執行第二個命令時，blob 名稱中的 **/** 分隔符號會在目的地資料夾下建立完整資料夾結構。 此資料夾結構將會用來下載 blob 並儲存為檔案。

若要有所選擇地下載 blob，請使用萬用字元。 例如︰

* 如果您有多個資料庫，並且只想下載其中名為 **CONTOSOCOSMOSDB3** 之資料庫的記錄，請使用下列命令：

    ```powershell
    Get-AzureStorageBlob -Container $container `
     -Context $sa.Context -Blob '*/DATABASEACCOUNTS/CONTOSOCOSMOSDB3
    ```

* 如果您有多個資源群組，並且只想下載其中某個資源群組的記錄，請使用 `-Blob '*/RESOURCEGROUPS/<resource group name>/*'` 命令：

    ```powershell
    Get-AzureStorageBlob -Container $container `
    -Context $sa.Context -Blob '*/RESOURCEGROUPS/CONTOSORESOURCEGROUP3/*'
    ```
* 如果您想下載 2017 年 7 月份的所有記錄，請使用 `-Blob '*/year=2017/m=07/*'` 命令：

    ```powershell
    Get-AzureStorageBlob -Container $container `
     -Context $sa.Context -Blob '*/year=2017/m=07/*'
    ```

您也可以執行下列命令：

* 若要查詢資料庫資源的診斷設定狀態，請使用 `Get-AzureRmDiagnosticSetting -ResourceId $account.ResourceId` 命令。
* 若要停用資料庫帳戶資源的 **DataPlaneRequests** 類別記錄，請使用 `Set-AzureRmDiagnosticSetting -ResourceId $account.ResourceId -StorageAccountId $sa.Id -Enabled $false -Categories DataPlaneRequests` 命令。


在這些查詢中傳回的 Blob 會儲存為文字，並格式化為 JSON Blob，如下列程式碼所示：

```json
{
    "records":
    [
        {
           "time": "Fri, 23 Jun 2017 19:29:50.266 GMT",
           "resourceId": "contosocosmosdb",
           "category": "DataPlaneRequests",
           "operationName": "Query",
           "resourceType": "Database",
           "properties": {"activityId": "05fcf607-6f64-48fe-81a5-f13ac13dd1eb",`
           "userAgent": "documentdb-dotnet-sdk/1.12.0 Host/64-bit MicrosoftWindowsNT/6.2.9200.0 AzureSearchIndexer/1.0.0",`
           "resourceType": "Database","statusCode": "200","documentResourceId": "",`
           "clientIpAddress": "13.92.241.0","requestCharge": "2.260","collectionRid": "",`
           "duration": "9250","requestLength": "72","responseLength": "209", "resourceTokenUserRid": ""}
        }
    ]
}
```

若要深入了解每個 JSON blob 中的資料，請參閱[解讀 Azure Cosmos DB 記錄](#interpret)。

## <a name="manage-your-logs"></a>管理您的記錄

Azure Cosmos DB 作業執行後兩個小時，就可以在您的帳戶中使用診斷記錄。 儲存體帳戶中的記錄由您全權管理：

* 請使用標準的 Azure 存取控制方法保護您的記錄，並限制可存取記錄的人員。
* 刪除不想繼續保留在儲存體帳戶中的記錄。
* 封存到儲存體帳戶之資料平面要求的保留期限，可在入口網站中藉由選取 [記錄 DataPlaneRequests] 設定來設定。 若要變更該設定，請參閱[在 Azure 入口網站中開啟記錄](#turn-on-logging-in-the-azure-portal)。


<a id="#view-in-loganalytics"></a>
## <a name="view-logs-in-log-analytics"></a>檢視 Log Analytics 中的記錄

如果您開啟診斷記錄時選取 [傳送至 Log Analytics] 選項，容器中的診斷資料會在兩個小時內轉送到 Log Analytics。 如果您在開啟記錄功能後立即查看 Log Analytics，將不會看到任何資料。 只需等待兩個小時，然後再試一次。 

檢視記錄前，請確認您的 Log Analytics 工作區是否已升級為使用新的 Log Analytics 查詢語言。 若要進行此確認，請開啟 [Azure 入口網站](https://portal.azure.com)，選取靠左側的 [Log Analytics]，然後選取工作區名稱，如下圖所示。 [OMS 工作區] 頁面隨即顯示：

![Azure 入口網站中的 Log Analytics](./media/logging/azure-portal.png)

如果您在 [OMS 工作區] 頁面上看到下列訊息，表示您的工作區尚未升級為使用新語言。 如需關於如何升級至新查詢語言的詳細資訊，請參閱[將 Azure Log Analytics 工作區升級至新的記錄搜尋](../log-analytics/log-analytics-log-search-upgrade.md)。 

![Log Analytics 升級訊息](./media/logging/upgrade-notification.png)

若要檢視 Log Analytics 中的診斷資料，請從左側功能表或此頁面的 [管理] 區域開啟 [記錄搜尋] 頁面，如下圖所示：

![Azure 入口網站中的記錄搜尋選項](./media/logging/log-analytics-open-log-search.png)

現在您已啟用資料收集，請使用新的查詢語言執行下列記錄搜尋範例，查看最新的 10 項記錄 `AzureDiagnostics | take 10`。

![10 項最新記錄的範例記錄搜尋](./media/logging/log-analytics-query.png)

<a id="#queries"></a>
### <a name="queries"></a>查詢

以下是一些您可以在 [記錄搜尋] 方塊中輸入以利監視 Azure Cosmos DB 容器的其他查詢。 這些查詢使用[新語言](../log-analytics/log-analytics-log-search-upgrade.md)。 

若要深入了解每個記錄搜尋傳回的資料所代表的涵義，請參閱[解讀 Azure Cosmos DB 記錄](#interpret)。

* 若要查詢 Azure Cosmos DB 中在指定的時段內的所有診斷記錄：

    ```
    AzureDiagnostics | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests"
    ```

* 若要查詢 10 個最近記錄的事件：

    ```
    AzureDiagnostics | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" | take 10
    ```

* 若要查詢所有作業 (依作業類型分組)：

    ```
    AzureDiagnostics | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" | summarize count() by OperationName
    ```

* 若要查詢所有作業 (依**資源**分組)：

    ```
    AzureActivity | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" | summarize count() by Resource
    ```

* 若要查詢所有使用者活動 (依資源分組)：

    ```
    AzureActivity | where Caller == "test@company.com" and ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" | summarize count() by Resource
    ```
    > [!NOTE]
    > 此命令適用於活動記錄，而非診斷記錄。

* 若要查詢哪些作業費時超過 3 毫秒：

    ```
    AzureDiagnostics | where toint(duration_s) > 30000 and ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" | summarize count() by clientIpAddress_s, TimeGenerated
    ```

* 若要查詢哪些代理程式正在執行此作業：

    ```
    AzureDiagnostics | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" | summarize count() by OperationName, userAgent_s
    ```

* 若要查詢長時間執行的作業於何時執行：

    ```
    AzureDiagnostics | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" | project TimeGenerated , toint(duration_s)/1000 | render timechart
    ```

如需關於如何使用新記錄搜尋語言的詳細資訊，請參閱[了解 Log Analytics 中的記錄搜尋](../log-analytics/log-analytics-log-search-new.md)。 

## <a id="interpret"></a>解讀記錄

儲存在 Azure 儲存體和 Log Analytics 中的診斷資料會使用類似的結構描述。 

下表說明每個記錄項目的內容。

| Azure 儲存體欄位或屬性 | Log Analytics 屬性 | 說明 |
| --- | --- | --- |
| **time** | **TimeGenerated** | 作業發生的日期和時間 (UTC)。 |
| **resourceId** | **Resource** | 啟用記錄的 Azure Cosmos DB 帳戶。|
| **類別** | **類別** | 對於 Azure Cosmos DB 記錄，**DataPlaneRequests** 是唯一的可用值。 |
| **operationName** | **OperationName** | 作業名稱。 這個值可以是下列任一作業：Create、Update、Read、ReadFeed、Delete、Replace、Execute、SqlQuery、Query、JSQuery、Head、HeadFeed 或 Upsert。   |
| **properties** | n/a | 此欄位的內容說明於下列資料列中。 |
| **activityId** | **activityId_g** | 所記錄作業的唯一 GUID。 |
| **userAgent** | **userAgent_s** | 此字串指定執行要求的用戶端使用者代理程式。 格式為 {使用者代理程式名稱}/{版本}。|
| **resourceType** | **ResourceType** | 存取的資源類型。 這個值可以是下列任一資源類型：Database、Container、Document、Attachment、User、Permission、StoredProcedure、Trigger、UserDefinedFunction 或 Offer。 |
| **statusCode** | **statusCode_s** | 作業的回應狀態。 |
| **requestResourceId** | **ResourceId** | 關於要求的 resourceId。 根據執行的作業，此值可能表示 databaseRid、collectionRid 或 documentRid。|
| **clientIpAddress** | **clientIpAddress_s** | 用戶端的 IP 位址。 |
| **requestCharge** | **requestCharge_s** | 作業使用的 RU 數 |
| **collectionRid** | **collectionId_s** | 集合的唯一識別碼。|
| **duration** | **duration_s** | 以刻度為單位的作業持續時間。 |
| **requestLength** | **requestLength_s** | 以位元組為單位的要求長度。 |
| **responseLength** | **responseLength_s** | 以位元組為單位的回應長度。|
| **resourceTokenUserRid** | **resourceTokenUserRid_s** | 使用[資源權杖](https://docs.microsoft.com/azure/cosmos-db/secure-access-to-data#resource-tokens)進行驗證時，此值為非空白值。 此值表示使用者的資源識別碼。 |

## <a name="next-steps"></a>後續步驟

- 若要了解如何啟用記錄，以及各種 Azure 服務支援的計量和記錄類別，請閱讀 [Microsoft Azure 中的計量概觀](../monitoring-and-diagnostics/monitoring-overview-metrics.md)和 [Azure 診斷記錄概觀](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)文章。
- 閱讀下列文章來了解事件中樞：
   - [Azure 事件中樞是什麼？](../event-hubs/event-hubs-what-is-event-hubs.md)
   - [開始使用事件中心](../event-hubs/event-hubs-csharp-ephcs-getstarted.md)
- 閱讀[從 Azure 儲存體下載計量和診斷記錄](../storage/blobs/storage-quickstart-blobs-dotnet.md#download-blobs)。
- 閱讀[了解 Log Analytics 中的記錄搜尋](../log-analytics/log-analytics-log-search-new.md)。
