---
title: Azure Data Factory 中的資料集和已連結的服務 | Microsoft Docs
description: 了解 Data Factory 中的資料集和已連結的服務。 已連結的服務會將計算/資料存放區連結至資料處理站。 資料集代表輸入/輸出資料。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: shlo
ms.openlocfilehash: d5cf4005ad50c9c75f22b2fa2719925afbe69f26
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38581261"
---
# <a name="datasets-and-linked-services-in-azure-data-factory"></a>Azure Data Factory 中的資料集和已連結的服務 
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-create-datasets.md)
> * [目前的版本](concepts-datasets-linked-services.md)

本文說明什麼是資料集、如何以 JSON 格式定義它們，以及如何在 Azure Data Factory 管線中使用它們。 

如果您不熟悉 Data Factory，請參閱 [Azure Data Factory 簡介](introduction.md)來概略了解。 

## <a name="overview"></a>概觀
資料處理站可以有一或多個管線。 「管線」是一起執行某個工作的「活動」所組成的邏輯群組。 管線中的活動會定義要在資料上執行的動作。 例如，您可以使用複製活動將資料從內部部署 SQL Server 複製到 Azure Blob 儲存體。 接著，您可以使用在 Azure HDInsight 叢集上執行 Hive 指令碼的 Hive 活動，來處理來自 Blob 儲存體的資料以產生輸出資料。 最後，您可以使用第二個複製活動將輸出資料複製到「Azure SQL 資料倉儲」，以在該處建置商業智慧 (BI) 報表解決方案。 如需有關管線和活動的詳細資訊，請參閱 Azure Data Factory 中的[管線和活動](concepts-pipelines-activities.md)。

現在，「資料集」是一個具名的資料檢視，指向或參考您想要在「活動」中用來作為輸入或輸出的資料。 資料集可識別資料表、檔案、資料夾和文件等各種資料存放區中的資料。 例如，Azure Blob 資料集會指定活動應從中讀取資料之 Blob 儲存體中的 Blob 容器和資料夾。

在您建立資料集之前，您必須建立一個「已連結的服務」，以將資料存放區連結到資料處理站。 已連結的服務非常類似連接字串，可定義 Data Factory 連接到外部資源所需的連線資訊。 這麼說吧：資料集代表已連結之資料存放區內的資料結構，而已連結的服務則定義與資料來源的連線。 例如，「Azure 儲存體」已連結服務會將儲存體帳戶連結到 Data Factory。 Azure Blob 資料集代表該 Azure 儲存體帳戶內包含要處理之輸入 Blob 的 Blob 容器和資料夾。

以下是一個範例案例。 若要將資料從 Blob 儲存體複製到 SQL Database，您需建立兩個已連結的服務：「Azure 儲存體」和 Azure SQL Database。 接著，建立兩個資料集：Azure Blob 資料集 (此資料集參考「Azure 儲存體」已連結服務) 和「Azure SQL 資料表」資料集 (此資料集參考 Azure SQL Database 已連結服務)。 「Azure 儲存體」和 Azure SQL Database 已連結服務包含 Data Factory 在執行階段分別用來連接到「Azure 儲存體」和 Azure SQL Database 的連接字串。 Azure Blob 資料集會指定包含 Blob 儲存體中輸入 Blob 的 Blob 容器和 Blob 資料夾。 「Azure SQL 資料表」資料集會指定作為資料複製目的地的 SQL Database 中 SQL 資料表。

下圖顯示 Data Factory 中管線、活動、資料集及已連結服務之間的關聯性：

![管線、活動、資料集、已連結的服務之間的關聯性](media/concepts-datasets-linked-services/relationship-between-data-factory-entities.png)

## <a name="linked-service-json"></a>連結服務 JSON
Data Factory 中的連結服務會以 JSON 格式定義如下：

