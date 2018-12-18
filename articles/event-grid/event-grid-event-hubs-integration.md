---
title: Azure Event Grid 和事件中樞整合
description: 說明如何使用 Azure Event Grid 和事件中樞將資料移轉到 SQL 資料倉儲
services: event-grid
author: tfitzmac
manager: timlt
ms.service: event-grid
ms.topic: tutorial
ms.date: 08/22/2018
ms.author: tomfitz
ms.openlocfilehash: aad7a24d8b0e0bc74815cad3604db1cc21a6db96
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44163221"
---
# <a name="stream-big-data-into-a-data-warehouse"></a>將巨量資料串流處理至資料倉儲

Azure [Event Grid](overview.md) 是一項智慧型的事件路由服務，它能讓您針對應用程式和服務發出的通知做出反應。 例如，它可以觸發 Azure 函式以處理已擷取至 Azure Blob 儲存體或 Data Lake Store 的事件中樞資料，並將資料移轉至其他資料存放庫。 此[事件中樞擷取和事件格線範例](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo)說明如何使用事件中樞擷取搭配事件格線，將事件中樞資料順暢地從 Blob 儲存體移轉至 SQL 資料倉儲。

![應用程式概觀](media/event-grid-event-hubs-integration/overview.png)

由於資料會傳送到事件中樞，所以擷取能從資料流中提取資料，再以 Avro 格式的資料產生儲存體 Blob。 當擷取產生 Blob 時會觸發事件。 Event Grid 會將事件的相關資料散發給訂閱者。 在本案例中，事件資料會傳送到 Azure Functions 端點。 事件資料包括產生之 Blob 的路徑。 函式會使用該 URL 來取出檔案，然後將其傳送到資料倉儲。

在本文章中，您將：

* 部署下列基礎結構：
  * 啟用擷取的事件中樞
  * 儲存體帳戶，用來容納擷取傳出的檔案
  * Azure App Service 方案，用來裝載函式應用程式
  * 函式應用程式，用來處理事件
  * SQL Server，用來裝載資料倉儲
  * SQL 資料倉儲，用來儲存移轉的資料
* 在資料倉儲中建立資料表
* 將程式碼新增到函數應用程式
* 訂閱事件
* 執行應用程式，將資料傳送到事件中樞
* 在資料倉儲中檢視移轉的資料

## <a name="about-the-event-data"></a>關於事件資料

Event Grid 會將事件資料散發給訂閱者。 以下範例展示用來建立擷取檔案的事件資料。 請特別注意 `data` 物件中的 `fileUrl` 屬性。 函數應用程式會取得該值，並使用它來取出擷取檔案。

