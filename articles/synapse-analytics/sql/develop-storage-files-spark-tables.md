---
title: 在無伺服器 SQL 集區中同步處理 Apache Spark 外部資料表定義
description: 概述如何使用無伺服器 SQL 集區查詢 Spark 資料表
services: synapse-analytics
author: julieMSFT
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.openlocfilehash: 057a69881b8b407e5d75fa3510ca1c3eb1830bc7
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/01/2020
ms.locfileid: "96446488"
---
# <a name="synchronize-apache-spark-for-azure-synapse-external-table-definitions-in-serverless-sql-pool"></a>在無伺服器 SQL 集區中同步處理 Apache Spark for Azure Synapse 的外部資料表定義

無伺服器 SQL 集區可以自動同步處理來自 Apache Spark 的中繼資料。 系統會針對 Spark 集區中的每個資料庫建立無伺服器 SQL 集區資料庫。 

針對以 Parquet 為基礎且位於 Azure 儲存體中的每個 Spark 外部資料表，系統會在無伺服器 SQL 集區資料庫中建立一個外部資料表。 因此，您可以在關閉 Spark 集區後，繼續從無伺服器 SQL 集區查詢 Spark 外部資料表。

在 Spark 中分割資料表時，儲存體中的檔案會依資料夾組織。 無伺服器 SQL 集區會使用分割區中繼資料，而且只會以相關的資料夾和檔案作為查詢目標。

系統已針對 Azure Synapse 工作區中佈建的每個無伺服器 Apache Spark 集區自動設定中繼資料同步。 您可以立即開始查詢 Spark 外部資料表。

位於 Azure 儲存體中的每個 Spark Parquet 外部資料表都會以 dbo 結構描述中對應至無伺服器 SQL 集區資料庫的外部資料表來表示。 

針對 Spark 外部資料表查詢，請執行以外部 [spark_table] 為目標的查詢。 在執行下列範例之前，請確定您可以正確地[存取檔案所在的儲存體帳戶](develop-storage-files-storage-access-control.md)。

```sql
SELECT * FROM [db].dbo.[spark_table]
```

> [!NOTE]
> 新增、置放或變更資料行的 Spark 外部資料表命令都不會在無伺服器 SQL 集區的外部資料表中反映。

## <a name="apache-spark-data-types-to-sql-data-types-mapping"></a>Apache Spark 資料類型與 SQL 資料類型的對應

| Spark 資料類型 | SQL 資料類型               |
| --------------- | --------------------------- |
| ByteType        | SMALLINT                    |
| ShortType      | SMALLINT                    |
| IntegerType     | int                         |
| LongType        | BIGINT                      |
| FloatType       | real                        |
| DoubleType      | FLOAT                       |
| DecimalType     | decimal                     |
| TimestampType   | datetime2                   |
| DateType        | date                        |
| StringType      | varchar(max)\*               |
| BinaryType      | varbinary                   |
| BooleanType     | bit                         |
| ArrayType       | varchar(max)\* (into JSON)\** |
| MapType         | varchar(max)\* (into JSON)\** |
| StructType      | varchar(max)\* (into JSON)\** |

\* 使用的定序為 Latin1_General_100_BIN2_UTF8。

\** ArrayType、MapType 和 StructType 會以 JSON 表示。



## <a name="next-steps"></a>後續步驟

若要深入了解儲存體的存取控制，請前往[儲存體存取控制](develop-storage-files-storage-access-control.md)一文。
