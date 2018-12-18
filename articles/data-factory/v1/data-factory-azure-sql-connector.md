---
title: 從 Azure SQL Database 來回複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從 Azure SQL 資料庫來回複製資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 484f735b-8464-40ba-a9fc-820e6553159e
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: d010fd90d96409b262be59f1db4fac9e4cec835c
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37051554"
---
# <a name="copy-data-to-and-from-azure-sql-database-using-azure-data-factory"></a>使用 Azure Data Factory 從 Azure SQL Database 來回複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-azure-sql-connector.md)
> * [第 2 版 (目前的版本)](../connector-azure-sql-database.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您是使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 Azure SQL Database 連接器](../connector-azure-sql-database.md)。

本文說明如何使用 Azure Data Factory 中的「複製活動」，將資料移進/移出 Azure SQL Database。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。  

## <a name="supported-scenarios"></a>支援的案例
您可以**從 Azure SQL Database**將資料複製到下列資料存放區：

[!INCLUDE [data-factory-supported-sinks](../../../includes/data-factory-supported-sinks.md)]

您可以從下列資料存放區將資料複製**到 Azure SQL Database**：

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

## <a name="supported-authentication-type"></a>支援的驗證類型
Azure SQL Database 連接器支援基本驗證。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以將資料移進/移出 Azure SQL Database。

若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線： 

