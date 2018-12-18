---
title: 從 SQL Server 來回移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory，從內部部署或 Azure VM 中的 SQL Server 資料庫來回移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 864ece28-93b5-4309-9873-b095bbe6fedd
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: c4038ea5a450f32a46f24a306d1ee30bd61308a5
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37054580"
---
# <a name="move-data-to-and-from-sql-server-on-premises-or-on-iaas-azure-vm-using-azure-data-factory"></a>使用 Azure Data Factory 從 SQL Server 內部部署或 IaaS (Azure VM) 上來回移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-sqlserver-connector.md)
> * [第 2 版 (目前的版本)](../connector-sql-server.md)

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 SQL Server 連接器](../connector-sql-server.md)。

本文說明如何使用 Azure Data Factory 中的「複製活動」，將資料移進/移出內部部署 SQL Server 資料庫。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。 

## <a name="supported-scenarios"></a>支援的案例
您可以**從 SQL Server 資料庫**將資料複製到下列資料存放區：

[!INCLUDE [data-factory-supported-sink](../../../includes/data-factory-supported-sinks.md)]

您可以從下列資料存放區將資料複製**到 SQL Server 資料庫**：

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

## <a name="supported-sql-server-versions"></a>支援的 SQL Server 版本
這個 SQL Server 連接器支援使用 SQL 驗證和 Windows 驗證，在裝載於內部部署或在 Azure IaaS 中的下列版本個體間往返複製資料︰SQL Server 2016、SQL Server 2014、SQL Server 2012、SQL Server 2008 R2、SQL Server 2008、SQL Server 2005

## <a name="enabling-connectivity"></a>啟用連線
與裝載於內部部署或 Azure IaaS (基礎結構即為服務) VM 中的 SQL Server 連線所需的概念和步驟都相同。 在這兩種情況下，您都需要使用「資料管理閘道」來進行連線。

