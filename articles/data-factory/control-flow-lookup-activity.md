---
title: Azure Data Factory 中的查閱活動 | Microsoft Docs
description: 了解如何使用查閱活動查閱外部來源的值。 此輸出可供後續活動進一步參考。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/15/2018
ms.author: shlo
ms.openlocfilehash: e437e7b7d5298af325ae2a5e2ba689b417bad022
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39002915"
---
# <a name="lookup-activity-in-azure-data-factory"></a>Azure Data Factory 中的查閱活動

查閱活動可從任何 Azure Data Factory 所支援的資料來源擷取資料集。 請在下列狀況使用查閱活動：
- 動態判斷在後續活動中要處理哪些物件，而不是將物件名稱寫入程式碼。 某些物件範例是檔案和資料表。

查閱活動會讀取並傳回組態檔或資料表的內容。 也會傳回執行查詢或預存程序的結果。 查閱活動的輸出可用於後續的複製或轉換活動中 (如果輸出是單一值)。 輸出也可用於 ForEach 活動中 (如果輸出是屬性陣列)。

## <a name="supported-capabilities"></a>支援的功能

查閱活動支援下列資料來源。 查閱活動能傳回的資料列數目上限是 5000，大小最多為 2 MB。 目前，查閱活動在逾時之前的最長持續時間為一小時。

[!INCLUDE [data-factory-v2-supported-data-stores](../../includes/data-factory-v2-supported-data-stores-for-lookup-activity.md)]

## <a name="syntax"></a>語法

```json
{
    "name": "LookupActivity",
    "type": "Lookup",
    "typeProperties": {
        "source": {
            "type": "<source type>"
            <additional source specific properties (optional)>
        },
        "dataset": { 
            "referenceName": "<source dataset name>",
            "type": "DatasetReference"
        },
        "firstRowOnly": false
    }
}
```

## <a name="type-properties"></a>類型屬性
Name | 說明 | 類型 | 必要？
---- | ----------- | ---- | --------
資料集 | 提供查閱的資料集參考。 如需詳細資料，請參閱每個對應連接器文章中的＜**資料集屬性**＞一節。 | 索引鍵/值組 | 是
來源 | 包含資料集特定的來源屬性，與複製活動來源相同。 如需詳細資料，請參閱每個對應連接器文章中的＜**複製活動屬性**＞一節。 | 索引鍵/值組 | 是
firstRowOnly | 指出是否只傳回第一個資料列或傳回所有資料列。 | BOOLEAN | 否。 預設值為 `true`。

> [!NOTE]

> * 不支援 **ByteArray** 類型的來源資料行。
> * 資料集定義不支援**結構**。 針對文字格式檔案，可以使用標頭資料列來提供資料行名稱。
> * 如果您的查閱來源是 JSON 檔案，不支援用於調整 JSON 物件的 `jsonPathDefinition` 設定。 將會擷取整個物件。

## <a name="use-the-lookup-activity-result-in-a-subsequent-activity"></a>在後續活動中使用查閱活動結果

查閱結果會在活動執行結果的 `output` 區段中傳回。

* **當 `firstRowOnly` 設為 `true` (預設值)**，輸出格式如下列程式碼所示。 查閱結果就在固定的 `firstRow` 索引鍵底下。 若要在後續活動中使用結果，請使用 `@{activity('MyLookupActivity').output.firstRow.TableName}` 模式。

    ```json
    {
        "firstRow":
        {
            "Id": "1",
            "TableName" : "Table1"
        }
    }
    ```

* **當 `firstRowOnly` 設為 `false`**，輸出格式如下列程式碼所示。 `count` 欄位會指出傳回多少筆記錄。 詳細的值會顯示在固定 `value` 陣列下。 在這種情況下，查閱活動後面會接著 [Foreach 活動](control-flow-for-each-activity.md)。 您可以使用 `@activity('MyLookupActivity').output.value` 模式，將 `value` 陣列傳遞至 ForEach 活動的 `items` 欄位。 若要存取 `value` 陣列中的元素，請使用下列語法：`@{activity('lookupActivity').output.value[zero based index].propertyname}`。 例如 `@{activity('lookupActivity').output.value[0].tablename}`。

    ```json
    {
        "count": "2",
        "value": [
            {
                "Id": "1",
                "TableName" : "Table1"
            },
            {
                "Id": "2",
                "TableName" : "Table2"
            }
        ]
    } 
    ```

### <a name="copy-activity-example"></a>複製活動範例
在此範例中，複製活動會將資料從 Azure SQL Database 執行個體中的 SQL 資料表複製到 Azure Blob 儲存體。 SQL 資料表的名稱會儲存在 Blob 儲存體的 JSON 檔案中。 查閱活動在執行階段會查閱表格名稱。 使用這種方法可動態修改 JSON。 您不需要重新部署管線或資料集。 

此範例僅示範查閱第一個資料列。 若要查閱所有資料列並鏈結結果與 ForEach 活動，請參閱[使用 Azure Data Factory 大量複製多個資料表](tutorial-bulk-copy.md)中的範例。

### <a name="pipeline"></a>管線
這個管線包含兩個活動：「查閱」和「複製」。 

