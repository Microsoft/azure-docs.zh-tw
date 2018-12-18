---
title: 匯入資料以用於 Azure Cosmos DB 資料表 API | Microsoft Docs
description: 了解如何匯入資料以用於 Azure Cosmos DB 資料表 API。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: na
ms.topic: tutorial
ms.date: 11/28/2017
ms.author: sngun
ms.openlocfilehash: 8614bf7009afab8c005d2ffb8b1a78f3694d2af7
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48247504"
---
# <a name="migrate-your-data-to-azure-cosmos-db-table-api-account"></a>將您的資料移轉至 Azure Cosmos DB 資料表 API 帳戶

本教學課程提供如何匯入資料以用於 Azure Cosmos DB [資料表 API](table-introduction.md) 的指示。 如果您將資料儲存在 Azure 資料表儲存體中，您可以使用資料移轉工具或 AzCopy 將資料匯入 Azure Cosmos DB 資料表 API 中。 如果您將資料儲存在 Azure Cosmos DB 資料表 API (預覽) 帳戶，您必須使用資料移轉工具來移轉資料。 

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 使用資料移轉工具匯入資料
> * 使用 AzCopy 匯入資料
> * 從資料表 API (預覽) 移轉至資料表 API 

## <a name="prerequisites"></a>必要條件

* **增加輸送量︰** 資料移轉的時間長短取決於您為個別容器或一組容器設定的輸送量。 針對較大資料移轉，請務必增加輸送量。 完成移轉之後，再降低輸送量以節省成本。 如需在 Azure 入口網站增加輸送量的詳細資訊，請參閱 Azure Cosmos DB 中的效能等級和定價層。

* **建立 Azure Cosmos DB 資源：** 在您開始遷移資料之前，請先從 Azure 入口網站預先建立所有資料表。 如果您要遷移至具有資料庫層級輸送量的 Azure Cosmos DB 帳戶，請務必在建立 Azure Cosmos DB 資料表時提供分割區索引鍵。

## <a name="data-migration-tool"></a>資料移轉工具

命令列 Azure Cosmos DB 資料移轉工具 (dt.exe) 可用來將您現有的 Azure 資料表儲存體資料匯入到資料表 API GA 帳戶，或將資料從資料表 API (預覽) 帳戶移轉至資料表 API GA 帳戶。 目前不支援其他來源。 UI 型資料移轉工具 (dtui.exe) 目前不支援資料表 API 帳戶。 

若要執行資料表資料移轉，請完成下列工作：

