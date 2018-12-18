---
title: 在 Azure Data Factory 中對應資料集資料行 | Microsoft Docs
description: 了解如何將來源資料行對應至目的地資料行。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: f1dd8c4c5e38547492887425bb5732d87d5f8071
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37045879"
---
# <a name="map-source-dataset-columns-to-destination-dataset-columns"></a>將來源資料集資料行對應至目的地資料集資料行
> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 

資料行對應可用於指定將來源資料表「 結構 」中指定資料行對應至接收器資料表 「 結構 」 中指定資料行的方式。 複製活動的 **typeProperties** 區段中可使用 **columnMapping** 屬性。

資料行對應支援下列案例：

* 來源資料集結構中的所有資料行對應至接收資料集結構中的所有資料行。
* 來源資料集結構中的資料行子集對應至接收資料集結構中的所有資料行。

以下是會導致發生例外狀況的錯誤狀況：

* 接收器資料表「結構」中的資料行數量多於或少於對應中所指定的數量。
* 重複的對應。
* SQL 查詢結果中沒有對應中所指定的資料行名稱。

> [!NOTE]
> 下列範例是針對 Azure SQL 和 Azure Blob，但是也適用於任何支援矩形資料集的資料存放區。 請調整範例中的資料集和已連結服務定義，以指向相關資料來源中的資料。

## <a name="sample-1--column-mapping-from-azure-sql-to-azure-blob"></a>範例 1 – 從 Azure SQL 到 Azure Blob 的資料行對應
在此範例中，輸入資料表有一個結構，且指向 Azure SQL 資料庫中的 SQL 資料表。

```json
{
    "name": "AzureSQLInput",
    "properties": {
        "structure": 
         [
           { "name": "userid"},
           { "name": "name"},
           { "name": "group"}
         ],
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

在此範例中，輸出資料表有一個結構，且指向 Azure Blob 儲存體中的 Blob。

```json
{
    "name": "AzureBlobOutput",
    "properties":
    {
         "structure": 
          [
                { "name": "myuserid"},
                { "name": "myname" },
                { "name": "mygroup"}
          ],
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder",
            "fileName":"myfile.csv",
            "format":
            {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

下列 JSON 定義了管線中的複製活動。 來自來源的資料行是使用 **Translator** 屬性來對應至接收器中的資料行 (**columnMappings**)。

```json
{
    "name": "CopyActivity",
    "description": "description", 
    "type": "Copy",
    "inputs":  [ { "name": "AzureSQLInput"  } ],
    "outputs":  [ { "name": "AzureBlobOutput" } ],
    "typeProperties":    {
        "source":
        {
            "type": "SqlSource"
        },
        "sink":
        {
            "type": "BlobSink"
        },
        "translator": 
        {
            "type": "TabularTranslator",
            "ColumnMappings": "UserId: MyUserId, Group: MyGroup, Name: MyName"
        }
    },
   "scheduler": {
          "frequency": "Hour",
          "interval": 1
        }
}
```
**資料行對應流程：**

![資料行對應流程](./media/data-factory-map-columns/column-mapping-flow.png)

## <a name="sample-2--column-mapping-with-sql-query-from-azure-sql-to-azure-blob"></a>範例 2 – 利用 SQL 查詢從 Azure SQL 至 Azure Blob 的資料行對應
在此範例中，使用 SQL 查詢從 Azure SQL 擷取資料，而非只在 「 結構 」 區段中指定資料表名稱和資料行名稱。 

```json
{
    "name": "CopyActivity",
    "description": "description", 
    "type": "CopyActivity",
    "inputs":  [ { "name": " AzureSQLInput"  } ],
    "outputs":  [ { "name": " AzureBlobOutput" } ],
    "typeProperties":
    {
        "source":
        {
            "type": "SqlSource",
            "SqlReaderQuery": "$$Text.Format('SELECT * FROM MyTable WHERE StartDateTime = \\'{0:yyyyMMdd-HH}\\'', WindowStart)"
        },
        "sink":
        {
            "type": "BlobSink"
        },
        "Translator": 
        {
            "type": "TabularTranslator",
            "ColumnMappings": "UserId: MyUserId, Group: MyGroup,Name: MyName"
        }
    },
    "scheduler": {
          "frequency": "Hour",
          "interval": 1
        }
}
```
在此情況下，查詢結果會先對應至來源「 結構 」 中所指定的資料行。 接下來，來源 「 結構 」 中的資料行會使用 columnMappings 中指定的規則對應至接收器 「 結構 」 中的資料行。  假設該查詢傳回 5 個資料行，比來源的 “structure” 中所指定的多 2 個資料行。

**資料行對應流程**

![資料行對應流程 -2](./media/data-factory-map-columns/column-mapping-flow-2.png)

## <a name="next-steps"></a>後續步驟
請參閱有關使用「複製活動」的教學課程文章： 

- [將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
