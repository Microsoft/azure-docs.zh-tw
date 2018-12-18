---
title: Azure SQL Database 記憶體內部技術 | Microsoft Docs
description: SQL Database 記憶體內部技術大幅提升交易和分析工作負載的效能。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: jodebrui
ms.author: jodebrui
ms.reviewer: ''
manager: craigg
ms.date: 07/16/2018
ms.openlocfilehash: 8fd430aa710c7e36133b40c7079b9d727774c68f
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166923"
---
# <a name="optimize-performance-by-using-in-memory-technologies-in-sql-database"></a>使用 SQL Database 中的記憶體內部技術將效能最佳化

您可以使用 Azure SQL Database 中的記憶體內部技術，以改善各種工作負載的效能︰交易式 (線上交易處理 (OLTP))、分析 (線上分析處理 (OLAP)) 及混合 (混合式交易/分析處理 (HTAP))。 因為可讓查詢和交易處理變得更有效率，記憶體內部技術也有助於降低成本。 您通常不需要升級資料庫的定價層，就能提升效能。 在某些情況下，您甚至可以降低定價層，而仍然發現記憶體內部技術可以提升效能。

以下是記憶體內部 OLTP 如何幫助大幅提升效能的兩個範例︰

- 由於使用記憶體內部 OLTP，[Quorum Business Solutions 能使其工作負載倍增，同時將 DTU 提高 70%](https://customers.microsoft.com/story/quorum-doubles-key-databases-workload-while-lowering-dtu-with-sql-database)。
    - DTU 表示「資料庫交易單位」，其中包含對資源耗用的測量。
- 下列影片以範例工作負載示範資源耗用量的重大改進︰[Azure SQL Database 影片中的記憶體內部 OLTP](https://channel9.msdn.com/Shows/Data-Exposed/In-Memory-OTLP-in-Azure-SQL-DB)。
    - 如需詳細資訊，請參閱部落格文章︰[Azure SQL Database 中的記憶體內 OLTP 部落格文章](https://azure.microsoft.com/blog/in-memory-oltp-in-azure-sql-database/) \(英文\)

記憶體內部技術適用於高階定價層的所有資料庫，包括高階彈性集區中的資料庫。

下列影片說明 Azure SQL Database 中的記憶體內部技術所可能提升的效能。 請記住，能提升多少效能永遠取決於許多因素，包括工作負載和資料的性質、資料庫的存取模式等等。

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Database-In-Memory-Technologies/player]
>
>

Azure SQL Database 擁有下列記憶體內部技術︰

- 「記憶體內部 OLTP」可增加交易並減少交易處理的延遲。 受益於記憶體內部 OLTP 的案例包括︰高輸送量的交易處理 (例如股票交易和網路遊戲)、從事件或 IoT 裝置擷取資料、快取、資料載入，以及暫存資料表和資料表變數等案例。
- 「叢集資料行存放區索引」可減少儲存體使用量 (最多 10 倍)，並提升報告和分析查詢的效能。 您可以將它用於資料超市中的事實資料表，在資料庫中容納更多資料並提升效能。 另外，您還可以將它用於操作資料庫中的歷史資料，則可封存並查詢多達 10 倍以上的資料。
- 「非叢集資料行存放區索引」 (適用於 HTAP) 可讓您透過直接查詢操作資料庫，即時深入了解您的業務，而不必執行昂貴的擷取、轉換和載入 (ETL) 程序並等候資料倉儲填入資料。 非叢集資料行存放區索引可極為快速地對 OLTP 資料庫執行分析查詢，同時降低對操作工作負載的影響。
- 您也可以結合記憶體最佳化資料表與資料行存放區索引。 結合它們可讓您執行非常快速的交易處理，並快速地*同時*對相同的資料執行分析查詢。

資料行存放區和記憶體內部 OLTP 已分別於 2012 年和 2014 年起納入成為 SQL Server 產品的一部分。 Azure SQL Database 和 SQL Server 共用相同的記憶體內部技術實作。 未來，這些技術的新功能會先在 Azure SQL Database 中推出，然後才在 SQL Server 中推出。

本文從各個面向說明 Azure SQL Database 特有的記憶體內部 OLTP 和資料行存放區索引，還包含範例：
- 您將了解這些技術對儲存體和資料大小限制的影響。
- 您將了解如何管理在不同定價層之間移動採用這些技術的資料庫。
- 您將看到兩個範例，其分別示範如何在 Azure SQL Database 中使用記憶體內部 OLTP 以及資料行存放區索引。

如需詳細資訊，請參閱下列資源。

有關這些技術的深入資訊：

- [記憶體內部 OLTP 概觀和使用案例](https://msdn.microsoft.com/library/mt774593.aspx) (包括客戶案例研究參考和入門資訊)
- [記憶體內部 OLTP 的文件](http://msdn.microsoft.com/library/dn133186.aspx)
- [資料行存放區索引指南](https://msdn.microsoft.com/library/gg492088.aspx)
- 混合式交易/分析處理 (HTAP)，也稱為[即時作業分析](https://msdn.microsoft.com/library/dn817827.aspx)

記憶體內部 OLTP 快速入門：[快速入門 1：獲得更快 T-SQL 效能的記憶體內部 OLTP 技術)](http://msdn.microsoft.com/library/mt694156.aspx) (另一篇協助您開始使用的文章)

技術的相關深入介紹影片︰

- [Azure SQL Database 中的記憶體內部 OLTP](https://channel9.msdn.com/Shows/Data-Exposed/In-Memory-OTLP-in-Azure-SQL-DB) (包含效能優點的示範，以及自行重現這些結果的步驟)
- [記憶體內部 OLTP 影片︰其功能、使用時機和使用方式](https://blogs.msdn.microsoft.com/sqlserverstorageengine/2016/10/03/in-memory-oltp-video-what-it-is-and-whenhow-to-use-it/)
- [資料行存放區索引：記憶體內部分析影片 (來源：Ignite 2016)](https://blogs.msdn.microsoft.com/sqlserverstorageengine/2016/10/04/columnstore-index-in-memory-analytics-i-e-columnstore-index-videos-from-ignite-2016/)

## <a name="storage-and-data-size"></a>儲存體和資料大小

### <a name="data-size-and-storage-cap-for-in-memory-oltp"></a>記憶體內部 OLTP 的資料大小和儲存體上限

記憶體內部 OLTP 包含記憶體最佳化資料表，以用來儲存使用者資料。 這些資料表必須可容納於記憶體。 因為您是直接在 SQL Database 服務中管理記憶體，我們有使用者資料配額的概念。 這個概念稱為「記憶體內部 OLAP 儲存體」。

每個受支援的單一資料庫定價層以及每個彈性集區定價層都包含一定數量的記憶體內部 OLTP 儲存體。 請參閱[以 DTU 為基礎的資源限制 - 單一資料庫](sql-database-dtu-resource-limits-single-databases.md)、[以 DTU 為基礎的資源限制 - 彈性集區](sql-database-dtu-resource-limits-elastic-pools.md)、[以虛擬核心為基礎的資源限制 - 單一資料庫](sql-database-vcore-resource-limits-single-databases.md)及[以虛擬核心為基礎的資源限制 - 彈性集區](sql-database-vcore-resource-limits-elastic-pools.md)。

下列項目計入記憶體內部 OLTP 儲存體容量上限︰

- 記憶體最佳化資料表和資料表變數中的作用中使用者資料列。 請注意，舊資料列版本不計入上限。
- 記憶體最佳化資料表上的索引。
- ALTER TABLE 作業的作業負荷。

如果您達到上限，則會收到超出配額錯誤，並再也無法插入或更新資料。 為避免此錯誤，您可以刪除資料或增加資料庫或集區的定價層。

如需監視記憶體內部 OLTP 儲存體使用量以及設定要在幾乎達到上限時發出警示的詳細資訊，請參閱[監視記憶體內部儲存體](sql-database-in-memory-oltp-monitoring.md)。

#### <a name="about-elastic-pools"></a>關於彈性集區

使用彈性集區時，集區中的所有資料庫會共用記憶體內部 OLTP 儲存體。 因此，一個資料庫中的使用量可能會影響其他資料庫。 對此，您可以採取以下兩個對策︰

- 將資料庫的 `Max-eDTU` 或 `MaxvCore` 設定為低於整個集區的 eDTU 或虛擬核心計數。 此最大值會將集區中任何資料庫的記憶體內部 OLTP 儲存體使用量上限，限制為對應到 eDTU 計數的大小。
- 設定大於 0 的 `Min-eDTU` 或 `MinvCore`。 此最小值可確保集區中的每個資料庫，都能擁有與所設定 `Min-eDTU` 或 `vCore` 相對應的可用記憶體內部 OLTP 儲存體數量。

### <a name="data-size-and-storage-for-columnstore-indexes"></a>資料行存放區索引的資料大小和儲存體

資料行存放區索引不需要納入記憶體中。 因此，索引大小的唯一上限是整體資料庫大小上限，相關說明請參閱 [DTU 形式的購買模型](sql-database-service-tiers-dtu.md)和[虛擬核心形式的購買模型](sql-database-service-tiers-vcore.md) 一文。

當您使用叢集資料行存放區索引時，基底表格儲存體會使用單資料行式壓縮。 壓縮可大幅降低使用者資料的儲存體使用量，這表示您可以在資料庫中容納更多資料。 若要再進一步壓縮，您可以使用[單資料行式封存壓縮](https://msdn.microsoft.com/library/cc280449.aspx#Using Columnstore and Columnstore Archive Compression)。 能達到多大壓縮量取決於資料性質，但 10 倍的壓縮並不罕見。

例如，如果您的資料庫大小上限是 1 TB，而且您使用資料行存放區達到 10 倍壓縮，您總共可以在資料庫中容納 10 TB 的使用者資料。

當您使用非叢集資料行存放區索引時，基底資料表仍然會以傳統資料列存放區格式儲存。 因此，節省的儲存空間沒有像使用叢集資料行存放區索引時那樣大。 不過，如果您以單一資料行存放區索引來取代部分的傳統非叢集索引，整體來說仍可節省資料表的儲存體使用量。

## <a name="moving-databases-that-use-in-memory-technologies-between-pricing-tiers"></a>在不同定價層之間移動使用記憶體內部技術的資料庫

當您升級到較高的定價層時 (例如從標準升級到進階)，絕不會有任何相容性問題或其他問題， 可用的功能和資源只會增加。

但是將定價層降級可能會對資料庫有負面影響。 當您的資料庫包含記憶體內部 OLTP 物件時，從進階降級到標準或基本會有特別顯著的影響。 降級之後將無法使用記憶體最佳化資料表 (即使它們依然可見)。 同樣的考量也適用於降低彈性集區的定價層時，或將使用記憶體內部技術的資料庫移至標準或基本的彈性集區時。

### <a name="in-memory-oltp"></a>記憶體內部 OLTP

降級至基本/標準：標準層或基本層的資料庫不支援記憶體內部 OLTP。 此外，也不可以將具有任何記憶體內部 OLTP 物件的資料庫移至標準層或基本層。

您可以透過程式設計的方式，來了解給定資料庫是否支援記憶體內部 OLTP。 您可以執行下列 Transact-SQL 查詢︰

```
SELECT DatabasePropertyEx(DB_NAME(), 'IsXTPSupported');
```

如果查詢傳回 **1**，則此資料庫支援記憶體內部 OLTP。

在將資料庫降級至標準層或基本層時，請移除所有記憶體最佳化資料表和資料表類型，以及所有原生編譯的 T-SQL 模組。 下列查詢會識別在資料庫可以降級至標準/基本之前，需要移除的所有物件：

```
SELECT * FROM sys.tables WHERE is_memory_optimized=1
SELECT * FROM sys.table_types WHERE is_memory_optimized=1
SELECT * FROM sys.sql_modules WHERE uses_native_compilation=1
```

降級至較低的進階層：記憶體最佳化資料表中的資料必須能夠容納於與資料庫定價層相關聯 (或彈性集區中可用) 的記憶體內部 OLTP 儲存體。 如果您嘗試降低定價層，或將資料庫移入沒有足夠之可用記憶體內部 OLTP 儲存體的集區內，則作業會失敗。

### <a name="columnstore-indexes"></a>資料行存放區索引

*降級為基本或標準*：只有在進階定價層和標準層、S3 及更新版本才支援資料行存放區索引，基本層則不支援。 當您將資料庫降級至不支援的層級時，資料行存放區索引將變成無法使用。 系統會維持您的資料行存放區索引，但它不會再利用索引。 如果您之後再升級為支援的層級，系統會立即重新利用您的資料行存放區索引。

如果您有「叢集」資料行存放區索引，則降級之後整個資料表會變成無法使用。 因此我們建議您在將資料庫降級至不支援的層級之前，先捨棄所有「叢集」資料行存放區索引。

*降級至支援較低的層級*：只要整個資料庫大小小於目標定價層的最大資料庫大小，或小於彈性集區中可用儲存體的大小，此降級就會成功。 資料行存放區索引不會產生任何特定影響。


<a id="install_oltp_manuallink" name="install_oltp_manuallink"></a>

&nbsp;

## <a name="1-install-the-in-memory-oltp-sample"></a>1.安裝記憶體內部 OLTP 範例

在 [Azure 入口網站](https://portal.azure.com/)中按幾下滑鼠，即可建立 AdventureWorksLT 範例資料庫。 然後，本節中的步驟會說明如何使用記憶體內部 OLTP 物件擴充 AdventureWorksLT 資料庫，並示範效能優點。

如需更簡單、但更美觀的記憶體內部 OLTP 效能示範，請參閱︰

- 版本︰[in-memory-oltp-demo-v1.0](https://github.com/Microsoft/sql-server-samples/releases/tag/in-memory-oltp-demo-v1.0)
- 原始程式碼︰[in-memory-oltp-demo-source-code](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/in-memory/ticket-reservations)

#### <a name="installation-steps"></a>安裝步驟

1. 在 [Azure 入口網站](https://portal.azure.com/)中，於伺服器上建立進階或業務關鍵資料庫。 將 [來源]  設定為 AdventureWorksLT 範例資料庫。 如需詳細指示，請參閱[建立您的第一個 Azure SQL Database](sql-database-get-started-portal.md)。

2. 使用 SQL Server Management Studio [(SSMS.exe)](http://msdn.microsoft.com/library/mt238290.aspx)連接到資料庫。

3. 將 [In-Memory OLTP Transact-SQL 指令碼](https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/features/in-memory/t-sql-scripts/sql_in-memory_oltp_sample.sql) 複製到剪貼簿。 T-SQL 指令碼會在步驟 1 建立的 AdventureWorksLT 範例資料庫中建立所需的 In-Memory 物件。

4. 將 T-SQL 指令碼貼到 SSMS 中，然後執行該指令碼。 `MEMORY_OPTIMIZED = ON` 子句 CREATE TABLE 陳述式很重要。 例如︰


```
CREATE TABLE [SalesLT].[SalesOrderHeader_inmem](
    [SalesOrderID] int IDENTITY NOT NULL PRIMARY KEY NONCLUSTERED ...,
    ...
) WITH (MEMORY_OPTIMIZED = ON);
```


#### <a name="error-40536"></a>錯誤 40536


如果您在執行 T-SQL 指令碼時收到錯誤 40536，請執行下列 T-SQL 指令碼來確認資料庫是否支援 In-Memory：


```
SELECT DatabasePropertyEx(DB_Name(), 'IsXTPSupported');
```


結果為 **0** 表示不支援 In-Memory，而 **1** 表示提供支援。 若要診斷問題，請確定資料庫位於「進階」服務層。


#### <a name="about-the-created-memory-optimized-items"></a>關於已建立的記憶體最佳化項目

**資料表**：此範例包含下列記憶體最佳化資料表：

- SalesLT.Product_inmem
- SalesLT.SalesOrderHeader_inmem
- SalesLT.SalesOrderDetail_inmem
- Demo.DemoSalesOrderHeaderSeed
- Demo.DemoSalesOrderDetailSeed


您可以透過 SSMS 中的 [物件總管]，檢查記憶體最佳化資料表。 以滑鼠右鍵按一下 [資料表] > [篩選] > [篩選設定] > [記憶體已最佳化嗎]。 值等於 1。


或者您可以查詢目錄檢視，例如：


```
SELECT is_memory_optimized, name, type_desc, durability_desc
    FROM sys.tables
    WHERE is_memory_optimized = 1;
```


**原生編譯的預存程序**：您可以透過目錄檢視查詢來檢查 SalesLT.usp_InsertSalesOrder_inmem：


```
SELECT uses_native_compilation, OBJECT_NAME(object_id), definition
    FROM sys.sql_modules
    WHERE uses_native_compilation = 1;
```


&nbsp;

### <a name="run-the-sample-oltp-workload"></a>執行範例 OLTP 工作負載

下列兩個預存程序  的唯一差別在於第一個程序會使用記憶體最佳化資料表版本，而第二個程序會使用一般磁碟資料表：

- SalesLT **.** usp_InsertSalesOrder **_inmem**
- SalesLT **.** usp_InsertSalesOrder **_ondisk**


在本節中，您會了解如何使用便利的 **ostress.exe** 公用程式，在壓力層級執行兩個預存程序。 您可以比較完成兩個壓力回合所需的時間。


當您執行 ostress.exe 時，建議您將指定的參數值傳遞至下列兩者：

- 使用 -n100 來執行大量的並行連線。
- 使用 -r500 讓每個連線執行幾百次迴圈。


不過，您可能想從較小的值 (-n10 和 -r50) 開始，以確保一切都運作正常。


### <a name="script-for-ostressexe"></a>Ostress.exe 的指令碼


本節顯示 ostress.exe 命令列中內嵌的 T-SQL 指令碼。 此指令碼會使用您稍早安裝的 T-SQL 指令碼所建立的項目。


下列指令碼會在下列記憶體最佳化資料表 中插入有 5 個細項的範例銷售訂單：

- SalesLT.SalesOrderHeader_inmem
- SalesLT.SalesOrderDetail_inmem


```
DECLARE
    @i int = 0,
    @od SalesLT.SalesOrderDetailType_inmem,
    @SalesOrderID int,
    @DueDate datetime2 = sysdatetime(),
    @CustomerID int = rand() * 8000,
    @BillToAddressID int = rand() * 10000,
    @ShipToAddressID int = rand() * 10000;

INSERT INTO @od
    SELECT OrderQty, ProductID
    FROM Demo.DemoSalesOrderDetailSeed
    WHERE OrderID= cast((rand()*60) as int);

WHILE (@i < 20)
begin;
    EXECUTE SalesLT.usp_InsertSalesOrder_inmem @SalesOrderID OUTPUT,
        @DueDate, @CustomerID, @BillToAddressID, @ShipToAddressID, @od;
    SET @i = @i + 1;
end
```


若要針對 ostress.exe 製作上述 T-SQL 指令碼的 *_ondisk* 版本，請以 *_ondisk* 取代兩個出現的 *_inmem* 子字串。 這類取代會影響資料表和預存程序的名稱。


### <a name="install-rml-utilities-and-ostress"></a>安裝 RML 公用程式和 ostress


您最好規劃在 Azure 虛擬機器 (VM) 上執行 ostress.exe。 您會在 AdventureWorksLT 資料庫所在的相同 Azure 地理區域中建立 [Azure VM](https://azure.microsoft.com/documentation/services/virtual-machines/)。 但是您可以改在您的膝上型電腦上執行 ostress.exe。


在 VM 上或你選擇的任何主機上，安裝 Replay Markup Language (RML) 公用程式。 這些公用程式包括 ostress.exe。

如需詳細資訊，請參閱
- [記憶體內部 OLTP 的範例資料庫](http://msdn.microsoft.com/library/mt465764.aspx)中的 ostress.exe 討論。
- [記憶體內部 OLTP 的範例資料庫](http://msdn.microsoft.com/library/mt465764.aspx)。
- [安裝 ostress.exe 的部落格](http://blogs.msdn.com/b/psssql/archive/2013/10/29/cumulative-update-2-to-the-rml-utilities-for-microsoft-sql-server-released.aspx)。



<!--
dn511655.aspx is for SQL 2014,
[Extensions to AdventureWorks to Demonstrate In-Memory OLTP]
(http://msdn.microsoft.com/library/dn511655&#x28;v=sql.120&#x29;.aspx)

whereas for SQL 2016+
[Sample Database for In-Memory OLTP]
(http://msdn.microsoft.com/library/mt465764.aspx)
-->



### <a name="run-the-inmem-stress-workload-first"></a>先執行 _inmem 壓力工作負載


您可以使用 RML 命令提示字元  視窗來執行 ostress.exe 命令列。 命令列參數會將 ostress 導向至：

- 同時執行 100 個連線 (-n100)。
- 每個連線會執行 T-SQL 指令碼 50 次 (-r50)。


```
ostress.exe -n100 -r50 -S<servername>.database.windows.net -U<login> -P<password> -d<database> -q -Q"DECLARE @i int = 0, @od SalesLT.SalesOrderDetailType_inmem, @SalesOrderID int, @DueDate datetime2 = sysdatetime(), @CustomerID int = rand() * 8000, @BillToAddressID int = rand() * 10000, @ShipToAddressID int = rand()* 10000; INSERT INTO @od SELECT OrderQty, ProductID FROM Demo.DemoSalesOrderDetailSeed WHERE OrderID= cast((rand()*60) as int); WHILE (@i < 20) begin; EXECUTE SalesLT.usp_InsertSalesOrder_inmem @SalesOrderID OUTPUT, @DueDate, @CustomerID, @BillToAddressID, @ShipToAddressID, @od; set @i += 1; end"
```


若要執行上述的 ostress.exe 命令列：


1. 在 SSMS 中執行下列命令來重設資料庫資料內容，以刪除先前執行插入的所有資料：

    ``` tsql
    EXECUTE Demo.usp_DemoReset;
    ```

2. 將上述 ostress.exe 命令列的文字複製到剪貼簿。

3. 以正確的實數值取代參數 -S -U -P -d 的 `<placeholders>` 。

4. 在 [RML 命令] 視窗中執行已編輯的命令列。


#### <a name="result-is-a-duration"></a>結果是持續時間


當 ostress.exe 完成時，它會在 RML 命令視窗中寫入執行持續時間做為輸出的最後一行。 例如，較短的測試回合持續大約 1.5 分鐘：

`11/12/15 00:35:00.873 [0x000030A8] OSTRESS exiting normally, elapsed time: 00:01:31.867`


#### <a name="reset-edit-for-ondisk-then-rerun"></a>重設，針對 _ondisk 編輯，然後重新執行


在獲得 _inmem 執行的結果之後，請針對 _ondisk 執行回合執行下列步驟：


1. 在 SSMS 中執行下列命令來重設資料庫，以刪除先前執行插入的所有資料：
```
EXECUTE Demo.usp_DemoReset;
```

2. 編輯 ostress.exe 命令列，以 *_ondisk* 取代所有的 *_inmem*。

3. 第二次重新執行 ostress.exe，並擷取持續時間結果。

4. 再次重設資料庫 (以負責刪除可能的大量測試資料)。


#### <a name="expected-comparison-results"></a>預期的比較結果

就這個過度簡單的工作負載而言，我們的「記憶體內部」測試顯示當 ostress 是在與資料庫相同 Azure 區域中的 Azure VM 上執行時，可獲得「九倍」的效能改善。

<a id="install_analytics_manuallink" name="install_analytics_manuallink"></a>

&nbsp;

## <a name="2-install-the-in-memory-analytics-sample"></a>2.安裝記憶體內部分析範例


在本節中，您將比較使用資料行存放區索引與使用傳統 B 型樹狀結構索引時的 IO 和統計資料結果。


針對 OLTP 工作負載的即時分析，通常最好使用非叢集式資料行存放區索引。 如需詳細資訊，請參閱[已描述的資料行存放區索引](http://msdn.microsoft.com/library/gg492088.aspx)。



### <a name="prepare-the-columnstore-analytics-test"></a>準備資料行存放區分析測試


1. 使用 Azure 入口網站，從範例建立全新的 AdventureWorksLT 資料庫。
 - 使用相同的名稱。
 - 選擇任何進階服務層。

2. 將 [sql_in-memory_analytics_sample](https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/features/in-memory/t-sql-scripts/sql_in-memory_analytics_sample.sql) 複製到剪貼簿。
 - T-SQL 指令碼會在步驟 1 建立的 AdventureWorksLT 範例資料庫中建立所需的 In-Memory 物件。
 - 此指令碼會建立維度資料表和兩個事實資料表。 每個事實資料表會填入 350 萬個資料列。
 - 此指令碼可能需要 15 分鐘才能完成。

3. 將 T-SQL 指令碼貼到 SSMS 中，然後執行該指令碼。 **CREATE INDEX** 陳述式中的 **COLUMNSTORE** 關鍵字很重要，如下所示：<br/>`CREATE NONCLUSTERED COLUMNSTORE INDEX ...;`

4. 將 AdventureWorksLT 設為相容性層級 130：<br/>`ALTER DATABASE AdventureworksLT SET compatibility_level = 130;`

    層級 130 並未與 In-Memory 功能直接相關。 但層級 130 通常會提供比層級 120 更快的查詢效能。


#### <a name="key-tables-and-columnstore-indexes"></a>重要資料表和資料行存放區索引


- dbo.FactResellerSalesXL_CCI 是具有叢集資料行存放區索引的資料表，此資料表已在「資料」層級進一步壓縮。

- dbo.FactResellerSalesXL_PageCompressed 是具有對等一般叢集式索引的資料表，此資料表只在「頁面」層級壓縮。


#### <a name="key-queries-to-compare-the-columnstore-index"></a>用來比較資料行存放區索引的重要查詢


有[您可以執行的數種 T-SQL 查詢類型](https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/features/in-memory/t-sql-scripts/clustered_columnstore_sample_queries.sql)可用來查看效能改進。 在步驟 2 的 T-SQL 指令碼中，請注意這一組查詢。 其中的不同之處只有一行：


- `FROM FactResellerSalesXL_PageCompressed a`
- `FROM FactResellerSalesXL_CCI a`


叢集資料行存放區索引位於 FactResellerSalesXL\_CCI 資料表上。

下列 T-SQL 指令碼摘錄列出每個資料表查詢的 IO 和 TIME 統計資料。


```
/*********************************************************************
Step 2 -- Overview
-- Page Compressed BTree table v/s Columnstore table performance differences
-- Enable actual Query Plan in order to see Plan differences when Executing
*/
-- Ensure Database is in 130 compatibility mode
ALTER DATABASE AdventureworksLT SET compatibility_level = 130
GO

-- Execute a typical query that joins the Fact Table with dimension tables
-- Note this query will run on the Page Compressed table, Note down the time
SET STATISTICS IO ON
SET STATISTICS TIME ON
GO

SELECT c.Year
    ,e.ProductCategoryKey
    ,FirstName + ' ' + LastName AS FullName
    ,count(SalesOrderNumber) AS NumSales
    ,sum(SalesAmount) AS TotalSalesAmt
    ,Avg(SalesAmount) AS AvgSalesAmt
    ,count(DISTINCT SalesOrderNumber) AS NumOrders
    ,count(DISTINCT a.CustomerKey) AS CountCustomers
FROM FactResellerSalesXL_PageCompressed a
INNER JOIN DimProduct b ON b.ProductKey = a.ProductKey
INNER JOIN DimCustomer d ON d.CustomerKey = a.CustomerKey
Inner JOIN DimProductSubCategory e on e.ProductSubcategoryKey = b.ProductSubcategoryKey
INNER JOIN DimDate c ON c.DateKey = a.OrderDateKey
GROUP BY e.ProductCategoryKey,c.Year,d.CustomerKey,d.FirstName,d.LastName
GO
SET STATISTICS IO OFF
SET STATISTICS TIME OFF
GO


-- This is the same Prior query on a table with a clustered columnstore index CCI
-- The comparison numbers are even more dramatic the larger the table is (this is an 11 million row table only)
SET STATISTICS IO ON
SET STATISTICS TIME ON
GO
SELECT c.Year
    ,e.ProductCategoryKey
    ,FirstName + ' ' + LastName AS FullName
    ,count(SalesOrderNumber) AS NumSales
    ,sum(SalesAmount) AS TotalSalesAmt
    ,Avg(SalesAmount) AS AvgSalesAmt
    ,count(DISTINCT SalesOrderNumber) AS NumOrders
    ,count(DISTINCT a.CustomerKey) AS CountCustomers
FROM FactResellerSalesXL_CCI a
INNER JOIN DimProduct b ON b.ProductKey = a.ProductKey
INNER JOIN DimCustomer d ON d.CustomerKey = a.CustomerKey
Inner JOIN DimProductSubCategory e on e.ProductSubcategoryKey = b.ProductSubcategoryKey
INNER JOIN DimDate c ON c.DateKey = a.OrderDateKey
GROUP BY e.ProductCategoryKey,c.Year,d.CustomerKey,d.FirstName,d.LastName
GO

SET STATISTICS IO OFF
SET STATISTICS TIME OFF
GO
```

在定價層為 P2 的資料庫中，相較於傳統索引，使用叢集資料行存放區索引進行此查詢預期可提升大約九倍的效能。 使用 P15 時，您可以預期使用資料行存放區索引大約可提升 57 倍的效能。



## <a name="next-steps"></a>後續步驟

- [快速入門 1：可讓 Transact-SQL 擁有更快效能的記憶體內部 OLTP 技術](http://msdn.microsoft.com/library/mt694156.aspx)

- [在現有的 Azure SQL 應用程式中使用記憶體內部 OLTP](sql-database-in-memory-oltp-migration.md)

- 針對記憶體內部 OLAP [監視記憶體內部 OLTP 儲存體](sql-database-in-memory-oltp-monitoring.md)


## <a name="additional-resources"></a>其他資源

#### <a name="deeper-information"></a>更深入的資訊

- [了解仲裁如何透過 SQL Database 中的記憶體內部 OLTP，讓重要資料庫的工作負載加倍，同時降低 70% 的 DTU](https://customers.microsoft.com/story/quorum-doubles-key-databases-workload-while-lowering-dtu-with-sql-database)

- [Azure SQL Database 中的記憶體內部 OLTP 部落格文章](https://azure.microsoft.com/blog/in-memory-oltp-in-azure-sql-database/)

- [了解記憶體內部 OLTP](http://msdn.microsoft.com/library/dn133186.aspx)

- [了解資料行存放區索引](https://msdn.microsoft.com/library/gg492088.aspx)

- [了解即時作業分析](http://msdn.microsoft.com/library/dn817827.aspx)

- 請參閱[一般工作負載模式和移轉考量](http://msdn.microsoft.com/library/dn673538.aspx) (其中描述記憶體內部 OLTP 經常提供顯著效能改善的工作負載模式)

#### <a name="application-design"></a>應用程式設計

- [In-Memory OLTP (In-Memory Optimization)](http://msdn.microsoft.com/library/dn133186.aspx)

- [在現有的 Azure SQL 應用程式中使用記憶體內部 OLTP](sql-database-in-memory-oltp-migration.md)

#### <a name="tools"></a>工具

- [Azure 入口網站](https://portal.azure.com/)

- [SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/library/mt238290.aspx)

- [SQL Server Data Tools (SSDT)](http://msdn.microsoft.com/library/mt204009.aspx)