1. 請從 [GitHub](https://github.com/azure/azure-documentdb-datamigrationtool) 下載移轉工具。
2. 使用您案例的命令列引數執行 `dt.exe`。 `dt.exe` 會採用下列格式的命令：

   ```bash
    dt.exe [/<option>:<value>] /s:<source-name> [/s.<source-option>:<value>] /t:<target-name> [/t.<target-option>:<value>] 
```

此命令的選項如下：

    /ErrorLog: Optional. Name of the CSV file to redirect data transfer failures
    /OverwriteErrorLog: Optional. Overwrite error log file
    /ProgressUpdateInterval: Optional, default is 00:00:01. Time interval to refresh on-screen data transfer progress
    /ErrorDetails: Optional, default is None. Specifies that detailed error information should be displayed for the following errors: None, Critical, All

### <a name="command-line-source-settings"></a>命令列來源設定

將 Azure 資料表儲存體或資料表 API 預覽版定義為移轉來源時，請使用下列來源選項。

    /s:AzureTable: Reads data from Azure Table storage
    /s.ConnectionString: Connection string for the table endpoint. This can be retrieved from the Azure portal
    /s.LocationMode: Optional, default is PrimaryOnly. Specifies which location mode to use when connecting to Azure Table storage: PrimaryOnly, PrimaryThenSecondary, SecondaryOnly, SecondaryThenPrimary
    /s.Table: Name of the Azure Table
    /s.InternalFields: Set to All for table migration as RowKey and PartitionKey are required for import.
    /s.Filter: Optional. Filter string to apply
    /s.Projection: Optional. List of columns to select

在從 Azure 資料表儲存體匯入時若要擷取來源的連接字串，請開啟 Azure 入口網站，按一下 [儲存體帳戶] > [帳戶] > [存取金鑰]，然後使用 [複製] 按鈕複製 [連接字串]。

![HBase 檔案來源選項的螢幕擷取畫面](./media/table-import/storage-table-access-key.png)

在從 Azure Cosmos DB 資料表 API (預覽) 帳戶匯入時若要擷取來源的連接字串，請開啟 Azure 入口網站，按一下 [Azure Cosmos DB] > [帳戶] > [連接字串]，然後使用 [複製] 按鈕複製 [連接字串]。

![HBase 檔案來源選項的螢幕擷取畫面](./media/table-import/cosmos-connection-string.png)

[Azure 資料表儲存體的命令範例](#azure-table-storage)

[Azure Cosmos DB 資料表 API (預覽) 的命令範例](#table-api-preview)

### <a name="command-line-target-settings"></a>命令列目標設定

將 Azure Cosmos DB 資料表 API 定義為移轉目標時，請使用下列目標選項。

    /t:TableAPIBulk: Uploads data into Azure CosmosDB Table in batches
    /t.ConnectionString: Connection string for the table endpoint
    /t.TableName: Specifies the name of the table to write to
    /t.Overwrite: Optional, default is false. Specifies if existing values should be overwritten
    /t.MaxInputBufferSize: Optional, default is 1GB. Approximate estimate of input bytes to buffer before flushing data to sink
    /t.Throughput: Optional, service defaults if not specified. Specifies throughput to configure for table
    /t.MaxBatchSize: Optional, default is 2MB. Specify the batch size in bytes

<a id="azure-table-storage"></a>
### <a name="sample-command-source-is-azure-table-storage"></a>命令範例：來源是 Azure 資料表儲存體

以下命令列範例會說明如何從 Azure 資料表儲存體匯入到資料表 API：

```
dt /s:AzureTable /s.ConnectionString:DefaultEndpointsProtocol=https;AccountName=<Azure Table storage account name>;AccountKey=<Account Key>;EndpointSuffix=core.windows.net /s.Table:<Table name> /t:TableAPIBulk /t.ConnectionString:DefaultEndpointsProtocol=https;AccountName=<Azure Cosmos DB account name>;AccountKey=<Azure Cosmos DB account key>;TableEndpoint=https://<Account name>.table.cosmosdb.azure.com:443 /t.TableName:<Table name> /t.Overwrite
```
<a id="table-api-preview"></a>
### <a name="sample-command-source-is-azure-cosmos-db-table-api-preview"></a>命令範例：來源是 Azure Cosmos DB 資料表 API (預覽)

以下是從資料表 API 預覽版匯入到資料表 API GA 的命令列範例：

```
dt /s:AzureTable /s.ConnectionString:DefaultEndpointsProtocol=https;AccountName=<Table API preview account name>;AccountKey=<Table API preview account key>;TableEndpoint=https://<Account Name>.documents.azure.com; /s.Table:<Table name> /t:TableAPIBulk /t.ConnectionString:DefaultEndpointsProtocol=https;AccountName=<Azure Cosmos DB account name>;AccountKey=<Azure Cosmos DB account key>;TableEndpoint=https://<Account name>.table.cosmosdb.azure.com:443 /t.TableName:<Table name> /t.Overwrite
```

## <a name="migrate-data-by-using-azcopy"></a>使用 AzCopy 移轉資料

使用 AzCopy 命令列公用程式是另一個用來將資料從 Azure 資料表儲存體移轉到 Azure Cosmos DB 資料表 API 的選項。 若要使用 AzCopy，您必須先匯出資料 (如[從資料表儲存體匯出資料](../storage/common/storage-use-azcopy.md#export-data-from-table-storage)所述)，然後將資料匯入到 Azure Cosmos DB (如 [Azure Cosmos DB 資料表 API](../storage/common/storage-use-azcopy.md#import-data-into-table-storage) 所述)。

在執行目標為 Azure Cosmos DB 的匯入作業時，請參閱下列範例。 請注意，/Dest 值使用 cosmosdb，而非 core。

匯入命令範例：

```
AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.table.cosmosdb.windows.net/mytable1/ /DestKey:key /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:InsertOrReplace
```

## <a name="migrate-from-table-api-preview-to-table-api"></a>從資料表 API (預覽) 移轉至資料表 API

> [!WARNING]
> 如果您想要立即享受正式推出之資料表的好處，請依本節所述地移轉您現有的預覽版資料表，否則，我們將在未來幾週為現有的預覽版客戶執行自動移轉，但請注意，自動移轉的預覽版資料表會受到某些限制，而新建立的資料表則沒有這些限制。
> 

資料表 API 現已正式推出 (GA)。 在執行於雲端的程式碼中以及在執行於用戶端的程式碼中，預覽版和 GA 版的資料表皆會有差異。 因此，不建議您嘗試混合使用預覽版 SDK 用戶端與 GA 版資料表 API 帳戶，反之亦然。 想要繼續使用其現有資料表，但環境改為生產環境的資料表 API 預覽版客戶，必須從預覽版移轉至 GA 版環境，或者，他們也可以等候自動移轉。 如果您要等候自動移轉，系統會通知您移轉的資料表會受到限制。 移轉之後，您就可以在現有帳戶上毫無限制地建立新資料表 (只有經過移轉的資料表會有限制)。

若要從資料表 API (預覽版) 移轉至正式推出版資料表 API：

1. 建立新的 Azure Cosmos DB 帳戶，並將其 API 類型設定為 Azure 資料表，如[建立資料庫帳戶](create-table-dotnet.md#create-a-database-account)所述。

2. 將用戶端變更為使用 GA 版的[資料表 API SDK](table-sdk-dotnet.md)。

3. 使用資料移轉工具，將用戶端資料從預覽版資料表移轉至 GA 版資料表。 如需如何使用資料移轉工具來進行此用途的指示，請參閱[資料移轉工具](#data-migration-tool)的說明。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 使用資料移轉工具匯入資料
> * 使用 AzCopy 匯入資料
> * 從資料表 API (預覽) 移轉至資料表 API

您現在可以繼續進行下一個教學課程，了解如何使用 Azure Cosmos DB 資料表 API 查詢資料。 

> [!div class="nextstepaction"]
>[如何查詢資料？](../cosmos-db/tutorial-query-table.md)
