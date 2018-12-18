---
title: 對不同結構描述的雲端資料庫執行查詢 | Microsoft Docs
description: 如何透過垂直資料分割設定跨資料庫查詢
services: sql-database
ms.service: sql-database
subservice: elastic-scale
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: MladjoA
ms.author: mlandzic
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: 3d94a52aaafca91d45d7f1a6406d3f998fbffe9f
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161755"
---
# <a name="query-across-cloud-databases-with-different-schemas-preview"></a>對不同結構描述的雲端資料庫執行查詢 (預覽)
![在不同資料庫中跨資料表查詢][1]

垂直資料分割資料庫使用在不同資料庫的不同資料表集。 這表示不同資料庫的結構描述不同。 比方說，庫存的所有資料表都位於一個資料庫上，而所有會計相關資料表則位於另一個資料庫上。 

## <a name="prerequisites"></a>必要條件
* 使用者必須擁有 ALTER ANY EXTERNAL DATA SOURCE 權限。 這個權限包含在 ALTER DATABASE 權限中。
* 需有 ALTER ANY EXTERNAL DATA SOURCE 權限，才能參考基礎資料來源。

## <a name="overview"></a>概觀

> [!NOTE]
> 與水平資料分割不同，這些 DDL 陳述式並不倚賴透過彈性資料庫用戶端程式庫來定義帶有分區對應的資料層。
>

