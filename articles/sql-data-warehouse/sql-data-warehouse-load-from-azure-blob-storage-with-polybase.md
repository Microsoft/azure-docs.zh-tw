---
title: 將 Contoso 零售資料載入 Azure SQL 資料倉儲 | Microsoft Docs
description: 使用 PolyBase 和 T-SQL 命令來將 Contoso 零售資料的兩個資料表載入至 Azure SQL 資料倉儲。
services: sql-data-warehouse
author: ckarst
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: cakarst
ms.reviewer: igorstan
ms.openlocfilehash: cf484d5d7a48117bac659dcfad356238f1e9f8a2
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307432"
---
# <a name="load-contoso-retail-data-to-azure-sql-data-warehouse"></a>將 Contoso 零售資料載入 Azure SQL 資料倉儲

使用 PolyBase 和 T-SQL 命令來將 Contoso 零售資料的兩個資料表載入至 Azure SQL 資料倉儲。 若要載入完整的資料集，請從 Microsoft SQL Server 範例儲存機制執行[載入完整 Contoso 零售資料倉儲](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/contoso-data-warehouse/readme.md)範例。

在本教學課程中，您將：

1. 設定 PolyBase 以從 Azure Blob 儲存體載入
2. 將公用資料載入您的資料庫
3. 在完成載入後執行最佳化。

## <a name="before-you-begin"></a>開始之前
若要執行本教學課程，您需要已經擁有 SQL 資料倉儲資料庫的 Azure 帳戶。 如果您尚未擁有此資料庫，請參閱[建立 SQL 資料倉儲][Create a SQL Data Warehouse]。

## <a name="1-configure-the-data-source"></a>1.設定資料來源
PolyBase 使用 T-SQL 外部物件以定義外部資料的位置和屬性。 外部物件定義會儲存在 SQL 資料倉儲中。 資料本身則會儲存在外部。

### <a name="11-create-a-credential"></a>1.1. 建立認證
**略過此步驟** 。 您並不需要安全地存取公用資料，因為該資料已經可供所有人存取。

**不要略過此步驟** 。 若要透過認證存取資料，請使用下列指令碼來建立資料庫範圍的認證，然後在定義資料來源的位置時使用它。

```sql
-- A: Create a master key.
-- Only necessary if one does not already exist.
-- Required to encrypt the credential secret in the next step.

CREATE MASTER KEY;


-- B: Create a database scoped credential
-- IDENTITY: Provide any string, it is not used for authentication to Azure storage.
-- SECRET: Provide your Azure storage account key.


CREATE DATABASE SCOPED CREDENTIAL AzureStorageCredential
WITH
    IDENTITY = 'user',
    SECRET = '<azure_storage_account_key>'
;


-- C: Create an external data source
-- TYPE: HADOOP - PolyBase uses Hadoop APIs to access data in Azure blob storage.
-- LOCATION: Provide Azure storage account name and blob container name.
-- CREDENTIAL: Provide the credential created in the previous step.

CREATE EXTERNAL DATA SOURCE AzureStorage
WITH (
    TYPE = HADOOP,
    LOCATION = 'wasbs://<blob_container_name>@<azure_storage_account_name>.blob.core.windows.net',
    CREDENTIAL = AzureStorageCredential
);
```

跳到步驟 2。

### <a name="12-create-the-external-data-source"></a>1.2. 建立外部資料來源
使用此 [CREATE EXTERNAL DATA SOURCE][CREATE EXTERNAL DATA SOURCE] 命令以儲存資料的位置及類型。 

```sql
CREATE EXTERNAL DATA SOURCE AzureStorage_west_public
WITH 
(  
    TYPE = Hadoop 
,   LOCATION = 'wasbs://contosoretaildw-tables@contosoretaildw.blob.core.windows.net/'
); 
```

> [!IMPORTANT]
> 如果您選擇將 Azure Blob 儲存體容器設為公用，請記住，當資料離開資料中心時，您身為資料擁有者將必須支付資料流出費用。 
> 
> 

## <a name="2-configure-data-format"></a>2.設定資料格式
資料將會以文字檔儲存在 Azure Blob 儲存體中，每個欄位都會以分隔符號分隔。 執行此 [CREATE EXTERNAL FILE FORMAT][CREATE EXTERNAL FILE FORMAT] 命令以指定文字檔中資料的格式。 Contoso 資料為未壓縮且以直立線符號分隔。