請參閱 [在內部部署位置與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文來了解資料管理閘道和設定閘道的逐步指示。 設定閘道器執行個體是與 SQL Server 連線的必要條件。

雖然您可以將閘道安裝在與 SQL Server 相同的內部部署機器或雲端 VM 執行個體上來獲得較佳的效能，但仍建議您將它們安裝在個別的機器上。 將閘道與 SQL Server 置於個別的機器上可降低發生資源競爭的情況。

## <a name="getting-started"></a>開始使用
您可以建立內含複製活動的管線，使用不同的工具/API 將資料移進/移出內部部署 SQL Server 資料庫。

若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線： 

1. 建立 **Data Factory**。 資料處理站可包含一或多個管線。 
2. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。 例如，如果您從 SQL Server 資料庫將資料複製到 Azure Blob 儲存體，您會建立兩個連結服務，將 SQL Server 資料庫和 Azure 儲存體帳戶連結至資料處理站。 有關 SQL Server 資料庫專屬的連結服務屬性，請參閱[連結服務屬性](#linked-service-properties)一節。 
3. 建立**資料集**，代表複製作業的輸入和輸出資料。 在上一個步驟所述的範例中，您會建立資料集來指定 SQL Server 資料庫中包含輸入資料的 SQL 資料表。 您還會建立另一個資料集來指定 blob 容器和資料夾，以保存從 SQL Server 資料庫複製的資料。 有關 SQL Server 資料庫專屬的資料集屬性，請參閱[資料集屬性](#dataset-properties)一節。
4. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 在稍早所述的範例中，您使用 SqlSource 作為來源，以及使用 BlobSink 作為複製活動的接收器。 同樣地，如果您是從 Azure Blob 儲存體複製到 SQL Server 資料庫，則需要在複製活動中使用 BlobSource 和 SqlSink。 有關 SQL Server 資料庫專屬的複製活動屬性，請參閱[複製活動屬性](#copy-activity-properties)一節。 如需有關如何使用資料存放區作為來源或接收器的詳細資訊，按一下上一節中資料存放區的連結。 

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來將資料複製到內部部署 SQL Server 資料庫 (或從內部部署 SQL Server 資料庫複製資料) 之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例](#json-examples-for-copying-data-from-and-to-sql-server)一節。 

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 SQL Server 特有的 Data Factory 實體： 

## <a name="linked-service-properties"></a>連結服務屬性
您可以建立 **OnPremisesSqlServer** 類型的連結服務，以將內部部署 SQL Server 資料庫連結至資料處理站。 下表提供內部部署 SQL Server 連結服務專屬 JSON 元素的描述。

下表提供 SQL Server 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性應設為： **OnPremisesSqlServer**。 |yes |
| connectionString |指定使用 SQL 驗證或 Windows 驗證連接至內部部署 SQL Server 資料庫所需的 connectionString 資訊。 |yes |
| gatewayName |Data Factory 服務應該用來連接到內部部署 SQL Server 資料庫的閘道器名稱。 |yes |
| username |如果您使用「Windows 驗證」，請指定使用者名稱。 範例︰**domainname\\username**。 |否 |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 |否 |

您可以使用 **New-AzureRmDataFactoryEncryptValue** Cmdlet 加密認證，並在連接字串中使用這些認證，如下列範例所示 (**EncryptedCredential** 屬性)：  

```JSON
"connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",
```

### <a name="samples"></a>範例
**使用 SQL 驗證的 JSON**

```json
{
    "name": "MyOnPremisesSQLDB",
    "properties":
    {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=False;User ID=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```
**使用 Windows 驗證的 JSON**

資料管理閘道會模擬指定的使用者帳戶，以連線到內部部署 SQL Server 資料庫。 

```json
{
     "Name": " MyOnPremisesSQLDB",
     "Properties":
     {
         "type": "OnPremisesSqlServer",
         "typeProperties": {
             "ConnectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=True;",
             "username": "<domain\\username>",
             "password": "<password>",
             "gatewayName": "<gateway name>"
        }
     }
}
```

## <a name="dataset-properties"></a>資料集屬性
範例中使用使用 **SqlServerTable** 類型的資料集來表示 SQL Server 資料庫中的資料表。  

如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 所有資料集類型 (SQL Server、Azure Blob、Azure 資料表等) 的資料集 JSON 區段 (例如 structure、availability 及 policy) 都相似。

每個資料集類型的 typeProperties 區段都不同，可提供資料存放區中資料的位置相關資訊。 **SqlServerTable** 類型資料集的 **typeProperties** 區段有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |SQL Server Database 執行個體中連結服務所參照的資料表或檢視名稱。 |yes |

## <a name="copy-activity-properties"></a>複製活動屬性
如果您要將資料從 SQL Server 資料庫移出，請將複製活動中的來源類型設定為 **SqlSource**。 同樣的，如果您要將資料移進 SQL Server 資料庫，請將複製活動中的接收器類型設定為 **SqlSink**。 本節提供 SqlSource 和 SqlSink 支援的屬性清單。

如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

> [!NOTE]
> 複製活動只會採用一個輸入，而且只產生一個輸出。

而活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

### <a name="sqlsource"></a>SqlSource
當複製活動中的來源類型為 **SqlSource** 時，**typeProperties** 區段會有下列可用屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| SqlReaderQuery |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如：select * from MyTable。 可以參考輸入資料集所參考資料庫中的多個資料表。 如果未指定，執行的 SQL 陳述式：select from MyTable。 |否 |
| sqlReaderStoredProcedureName |從來源資料表讀取資料的預存程序名稱。 |預存程序的名稱。 最後一個 SQL 陳述式必須是預存程序中的 SELECT 陳述式。 |否 |
| storedProcedureParameters |預存程序的參數。 |名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |

如果已為 SqlSource 指定 **sqlReaderQuery** ，複製活動會針對 SQL Server 資料庫來源執行這項查詢以取得資料。

或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。

如果您未指定 sqlReaderQuery 或 sqlReaderStoredProcedureName，系統就會使用 structure 區段中定義的資料行來建立一個要對「SQL Server 資料庫」執行的 select 查詢。 如果資料集定義沒有結構，則會從資料表中選取所有資料行。

> [!NOTE]
> 當您使用 **sqlReaderStoredProcedureName** 時，仍必須為資料集 JSON 中的 **tableName** 屬性指定值。 雖然目前尚未針對此資料表來進行驗證。

### <a name="sqlsink"></a>管線
**SqlSink** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| writeBatchTimeout |在逾時前等待批次插入作業完成的時間。 |時間範圍<br/><br/> 範例：“00:30:00” (30 分鐘)。 |否 |
| writeBatchSize |當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中 |整數 (資料列數目) |否 (預設值：10000) |
| sqlWriterCleanupScript |指定要讓「複製活動」執行的查詢，以便清除特定分割的資料。 如需詳細資訊，請參閱[可重複複製](#repeatable-copy)一節。 |查詢陳述式。 |否 |
| sliceIdentifierColumnName |指定要讓「複製活動」以自動產生的分割識別碼填入的資料行名稱，這可在重新執行時用來清除特定分割的資料。 如需詳細資訊，請參閱[可重複複製](#repeatable-copy)一節。 |資料類型為 binary(32) 之資料行的資料行名稱。 |否 |
| sqlWriterStoredProcedureName |預存程序的名稱，此預存程序定義如何將來源資料套用至目標資料表，例如使用您自己的商務邏輯來執行更新插入或轉換。 <br/><br/>請注意，將會**依批次叫用**此預存程序。 如果您想要進行只執行一次且與來源資料無關的作業 (例如刪除/截斷)，請使用 `sqlWriterCleanupScript` 屬性。 |預存程序的名稱。 |否 |
| storedProcedureParameters |預存程序的參數。 |名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |
| sqlWriterTableType |指定要在預存程序中使用的資料表類型名稱。 複製活動可讓正在移動的資料可用於此資料表類型的暫存資料表。 然後，預存程序程式碼可以合併正在複製的資料與現有的資料。 |資料表類型名稱。 |否 |


## <a name="json-examples-for-copying-data-from-and-to-sql-server"></a>往返 SQL Server 複製資料的 JSON 範例
以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。 下列範例說明如何將資料複製到 SQL Server 和「Azure Blob 儲存體」，以及從這兩處複製資料。 不過，您可以在 Azure Data Factory 中使用複製活動，從任何來源 **直接** 將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。     

## <a name="example-copy-data-from-sql-server-to-azure-blob"></a>範例：將資料從 SQL Server 複製到 Azure Blob
下列範例顯示︰

1. [OnPremisesSqlServer](#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [SqlServerTable](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [SqlSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將時間序列資料從 SQL Server 資料表複製到 Azure Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

第一步是設定資料管理閘道。 如需相關指示，請參閱 [在內部部署位置和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 。

**SQL Server 連結服務**
```json
{
  "Name": "SqlServerLinkedService",
  "properties": {
    "type": "OnPremisesSqlServer",
    "typeProperties": {
      "connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;",
      "gatewayName": "<gatewayname>"
    }
  }
}
```
**Azure Blob 儲存體連結服務**

```json
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
**SQL Server 輸入資料集**

此範例假設您已在 SQL Server 中建立資料表 "MyTable"，其中包含時間序列資料的資料行 (名稱為 "timestampcolumn")。 您可以使用單一資料集來查詢相同資料庫內的多個資料表，但針對資料集的 tableName typeProperty 必須使用單一資料表。

設定 “external”: ”true” 可讓 Data Factory 服務知道資料集是在 Data Factory 外部，而不是由 Data Factory 中的活動所產生。

```json
{
  "name": "SqlServerInput",
  "properties": {
    "type": "SqlServerTable",
    "linkedServiceName": "SqlServerLinkedService",
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
**Azure Blob 輸出資料集**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```json
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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
**具有複製活動的管線**

此管線包含「複製活動」，該活動已設定為使用這些輸入和輸出資料集，並且排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **SqlSource**，而 **sink** 類型設為 **BlobSink**。 針對 **SqlReaderQuery** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2016-06-01T18:00:00",
    "end":"2016-06-01T19:00:00",
    "description":"pipeline for copy activity",
    "activities":[  
      {
        "name": "SqlServertoBlob",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": " SqlServerInput"
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
在此範例中，已為 SqlSource 指定 **sqlReaderQuery** 。 複製活動會針對 SQL Server 資料庫來源執行這項查詢以取得資料。 或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。 sqlReaderQuery 可以參考輸入資料集所參考之資料庫內的多個資料表。 這不限於只有設定為資料集之 tableName typeProperty 的資料表。

如果您未指定 sqlReaderQuery 或 sqlReaderStoredProcedureName，系統就會使用 structure 區段中定義的資料行來建立一個要對「SQL Server 資料庫」執行的 select 查詢。 如果資料集定義沒有結構，則會從資料表中選取所有資料行。

如需 SqlSource 和 BlobSink 所支援屬性的清單，請參閱 [SQL 來源](#sqlsource)小節和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties)。

## <a name="example-copy-data-from-azure-blob-to-sql-server"></a>範例：將資料從 Azure Blob 複製到 SQL Server
下列範例顯示︰

1. [OnPremisesSqlServer](#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties) 類型的連結服務。
3. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [SqlServerTable](data-factory-sqlserver-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) 和 [SqlSink](#sql-server-copy-activity-type-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將時間序列資料從 Azure Blob 複製到 SQL Server 資料表。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**SQL Server 連結服務**

```json
{
  "Name": "SqlServerLinkedService",
  "properties": {
    "type": "OnPremisesSqlServer",
    "typeProperties": {
      "connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;",
      "gatewayName": "<gatewayname>"
    }
  }
}
```
**Azure Blob 儲存體連結服務**

```json
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
**Azure Blob 輸入資料集**

每小時從新的 Blob 挑選資料 (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑和檔案名稱。 資料夾路徑會使用開始時間的年、月及日部分，而檔案名稱則使用開始時間的小時部分。 “external”: “true” 設定可讓 Data Factory 服務知道資料集是在 Data Factory 外部，而不是由 Data Factory 中的活動所產生。

```json
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
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
**SQL Server 輸出資料集**

此範例會將資料複製到 SQL Server 中名為 "MyTable" 的資料表。 請在 SQL Server 中建立此資料表，其資料行的數目須與您預期 Blob CSV 檔案要包含的數目相同。 此資料表會每小時加入新的資料列。

```json
{
  "name": "SqlServerOutput",
  "properties": {
    "type": "SqlServerTable",
    "linkedServiceName": "SqlServerLinkedService",
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
**具有複製活動的管線**

此管線包含「複製活動」，該活動已設定為使用這些輸入和輸出資料集，並且排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **BlobSource**，而 **sink** 類型設為 **SqlSink**。

```json
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
            "name": " SqlServerOutput "
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

## <a name="troubleshooting-connection-issues"></a>疑難排解連線問題
1. 將 SQL Server 設定成接受遠端連線。 啟動 [SQL Server Management Studio]、用滑鼠右鍵按一下 [伺服器]，然後按一下 [屬性]。 選取清單中 [連接]，然後核取 [允許此伺服器的遠端連接]。

    ![啟用遠端連線](./media/data-factory-sqlserver-connector/AllowRemoteConnections.png)

    如需詳細步驟，請參閱 [設定 remote access 伺服器組態選項](https://msdn.microsoft.com/library/ms191464.aspx) 。
2. 啟動 [SQL Server 組態管理員] 。 展開您想要之執行個體的 [SQL Server 網路組態]，然後選取 [MSSQLSERVER 的通訊協定]。 您應該會在右窗格中看到通訊協定。 用滑鼠右鍵按一下 [TCP/IP]，然後按一下 [啟用] 來啟用 TCP/IP。

    ![啟用 TCP/IP](./media/data-factory-sqlserver-connector/EnableTCPProptocol.png)

    如需啟用 TCP/IP 通訊協定的詳細資料及替代方式，請參閱 [啟用或停用伺服器網路通訊協定](https://msdn.microsoft.com/library/ms191294.aspx) 。
3. 在相同的視窗中，按兩下 [TCP/IP] 來啟動 [TCP/IP 屬性] 視窗。
4. 切換到 [IP 位址] 索引標籤。向下捲動到 [IPAll] 區段。 記下 **TCP 通訊埠** (預設值是 **1433**)。
5. 在電腦上建立 **Windows 防火牆規則** ，來允許透過此連接埠的連入流量。  
6. **確認連線**：若要使用完整名稱來連線到 SQL Server，請使用來自不同機器的 SQL Server Management Studio。 例如："<machine><domain>.corp<company>.com,1433"。

   > [!IMPORTANT]

   > 如需詳細資訊，請參閱[利用資料管理閘道在內部部署來源和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)。
   >
   > 如需連接/閘道器相關問題的疑難排解秘訣，請參閱 [針對閘道問題進行疑難排解](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) 。
   >
   >


## <a name="identity-columns-in-the-target-database"></a>目標資料庫中的身分識別資料行
本節提供一個範例，此範例會將資料從沒有身分識別資料行的來源資料表，複製到具有身分識別資料行的目的地資料表。

**來源資料表：**

```sql
create table dbo.SourceTbl
(
       name varchar(100),
       age int
)
```
**目的地資料表：**

```sql
create table dbo.TargetTbl
(
       identifier int identity(1,1),
       name varchar(100),
       age int
)
```

請注意，目標資料表具有身分識別資料行。

**來源資料集 JSON 定義**

```json
{
    "name": "SampleSource",
    "properties": {
        "published": false,
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

```json
{
    "name": "SampleTarget",
    "properties": {
        "structure": [
            { "name": "name" },
            { "name": "age" }
        ],
        "published": false,
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

## <a name="type-mapping-for-sql-server"></a>SQL Server 的類型對應
如同 [資料移動活動](data-factory-data-movement-activities.md) 一文所述，「複製活動」會藉由含有下列 2 個步驟的方法，執行從來源類型轉換成接收類型的自動類型轉換：

1. 從原生來源類型轉換成 .NET 類型
2. 從 .NET 類型轉換成原生接收類型

從 SQL Server 來回移動資料時，會使用下列從 SQL 類型到 .NET 類型的對應，以及反向的對應。

此對應與 ADO.NET 的 SQL Server 資料類型對應相同。

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

## <a name="mapping-source-to-sink-columns"></a>將來源對應到接收資料行
若要將來源資料集中的資料行對應至接收資料集中的資料行，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="repeatable-copy"></a>可重複複製
將資料複製到 SQL Server 資料庫時，複製活動預設會將資料附加至接收資料表。 若要改為執行 UPSERT，請參閱[對 SqlSink 進行可重複的寫入](data-factory-repeatable-copy.md#repeatable-write-to-sqlsink)一文。 

從關聯式資料存放區複製資料時，請將可重複性謹記在心，以避免產生非預期的結果。 在 Azure Data Factory 中，您可以手動重新執行配量。 您也可以為資料集設定重試原則，使得在發生失敗時，重新執行配量。 以上述任一方式重新執行配量時，您必須確保不論將配量執行多少次，都會讀取相同的資料。 請參閱[從關聯式來源進行可重複的讀取](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。