- 查閱活動會設定為使用 **LookupDataset**，它會參考 Azure Blob 儲存體中的位置。 查閱活動會從這個位置的 JSON 檔案讀取 SQL 資料表的名稱。 
- 複製活動會使用查閱活動的輸出，亦即 SQL 資料表的名稱。 **SourceDataset** 中的 **tableName** 屬性會設定為使用查閱活動的輸出。 複製活動會將資料從 SQL 資料表複製到 Azure Blob 儲存體 中的位置。 此位置會由 **SinkDataset** 屬性指定。 

```json
{
    "name": "LookupPipelineDemo",
    "properties": {
        "activities": [
            {
                "name": "LookupActivity",
                "type": "Lookup",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "dataset": { 
                        "referenceName": "LookupDataset", 
                        "type": "DatasetReference" 
                    }
                }
            },
            {
                "name": "CopyActivity",
                "type": "Copy",
                "typeProperties": {
                    "source": { 
                        "type": "SqlSource", 
                        "sqlReaderQuery": "select * from @{activity('LookupActivity').output.firstRow.tableName}" 
                    },
                    "sink": { 
                        "type": "BlobSink" 
                    }
                },                
                "dependsOn": [ 
                    { 
                        "activity": "LookupActivity", 
                        "dependencyConditions": [ "Succeeded" ] 
                    }
                 ],
                "inputs": [ 
                    { 
                        "referenceName": "SourceDataset", 
                        "type": "DatasetReference" 
                    } 
                ],
                "outputs": [ 
                    { 
                        "referenceName": "SinkDataset", 
                        "type": "DatasetReference" 
                    } 
                ]
            }
        ]
    }
}
```

### <a name="lookup-dataset"></a>查閱資料集
**查閱**資料集會是 Azure 儲存體查閱資料夾中的 **sourcetable.json** 檔案 (此檔案是由 **AzureStorageLinkedService** 類型指定)。 

```json
{
    "name": "LookupDataset",
    "properties": {
        "type": "AzureBlob",
        "typeProperties": {
            "folderPath": "lookup",
            "fileName": "sourcetable.json",
            "format": {
                "type": "JsonFormat",
                "filePattern": "SetOfObjects"
            }
        },
        "linkedServiceName": {
            "referenceName": "AzureStorageLinkedService",
            "type": "LinkedServiceReference"
        }
    }
}
```

### <a name="source-dataset-for-copy-activity"></a>複製活動的**來源**資料集
**來源**資料集會使用查閱活動的輸出，亦即 SQL 資料表的名稱。 複製活動會將資料從此 SQL 資料表複製到 Azure Blob 儲存體 中的位置。 此位置會由**接收**資料集指定。 

```json
{
    "name": "SourceDataset",
    "properties": {
        "type": "AzureSqlTable",
        "typeProperties":{
            "tableName": "@{activity('LookupActivity').output.firstRow.tableName}"
        },
        "linkedServiceName": {
            "referenceName": "AzureSqlLinkedService",
            "type": "LinkedServiceReference"
        }
    }
}
```

### <a name="sink-dataset-for-copy-activity"></a>複製活動的**接收**資料集
複製活動會將資料從 SQL 資料表複製到 Azure 儲存體中 **csv** 資料夾內的 **filebylookup.csv** 檔案。 此檔案會由 **AzureStorageLinkedService** 屬性指定。 

```json
{
    "name": "SinkDataset",
    "properties": {
        "type": "AzureBlob",
        "typeProperties": {
            "folderPath": "csv",
            "fileName": "filebylookup.csv",
            "format": {
                "type": "TextFormat"                                                                    
            }
        },
        "linkedServiceName": {
            "referenceName": "AzureStorageLinkedService",
            "type": "LinkedServiceReference"
        }
    }
}
```

### <a name="azure-storage-linked-service"></a>Azure 儲存體連結服務
這個儲存體帳戶包含具有 SQL 資料表名稱的 JSON 檔案。 

```json
{
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": {
                "value": "DefaultEndpointsProtocol=https;AccountName=<StorageAccountName>;AccountKey=<StorageAccountKey>",
                "type": "SecureString"
            }
        }
    },
        "name": "AzureStorageLinkedService"
}
```

### <a name="azure-sql-database-linked-service"></a>Azure SQL Database 的連結服務
這個 Azure SQL 資料庫執行個體包含要複製到 Blob 儲存體的資料。 

```json
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "description": "",
        "typeProperties": {
            "connectionString": {
                "value": "Server=<server>;Initial Catalog=<database>;User ID=<user>;Password=<password>;",
                "type": "SecureString"
            }
        }
    }
}
```

### <a name="sourcetablejson"></a>sourcetable.json

#### <a name="set-of-objects"></a>物件集合

```json
{
  "Id": "1",
  "tableName": "Table1"
}
{
   "Id": "2",
  "tableName": "Table2"
}
```

#### <a name="array-of-objects"></a>物件的陣列

```json
[ 
    {
        "Id": "1",
        "tableName": "Table1"
    },
    {
        "Id": "2",
        "tableName": "Table2"
    }
]
```

## <a name="next-steps"></a>後續步驟
請參閱 Data Factory 支援的其他控制流程活動： 

- [執行管線活動](control-flow-execute-pipeline-activity.md)
- [ForEach 活動](control-flow-for-each-activity.md)
- [GetMetadata 活動](control-flow-get-metadata-activity.md)
- [Web 活動](control-flow-web-activity.md)
