---
title: Data Factory - .NET API 變更記錄 | Microsoft Docs
description: 描述 Azure Data Factory 的特定版本 NET API 中的重大變更、新增功能、錯誤修正等等...
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
editor: ''
ms.assetid: 8208271b-7f4c-4214-b665-d2ff503c4470
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: f53c9e2b21e4758bccb6b0f89eb69501df2a6009
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37051394"
---
# <a name="azure-data-factory---net-api-change-log"></a>Azure Data Factory - .NET API 變更記錄
> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 

本文章提供 Azure Data Factory SDK 在特定版本中有何變更的相關資訊。 您可以在 [這裡](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactories)

## <a name="version-4110"></a>版本 4.11.0
新增功能︰

* 已加入下列連結服務類型：
  * [OnPremisesMongoDbLinkedService](https://msdn.microsoft.com/library/mt765129.aspx)
  * [AmazonRedshiftLinkedService](https://msdn.microsoft.com/library/mt765121.aspx)
  * [AwsAccessKeyLinkedService](https://msdn.microsoft.com/library/mt765144.aspx)
* 已加入下列資料集類型：
  * [MongoDbCollectionDataset](https://msdn.microsoft.com/library/mt765145.aspx)
  * [AmazonS3Dataset](https://msdn.microsoft.com/library/mt765112.aspx)
* 已加入下列複製來源類型：
  * [MongoDbSource](https://msdn.microsoft.com/library/mt765123.aspx)

## <a name="version-4100"></a>版本 4.10.0
* 下列選擇性屬性已新增至 TextFormat：
  * [SkipLineCount](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.textformat.skiplinecount.aspx)
  * [FirstRowAsHeader](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.textformat.firstrowasheader.aspx)
  * [TreatEmptyAsNull](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.textformat.treatemptyasnull.aspx)
* 已加入下列連結服務類型：
  * [OnPremisesCassandraLinkedService](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.onpremisescassandralinkedservice.aspx)
  * [SalesforceLinkedService](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.salesforcelinkedservice.aspx)
* 已加入下列資料集類型：
  * [OnPremisesCassandraTableDataset](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.onpremisescassandratabledataset.aspx)
* 已加入下列複製來源類型：
  * [CassandraSource](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.cassandrasource.aspx)
* 新增 [WebServiceInputs](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.azuremlbatchexecutionactivity.webserviceinputs.aspx) 屬性至 AzureMLBatchExecutionActivity
  * 可將多個 Web 服務輸入傳遞到 Azure Machine Learning 實驗

## <a name="version-491"></a>版本 4.9.1
### <a name="bug-fix"></a>錯誤修正
* 取代 [WebLinkedService](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.weblinkedservice.authenticationtype.aspx)的 WebApi 型驗證。

## <a name="version-490"></a>版本 4.9.0
### <a name="feature-additions"></a>新增功能
* 新增 CopyActivity 的 [EnableStaging](https://msdn.microsoft.com/library/mt767916.aspx) 和 [StagingSettings](https://msdn.microsoft.com/library/mt767918.aspx) 屬性。 如需功能的詳細資訊，請參閱 [分段複製](data-factory-copy-activity-performance.md#staged-copy) 。

### <a name="bug-fix"></a>錯誤修正
* 導入 [ActivityWindowOperationExtensions.List](https://msdn.microsoft.com/library/mt767915.aspx) 方法的多載，它會採用 [ActivityWindowsByActivityListParameters](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.activitywindowsbyactivitylistparameters.aspx) 執行個體。
* 將 [WriteBatchSize](https://msdn.microsoft.com/library/dn884293.aspx) 和 [WriteBatchTimeout](https://msdn.microsoft.com/library/dn884245.aspx) 標示為 CopySink 中的選擇性項目。

## <a name="version-480"></a>版本 4.8.0
### <a name="feature-additions"></a>新增功能
* 複製活動類型中已新增下列選擇性屬性，以啟用複製效能的微調︰
  * [ParallelCopies](https://msdn.microsoft.com/library/mt767910.aspx)
  * [CloudDataMovementUnits](https://msdn.microsoft.com/library/mt767912.aspx)

## <a name="version-470"></a>4.7.0 版
### <a name="feature-additions"></a>新增功能
* 加入了新的 StorageFormat 類型 [OrcFormat](https://msdn.microsoft.com/library/mt723391.aspx) 類型，可用來複製最佳化資料列單欄式 (ORC) 格式的檔案。
* 為 SqlDWSink 新增 [AllowPolyBase](https://msdn.microsoft.com/library/mt723396.aspx) 和 PolyBaseSettings 屬性。
  * 允許使用 PolyBase 將資料複製到「SQL 資料倉儲」。

## <a name="version-461"></a>4.6.1 版
### <a name="bug-fixes"></a>錯誤修正
* 修正用於列出活動時段的 HTTP 要求。
  * 從要求承載移除資源群組名稱和 Data Factory 名稱。

## <a name="version-460"></a>4.6.0 版
### <a name="feature-additions"></a>新增功能
* 下列屬性已新增至 [PipelineProperties](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.pipelineproperties_properties.aspx)：
  * [PipelineMode](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.pipelineproperties.pipelinemode.aspx)
  * [ExpirationTime](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.pipelineproperties.expirationtime.aspx)
  * [資料集](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.pipelineproperties.datasets.aspx)
* 下列屬性已新增至 [PipelineRuntimeInfo](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.common.models.pipelineruntimeinfo.aspx)：
  * [PipelineState](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.common.models.pipelineruntimeinfo.pipelinestate.aspx)
* 加入了新的 [StorageFormat](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.storageformat.aspx) 類型 [JsonFormat](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.jsonformat.aspx) 類型，可用來定義資料採用 JSON 格式的資料集。

## <a name="version-450"></a>4.5.0 版
### <a name="feature-additions"></a>新增功能
* 加入了 [活動時段的清單作業](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.activitywindowoperationsextensions.aspx)。
  * 已根據實體類型 (即資料處理站、資料集、管線和活動)，加入了擷取有篩選器活動視窗的方法。
* 已加入下列連結服務類型：
  * [ODataLinkedService](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.odatalinkedservice.aspx)、[WebLinkedService](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.weblinkedservice.aspx)
* 已加入下列資料集類型：
  * [ODataResourceDataset](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.odataresourcedataset.aspx)、[WebTableDataset](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.webtabledataset.aspx)
* 已加入下列複製來源類型：     
  * [WebSource](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.websource.aspx)

## <a name="version-440"></a>4.4.0 版
### <a name="feature-additions"></a>新增功能
* 已新增下列連結服務類型來做為複製活動的資料來源和接收器：
  * [AzureStorageSasLinkedService](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.azurestoragesaslinkedservice.aspx)。 如需概念性資訊和範例，請參閱 [Azure 儲存體 SAS 連結服務](data-factory-azure-blob-connector.md#azure-storage-sas-linked-service) 。

## <a name="version-430"></a>4.3.0 版
### <a name="feature-additions"></a>新增功能
* 已新增下列連結服務類型來做為複製活動的資料來源：
  * [HdfsLinkedService](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.hdfslinkedservice.aspx)。 如需概念性資訊和範例，請參閱 [使用 Data Factory 從 HDFS 移動資料](data-factory-hdfs-connector.md) 。
  * [OnPremisesOdbcLinkedService](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.onpremisesodbclinkedservice.aspx)。 如需概念性資訊和範例，請參閱 [使用 Azure Data Factory 從 ODBC 資料存放區移動資料](data-factory-odbc-connector.md) 。

## <a name="version-420"></a>4.2.0 版
### <a name="feature-additions"></a>新增功能
* 新增下列新的活動類型： [AzureMLUpdateResourceActivity](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuremlupdateresourceactivity.aspx)。 如需有關活動的詳細資訊，請參閱[使用更新資源活動更新 Azure ML 模型](data-factory-azure-ml-batch-execution-activity.md)。
* 新的選擇性屬性 [updateResourceEndpoint](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuremllinkedservice.updateresourceendpoint.aspx) 已加入至 [AzureMLLinkedService 類別](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuremllinkedservice.aspx)。
* [LongRunningOperationInitialTimeout](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.datafactorymanagementclient.longrunningoperationinitialtimeout.aspx) 和 [LongRunningOperationRetryTimeout](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.datafactorymanagementclient.longrunningoperationretrytimeout.aspx) 屬性已加入至 [DataFactoryManagementClient](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.datafactorymanagementclient.aspx) 類別。
* 允許設定用戶端呼叫 Data Factory 服務的逾時值。

## <a name="version-410"></a>4.1.0 版
### <a name="feature-additions"></a>新增功能
* 已加入下列連結服務類型：
  * [AzureDataLakeStoreLinkedService](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice.aspx)
  * [AzureDataLakeAnalyticsLinkedService](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice.aspx)
* 已加入下列活動類型：
  * [DataLakeAnalyticsUSQLActivity](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.datalakeanalyticsusqlactivity.aspx)
* 已加入下列資料集類型：
  * [AzureDataLakeStoreDataset](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestoredataset.aspx)
* 已加入複製活動的下列來源和接收器類型：
  * [AzureDataLakeStoreSource](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestoresource.aspx)
  * [AzureDataLakeStoreSink](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestoresink.aspx)

## <a name="version-401"></a>4.0.1 版
### <a name="breaking-changes"></a>重大變更
下列類別已重新命名。 新的名稱是 4.0.0 版之前的原始類別名稱。

| 4.0.0 中的名稱 | 4.0.1 中的名稱 |
|:--- |:--- |
| AzureSqlDataWarehouseDataset |[AzureSqlDataWarehouseTableDataset](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuresqldatawarehousetabledataset.aspx) |
| AzureSqlDataset |[AzureSqlTableDataset](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuresqltabledataset.aspx) |
| AzureDataset |[AzureTableDataset](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuretabledataset.aspx) |
| OracleDataset |[OracleTableDataset](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.oracletabledataset.aspx) |
| RelationalDataset |[RelationalTableDataset](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.relationaltabledataset.aspx) |
| SqlServerDataset |[SqlServerTableDataset](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.sqlservertabledataset.aspx) |

## <a name="version-400"></a>4.0.0 版
### <a name="breaking-changes"></a>重大變更
* 下列類別/介面已重新命名。

| 舊名稱 | 新名稱 |
|:--- |:--- |
| ITableOperations |[IDatasetOperations](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.idatasetoperations.aspx) |
| 資料表 |[Dataset](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.dataset.aspx) |
| TableProperties |[DatasetProperties](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.datasetproperties.aspx) |
| TableTypeProprerties |[DatasetTypeProperties](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.datasettypeproperties.aspx) |
| TableCreateOrUpdateParameters |[DatasetCreateOrUpdateParameters](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.datasetcreateorupdateparameters.aspx) |
| TableCreateOrUpdateResponse |[DatasetCreateOrUpdateResponse](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.datasetcreateorupdateresponse.aspx) |
| TableGetResponse |[DatasetGetResponse](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.datasetgetresponse.aspx) |
| TableListResponse |[DatasetListResponse](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.datasetlistresponse.aspx) |
| CreateOrUpdateWithRawJsonContentParameters |[DatasetCreateOrUpdateWithRawJsonContentParameters](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.datasetcreateorupdatewithrawjsoncontentparameters.aspx) |

* **List** 方法現在會傳回分頁式結果。 如果回應包含非空白的 **NextLink** 屬性，用戶端應用程式需要繼續提取下一個頁面，直到傳回了所有頁面為止。  下列是一個範例：

    ```csharp
    PipelineListResponse response = client.Pipelines.List("ResourceGroupName", "DataFactoryName");
    var pipelines = new List<Pipeline>(response.Pipelines);

    string nextLink = response.NextLink;
    while (!string.IsNullOrEmpty(nextLink))
    {
        PipelineListResponse nextResponse = client.Pipelines.ListNext(nextLink);
        pipelines.AddRange(nextResponse.Pipelines);

        nextLink = nextResponse.NextLink;
    }
    ```
* **List** 管線 API 只會傳回管線的摘要，而不是完整的詳細資料。 例如，管線摘要中的活動只包含名稱和類型。

### <a name="feature-additions"></a>新增功能
* [SqlDWSink](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.sqldwsink.aspx) 類別支援兩個新屬性 ( **SliceIdentifierColumnName** 和 **SqlWriterCleanupScript**)，以支援將資料等冪複製到 Azure SQL 資料倉儲。 如需有關這些屬性的詳細資料，請參閱 [Azure SQL 資料倉儲](data-factory-azure-sql-data-warehouse-connector.md)文章。
* 我們現在支援對 Azure SQL Database 和 Azure SQL 資料倉儲的來源執行預存程序，做為複製活動的一部分。 [SqlSource](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.sqlsource.aspx) 和 [SqlDWSource](https://msdn.microsoft.com/library/azure/microsoft.azure.management.datafactories.models.sqldwsource.aspx) 類別具有下列屬性：**SqlReaderStoredProcedureName** 和 **StoredProcedureParameters**。 如需這些屬性的詳細資料，請參閱 Azure.com 上的 [Azure SQL Database](data-factory-azure-sql-connector.md#sqlsource) 和 [Azure SQL 資料倉儲](data-factory-azure-sql-data-warehouse-connector.md#sqldwsource)文章。  