```sql
CREATE EXTERNAL FILE FORMAT TextFileFormat 
WITH 
(   FORMAT_TYPE = DELIMITEDTEXT
,    FORMAT_OPTIONS    (   FIELD_TERMINATOR = '|'
                    ,    STRING_DELIMITER = ''
                    ,    DATE_FORMAT         = 'yyyy-MM-dd HH:mm:ss.fff'
                    ,    USE_TYPE_DEFAULT = FALSE 
                    )
);
``` 

## <a name="3-create-the-external-tables"></a>3.建立外部資料表
您在指定資料來源和檔案格式之後，便可以開始建立外部資料表。 

### <a name="31-create-a-schema-for-the-data"></a>3.1. 建立資料的結構描述。
若要在您的資料庫中建立儲存 Contoso 資料的位置，請建立結構描述。

```sql
CREATE SCHEMA [asb]
GO
```

### <a name="32-create-the-external-tables"></a>3.2. 建立外部資料表。
執行此指令碼來建立 DimProduct 和 FactOnlineSales 外部資料表。 我們在這邊只需要定義資料行名稱和資料類型，並將它們繫結至 Azure Blob 儲存體檔案的位置及格式。 定義會儲存在 SQL 資料倉儲中，而資料則仍然儲存在 Azure 儲存體 Blob 中。

**LOCATION** 參數為 Azure 儲存體 Blob 中根目錄下方的資料夾。 每個資料表都位於不同的資料夾中。

```sql

--DimProduct
CREATE EXTERNAL TABLE [asb].DimProduct (
    [ProductKey] [int] NOT NULL,
    [ProductLabel] [nvarchar](255) NULL,
    [ProductName] [nvarchar](500) NULL,
    [ProductDescription] [nvarchar](400) NULL,
    [ProductSubcategoryKey] [int] NULL,
    [Manufacturer] [nvarchar](50) NULL,
    [BrandName] [nvarchar](50) NULL,
    [ClassID] [nvarchar](10) NULL,
    [ClassName] [nvarchar](20) NULL,
    [StyleID] [nvarchar](10) NULL,
    [StyleName] [nvarchar](20) NULL,
    [ColorID] [nvarchar](10) NULL,
    [ColorName] [nvarchar](20) NOT NULL,
    [Size] [nvarchar](50) NULL,
    [SizeRange] [nvarchar](50) NULL,
    [SizeUnitMeasureID] [nvarchar](20) NULL,
    [Weight] [float] NULL,
    [WeightUnitMeasureID] [nvarchar](20) NULL,
    [UnitOfMeasureID] [nvarchar](10) NULL,
    [UnitOfMeasureName] [nvarchar](40) NULL,
    [StockTypeID] [nvarchar](10) NULL,
    [StockTypeName] [nvarchar](40) NULL,
    [UnitCost] [money] NULL,
    [UnitPrice] [money] NULL,
    [AvailableForSaleDate] [datetime] NULL,
    [StopSaleDate] [datetime] NULL,
    [Status] [nvarchar](7) NULL,
    [ImageURL] [nvarchar](150) NULL,
    [ProductURL] [nvarchar](150) NULL,
    [ETLLoadID] [int] NULL,
    [LoadDate] [datetime] NULL,
    [UpdateDate] [datetime] NULL
)
WITH
(
    LOCATION='/DimProduct/' 
,   DATA_SOURCE = AzureStorage_west_public
,   FILE_FORMAT = TextFileFormat
,   REJECT_TYPE = VALUE
,   REJECT_VALUE = 0
)
;

--FactOnlineSales
CREATE EXTERNAL TABLE [asb].FactOnlineSales 
(
    [OnlineSalesKey] [int]  NOT NULL,
    [DateKey] [datetime] NOT NULL,
    [StoreKey] [int] NOT NULL,
    [ProductKey] [int] NOT NULL,
    [PromotionKey] [int] NOT NULL,
    [CurrencyKey] [int] NOT NULL,
    [CustomerKey] [int] NOT NULL,
    [SalesOrderNumber] [nvarchar](20) NOT NULL,
    [SalesOrderLineNumber] [int] NULL,
    [SalesQuantity] [int] NOT NULL,
    [SalesAmount] [money] NOT NULL,
    [ReturnQuantity] [int] NOT NULL,
    [ReturnAmount] [money] NULL,
    [DiscountQuantity] [int] NULL,
    [DiscountAmount] [money] NULL,
    [TotalCost] [money] NOT NULL,
    [UnitCost] [money] NULL,
    [UnitPrice] [money] NULL,
    [ETLLoadID] [int] NULL,
    [LoadDate] [datetime] NULL,
    [UpdateDate] [datetime] NULL
)
WITH
(
    LOCATION='/FactOnlineSales/' 
,   DATA_SOURCE = AzureStorage_west_public
,   FILE_FORMAT = TextFileFormat
,   REJECT_TYPE = VALUE
,   REJECT_VALUE = 0
)
;
```

