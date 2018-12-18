---
title: 使用 Azure Data Factory 中的預存程序活動轉換資料 | Microsoft Docs
description: 說明如何使用 SQL Server 預存程序活動，以從 Data Factory 管線叫用 Azure SQL Database/資料倉儲中的預存程序。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/16/2018
ms.author: douglasl
ms.openlocfilehash: e8e0f8352404892ea8af6a0fa176c336dd2c1659
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37054019"
---
# <a name="transform-data-by-using-the-sql-server-stored-procedure-activity-in-azure-data-factory"></a>使用 Azure Data Factory 中的 SQL Server 預存程序活動轉換資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-stored-proc-activity.md)
> * [目前的版本](transform-data-using-stored-procedure.md)

您在 Data Factory [管線](concepts-pipelines-activities.md)中使用資料轉換活動，以轉換和處理您的原始資料以進行預測和深入了解。 預存程序活動是 Data Factory 支援的其中一個轉換活動。 本文是以[轉換資料](transform-data.md)一文為基礎，提供 Data Factory 中資料轉換及所支援轉換活動的一般概觀。

> [!NOTE]
> 如果您是 Azure Data Factory 的新手，請在閱讀本文之前先閱讀 [Azure Data Factory 簡介](introduction.md)，以及研習教學課程：[教學課程：轉換資料](tutorial-transform-data-spark-powershell.md)。 

您可以使用「預存程序活動」來叫用您企業或 Azure 虛擬機器 (VM) 中的下列其中一個資料存放區： 

- 連接字串
- Azure SQL 資料倉儲
- SQL Server Database。  如果您使用 SQL Server，請在裝載資料庫的同一部電腦上或可存取資料庫的個別電腦上安裝自我裝載整合執行階段。 自我裝載整合執行階段是一套透過安全且可管理的方式，將內部部署/Azure VM 上的資料來源連結至雲端服務的元件。 如需詳細資料，請參閱[自我裝載整合執行階段](create-self-hosted-integration-runtime.md)一文。

> [!IMPORTANT]
> 將資料複製到 Azure SQL Database 或 SQL Server 時，您可以使用 **sqlWriterStoredProcedureName** 屬性在複製活動中設定 **SqlSink** 以叫用預存程序。 如需有關此屬性的詳細資料，請參閱下列連接器文章：[Azure SQL Database](connector-azure-sql-database.md)、[SQL Server](connector-sql-server.md)。 不支援在使用複製活動將資料複製到「Azure SQL 資料倉儲」時叫用預存程序。 但是，您可以使用預存程序活動來叫用「SQL 資料倉儲」中的預存程序。 
>
> 從 Azure SQL Database、SQL Server 或「Azure SQL 資料倉儲」複製資料時，您可以使用 **sqlReaderStoredProcedureName** 屬性在複製活動中設定 **SqlSource**，以叫用預存程序從來源資料庫讀取資料。 如需詳細資訊，請參閱下列連接器文章：[Azure SQL Database](connector-azure-sql-database.md)、[SQL Server](connector-sql-server.md)、[Azure SQL 資料倉儲](connector-azure-sql-data-warehouse.md)          

 

## <a name="syntax-details"></a>語法詳細資料
以下是 JSON 格式來定義預存程序活動︰

```json
{
    "name": "Stored Procedure Activity",
    "description":"Description",
    "type": "SqlServerStoredProcedure",
    "linkedServiceName": {
        "referenceName": "AzureSqlLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "storedProcedureName": "sp_sample",
        "storedProcedureParameters": {
            "identifier": { "value": "1", "type": "Int" },
            "stringData": { "value": "str1" }

        }
    }
}
```

下表說明這些 JSON 屬性：

| 屬性                  | 說明                              | 必要 |
| ------------------------- | ---------------------------------------- | -------- |
| name                      | 活動的名稱                     | yes      |
| 說明               | 說明活動用途的文字 | 否       |
| type                      | 對於預存程序活動，活動類型為 **SqlServerStoredProcedure** | yes      |
| 預設容器         | 參考 **Azure SQL Database** 或 **Azure SQL 資料倉儲**或 **SQL Server**，註冊為 Data Factory 中的連結服務。 若要深入了解此已連結的服務，請參閱[計算已連結的服務](compute-linked-services.md)一文。 | yes      |
| storedProcedureName       | 指定要叫用的預存程序名稱。 | yes      |
| storedProcedureParameters | 指定預存程序參數的值。 使用 `"param1": { "value": "param1Value","type":"param1Type" }` 來傳遞資料來源所支援的參數值及其類型。 如果您需要為參數傳遞 Null，請使用 `"param1": { "value": null }` (全部小寫)。 | 否       |

## <a name="next-steps"></a>後續步驟
請參閱下列文章，其說明如何以其他方式轉換資料： 

* [U-SQL 活動](transform-data-using-data-lake-analytics.md)
* [Hive 活動](transform-data-using-hadoop-hive.md)
* [Pig 活動](transform-data-using-hadoop-pig.md)
* [MapReduce 活動](transform-data-using-hadoop-map-reduce.md)
* [Hadoop 串流活動](transform-data-using-hadoop-streaming.md)
* [Spark 活動](transform-data-using-spark.md)
* [.NET 自訂活動](transform-data-using-dotnet-custom-activity.md)
* [Machine Learning 批次執行活動](transform-data-using-machine-learning.md)
* [預存程序活動](transform-data-using-stored-procedure.md)