```json
{
    "name": "<Name of the linked service>",
    "properties": {
        "type": "<Type of the linked service>",
        "typeProperties": {
              "<data store or compute-specific type properties>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

下表描述上述 JSON 的屬性：

屬性 | 說明 | 必要 |
-------- | ----------- | -------- |
name | 連結服務的名稱。 請參閱 [Azure Data Factory - 命名規則](naming-rules.md)。 |  yes |
type | 連結服務的類型。 例如：AzureStorage (資料存放區) 或 AzureBatch (計算)。 請參閱 typeProperties 的描述。 | yes |
typeProperties | 每個資料存放區和計算的類型屬性都不同。 <br/><br/> 如需支援的資料存放區類型及其類型屬性，請參閱本文章的[資料集類型](#dataset-type)表格。 請瀏覽資料存放區連接器的文章，以了解資料存放區特有的類型屬性。 <br/><br/> 如需支援的計算類型與其類型屬性，請參閱[計算連結服務](compute-linked-services.md)。 | yes |
connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或自我裝載整合執行階段 (如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 | 否

## <a name="linked-service-example"></a>已連結的服務範例
以下連結服務是 Azure 儲存體連結服務。 請注意，類型已設為 AzureStorage。 Azure 儲存體連結服務的類型屬性包含連接字串。 Data Factory 服務會在執行階段使用連接字串來連線至資料存放區。 

```json
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-json"></a>資料集 JSON
Data Factory 中的資料集會以 JSON 格式定義如下：

```json
{
    "name": "<name of dataset>",
    "properties": {
        "type": "<type of dataset: AzureBlob, AzureSql etc...>",
        "linkedServiceName": {
                "referenceName": "<name of linked service>",
                 "type": "LinkedServiceReference",
        },
        "structure": [
            {
                "name": "<Name of the column>",
                "type": "<Name of the type>"
            }
        ],
        "typeProperties": {
            "<type specific property>": "<value>",
            "<type specific property 2>": "<value 2>",
        }
    }
}

```
下表描述上述 JSON 的屬性：