```json
[
    {
        "topic": "/subscriptions/<guid>/resourcegroups/rgDataMigrationSample/providers/Microsoft.EventHub/namespaces/tfdatamigratens",
        "subject": "eventhubs/hubdatamigration",
        "eventType": "Microsoft.EventHub.CaptureFileCreated",
        "eventTime": "2017-08-31T19:12:46.0498024Z",
        "id": "14e87d03-6fbf-4bb2-9a21-92bd1281f247",
        "data": {
            "fileUrl": "https://tf0831datamigrate.blob.core.windows.net/windturbinecapture/tfdatamigratens/hubdatamigration/1/2017/08/31/19/11/45.avro",
            "fileType": "AzureBlockBlob",
            "partitionId": "1",
            "sizeInBytes": 249168,
            "eventCount": 1500,
            "firstSequenceNumber": 2400,
            "lastSequenceNumber": 3899,
            "firstEnqueueTime": "2017-08-31T19:12:14.674Z",
            "lastEnqueueTime": "2017-08-31T19:12:44.309Z"
        }
    }
]
```

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您必須具備：

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。
* [Visual Studio 2017 15.3.2 或更新版本](https://www.visualstudio.com/vs/)，搭配適用於 .NET 桌面開發、Azure 開發、ASP.NET 和 Web 開發、Node.js 開發及 Python 開發的工作負載。
* 已下載到電腦的 [EventHubsCaptureEventGridDemo 範例專案](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo)。

## <a name="deploy-the-infrastructure"></a>部署基礎結構

若要簡化本文章的內容，您可以利用 Resource Manager 範本部署必要的基礎結構。 若要查看已部署的資源，請檢視[範本](https://github.com/Azure/azure-docs-json-samples/blob/master/event-grid/EventHubsDataMigration.json)。

對於 Azure CLI，請使用：

```azurecli-interactive
az group create -l westcentralus -n rgDataMigrationSample

az group deployment create \
  --resource-group rgDataMigrationSample \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/event-grid/EventHubsDataMigration.json \
  --parameters eventHubNamespaceName=<event-hub-namespace> eventHubName=hubdatamigration sqlServerName=<sql-server-name> sqlServerUserName=<user-name> sqlServerPassword=<password> sqlServerDatabaseName=<database-name> storageName=<unique-storage-name> functionAppName=<app-name>
```

對於 PowerShell，請使用：

```powershell
New-AzureRmResourceGroup -Name rgDataMigration -Location westcentralus

New-AzureRmResourceGroupDeployment -ResourceGroupName rgDataMigration -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/event-grid/EventHubsDataMigration.json -eventHubNamespaceName <event-hub-namespace> -eventHubName hubdatamigration -sqlServerName <sql-server-name> -sqlServerUserName <user-name> -sqlServerDatabaseName <database-name> -storageName <unique-storage-name> -functionAppName <app-name>
```

出現提示時，請提供密碼值。

## <a name="create-a-table-in-sql-data-warehouse"></a>在 SQL 資料倉儲中建立資料表

執行 [CreateDataWarehouseTable.sql](https://github.com/Azure/azure-event-hubs/blob/master/samples/e2e/EventHubsCaptureEventGridDemo/scripts/CreateDataWarehouseTable.sql) 指令碼，將資料表新增到資料倉儲。 若要執行指令碼，請在入口網站中使用 Visual Studio 或查詢編輯器。

要執行的指令碼是：

```sql
CREATE TABLE [dbo].[Fact_WindTurbineMetrics] (
    [DeviceId] nvarchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL, 
    [MeasureTime] datetime NULL, 
    [GeneratedPower] float NULL, 
    [WindSpeed] float NULL, 
    [TurbineSpeed] float NULL
)
WITH (CLUSTERED COLUMNSTORE INDEX, DISTRIBUTION = ROUND_ROBIN);
```

## <a name="publish-the-azure-functions-app"></a>發佈 Azure Functions 應用程式

1. 在 Visual Studio 2017 (15.3.2 或更新版本) 中開啟 [EventHubsCaptureEventGridDemo 範例專案](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo)。

1. 在 [方案總管] 中，以滑鼠右鍵按一下 [FunctionEGDWDumper]，然後選取 [發佈]。

   ![發佈函數應用程式](media/event-grid-event-hubs-integration/publish-function-app.png)

1. 選取 [Azure 函數應用程式] 和 [選取現有的]。 選取 [發佈] 。

   ![目標函數應用程式](media/event-grid-event-hubs-integration/pick-target.png)

1. 選取透過範本部署的函數應用程式。 選取 [確定] 。

   ![選取函數應用程式](media/event-grid-event-hubs-integration/select-function-app.png)

1. 當 Visual Studio 完成設定檔設定時，選取 [發佈]。

   ![Select publish](media/event-grid-event-hubs-integration/select-publish.png)

發行函式之後，您已準備好訂閱事件。

## <a name="subscribe-to-the-event"></a>訂閱事件

1. 移至 [Azure 入口網站](https://portal.azure.com/)。 選取您的資源群組和函數應用程式。

   ![檢視函數應用程式](media/event-grid-event-hubs-integration/view-function-app.png)

1. 選取函式。

   ![選取函式](media/event-grid-event-hubs-integration/select-function.png)

1. 選取 [新增事件方格訂用帳戶]。

   ![加入訂閱](media/event-grid-event-hubs-integration/add-event-grid-subscription.png)

9. 為事件方格訂用帳戶指定名稱。 使用 [事件中樞命名空間] 作為事件類型。 提供值以選取您的事件中樞命名空間執行個體。 將訂閱者端點保留為提供的值。 選取 [建立] 。

   ![建立訂用帳戶](media/event-grid-event-hubs-integration/set-subscription-values.png)

## <a name="run-the-app-to-generate-data"></a>執行應用程式以產生資料

您已完成事件中樞、SQL 資料倉儲、Azure 函數應用程式及事件訂閱的設定。 解決方案已準備好將資料從事件中樞移轉到資料倉儲。 在執行產生事件中樞資料的應用程式之前，您需要設定幾個值。

1. 在入口網站中，選取您的事件中樞命名空間。 選取 [連接字串]。

   ![選取連接字串](media/event-grid-event-hubs-integration/event-hub-connection.png)

2. 選取 [RootManageSharedAccessKey]

   ![選取索引鍵](media/event-grid-event-hubs-integration/show-root-key.png)

3. 複製 [連接字串 - 主索引鍵]

   ![複製金鑰](media/event-grid-event-hubs-integration/copy-key.png)

4. 返回 Visual Studio 專案。 在 WindTurbineDataGenerator 專案中，開啟 **program.cs**。

5. 取代兩個常數值。 使用複製的值來取代 **EventHubConnectionString**。 使用 **hubdatamigration** 作為事件中樞名稱。

   ```cs
   private const string EventHubConnectionString = "Endpoint=sb://demomigrationnamespace.servicebus.windows.net/...";
   private const string EventHubName = "hubdatamigration";
   ```

6. 建置方案。 執行 WindTurbineGenerator.exe 應用程式。 在幾分鐘之後，查詢資料倉儲中的資料表來找出移轉的資料。

## <a name="next-steps"></a>後續步驟

* 若要了解 Azure 傳訊服務的差異，請參閱[在傳遞訊息的 Azure 服務之間做選擇](compare-messaging-services.md)。
* 如需 Event Grid 的簡介，請參閱[關於 Event Grid](overview.md)。
* 如需事件中樞擷取的簡介，請參閱[使用 Azure 入口網站啟用事件中樞擷取](../event-hubs/event-hubs-capture-enable-through-portal.md)。
* 如需設定及執行範例的詳細資訊，請參閱[事件中樞擷取和 Event Grid 範例](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo)。