## <a name="4-load-the-data"></a>4.載入資料
存取外部資料有很多不同的方式。  您可以直接從外部資料表查詢資料、將資料載入新的資料庫資料表，或將外部資料新增到現有資料庫資料表。  

### <a name="41-create-a-new-schema"></a>4.1. 建立新的結構描述
CTAS 會建立包含資料的新資料表。  首先，請建立 Contoso 資料的結構描述。

```sql
CREATE SCHEMA [cso]
GO
```

### <a name="42-load-the-data-into-new-tables"></a>4.2. 將資料載入新資料表
若要從 Azure Blob 儲存體載入資料，並將它儲存在資料庫內的資料表中，請使用 [CREATE TABLE AS SELECT (Transact-SQL)][CREATE TABLE AS SELECT (Transact-SQL)] 陳述式。 以 CTAS 載入將能利用您剛剛建立的強型別外部資料表。針對每個資料表，請使用一個 [CTAS][CTAS] 陳述式。 
 
CTAS 建立新的資料表，並將選取陳述式的結果填入該資料表。 CTAS 定義新資料表，以使它擁有和選取陳述式之結果相同的資料行和資料類型。 如果您選取外部資料表上的所有資料行，則新資料表將會是外部資料表中資料行和資料類型的複本。

在此範例中，我們同時將維度和事實資料表建立為雜湊分散式資料表。 

```sql
SELECT GETDATE();
GO

CREATE TABLE [cso].[DimProduct]            WITH (DISTRIBUTION = HASH([ProductKey]  ) ) AS SELECT * FROM [asb].[DimProduct]             OPTION (LABEL = 'CTAS : Load [cso].[DimProduct]             ');
CREATE TABLE [cso].[FactOnlineSales]       WITH (DISTRIBUTION = HASH([ProductKey]  ) ) AS SELECT * FROM [asb].[FactOnlineSales]        OPTION (LABEL = 'CTAS : Load [cso].[FactOnlineSales]        ');
```

### <a name="43-track-the-load-progress"></a>4.3 追蹤載入進度
您可以使用動態管理檢視 (DMV) 來追蹤載入進度。 

```sql
-- To see all requests
SELECT * FROM sys.dm_pdw_exec_requests;

-- To see a particular request identified by its label
SELECT * FROM sys.dm_pdw_exec_requests as r
WHERE r.[label] = 'CTAS : Load [cso].[DimProduct]             '
      OR r.[label] = 'CTAS : Load [cso].[FactOnlineSales]        '
;

-- To track bytes and files
SELECT
    r.command,
    s.request_id,
    r.status,
    count(distinct input_name) as nbr_files, 
    sum(s.bytes_processed)/1024/1024/1024 as gb_processed
FROM
    sys.dm_pdw_exec_requests r
    inner join sys.dm_pdw_dms_external_work s
        on r.request_id = s.request_id
WHERE 
    r.[label] = 'CTAS : Load [cso].[DimProduct]             '
    OR r.[label] = 'CTAS : Load [cso].[FactOnlineSales]        '
GROUP BY
    r.command,
    s.request_id,
    r.status
ORDER BY
    nbr_files desc,
    gb_processed desc;
```

## <a name="5-optimize-columnstore-compression"></a>5.最佳化資料行存放區壓縮
根據預設，SQL 資料倉儲會將資料表儲存為叢集資料行存放區索引。 載入完成後，某些資料列可能不會被壓縮為資料行存放區。  有許多原因會導致發生此情況。 若要深入了解，請參閱[管理資料行存放區索引][manage columnstore indexes]。