1. 建立 **Data Factory**。 資料處理站可包含一或多個管線。 
2. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。 例如，如果您將資料從 Azure blob 儲存體複製到 Azure SQL 資料庫，您會建立兩個連結服務，將 Azure 儲存體帳戶和 Azure SQL 資料庫連結至資料處理站。 有關 Azure SQL Database 專屬的連結服務屬性，請參閱[連結服務屬性](#linked-service-properties)一節。 
3. 建立**資料集**，代表複製作業的輸入和輸出資料。 在上一個步驟所述的範例中，您會建立資料集來指定 blob 容器和包含輸入資料的資料夾。 您還會建立另一個資料集來指定 Azure SQL Database 中的 SQL 資料表，以保存從 Blob 儲存體複製的資料。 有關 Azure Data Lake Store 專屬的資料集屬性，請參閱[資料集屬性](#dataset-properties)一節。
4. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 在稍早所述的範例中，您使用 BlobSource 作為來源，以及使用 SqlSink 作為複製活動的接收器。 同樣地，如果您是從 Azure SQL Database 複製到 Azure Blob 儲存體，則需要在複製活動中使用 SqlSource 和 BlobSink。 有關 Azure SQL Database 專屬的複製活動屬性，請參閱[複製活動屬性](#copy-activity-properties)一節。 如需有關如何使用資料存放區作為來源或接收器的詳細資訊，請在上一節按一下適用於您的資料存放區的連結。

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來將資料複製到 Azure SQL Database (或從 Azure SQL Database 複製資料) 之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例](#json-examples-for-copying-data-to-and-from-sql-database)一節。 

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 Azure SQL Database 特定的 Data Factory 實體： 

## <a name="linked-service-properties"></a>連結服務屬性
Azure SQL 已連結服務可將 Azure SQL Database 連結到您的 Data Factory。 下表提供 Azure SQL 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |type 屬性必須設為： **AzureSqlDatabase** |yes |
| connectionString |針對 connectionString 屬性指定連接到 Azure SQL Database 執行個體所需的資訊。 僅支援基本驗證。 |yes |

> [!IMPORTANT]
> 設定 [Azure SQL Database 防火牆](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure)和資料庫伺服器，以[允許 Azure 服務存取伺服器](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure)。 此外，如果您要從 Azure 外部 (包括從具有 Fata Factory 閘道器的內部部署資料來源) 將資料複製到 Azure SQL Database，請為傳送資料到 Azure SQL Database 的機器設定適當的 IP 位址範圍。

## <a name="dataset-properties"></a>資料集屬性
若要指定資料集以代表 Azure SQL Database 中的輸入或輸出資料，您需將資料集的類型屬性設定成：**AzureSqlTable**。 請將資料集的 **linkedServiceName** 屬性設定成 Azure SQL 已連結服務的名稱。  

如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 typeProperties 區段都不同，可提供資料存放區中資料的位置相關資訊。 **AzureSqlTable** 類型資料集的 **typeProperties** 區段具有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Azure SQL Database 執行個體中連結服務所參考的資料表或檢視的名稱。 |yes |

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

> [!NOTE]
> 複製活動只會採用一個輸入，而且只產生一個輸出。

而活動的 **typeProperties** 區段中，可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

如果您要將資料從 Azure SQL Database 移出，請將複製活動中的來源類型設定為 **SqlSource**。 同樣的，如果您要將資料移進 Azure SQL Database，請將複製活動中的接收器類型設定為 **SqlSink**。 本節提供 SqlSource 和 SqlSink 支援的屬性清單。

### <a name="sqlsource"></a>SqlSource
在複製活動中，如果來源的類型為 **SqlSource**，則 **typeProperties** 區段有下列可用屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| SqlReaderQuery |使用自訂查詢來讀取資料。 |SQL 查詢字串。 範例： `select * from MyTable`. |否 |
| sqlReaderStoredProcedureName |從來源資料表讀取資料的預存程序名稱。 |預存程序的名稱。 最後一個 SQL 陳述式必須是預存程序中的 SELECT 陳述式。 |否 |
| storedProcedureParameters |預存程序的參數。 |名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |

如果已為 SqlSource 指定 **sqlReaderQuery** ，複製活動會針對 Azure SQL Database 來源執行這項查詢以取得資料。 或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。

如果您未指定 sqlReaderQuery 或 sqlReaderStoredProcedureName，就會使用資料集 JSON 的結構區段中定義的資料行來建立一個查詢 (`select column1, column2 from mytable`)，以對 Azure SQL Database 執行。 如果資料集定義沒有結構，則會從資料表中選取所有資料行。

> [!NOTE]
> 當您使用 **sqlReaderStoredProcedureName** 時，仍必須為資料集 JSON 中的 **tableName** 屬性指定值。 雖然目前尚未針對此資料表來進行驗證。
>
>

### <a name="sqlsource-example"></a>SqlSource 範例

```JSON
"source": {
    "type": "SqlSource",
    "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
    "storedProcedureParameters": {
        "stringData": { "value": "str3" },
        "identifier": { "value": "$$Text.Format('{0:yyyy}', SliceStart)", "type": "Int"}
    }
}
```

**預存程序定義：**

```SQL
CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
(
    @stringData varchar(20),
    @identifier int
)
AS
SET NOCOUNT ON;
BEGIN
     select *
     from dbo.UnitTestSrcTable
     where dbo.UnitTestSrcTable.stringData != stringData
    and dbo.UnitTestSrcTable.identifier != identifier
END
GO
```

### <a name="sqlsink"></a>管線
**SqlSink** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| writeBatchTimeout |在逾時前等待批次插入作業完成的時間。 |時間範圍<br/><br/> 範例：“00:30:00” (30 分鐘)。 |否 |
| writeBatchSize |當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中 |整數 (資料列數目) |否 (預設值：10000) |
| sqlWriterCleanupScript |指定要讓「複製活動」執行的查詢，以便清除特定分割的資料。 如需詳細資訊，請參閱[可重複複製](#repeatable-copy)。 |查詢陳述式。 |否 |
| sliceIdentifierColumnName |指定要讓「複製活動」以自動產生的分割識別碼填入的資料行名稱，這可在重新執行時用來清除特定分割的資料。 如需詳細資訊，請參閱[可重複複製](#repeatable-copy)。 |資料類型為 binary(32) 之資料行的資料行名稱。 |否 |
| sqlWriterStoredProcedureName |預存程序的名稱，此預存程序定義如何將來源資料套用至目標資料表，例如使用您自己的商務邏輯來執行更新插入或轉換。 <br/><br/>請注意，將會**依批次叫用**此預存程序。 如果您想要進行只執行一次且與來源資料無關的作業 (例如刪除/截斷)，請使用 `sqlWriterCleanupScript` 屬性。 |預存程序的名稱。 |否 |
| storedProcedureParameters |預存程序的參數。 |名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |
| sqlWriterTableType |指定要在預存程序中使用的資料表類型名稱。 複製活動可讓正在移動的資料可用於此資料表類型的暫存資料表。 然後，預存程序程式碼可以合併正在複製的資料與現有的資料。 |資料表類型名稱。 |否 |

#### <a name="sqlsink-example"></a>SqlSink 範例

```JSON
"sink": {
    "type": "SqlSink",
    "writeBatchSize": 1000000,
    "writeBatchTimeout": "00:05:00",
    "sqlWriterStoredProcedureName": "CopyTestStoredProcedureWithParameters",
    "sqlWriterTableType": "CopyTestTableType",
    "storedProcedureParameters": {
        "identifier": { "value": "1", "type": "Int" },
        "stringData": { "value": "str1" },
        "decimalData": { "value": "1", "type": "Decimal" }
    }
}
```

## <a name="json-examples-for-copying-data-to-and-from-sql-database"></a>往返 SQL Database 複製資料的 JSON 範例
以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。 它們會示範如何將資料複製到 Azure SQL Database 和 Azure Blob 儲存體，以及複製其中的資料。 不過，您可以在 Azure Data Factory 中使用複製活動，從任何來源 **直接** 將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。

### <a name="example-copy-data-from-azure-sql-database-to-azure-blob"></a>範例：將資料從 Azure SQL Database 複製到 Azure Blob
相同內容會定義下列 Data Factory 實體：

1. [AzureSqlDatabase](#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [AzureSqlTable](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [Azure Blob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [SqlSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之「複製活動」的[管線](data-factory-create-pipelines.md)。

此範例會每小時將時間序列資料 (每小時、每日等等) 從 Azure SQL Database 中的資料表複製到 Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。  

**Azure SQL Database 已連結的服務：**

```JSON
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
如需這個連結的服務所支援屬性的清單，請參閱 [Azure SQL 連結的服務](#linked-service) 一文。

**Azure Blob 儲存體連結服務：**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
如需這個連結的服務所支援屬性的清單，請參閱 [Azure Blob](data-factory-azure-blob-connector.md#azure-storage-linked-service) 一文。


**Azure SQL 輸入資料集：**

此範例假設您已在 SQL Azure 中建立資料表 "MyTable"，其中包含時間序列資料的資料行 (名稱為 "timestampcolumn")。

設定 “external”: ”true” 會通知 Azure Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

```JSON
{
  "name": "AzureSqlInput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

如需這個資料集類型所支援屬性的清單，請參閱 [Azure SQL 資料集型別屬性](#dataset) 小節。  

**Azure Blob 輸出資料集：**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
如需這個資料集類型所支援屬性的清單，請參閱 [Azure Blob 資料集型別屬性](data-factory-azure-blob-connector.md#dataset-properties) 小節。  

**具有 SQL 來源和 Blob 接收器的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **SqlSource**，而 **sink** 類型設為 **BlobSink**。 針對 **SqlReaderQuery** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

```JSON
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline for copy activity",
    "activities":[  
      {
        "name": "AzureSQLtoBlob",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureSQLInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "SqlSource",
            "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
     ]
   }
}
```
在此範例中，已為 SqlSource 指定 **sqlReaderQuery** 。 複製活動會針對 Azure SQL Database 來源執行這項查詢以取得資料。 或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。

如果您未指定 sqlReaderQuery 或 sqlReaderStoredProcedureName，就會使用資料集 JSON 的結構區段中定義的資料行來建立一個查詢，以對 Azure SQL Database 執行。 例如： `select column1, column2 from mytable`。 如果資料集定義沒有結構，則會從資料表中選取所有資料行。

如需 SqlSource 和 BlobSink 所支援屬性的清單，請參閱 [SQL 來源](#sqlsource)小節和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties)。

### <a name="example-copy-data-from-azure-blob-to-azure-sql-database"></a>範例：將資料從 Azure Blob 複製到 Azure SQL Database
此範例會定義下列 Data Factory 實體：  

1. [AzureSqlDatabase](#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureSqlTable](#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) 和 [SqlSink](#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將時間序列資料 (每小時、每日等等) 從 Azure Blob 複製到 Azure SQL Database 中的資料表。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure SQL 連結服務：**

```JSON
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
如需這個連結的服務所支援屬性的清單，請參閱 [Azure SQL 連結的服務](#linked-service) 一文。

**Azure Blob 儲存體連結服務：**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
如需這個連結的服務所支援屬性的清單，請參閱 [Azure Blob](data-factory-azure-blob-connector.md#azure-storage-linked-service) 一文。


**Azure Blob 輸入資料集：**

每小時從新的 Blob 挑選資料 (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑和檔案名稱。 資料夾路徑會使用開始時間的年、月及日部分，而檔案名稱則使用開始時間的小時部分。 “external”: “true” 設定會通知 Data Factory 服務：這是 Data Factory 外部的資料表而且不是由 Data Factory 中的活動所產生。

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```
如需這個資料集類型所支援屬性的清單，請參閱 [Azure Blob 資料集型別屬性](data-factory-azure-blob-connector.md#dataset-properties) 小節。

**Azure SQL Database 輸出資料集：**

此範例會將資料複製到 Azure SQL 中名為 "MyTable" 的資料表。 請在Azure SQL 中建立此資料表，其資料行的數目如您預期 Blob CSV 檔案要包含的數目。 此資料表會每小時加入新的資料列。

```JSON
{
  "name": "AzureSqlOutput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
如需這個資料集類型所支援屬性的清單，請參閱 [Azure SQL 資料集型別屬性](#dataset) 小節。

**具有 Blob 來源和 SQL 接收器的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **BlobSource**，而 **sink** 類型設為 **SqlSink**。

```JSON
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[  
      {
        "name": "AzureBlobtoSQL",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource",
            "blobColumnSeparators": ","
          },
          "sink": {
            "type": "SqlSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
      ]
   }
}
```
如需 SqlSink 和 BlobSource 所支援屬性的清單，請參閱 [SQL 接收器](#sqlsink)小節和 [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties)。

## <a name="identity-columns-in-the-target-database"></a>目標資料庫中的身分識別資料行
本節提供的範例將示範，如何把資料從沒有身分識別資料行的來源資料表，複製到具有身分識別資料行的目的地資料表。

**來源資料表：**

```SQL
create table dbo.SourceTbl
(
       name varchar(100),
       age int
)
```
**目的地資料表：**

```SQL
create table dbo.TargetTbl
(
       identifier int identity(1,1),
       name varchar(100),
       age int
)
```
請注意，目標資料表具有身分識別資料行。

**來源資料集 JSON 定義**

```JSON
{
    "name": "SampleSource",
    "properties": {
        "type": " SqlServerTable",
        "linkedServiceName": "TestIdentitySQL",
        "typeProperties": {
            "tableName": "SourceTbl"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```
**目的地資料集 JSON 定義**

```JSON
{
    "name": "SampleTarget",
    "properties": {
        "structure": [
            { "name": "name" },
            { "name": "age" }
        ],
        "type": "AzureSqlTable",
        "linkedServiceName": "TestIdentitySQLSource",
        "typeProperties": {
            "tableName": "TargetTbl"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": false,
        "policy": {}
    }    
}
```

請注意，您的來源資料表與目標資料表的結構描述不同 (目標資料表有一個具有身分識別的額外資料行)。 在此案例中，您必須在目標資料集定義中指定 **structure** 屬性，這不包含身分識別資料行。

## <a name="invoke-stored-procedure-from-sql-sink"></a>從 SQL 接收器叫用預存程序
如需在管線的複製活動中從 SQL 接收器叫用預存程序的範例，請參閱[在複製活動中叫用 SQL 接收器的預存程序](data-factory-invoke-stored-procedure-from-copy-activity.md)一文。 

## <a name="type-mapping-for-azure-sql-database"></a>Azure SQL Database 的類型對應
如同 [資料移動活動](data-factory-data-movement-activities.md) 一文所述，複製活動會使用下列 2 個步驟的方法，執行自動類型轉換，將來源類型轉換成接收類型：

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

將資料移進和移出 Azure SQL Database 時，會使用下列從 SQL 類型到 .NET 類型的對應，以及反向的對應。 此對應與 ADO.NET 的 SQL Server 資料類型對應相同。

| SQL Server Database Engine 類型 | .NET Framework 類型 |
| --- | --- |
| bigint |Int64 |
| binary |Byte[] |
| bit |BOOLEAN |
| char |String、Char[] |
| 日期 |Datetime |
| DateTime |Datetime |
| datetime2 |Datetime |
| Datetimeoffset |DateTimeOffset |
| 十進位 |十進位 |
| FILESTREAM 屬性 (varbinary(max)) |Byte[] |
| Float |兩倍 |
| 映像 |Byte[] |
| int |Int32 |
| money |十進位 |
| nchar |String、Char[] |
| ntext |String、Char[] |
| numeric |十進位 |
| nvarchar |String、Char[] |
| real |單一 |
| rowversion |Byte[] |
| smalldatetime |Datetime |
| smallint |Int16 |
| smallmoney |十進位 |
| sql_variant |物件 * |
| text |String、Char[] |
| 分析 |時間範圍 |
| timestamp |Byte[] |
| tinyint |Byte |
| uniqueidentifier |Guid |
| varbinary |Byte[] |
| varchar |String、Char[] |
| xml |xml |

## <a name="map-source-to-sink-columns"></a>將來源對應到接收資料行
若要了解如何將來源資料集內的資料行對應至接收資料集內的資料行，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="repeatable-copy"></a>可重複複製
將資料複製到 SQL Server 資料庫時，複製活動預設會將資料附加至接收資料表。 若要改為執行 UPSERT，請參閱[對 SqlSink 進行可重複的寫入](data-factory-repeatable-copy.md#repeatable-write-to-sqlsink)一文。 

從關聯式資料存放區複製資料時，請將可重複性謹記在心，以避免產生非預期的結果。 在 Azure Data Factory 中，您可以手動重新執行配量。 您也可以為資料集設定重試原則，使得在發生失敗時，重新執行配量。 以上述任一方式重新執行配量時，您必須確保不論將配量執行多少次，都會讀取相同的資料。 請參閱[從關聯式來源進行可重複的讀取](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。