1. [CREATE MASTER KEY](https://msdn.microsoft.com/library/ms174382.aspx)
2. [建立資料庫範圍認證](https://msdn.microsoft.com/library/mt270260.aspx)
3. [CREATE EXTERNAL DATA SOURCE](https://msdn.microsoft.com/library/dn935022.aspx)
4. [CREATE EXTERNAL TABLE](https://msdn.microsoft.com/library/dn935021.aspx) 

## <a name="create-database-scoped-master-key-and-credentials"></a>建立資料庫範圍的主要金鑰和認證
彈性查詢使用認證連接到遠端資料庫。  

    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'master_key_password';
    CREATE DATABASE SCOPED CREDENTIAL <credential_name>  WITH IDENTITY = '<username>',  
    SECRET = '<password>'
    [;]

> [!NOTE]
> 請確定 `<username>` 不包含任何 **"\@servername"** 尾碼。 
>

## <a name="create-external-data-sources"></a>建立外部資料來源
語法：

    <External_Data_Source> ::=
    CREATE EXTERNAL DATA SOURCE <data_source_name> WITH 
               (TYPE = RDBMS,
                LOCATION = ’<fully_qualified_server_name>’,
                DATABASE_NAME = ‘<remote_database_name>’,  
                CREDENTIAL = <credential_name> 
                ) [;] 

> [!IMPORTANT]
> TYPE 參數必須設定為 **RDBMS**。 
>

### <a name="example"></a>範例
下列範例說明對外部資料來源使用 CREATE 陳述式。 

    CREATE EXTERNAL DATA SOURCE RemoteReferenceData 
    WITH 
    ( 
        TYPE=RDBMS, 
        LOCATION='myserver.database.windows.net', 
        DATABASE_NAME='ReferenceData', 
        CREDENTIAL= SqlUser 
    ); 

若要擷取目前的外部資料來源清單︰ 

    select * from sys.external_data_sources; 

### <a name="external-tables"></a>外部資料表
語法：

    CREATE EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name . ] table_name  
    ( { <column_definition> } [ ,...n ])     
    { WITH ( <rdbms_external_table_options> ) } 
    )[;] 

    <rdbms_external_table_options> ::= 
      DATA_SOURCE = <External_Data_Source>, 
      [ SCHEMA_NAME = N'nonescaped_schema_name',] 
      [ OBJECT_NAME = N'nonescaped_object_name',] 

### <a name="example"></a>範例
    CREATE EXTERNAL TABLE [dbo].[customer]( 
        [c_id] int NOT NULL, 
        [c_firstname] nvarchar(256) NULL, 
        [c_lastname] nvarchar(256) NOT NULL, 
        [street] nvarchar(256) NOT NULL, 
        [city] nvarchar(256) NOT NULL, 
        [state] nvarchar(20) NULL, 
        [country] nvarchar(50) NOT NULL, 
    ) 
    WITH 
    ( 
           DATA_SOURCE = RemoteReferenceData 
    ); 

下列範例顯示如何從目前資料庫擷取外部資料表清單： 

    select * from sys.external_tables; 

### <a name="remarks"></a>備註
彈性的查詢會延伸現有的外部資料表語法來定義使用 RDBMS 類型外部資料來源的外部資料表。 垂直資料分割的外部資料表定義包含下列各方面： 

* **結構描述**：外部資料表 DDL 會定義您的查詢可以使用的結構描述。 外部資料表定義中提供的結構描述必須符合實際資料儲存所在之遠端資料庫中資料表的結構描述。 
* **遠端資料庫參考**：外部資料表 DDL 指的是外部資料來源。 外部資料來源可指定邏輯伺服器名稱和實際資料表資料儲存所在之遠端資料庫的資料庫名稱。 

如上一節所述使用外部資料來源，建立外部資料表的語法如下： 

DATA_SOURCE 子句會定義用於外部資料表的外部資料來源 (亦即垂直資料分割情形中的遠端資料庫)。  

SCHEMA_NAME 和 OBJECT_NAME 子句提供的功能可將外部資料表定義分別對應至遠端資料庫上不同結構描述中的資料表，或對應至不同名稱的資料表。 不論是您想要為目錄檢視或遠端資料庫上的 DMV 定義外部資料表，還是在遠端資料表名稱在本機已被使用的任何其他情況下，這個方法都很實用。  

下列 DDL 陳述式會從本機目錄卸除現有的外部資料表定義。 它不會影響遠端資料庫。 

    DROP EXTERNAL TABLE [ [ schema_name ] . | schema_name. ] table_name[;]  

**CREATE/DROP EXTERNAL TABLE 的權限**：ALTER ANY EXTERNAL DATA SOURCE 權限對外部資料表 DDL 而言是必要的，而後者在參考基礎資料來源時也是必要的。  

## <a name="security-considerations"></a>安全性考量
可存取外部資料表的使用者可以在外部資料來源定義中所提供的認證下，自動取得基礎遠端資料表的存取權。 您應該仔細管理外部資料表的存取權，以避免透過外部資料來源的認證所造成的意外權限提升。 一般的 SQL 權限可用來授與或撤銷外部資料表的存取權，如同它是一般資料表那樣。  

## <a name="example-querying-vertically-partitioned-databases"></a>範例︰查詢垂直資料分割的資料庫
下列查詢會執行訂單和訂單明細的兩個本機資料表與客戶遠端資料表之間的三方聯結。 這是彈性查詢的參考資料使用案例的範例： 

    SELECT      
     c_id as customer,
     c_lastname as customer_name,
     count(*) as cnt_orderline, 
     max(ol_quantity) as max_quantity,
     avg(ol_amount) as avg_amount,
     min(ol_delivery_d) as min_deliv_date
    FROM customer 
    JOIN orders 
    ON c_id = o_c_id
    JOIN  order_line 
    ON o_id = ol_o_id and o_c_id = ol_c_id
    WHERE c_id = 100


## <a name="stored-procedure-for-remote-t-sql-execution-spexecuteremote"></a>用於遠端 T-SQL 執行的預存程序：sp\_execute_remote
彈性查詢也會介紹可供直接存取遠端資料庫的預存程序。 預存程序稱為 [sp\_execute\_remote](https://msdn.microsoft.com/library/mt703714)，可用來在遠端資料庫上執行遠端預存程序或 T-SQL 程式碼。 它需要以下參數： 

* 資料來源名稱 (nvarchar)：RDBMS 類型的外部資料來源名稱。 
* 查詢 (nvarchar)：要在遠端資料庫上執行的 T-SQL 查詢。 
* 參數宣告 (nvarchar) - 選用：含有查詢參數 (如 sp_executesql) 中所用參數的資料類型定義的字串。 
* 參數值清單 - 選用：以逗號分隔的參數值清單 (如 sp_executesql)。

sp\_execute\_remote 會使用叫用參數中提供的外部資料來源，在遠端資料庫上執行指定的 T-SQL 陳述式。 它會使用外部資料來源的認證來連線至遠端資料庫。  

範例： 

    EXEC sp_execute_remote
        N'MyExtSrc',
        N'select count(w_id) as foo from warehouse' 



## <a name="connectivity-for-tools"></a>工具的連線能力
您可以使用一般 SQL Server 連接字串，在啟用彈性查詢及定義外部資料表的 SQL DB 伺服器上，將 BI 和資料整合工具連接到資料庫。 請確定 SQL Server 可支援做為您的工具的資料來源。 然後參考彈性查詢資料庫和其外部資料表，就如同您會使用您的工具連接的任何其他 SQL Server 資料庫。 

## <a name="best-practices"></a>最佳作法
* 確保遠端資料庫已藉由在 Azure 服務的 SQL DB 防火牆組態中啟用其存取權，獲得彈性查詢端點資料庫的存取權。 也請確定外部資料來源定義中提供的認證可以成功登入遠端資料庫，而且具有存取遠端資料表的權限。  
* 彈性查詢最適合可在遠端資料庫上完成大部分運算的查詢。 使用可在遠端資料庫上評估的選擇性篩選述詞，或可在遠端資料庫上完全執行的聯結，通常可以獲得最佳查詢效能。 其他查詢模式可能需要從遠端資料庫載入大量的資料，而且執行效能可能會很差。 

## <a name="next-steps"></a>後續步驟

* 如需彈性查詢的概觀，請參閱[彈性查詢概觀](sql-database-elastic-query-overview.md)。
* 若要開始撰寫程式碼，請參閱 [開始使用跨資料庫查詢 (垂直資料分割)](sql-database-elastic-query-getting-started-vertical.md)。
* 如需水平資料分割 (分區化) 教學課程，請參閱[開始使用彈性查詢進行水平資料分割 (分區化)](sql-database-elastic-query-getting-started.md)。
* 如需水平資料分割之資料的語法和範例查詢，請參閱[查詢水平資料分割的資料](sql-database-elastic-query-horizontal-partitioning.md)
* 如需會在單一遠端 Azure SQL Database 或一組在水平資料分割配置中作為分區之資料庫上執行 Transact-SQL 陳述式的預存程序，請參閱 [sp\_execute \_remote](https://msdn.microsoft.com/library/mt703714)。


<!--Image references-->
[1]: ./media/sql-database-elastic-query-vertical-partitioning/verticalpartitioning.png


<!--anchors-->