若要最佳化載入後的查詢效能和資料行存放區壓縮，請重建資料表以強制資料行存放區索引對所有資料列進行壓縮。 

```sql
SELECT GETDATE();
GO

ALTER INDEX ALL ON [cso].[DimProduct]               REBUILD;
ALTER INDEX ALL ON [cso].[FactOnlineSales]          REBUILD;
```

如需維護資料行存放區索引的詳細資訊，請參閱[管理資料行存放區索引][manage columnstore indexes]一文。

## <a name="6-optimize-statistics"></a>6.最佳化統計資料
您最好在載入後立刻建立單一資料行統計資料。 針對統計資料，您將會有一些選項。 例如，如果您在每個資料行上建立單一資料行統計資料，可能會需要很長的時間才能重建所有統計資料。 如果您知道某些資料行不會被包含在查詢述詞中，您可以略過為那些資料行建立統計資料。

如果您決定要在每個資料表的每個資料行上建立單一資料行統計資料，您可以使用[統計資料][statistics]一文中的預存程序程式碼範例 `prc_sqldw_create_stats`。

下列範例為建立統計資料的好起點。 它會在維度資料表中的每個資料行上，以及在事實資料表中的每個聯結資料行上建立單一資料行統計資料。 您之後隨時可以將單一或多個資料行統計資料新增到其他事實資料表資料行上。

```sql
CREATE STATISTICS [stat_cso_DimProduct_AvailableForSaleDate] ON [cso].[DimProduct]([AvailableForSaleDate]);
CREATE STATISTICS [stat_cso_DimProduct_BrandName] ON [cso].[DimProduct]([BrandName]);
CREATE STATISTICS [stat_cso_DimProduct_ClassID] ON [cso].[DimProduct]([ClassID]);
CREATE STATISTICS [stat_cso_DimProduct_ClassName] ON [cso].[DimProduct]([ClassName]);
CREATE STATISTICS [stat_cso_DimProduct_ColorID] ON [cso].[DimProduct]([ColorID]);
CREATE STATISTICS [stat_cso_DimProduct_ColorName] ON [cso].[DimProduct]([ColorName]);
CREATE STATISTICS [stat_cso_DimProduct_ETLLoadID] ON [cso].[DimProduct]([ETLLoadID]);
CREATE STATISTICS [stat_cso_DimProduct_ImageURL] ON [cso].[DimProduct]([ImageURL]);
CREATE STATISTICS [stat_cso_DimProduct_LoadDate] ON [cso].[DimProduct]([LoadDate]);
CREATE STATISTICS [stat_cso_DimProduct_Manufacturer] ON [cso].[DimProduct]([Manufacturer]);
CREATE STATISTICS [stat_cso_DimProduct_ProductDescription] ON [cso].[DimProduct]([ProductDescription]);
CREATE STATISTICS [stat_cso_DimProduct_ProductKey] ON [cso].[DimProduct]([ProductKey]);
CREATE STATISTICS [stat_cso_DimProduct_ProductLabel] ON [cso].[DimProduct]([ProductLabel]);
CREATE STATISTICS [stat_cso_DimProduct_ProductName] ON [cso].[DimProduct]([ProductName]);
CREATE STATISTICS [stat_cso_DimProduct_ProductSubcategoryKey] ON [cso].[DimProduct]([ProductSubcategoryKey]);
CREATE STATISTICS [stat_cso_DimProduct_ProductURL] ON [cso].[DimProduct]([ProductURL]);
CREATE STATISTICS [stat_cso_DimProduct_Size] ON [cso].[DimProduct]([Size]);
CREATE STATISTICS [stat_cso_DimProduct_SizeRange] ON [cso].[DimProduct]([SizeRange]);
CREATE STATISTICS [stat_cso_DimProduct_SizeUnitMeasureID] ON [cso].[DimProduct]([SizeUnitMeasureID]);
CREATE STATISTICS [stat_cso_DimProduct_Status] ON [cso].[DimProduct]([Status]);
CREATE STATISTICS [stat_cso_DimProduct_StockTypeID] ON [cso].[DimProduct]([StockTypeID]);
CREATE STATISTICS [stat_cso_DimProduct_StockTypeName] ON [cso].[DimProduct]([StockTypeName]);
CREATE STATISTICS [stat_cso_DimProduct_StopSaleDate] ON [cso].[DimProduct]([StopSaleDate]);
CREATE STATISTICS [stat_cso_DimProduct_StyleID] ON [cso].[DimProduct]([StyleID]);
CREATE STATISTICS [stat_cso_DimProduct_StyleName] ON [cso].[DimProduct]([StyleName]);
CREATE STATISTICS [stat_cso_DimProduct_UnitCost] ON [cso].[DimProduct]([UnitCost]);
CREATE STATISTICS [stat_cso_DimProduct_UnitOfMeasureID] ON [cso].[DimProduct]([UnitOfMeasureID]);
CREATE STATISTICS [stat_cso_DimProduct_UnitOfMeasureName] ON [cso].[DimProduct]([UnitOfMeasureName]);
CREATE STATISTICS [stat_cso_DimProduct_UnitPrice] ON [cso].[DimProduct]([UnitPrice]);
CREATE STATISTICS [stat_cso_DimProduct_UpdateDate] ON [cso].[DimProduct]([UpdateDate]);
CREATE STATISTICS [stat_cso_DimProduct_Weight] ON [cso].[DimProduct]([Weight]);
CREATE STATISTICS [stat_cso_DimProduct_WeightUnitMeasureID] ON [cso].[DimProduct]([WeightUnitMeasureID]);
CREATE STATISTICS [stat_cso_FactOnlineSales_CurrencyKey] ON [cso].[FactOnlineSales]([CurrencyKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_CustomerKey] ON [cso].[FactOnlineSales]([CustomerKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_DateKey] ON [cso].[FactOnlineSales]([DateKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_OnlineSalesKey] ON [cso].[FactOnlineSales]([OnlineSalesKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_ProductKey] ON [cso].[FactOnlineSales]([ProductKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_PromotionKey] ON [cso].[FactOnlineSales]([PromotionKey]);
CREATE STATISTICS [stat_cso_FactOnlineSales_StoreKey] ON [cso].[FactOnlineSales]([StoreKey]);
```