屬性 | 說明 | 必要 |
-------- | ----------- | -------- |
name | 資料集的名稱。 請參閱 [Azure Data Factory - 命名規則](naming-rules.md)。 |  yes |
type | 資料集的類型。 指定 Data Factory 支援的其中一種類型 (例如︰AzureBlob、AzureSqlTable)。 <br/><br/>如需詳細資料，請參閱[資料集類型](#dataset-type)。 | yes |
structure | 資料集的結構描述。 如需詳細資料，請參閱[資料集結構](#dataset-structure)。 | 否 |
typeProperties | 每個類型 (例如：Azure Blob、Azure SQL 資料表) 的類型屬性都不同。 如需有關支援的類型及其屬性的詳細資料，請參閱[資料集類型](#dataset-type)。 | yes |

## <a name="dataset-example"></a>資料集範例
在下列範例中，資料集代表 SQL Database 中名為 MyTable 的資料表。

```json
{
    "name": "DatasetSample",
    "properties": {
        "type": "AzureSqlTable",
        "linkedServiceName": {
                "referenceName": "MyAzureSqlLinkedService",
                 "type": "LinkedServiceReference",
        },
        "typeProperties":
        {
            "tableName": "MyTable"
        },
    }
}

```
請注意下列幾點：

- 類型設為 AzureSqlTable。
- tableName 類型屬性 (針對 AzureSqlTable 類型) 設定為 MyTable。
- linkedServiceName 係指 AzureSqlDatabase 類型的已連結服務，在接下來的 JSON 程式碼片段中將會定義。

## <a name="dataset-type"></a>資料集類型
有許多不同類型的資料集，視您使用的資料存放區而定。 如需 Data Factory 所支援的資料存放區清單，請參閱下表。 按一下某個資料存放區，即可了解如何為該資料存放區建立已連結的服務和資料集。

[!INCLUDE [data-factory-v2-supported-data-stores](../../includes/data-factory-v2-supported-data-stores.md)]

在上一節的範例中，資料集的類型是設定為 **AzureSqlTable**。 同樣地，針對 Azure Blob 資料集，資料集的類型是設定為 **AzureBlob**，如以下 JSON 所示：

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": {
                "referenceName": "MyAzureStorageLinkedService",
                 "type": "LinkedServiceReference",
        }, 
 
        "typeProperties": {
            "fileName": "input.log",
            "folderPath": "adfgetstarted/inputdata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        }
    }
}
```
## <a name="dataset-structure"></a>資料集結構
**structure** 區段是選擇性區段。 它可透過包含資料行之名稱和資料類型的集合，定義資料集的結構描述。 您可以使用 structure 區段來提供類型資訊，此資訊會用來轉換類型並將資料行從來源對應到目的地。

structure 中的每個資料行都包含下列屬性︰

屬性 | 說明 | 必要
-------- | ----------- | --------
name | 資料行的名稱。 | yes
type | 資料行的資料類型。 Data Factory 支援將下列過渡期資料類型當作允許的值：**Int16、Int32、Int64、Single、Double、Decimal、Byte[]、Boolean、String、Guid、Datetime、Datetimeoffset 及 Timespan** | 否
culture | 當類型為 .NET 類型 (`Datetime` 或 `Datetimeoffset`) 時，所要使用的 .NET 型文化特性。 預設值為 `en-us`。 | 否
format | 當類型為 .NET 類型 (`Datetime` 或 `Datetimeoffset`) 時，所要使用的格式字串。 有關如何格式化日期時間的資訊，請參閱[自訂日期和時間格式字串](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 | 否

### <a name="example"></a>範例
在下列範例中，假設來源 Blob 資料是 CSV 格式，而且包含三個資料行：userid、name 和 lastlogindate。 這些是含有自訂日期時間格式 (使用法文縮寫星期幾名稱) 的 Int64、String 和 Datetime 類型。

請依下列方式，定義 Blob 資料集結構及資料行的類型定義：

```json
"structure":
[
    { "name": "userid", "type": "Int64"},
    { "name": "name", "type": "String"},
    { "name": "lastlogindate", "type": "Datetime", "culture": "fr-fr", "format": "ddd-MM-YYYY"}
]
```

### <a name="guidance"></a>指引

下列指引可協助您了解何時要包括 structure 資訊，以及在 **structure** 區段中要包含哪些資訊。 從[結構描述和類型對應](copy-activity-schema-and-type-mapping.md)深入了解資料處理站如何將來源資料對應至接收，以及何時該指定結構資訊。

- **針對強式結構描述的資料來源**，請只有在您想要將來源資料行對應到接收資料行且其名稱不相同時，才指定 structure 區段。 這類結構化資料來源會將資料結構描述和類型資訊與資料本身儲存在一起。 結構化資料來源的範例包括 SQL Server、Oracle 及 Azure SQL Database。<br/><br/>由於結構化資料來源已經有可用的類型資訊，因此當您包含 structure 區段時，便不應包含類型資訊。
- **針對無/弱式結構描述的資料來源，例如 Blob 儲存體中的文字檔案**，請在資料集是複製活動的輸入，並且來源資料集的資料類型應該轉換成接收器的原生類型時，包含 structure。 並且也在您想要將來源資料行與接收資料行對應時，包含 structure。

## <a name="create-datasets"></a>建立資料集
您可以使用下列其中一個工具或 SDK 來建立資料集：[.NET API](quickstart-create-data-factory-dot-net.md)、[PowerShell](quickstart-create-data-factory-powershell.md)、[REST API](quickstart-create-data-factory-rest-api.md)、Azure Resource Manager 範本及 Azure 入口網站

## <a name="current-version-vs-version-1-datasets"></a>目前版本與版本 1 資料集

以下是 Data Factory 與 Data Factory 版本 1 資料集之間的一些差異： 

- 目前版本中不支援外部屬性。 已由[觸發程序](concepts-pipeline-execution-triggers.md)取代它。
- 目前版本中不支援原則和可用性屬性。 管線的開始時間取決於[觸發程序](concepts-pipeline-execution-triggers.md)。
- 目前版本中不支援範圍資料集 (管線中定義的資料集)。 

## <a name="next-steps"></a>後續步驟
如需使用上述其中一個工具或 SDK 來建立管線和資料集的逐步指示，請參閱下列教學課程。 

- [快速入門：使用 .NET 來建立資料處理站](quickstart-create-data-factory-dot-net.md)
- [快速入門：使用 PowerShell 來建立資料處理站](quickstart-create-data-factory-powershell.md)
- [快速入門：使用 REST API 來建立資料處理站](quickstart-create-data-factory-rest-api.md)
- 快速入門：使用 Azure 入口網站來建立資料處理站
