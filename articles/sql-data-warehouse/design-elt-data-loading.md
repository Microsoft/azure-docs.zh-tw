---
title: 針對 Azure SQL 資料倉儲設計 ELT 而不是 ETL | Microsoft Docs
description: 針對「Azure SQL 資料倉儲」的資料載入設計「擷取」、「載入」及「轉換」(ELT) 程序，而不是 ETL。
services: sql-data-warehouse
author: ckarst
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: design
ms.date: 04/17/2018
ms.author: cakarst
ms.reviewer: igorstan
ms.openlocfilehash: facd9a98b2f2d866574dd1f9bf180fa2a9618f31
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43301325"
---
# <a name="designing-extract-load-and-transform-elt-for-azure-sql-data-warehouse"></a>設計 Azure SQL 資料倉儲的擷取、載入及轉換 (ELT)

設計「擷取」、「載入」及「轉換」(ELT) 程序 (而不是 ETL) 來將資料載入至「Azure SQL 資料倉儲」。 本文介紹如何設計可將資料移至 Azure 資料倉儲中的 ELT 程序。

> [!VIDEO https://www.youtube.com/embed/l9-wP7OdhDk]

## <a name="what-is-elt"></a>什麼是 ELT?

擷取、載入及轉換 (ELT) 是一個程序，讓資料從來源系統移至目的地資料倉儲。 此程序定期執行，例如每小時或每日，讓新產生的資料載入資料倉儲。 從來源將資料載入資料倉儲最理想的方式是開發 ELT 程序，該程序會使用 PolyBase 將資料載入 SQL 資料倉儲。

ELT 會先載入，然後轉換資料，而擷取、轉換及載入 (ETL) 則是在載入之前轉換資料。 執行 ELT 而不是 ETL，可以節省提供您自己的資源以在資料載入之前轉換的成本。 當使用 SQL 資料倉儲時，ELT 會利用 MPP 系統以執行轉換。

雖然針對 SQL 資料倉儲實作 ELT 有許多變化，以下是幾個基本步驟：  

1. 將來源資料擷取至文字檔。
2. 讓資料登陸到 Azure Blob 儲存體或 Azure Data Lake Store。
3. 準備要載入的資料。
2. 使用 PolyBase 將資料載入 SQL 資料倉儲暫存表格。
3. 轉換資料。
4. 將資料插入生產資料表。


如需載入教學課程，請參閱[使用 PolyBase 將資料從 Azure Blob 儲存體載入 SQL 資料倉儲中](load-data-from-azure-blob-storage-using-polybase.md)。

如需詳細資訊，請參閱[載入模式部落格](http://blogs.msdn.microsoft.com/sqlcat/2017/05/17/azure-sql-data-warehouse-loading-patterns-and-strategies/)。 

## <a name="options-for-loading-with-polybase"></a>以 PolyBase 載入的選項

PolyBase 是一種技術，它會透過 T-SQL 語言存取資料庫外部的資料。 它是將資料載入 SQL 資料倉儲的最佳方式。 使用 PolyBase，資料會以平行方式從資料來源直接載入運算節點。 

若要使用 PolyBase 載入資料，您可以使用任何一種載入選項。

- [PolyBase 與 T-SQL](load-data-from-azure-blob-storage-using-polybase.md) 非常適合於當您的資料是在 Azure Blob 儲存體或 Azure Data Lake Store 中的時候。 它給予您對於載入程序最多的控制權，但是也需要您定義外部資料物件。 其他方法會在您將來源資料表對應至目的地資料表時，在幕後定義這些物件。  若要協調 T-SQL 載入，您可以使用 Azure Data Factory、SSIS 或 Azure 函式。 
- [PolyBase 與 SSIS](/sql/integration-services/load-data-to-sql-data-warehouse) 非常適合於當您的來源資料是在 SQL Server 中的時候，無論是 SQL Server 內部部署或是在雲端。 SSIS 會定義來源至目的地資料表對應，也會協調載入。 如果您已經有 SSIS 套件，您可以將套件修改為搭配新的資料倉儲目的地。 
- [PolyBase 與 Azure Data Factory (ADF)](sql-data-warehouse-load-with-data-factory.md) 是另一個協調工具。  它會定義管線並排程作業。 
- 搭配使用 [PolyBase 與 Azure DataBricks](../azure-databricks/databricks-extract-load-sql-data-warehouse.md)，可將資料從 SQL 資料倉儲資料表移轉到 Databricks 資料框架和/或從 Databricks 資料框架移轉到 SQL 資料倉儲資料表。

### <a name="polybase-external-file-formats"></a>PolyBase 外部檔案格式

PolyBase 會從 UTF-8 和 UTF-16 編碼分隔符號文字檔載入資料。 除了分隔符號文字檔，它會從 Hadoop 檔案格式 RC 檔案、ORC 和 Parquet 載入。 PolyBase 可以從 Gzip 和 Snappy 壓縮檔案載入資料。 PolyBase 目前不支援延伸的 ASCII、固定寬度格式和巢狀格式，例如 WinZip、JSON 和 XML。

### <a name="non-polybase-loading-options"></a>非 PolyBase 載入選項
如果您的資料與 PolyBase 不相容，您可以使用 [bcp](/sql/tools/bcp-utility) 或 [SQLBulkCopy API](https://msdn.microsoft.com/library/system.data.sqlclient.sqlbulkcopy.aspx)。 bcp 會直接載入 SQL 資料倉儲而不需要透過 Azure Blob 儲存體，它僅適用於小型載入。 請注意，這些選項的載入效能會顯著低於 PolyBase。 


## <a name="extract-source-data"></a>擷取來源資料

從來源系統取得資料視來源而定。  目標是將資料移至分隔符號文字檔。 如果您使用 SQL Server，您可以使用 [bcp 命令列工具](/sql/tools/bcp-utility)來匯出資料。  

## <a name="land-data-to-azure-storage"></a>讓資料登陸到 Azure 儲存體

若要讓資料登陸到 Azure 儲存體，您可以將它移至 [Azure Blob 儲存體](../storage/blobs/storage-blobs-introduction.md)或 [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)。 在任一位置中，資料應該會儲存到文字檔。 Polybase 可以從任一位置載入。

這些是您可以用來將資料移至 Azure 儲存體的工具和服務。

- [Azure ExpressRoute](../expressroute/expressroute-introduction.md) 服務會增強網路輸送量、效能及可預測性。 ExpressRoute 是一項服務，它會透過專用私人連線將您的資料路由傳送至 Azure。 ExpressRoute 連線不會透過公用網際網路路由傳送資料。 相較於透過公用網際網路的一般連線，這個連線提供更為可靠、速度更快、延遲更低且安全性更高的網際網路連線。
- [AZCopy 公用程式](../storage/common/storage-moving-data.md)透過公用網際網路將資料移至 Azure 儲存體。 如果您的資料大小小於 10 TB，就適用這個選項。 若要使用 AZCopy 定期執行載入，請測試網路速度以查看是否可以接受。 
- [Azure Data Factory (ADF)](../data-factory/introduction.md) 具有閘道，您可以在本機伺服器上安裝。 然後您可以建立管線，將資料從本機伺服器移至 Azure 儲存體。 若要搭配使用 Data Factory 與 SQL 資料倉儲，請參閱[將資料載入 SQL 資料倉儲](/azure/data-factory/load-azure-sql-data-warehouse)。

## <a name="prepare-data"></a>準備資料

在將資料載入 SQL 資料倉儲之前，您可能需要在儲存體帳戶中準備及清除資料。 資料準備可以在您的資料是在來源中、當您將資料匯出到文字檔時，或是在資料在 Azure 儲存體之後執行。  盡可能儘早在程序中使用資料最簡單。  

### <a name="define-external-tables"></a>定義外部資料表
在您可以載入資料之前，您必須在您的資料倉儲中定義外部資料表。 PolyBase 使用外部資料表以定義及存取 Azure 儲存體中的資料。 外部資料表類似於一般資料表。 主要差異是外部資料表指向儲存在資料倉儲外部的資料。 

定義外部資料表牽涉到指定資料來源、文字檔格式和資料表定義。 以下是您需要的 T-SQL 語法主題：
- [CREATE EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql)
- [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql)
- [CREATE EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql)

如需建立外部物件的範例，請參閱載入教學課程中的[建立外部資料表](load-data-from-azure-blob-storage-using-polybase.md#create-external-tables-for-the-sample-data)步驟。

### <a name="format-text-files"></a>格式化文字檔

一旦定義外部物件，您必須對齊文字檔的資料列與外部資料表和檔案格式定義。 文字檔之每個資料列中的資料必須對齊資料表定義。

若要格式化文字檔：

- 如果您的資料是來自非關聯式來源，您必須將它轉換成資料列和資料行。 無論資料是來自關聯式或非關聯式來源，資料都必須轉換以對齊您打算將資料載入其中之資料表的資料行定義。 
- 格式化文字檔中的資料，以對齊 SQL 資料倉儲目的地資料表中的資料行和資料類型。 如果外部文字檔與資料倉儲資料表的的資料類型之間沒有對齊，會導致在載入期間資料列遭到拒絕。
- 使用結束字元分隔文字檔中的欄位。  請務必使用在來源資料中找不到的字元或字元序列。 搭配 [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql) 使用您指定的結束字元。

## <a name="load-to-a-staging-table"></a>載入至暫存表格
若要將資料載入資料倉儲，先將資料載入暫存表格相當合適。 藉由使用暫存表格，您可以處理錯誤而不會干擾生產資料表，而且可以避免在生產資料表上執行復原作業。 暫存表格也可讓您使用 SQL 資料倉儲，在將資料插入生產資料表之前先執行轉換。

若要使用 T-SQL 載入，請執行 [CREATE TABLE AS SELECT (CTAS)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) T-SQL 陳述式。 此命令會將 select 陳述式的結果插入新的資料表。 當陳述式從外部資料表選取時，它會匯入外部資料。 

在下列範例中，ext.Date 是外部資料表。 所有資料列會匯入名為 dbo.Date 的新資料表。

```sql
CREATE TABLE [dbo].[Date]
WITH
( 
    CLUSTERED COLUMNSTORE INDEX
)
AS SELECT * FROM [ext].[Date]
;
```

## <a name="transform-the-data"></a>轉換資料
當資料在暫存表格時，執行您的工作負載需要的轉換。 然後將資料移至生產資料表中。

## <a name="insert-data-into-production-table"></a>將資料插入生產資料表中

INSERT INTO ...SELECT 陳述式會從暫存表格將資料移至永久資料表。 

當您設計 ETL 程序時，嘗試在小型測試範例上執行程序。 嘗試從資料表將 1000 個資料列擷取至檔案，將它移至 Azure，然後嘗試將它載入暫存表格。 

## <a name="partner-loading-solutions"></a>合作夥伴載入解決方案
我們有許多合作夥伴皆提供載入解決方案。 若要深入了解，請參閱我們的[解決方案合作夥伴](sql-data-warehouse-partner-business-intelligence.md)清單。 

## <a name="next-steps"></a>後續步驟
如需載入指引，請參閱[載入資料的指引](guidance-for-loading-data.md)。