## <a name="achievement-unlocked"></a>成就解鎖！
您已成功將公用資料載入 Azure SQL 資料倉儲。 太棒了！

您現在可以使用與下面類似的查詢來開始查詢資料表︰

```sql
SELECT  SUM(f.[SalesAmount]) AS [sales_by_brand_amount]
,       p.[BrandName]
FROM    [cso].[FactOnlineSales] AS f
JOIN    [cso].[DimProduct]      AS p ON f.[ProductKey] = p.[ProductKey]
GROUP BY p.[BrandName]
```

## <a name="next-steps"></a>後續步驟
若要載入完整的 Contoso 零售資料倉儲資料，請使用指令碼。如需更多開發秘訣，請參閱 [SQL 資料倉儲開發概觀][SQL Data Warehouse development overview]。

<!--Image references-->

<!--Article references-->
[Create a SQL Data Warehouse]: sql-data-warehouse-get-started-provision.md
[Load data into SQL Data Warehouse]: sql-data-warehouse-overview-load.md
[SQL Data Warehouse development overview]: sql-data-warehouse-overview-develop.md
[manage columnstore indexes]: sql-data-warehouse-tables-index.md
[Statistics]: sql-data-warehouse-tables-statistics.md
[CTAS]: sql-data-warehouse-develop-ctas.md
[label]: sql-data-warehouse-develop-label.md

<!--MSDN references-->
[CREATE EXTERNAL DATA SOURCE]: https://msdn.microsoft.com/library/dn935022.aspx
[CREATE EXTERNAL FILE FORMAT]: https://msdn.microsoft.com/library/dn935026.aspx
[CREATE TABLE AS SELECT (Transact-SQL)]: https://msdn.microsoft.com/library/mt204041.aspx
[sys.dm_pdw_exec_requests]: https://msdn.microsoft.com/library/mt203887.aspx
[REBUILD]: https://msdn.microsoft.com/library/ms188388.aspx

<!--Other Web references-->
[Microsoft Download Center]: http://www.microsoft.com/download/details.aspx?id=36433
[Load the full Contoso Retail Data Warehouse]: https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/contoso-data-warehouse/readme.md
