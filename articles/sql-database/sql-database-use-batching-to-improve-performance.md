---
title: 如何使用批次處理來改善 Azure SQL Database 應用程式效能
description: 本主題提供證據，表明批次處理資料庫作業可大幅改善 Azure SQL Database 應用程式的速度和延展性。 雖然這些批次處理技術適用於任何 SQL Server 資料庫，但本文的重點在於 Azure。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: genemi
manager: craigg
ms.date: 09/20/2018
ms.openlocfilehash: 21dc28658f7f6f31bc7536df739a70238a3bcb8f
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47160803"
---
# <a name="how-to-use-batching-to-improve-sql-database-application-performance"></a>如何使用批次處理來改善 SQL Database 應用程式效能
批次處理 Azure SQL Database 的作業可大幅改善應用程式的效能和延展性。 為了瞭解優點，本文的第一個部分涵蓋一些範例測試結果，其中比較 SQL Database 的循序和批次要求。 本文其餘部分說明技術、案例和考量因素，協助您在 Azure 應用程式中順利使用批次處理。

## <a name="why-is-batching-important-for-sql-database"></a>為什麼批次處理對 SQL Database 很重要？
眾所周知，批次處理遠端服務的呼叫是一項可提升效能和延展性的策略。 與遠端服務進行任何互動都有固定的成本，例如序列化、網路傳輸和還原序列化。 將許多個別交易包裝成單一批次可將這些成本降到最低。

本文中，我們將探討 SQL Database 的各種批次處理策略和案例。 雖然這些策略對於使用 SQL Server 的內部部署應用程式也很重要，但基於一些理由，必須特別討論在 SQL Database 中使用批次處理：

* 存取 SQL Database 時網路延遲可能很嚴重，特別是從相同的 Microsoft Azure 資料中心外部存取 SQL Database。
* SQL Database 的多租用戶特性表示資料存取層的效率與資料庫的整體延展性有密切關係。 SQL Database 必須防止任何單一租用戶/使用者獨佔資料庫資源，而損害其他租用戶的權益。 為了避免使用量超出預先定義的配額，SQL Database 可以減少輸送量，或以節流例外狀況做出回應。 效率 (例如批次處理) 可讓您在達到這些限制之前，在 SQL Database 上完成更多工作。 
* 在使用多個資料庫 (分區化) 的架構下，批次處理也很有效益。 就整體延展性而言，與每個資料庫單位的互動效率仍然是關鍵因素。 

使用 SQL Database 的優點之一是您不必管理裝載著資料庫的伺服器。 不過，這種受控基礎結構也表示您必須以不同角度來思考資料庫最佳化。 您不必再設法改善資料庫硬體或網路基礎結構。 Microsoft Azure 會控制那些環境。 您可以控制的主要方面是應用程式與 SQL Database 之間的互動方式。 批次處理屬於這些最佳化作法之一。 

本文的第一個部分針對使用 SQL Database 的 .NET 應用程式，探討各種批次處理技術。 最後兩節涵蓋批次處理方針和案例。

## <a name="batching-strategies"></a>批次處理策略
### <a name="note-about-timing-results-in-this-article"></a>有關本文中計時結果的注意事項
> [!NOTE]
> 結果並不是基準，主要是示範 **相對效能**。 計時至少根據 10 個測試回合的平均值。 作業插入至空的資料表。 這些測試是在 V12 以前的版本中測量，不見得符合您在 V12 資料庫中使用新的 [DTU 服務層](sql-database-service-tiers-dtu.md)或[虛擬核心服務層](sql-database-service-tiers-vcore.md)時可能遇過的輸送量。 批次處理技術的相對優點應該類似。
> 
> 

### <a name="transactions"></a>交易
從討論交易來開始評論批次處理可能有點奇怪。 但使用用戶端交易也隱約有可改善效能的伺服器端批次處理效果。 只需要幾行程式碼就能新增交易，因此可快速改善循序作業的效能。

請考慮下列 C# 程式碼，其中對一個簡易資料表執行一連串插入和更新作業。

    List<string> dbOperations = new List<string>();
    dbOperations.Add("update MyTable set mytext = 'updated text' where id = 1");
    dbOperations.Add("update MyTable set mytext = 'updated text' where id = 2");
    dbOperations.Add("update MyTable set mytext = 'updated text' where id = 3");
    dbOperations.Add("insert MyTable values ('new value',1)");
    dbOperations.Add("insert MyTable values ('new value',2)");
    dbOperations.Add("insert MyTable values ('new value',3)");

下列 ADO.NET 程式碼會循序執行這些作業。

    using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
    {
        conn.Open();

        foreach(string commandString in dbOperations)
        {
            SqlCommand cmd = new SqlCommand(commandString, conn);
            cmd.ExecuteNonQuery();                   
        }
    }

若要將這段程式碼最佳化，最佳方式是針對這些呼叫，實作某種形式的用戶端批次處理。 還有更簡單的作法，只要將呼叫序列包裝在交易中，就能提升這段程式碼的效能。 以下是使用交易的相同程式碼。

    using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
    {
        conn.Open();
        SqlTransaction transaction = conn.BeginTransaction();

        foreach (string commandString in dbOperations)
        {
            SqlCommand cmd = new SqlCommand(commandString, conn, transaction);
            cmd.ExecuteNonQuery();
        }

        transaction.Commit();
    }

這兩個範例實際上都使用交易。 在第一個範例中，每個個別的呼叫就是隱含的交易。 在第二個範例中，明確的交易包裝所有的呼叫。 如 [預先寫入交易記錄](https://msdn.microsoft.com/library/ms186259.aspx)所述，交易認可時，記錄檔記錄會排清到磁碟。 因此，交易中包含越多呼叫，就越可能延遲到認可交易時，才會寫入交易記錄檔。 事實上，您是對寫入伺服器交易記錄檔的動作啟用批次處理。

下表顯示一些臨機操作測試結果。 這些測試分別以有交易和無交易，執行相同的循序插入。 為了進一步觀察，第一組測試是從遠端的膝上型電腦連到 Microsoft Azure 中的資料庫執行。 第二組測試是從位在相同 Microsoft Azure 資料中心 (美國西部) 內的雲端服務和資料庫執行。 下表分別以有交易和無交易，顯示循序插入的持續時間 (以毫秒為單位)。

**內部部署至 Azure**：

| 作業 | 無交易 (毫秒) | 交易 (毫秒) |
| --- | --- | --- |
| 1 |130 |402 |
| 10 |1208 |1226 |
| 100 |12662 |10395 |
| 1000 |128852 |102917 |

**Azure 至 Azure (相同資料中心)**：

| 作業 | 無交易 (毫秒) | 交易 (毫秒) |
| --- | --- | --- |
| 1 |21 |26 |
| 10 |220 |56 |
| 100 |2145 |341 |
| 1000 |21479 |2756 |

> [!NOTE]
> 結果並不是基準。 請參閱[有關此文章中計時結果的注意事項](#note-about-timing-results-in-this-article)。

根據先前的測試結果，將單一作業包裝在交易中確實會降低效能。 但隨著您增加單一交易內的作業數目，效能改善會變得越明顯。 當所有作業都在 Microsoft Azure 資料中心內發生時，效能差異也會更明顯。 從 Microsoft Azure 資料中心外部使用 SQL Database 而增加的延遲，讓使用交易所提升的效能黯然失色。

雖然使用交易可以提高效能，但要持續 [尋找交易和連接的最佳作法](https://msdn.microsoft.com/library/ms187484.aspx)。 交易儘可能越短越好，並於工作完成後立即關閉資料庫連接。 使用上述範例中的陳述式可確保後續的程式碼區塊完成時關閉連接。

上述範例示範只要使用兩行程式碼，就能將本機交易新增至任何 ADO.NET 程式碼。 對於執行循序插入、更新和刪除作業的程式碼，交易可以快速改善程式碼的效能。 不過，若要達到最快效能，請考慮進一步變更程式碼來利用用戶端批次處理，例如資料表值參數。

如需 ADO.NET 中的交易的詳細資訊，請參閱 [ADO.NET 中的本機交易](https://docs.microsoft.com/dotnet/framework/data/adonet/local-transactions)。

### <a name="table-valued-parameters"></a>資料表值參數
資料表值參數支援使用者定義資料表類型做為 Transact-SQL 陳述式、預存程序和函數中的參數。 此用戶端批次處理技術可讓您在資料表值參數內傳送很多列的資料。 若要使用資料表值參數，請先定義資料表類型。 下列 Transact-SQL 陳述式會建立名為 **MyTableType**的資料表類型。

    CREATE TYPE MyTableType AS TABLE 
    ( mytext TEXT,
      num INT );


在程式碼中，您使用與此資料表類型完全相同的名稱和類型建立 **DataTable** 。 將這個 **DataTable** 傳入文字查詢或預存程序呼叫中的參數。 下列範例示範這項技巧：

    using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
    {
        connection.Open();

        DataTable table = new DataTable();
        // Add columns and rows. The following is a simple example.
        table.Columns.Add("mytext", typeof(string));
        table.Columns.Add("num", typeof(int));    
        for (var i = 0; i < 10; i++)
        {
            table.Rows.Add(DateTime.Now.ToString(), DateTime.Now.Millisecond);
        }

        SqlCommand cmd = new SqlCommand(
            "INSERT INTO MyTable(mytext, num) SELECT mytext, num FROM @TestTvp",
            connection);

        cmd.Parameters.Add(
            new SqlParameter()
            {
                ParameterName = "@TestTvp",
                SqlDbType = SqlDbType.Structured,
                TypeName = "MyTableType",
                Value = table,
            });

        cmd.ExecuteNonQuery();
    }

在上述範例中，**SqlCommand** 物件從資料表值參數 **@TestTvp** 插入資料列。 先前建立的 **DataTable** 物件透過 **SqlCommand.Parameters.Add** 方法指派給此參數。 在一個呼叫中批次處理插入的效能明顯高於循序插入。

若要進一步改善上述範例，請使用預存程序代替文字式命令。 下列 Transact-SQL 命令會建立一個接受 **SimpleTestTableType** 資料表值參數的預存程序。

    CREATE PROCEDURE [dbo].[sp_InsertRows] 
    @TestTvp as MyTableType READONLY
    AS
    BEGIN
    INSERT INTO MyTable(mytext, num) 
    SELECT mytext, num FROM @TestTvp
    END
    GO

然後將上述程式碼範例中的 **SqlCommand** 物件宣告變更如下。

    SqlCommand cmd = new SqlCommand("sp_InsertRows", connection);
    cmd.CommandType = CommandType.StoredProcedure;

在大部分情況下，資料表值參數的效能同於或高於其他批次處理技術。 資料表值參數通常較適合，因為比其他選項更有彈性。 例如，其他技術 (例如 SQL 大量複製) 只允許插入新資料列。 但使用資料表值參數時，您可以在預存程序中使用邏輯，判斷哪些資料列是更新和哪些是插入。 也可以修改資料表類型來包含「作業」資料行，指出是否應該插入、更新或刪除指定的資料列。

下表顯示使用資料表值參數的臨機操作測試結果 (以毫秒為單位)。

| 作業 | 內部部署至 Azure (亳秒) | Azure 相同資料中心 (毫秒) |
| --- | --- | --- |
| 1 |124 |32 |
| 10 |131 |25 |
| 100 |338 |51 |
| 1000 |2615 |382 |
| 10000 |23830 |3586 |

> [!NOTE]
> 結果並不是基準。 請參閱[有關此文章中計時結果的注意事項](#note-about-timing-results-in-this-article)。
> 
> 

批次處理所提升的效能立即而明顯。 在先前的循序測試中，1000 個作業在資料中心外花費 129 秒，而從資料中心內花費 21 秒。 但使用資料表值參數時，1000 個作業在資料中心外只花費 2.6 秒，而在資料中心內只花費 0.4 秒。

如需資料表值參數的詳細資訊，請參閱 [資料表值參數](https://msdn.microsoft.com/library/bb510489.aspx)。

### <a name="sql-bulk-copy"></a>SQL 大量複製
SQL 大量複製是另一種將大量資料插入至目標資料庫的方式。 .NET 應用程式可以使用 **SqlBulkCopy** 類別執行大量插入作業。 **SqlBulkCopy** 在功能上類似於命令列工具 **Bcp.exe**，或 Transact-SQL 陳述式 **BULK INSERT**。 下列程式碼範例顯示如何將來源 **DataTable**資料表中的資料列，大量複製到 SQL Server 中的目的地資料表 MyTable。

    using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
    {
        connection.Open();

        using (SqlBulkCopy bulkCopy = new SqlBulkCopy(connection))
        {
            bulkCopy.DestinationTableName = "MyTable";
            bulkCopy.ColumnMappings.Add("mytext", "mytext");
            bulkCopy.ColumnMappings.Add("num", "num");
            bulkCopy.WriteToServer(table);
        }
    }

在某些情況下，大量複製比資料表值參數更適合。 請參閱[資料表值參數](https://msdn.microsoft.com/library/bb510489.aspx)一文中的資料表值參數與 BULK INSERT 作業的比較表。

下列臨機操作測試結果顯示 **SqlBulkCopy** 的批次處理效能 (以毫秒為單位)。

| 作業 | 內部部署至 Azure (亳秒) | Azure 相同資料中心 (毫秒) |
| --- | --- | --- |
| 1 |433 |57 |
| 10 |441 |32 |
| 100 |636 |53 |
| 1000 |2535 |341 |
| 10000 |21605 |2737 |

> [!NOTE]
> 結果並不是基準。 請參閱[有關此文章中計時結果的注意事項](#note-about-timing-results-in-this-article)。
> 
> 

批次較小時，使用資料表值參數的效能勝過 **SqlBulkCopy** 類別。 不過，在測試 1,000 和 10,000 個資料列時，**SqlBulkCopy** 的執行速度比資料表值參數快 12-31%。 就像資料表值參數一樣， **SqlBulkCopy** 是批次插入的理想選擇，尤其與非批次作業的效能相比較。

如需 ADO.NET 中的大量複製的詳細資訊，請參閱 [SQL Server 中的大量複製作業](https://msdn.microsoft.com/library/7ek5da1a.aspx)。

### <a name="multiple-row-parameterized-insert-statements"></a>多資料列參數化 INSERT 陳述式
對於小型批次，另一種作法是建構大型參數化 INSERT 陳述式來插入多個資料列。 下列程式碼範例示範這項技巧。

    using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
    {
        connection.Open();

        string insertCommand = "INSERT INTO [MyTable] ( mytext, num ) " +
            "VALUES (@p1, @p2), (@p3, @p4), (@p5, @p6), (@p7, @p8), (@p9, @p10)";

        SqlCommand cmd = new SqlCommand(insertCommand, connection);

        for (int i = 1; i <= 10; i += 2)
        {
            cmd.Parameters.Add(new SqlParameter("@p" + i.ToString(), "test"));
            cmd.Parameters.Add(new SqlParameter("@p" + (i+1).ToString(), i));
        }

        cmd.ExecuteNonQuery();
    }


此範例主要是示範基本概念。 在更實際的案例中會循環查看需要的實體，以同時建構查詢字串和命令參數。 總計以 2100 個查詢參數為限，此方法可處理的資料列總數受限於此。

下列臨機操作測試結果顯示這種插入陳述式的效能 (以毫秒為單位)。

| 作業 | 資料表值參數 (毫秒) | 單一陳述式 INSERT (毫秒) |
| --- | --- | --- |
| 1 |32 |20 |
| 10 |30 |25 |
| 100 |33 |51 |

> [!NOTE]
> 結果並不是基準。 請參閱[有關此文章中計時結果的注意事項](#note-about-timing-results-in-this-article)。
> 
> 

如果批次少於 100 的資料列，這種方法會稍微快一些。 雖然改善幅度很小，但在您的特殊應用案例中，這種技術可能是另一種適合的選項。

### <a name="dataadapter"></a>DataAdapter
**DataAdapter** 類別可讓您修改 **DataSet** 物件，然後以 INSERT、UPDATE 和 DELETE 作業的形式提交變更。 如果以此方式使用 **DataAdapter** ，必須注意每個不同的作業會執行個別的呼叫。 若要改善效能，請使用 **UpdateBatchSize** 屬性指定應該同時批次處理的作業數目。 如需詳細資訊，請參閱 [使用 Dataadapter 執行批次作業](https://msdn.microsoft.com/library/aadf8fk2.aspx)。

### <a name="entity-framework"></a>Entity Framework
Entity Framework 目前不支援批次處理。 社群中不同的開發人員已嘗試示範因應措施，例如覆寫 **SaveChanges** 方法。 但解決方案通常太複雜，而且都針對應用程式和資料模型來自訂。 Entity Framework codeplex 專案目前有這項功能要求的討論頁。 若要查看這項討論，請參閱[設計會議記錄 - 2012 年 8 月 2 日 (英文)](http://entityframework.codeplex.com/wikipage?title=Design%20Meeting%20Notes%20-%20August%202%2c%202012)。

### <a name="xml"></a>XML
為了完整說明，我們覺得有必要討論將 XML 當做批次處理策略。 但是，使用 XML 沒有比其他方法更好，而且還有幾個缺點。 此方法類似於資料表值參數，但傳給預存程序的是 XML 檔案或字串，而不是使用者定義的資料表。 預存程序會剖析預存程序中的命令。

這種方法有幾個缺點：

* 處理 XML 很麻煩又容易出錯。
* 在資料庫上剖析 XML 會耗用大量 CPU 資源。
* 在大部分情況下，這種方法比資料表值參數更慢。

基於這些理由，不建議使用 XML 進行批次查詢。

## <a name="batching-considerations"></a>批次處理考量
下列各節提供在 SQL Database 應用程式中使用批次處理的其他指引。

### <a name="tradeoffs"></a>權衡取捨
根據您的架構而定，批次處理可能需要在效能和恢復功能之間有所取捨。 例如，假設您的角色非預期地停止運作。 如果您失去一個資料列，影響程度會小於遺失一大批尚未提交的資料列。 將資料列傳送至資料庫之前，如果您將資料列緩衝一段指定的時間，則風險會很高。

由於有這種權衡取捨，請評估您要批次處理的作業類型。 對於較不重要的資料，應該更積極地批次處理 (較大的批次和較長的時間範圍)。

### <a name="batch-size"></a>批次大小
在我們的測試中，將大型批次分成小塊通常沒有好處。 事實上，這樣分割通常會導致效能比提交單一大型批次更慢。 例如，假設您想要插入 1000 個資料列。 下表顯示分割成較小的批次時，使用資料表值參數插入 1000 個資料列所需的時間。

| 批次大小 | 反覆運算次數 | 資料表值參數 (毫秒) |
| --- | --- | --- |
| 1000 |1 |347 |
| 500 |2 |355 |
| 100 |10 |465 |
| 50 |20 |630 |

> [!NOTE]
> 結果並不是基準。 請參閱[有關此文章中計時結果的注意事項](#note-about-timing-results-in-this-article)。
> 
> 

您可以看出，將 1000 個資料列一次全部提交，才能獲得最佳效能。 在其他測試中 (這裡未顯示)，將 10000 個資料列的批次分成兩個各有 5000 個資料列的批次會稍微提高效能。 但是這些測試的資料表結構描述相當簡單，您應該針對您的特定資料和批次大小進行測試，以確認這些研究結果。

另一個要考量的因素是如果整個批次變得太大，SQL Database 可能會節流並拒絕認可批次。 為了獲得最佳結果，請測試您的特定案例，以判斷是否有較理想的批次大小。 允許在執行階段設定批次大小，以根據效能或錯誤來快速調整。

最後，在批次大小與批次處理相關的風險之間找出平衡點。 如果發生暫時性錯誤或角色失敗，請考量重試作業或遺失批次中的資料的後果。

### <a name="parallel-processing"></a>平行處理
如果您已設法縮小批次，但使用多個執行緒來執行工作，情況又是如何？ 同樣地，我們的測試指出數個較小的多執行緒批次通常表現得比單一大型批次更差。 下列測試嘗試透過一或多個平行批次來插入 1000 個資料列。 此測試指出同時有多個批次實際上會降低效能。

| 批次大小 [反覆運算次數] | 兩個執行緒 (毫秒) | 四個執行緒 (毫秒) | 六個執行緒 (毫秒) |
| --- | --- | --- | --- |
| 1000 [1] |277 |315 |266 |
| 500 [2] |548 |278 |256 |
| 250 [4] |405 |329 |265 |
| 100 [10] |488 |439 |391 |

> [!NOTE]
> 結果並不是基準。 請參閱[有關此文章中計時結果的注意事項](#note-about-timing-results-in-this-article)。
> 
> 

由於平行處理而造成效能下降可能有幾個原因：

* 同時有多個網路呼叫，而不只一個。
* 對單一資料表執行多個作業會導致爭用和鎖定。
* 多執行緒處理會引起額外負荷。
* 開啟多個連接的成本超過平行處理的效益。

如果以不同資料表或資料庫為目標，這種策略可能會提高一些效能。 資料庫分區化或同盟就是適合這種方法的案例。 分區化會使用多個資料庫，並將不同資料路由傳送至每個資料庫。 如果每個小批次各送往不同的資料庫，則以平行方式執行作業會更有效率。 但是，效能提升不顯著，難以據此決定在您的解決方案中使用資料庫分區化。

在某些設計中，平行執行較小的批次可以在負載不足的系統中改善要求輸送量。 在此情況下，即使處理單一大型批次更快速，但以平行方式處理多個批次可能更有效率。

如果您使用平行執行，請考慮控制背景工作角色執行緒的數目上限。 數量較少可以減少爭用，並加快執行時間。 另外，也要考慮這在連接和交易中對目標資料庫所造成的額外負載。

### <a name="related-performance-factors"></a>相關的效能因素
資料庫效能的一般指引也會影響批次處理。 例如，如果資料表具有大型的主索引鍵或許多非叢集索引，插入效能會降低。

如果資料表值參數使用預存程序，您可以在程序的開頭使用命令 **SET NOCOUNT ON** 。 這個陳述式會防止傳回程序中受影響的資料列計數。 不過，在我們的測試中，使用 **SET NOCOUNT ON** 不是沒有效果，就是降低效能。 測試預存程序很簡單，只有來自資料表值參數的單一 **INSERT** 命令。 此陳述式可能有益於更複雜的預存程序。 但別以為將 **SET NOCOUNT ON** 新增至預存程序就會自動改善效能。 若要了解效果，請分別使用和不使用 **SET NOCOUNT ON** 陳述式來測試預存程序。

## <a name="batching-scenarios"></a>批次處理案例
下列各節說明如何在三個應用程式案例中使用資料表值參數。 第一個案例示範緩衝和批次處理如何一起運作。 第二個案例在單一預存程序呼叫中執行主要/詳細架構作業以提高效能。 最後一個案例示範如何在 "UPSERT" 作業中使用資料表值參數。

### <a name="buffering"></a>緩衝處理
雖然有些案例很明顯適合使用批次處理，但也有許多案例可以藉由延遲處理來利用批次處理。 不過，延遲處理也會帶來更大的風險，發生非預期的失敗時會遺失資料。 請務必了解這項風險並考慮後果。

例如，假設有一個 Web 應用程式會追蹤每一位使用者的瀏覽歷程記錄。 在每個頁面要求上，應用程式會執行資料庫呼叫來記錄使用者的頁面檢視。 但只要緩衝使用者的瀏覽活動，再分批將此資料傳送至資料庫，就能達到更高的效能和延展性。 您可以指定經歷時間和 (或) 緩衝區大小來觸發資料庫更新。 例如，規則可以指定應該在 20 秒之後或緩衝區達到 1000 個項目時處理批次。

下列程式碼範例使用 [Reactive Extensions - Rx](https://msdn.microsoft.com/data/gg577609) 處理監視類別所引發的緩衝事件。 當緩衝區填滿或達到逾時，就透過資料表值參數將使用者資料的批次傳送至資料庫。

下列 NavHistoryData 類別塑造使用者瀏覽詳細資料的模型。 它包含使用者識別碼、存取的 URL 和存取時間等基本資訊。

    public class NavHistoryData
    {
        public NavHistoryData(int userId, string url, DateTime accessTime)
        { UserId = userId; URL = url; AccessTime = accessTime; }
        public int UserId { get; set; }
        public string URL { get; set; }
        public DateTime AccessTime { get; set; }
    }

NavHistoryDataMonitor 類別負責將使用者瀏覽資料緩衝處理到資料庫。 它包含 RecordUserNavigationEntry 方法，以引發 **OnAdded** 事件做為回應。 下列程式碼顯示建構函式邏輯，其中使用 Rx 根據事件建立可觀察的集合。 它接著使用 Buffer 方法訂閱這個可觀察的集合。 此多載函式指定每隔 20 秒或 1000 個項目就應該傳送緩衝區。

    public NavHistoryDataMonitor()
    {
        var observableData =
            Observable.FromEventPattern<NavHistoryDataEventArgs>(this, "OnAdded");

        observableData.Buffer(TimeSpan.FromSeconds(20), 1000).Subscribe(Handler);           
    }

此處理常式會將所有緩衝項目轉換成資料表值類型，然後將此類型傳遞給處理批次的預存程序。 下列程式碼顯示 NavHistoryDataEventArgs 和 NavHistoryDataMonitor 類別的完整定義。

    public class NavHistoryDataEventArgs : System.EventArgs
    {
        public NavHistoryDataEventArgs(NavHistoryData data) { Data = data; }
        public NavHistoryData Data { get; set; }
    }

    public class NavHistoryDataMonitor
    {
        public event EventHandler<NavHistoryDataEventArgs> OnAdded;

        public NavHistoryDataMonitor()
        {
            var observableData =
                Observable.FromEventPattern<NavHistoryDataEventArgs>(this, "OnAdded");

            observableData.Buffer(TimeSpan.FromSeconds(20), 1000).Subscribe(Handler);           
        }

        public void RecordUserNavigationEntry(NavHistoryData data)
        {    
            if (OnAdded != null)
                OnAdded(this, new NavHistoryDataEventArgs(data));
        }

        protected void Handler(IList<EventPattern<NavHistoryDataEventArgs>> items)
        {
            DataTable navHistoryBatch = new DataTable("NavigationHistoryBatch");
            navHistoryBatch.Columns.Add("UserId", typeof(int));
            navHistoryBatch.Columns.Add("URL", typeof(string));
            navHistoryBatch.Columns.Add("AccessTime", typeof(DateTime));
            foreach (EventPattern<NavHistoryDataEventArgs> item in items)
            {
                NavHistoryData data = item.EventArgs.Data;
                navHistoryBatch.Rows.Add(data.UserId, data.URL, data.AccessTime);
            }

            using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
            {
                connection.Open();

                SqlCommand cmd = new SqlCommand("sp_RecordUserNavigation", connection);
                cmd.CommandType = CommandType.StoredProcedure;

                cmd.Parameters.Add(
                    new SqlParameter()
                    {
                        ParameterName = "@NavHistoryBatch",
                        SqlDbType = SqlDbType.Structured,
                        TypeName = "NavigationHistoryTableType",
                        Value = navHistoryBatch,
                    });

                cmd.ExecuteNonQuery();
            }
        }
    }

若要使用這個緩衝類別，應用程式需要建立靜態 NavHistoryDataMonitor 物件。 每次使用者存取頁面時，應用程式就呼叫 NavHistoryDataMonitor.RecordUserNavigationEntry 方法。 緩衝邏輯接著負責將這些項目分批傳送至資料庫。

### <a name="master-detail"></a>主要/詳細架構
資料表值參數適用於簡單的 INSERT 案例。 但是，如果插入作業涉及多個資料表，則批次處理會較具挑戰性。 「主要/詳細架構」案例是一個很好的例子。 主要資料表識別主要實體。 一或多個詳細資料表儲存實體的其他相關資料。 在此案例中，外部索引鍵關聯性會強制執行詳細資料與唯一主要實體的關聯性。 假設有一個簡化版的 PurchaseOrder 資料表及其相關聯的 OrderDetail 資料表。 下列 Transact-SQL 會建立具有四個資料行的 PurchaseOrder 資料表：OrderID、OrderDate、CustomerID 和 Status。

    CREATE TABLE [dbo].[PurchaseOrder](
    [OrderID] [int] IDENTITY(1,1) NOT NULL,
    [OrderDate] [datetime] NOT NULL,
    [CustomerID] [int] NOT NULL,
    [Status] [nvarchar](50) NOT NULL,
     CONSTRAINT [PrimaryKey_PurchaseOrder] 
    PRIMARY KEY CLUSTERED ( [OrderID] ASC ))

每筆訂單包含一或多個產品採購。 PurchaseOrderDetail 資料表中擷取這項資訊。 下列 Transact-SQL 會建立具有五個資料行的 PurchaseOrderDetail 資料表：OrderID、OrderDetailID、ProductID、UnitPrice 和 OrderQty。

    CREATE TABLE [dbo].[PurchaseOrderDetail](
    [OrderID] [int] NOT NULL,
    [OrderDetailID] [int] IDENTITY(1,1) NOT NULL,
    [ProductID] [int] NOT NULL,
    [UnitPrice] [money] NULL,
    [OrderQty] [smallint] NULL,
     CONSTRAINT [PrimaryKey_PurchaseOrderDetail] PRIMARY KEY CLUSTERED 
    ( [OrderID] ASC, [OrderDetailID] ASC ))

PurchaseOrderDetail 資料表中的 OrderID 資料行必須參考 PurchaseOrder 資料表中的訂單。 下列的外部索引鍵定義強制執行這個限制。

    ALTER TABLE [dbo].[PurchaseOrderDetail]  WITH CHECK ADD 
    CONSTRAINT [FK_OrderID_PurchaseOrder] FOREIGN KEY([OrderID])
    REFERENCES [dbo].[PurchaseOrder] ([OrderID])

若要使用資料表值參數，您必須針對每個目標資料表定義一個使用者定義資料表類型。

    CREATE TYPE PurchaseOrderTableType AS TABLE 
    ( OrderID INT,
      OrderDate DATETIME,
      CustomerID INT,
      Status NVARCHAR(50) );
    GO

    CREATE TYPE PurchaseOrderDetailTableType AS TABLE 
    ( OrderID INT,
      ProductID INT,
      UnitPrice MONEY,
      OrderQty SMALLINT );
    GO

然後，定義一個可接受這幾種資料表的預存程序。 此程序可讓應用程式以單一呼叫在本機批次處理一組訂單和訂單詳細資料。 下列 Transact-SQL 提供這個採購單範例的完整預存程序宣告。

    CREATE PROCEDURE sp_InsertOrdersBatch (
    @orders as PurchaseOrderTableType READONLY,
    @details as PurchaseOrderDetailTableType READONLY )
    AS
    SET NOCOUNT ON;

    -- Table that connects the order identifiers in the @orders
    -- table with the actual order identifiers in the PurchaseOrder table
    DECLARE @IdentityLink AS TABLE ( 
    SubmittedKey int, 
    ActualKey int, 
    RowNumber int identity(1,1)
    );

          -- Add new orders to the PurchaseOrder table, storing the actual
    -- order identifiers in the @IdentityLink table   
    INSERT INTO PurchaseOrder ([OrderDate], [CustomerID], [Status])
    OUTPUT inserted.OrderID INTO @IdentityLink (ActualKey)
    SELECT [OrderDate], [CustomerID], [Status] FROM @orders ORDER BY OrderID;

    -- Match the passed-in order identifiers with the actual identifiers
    -- and complete the @IdentityLink table for use with inserting the details
    WITH OrderedRows As (
    SELECT OrderID, ROW_NUMBER () OVER (ORDER BY OrderID) As RowNumber 
    FROM @orders
    )
    UPDATE @IdentityLink SET SubmittedKey = M.OrderID
    FROM @IdentityLink L JOIN OrderedRows M ON L.RowNumber = M.RowNumber;

    -- Insert the order details into the PurchaseOrderDetail table, 
          -- using the actual order identifiers of the master table, PurchaseOrder
    INSERT INTO PurchaseOrderDetail (
    [OrderID],
    [ProductID],
    [UnitPrice],
    [OrderQty] )
    SELECT L.ActualKey, D.ProductID, D.UnitPrice, D.OrderQty
    FROM @details D
    JOIN @IdentityLink L ON L.SubmittedKey = D.OrderID;
    GO

在此範例中，本機定義的 @IdentityLink 資料表會儲存新插入的資料列中的實際 OrderID 值。 這些訂單識別碼與 @orders 和 @details 資料表值參數中的暫時 OrderID 值不同。 因此，@IdentityLink 資料表就會將 @orders 參數中的 OrderID 值，連接到 PurchaseOrder 資料表中的新資料列的實際 OrderID 值。 完成這個步驟之後，@IdentityLink 資料表就能以滿足外部索引鍵條件約束的實際 OrderID，協助插入訂單詳細資料。

從程式碼或其他 Transact-SQL 呼叫中，都可以使用這個預存程序。 請參閱本文的資料表值參數一節，其中有一個程式碼範例。 下列 Transact-SQL 示範如何呼叫 sp_InsertOrdersBatch。

    declare @orders as PurchaseOrderTableType
    declare @details as PurchaseOrderDetailTableType

    INSERT @orders 
    ([OrderID], [OrderDate], [CustomerID], [Status])
    VALUES(1, '1/1/2013', 1125, 'Complete'),
    (2, '1/13/2013', 348, 'Processing'),
    (3, '1/12/2013', 2504, 'Shipped')

    INSERT @details
    ([OrderID], [ProductID], [UnitPrice], [OrderQty])
    VALUES(1, 10, $11.50, 1),
    (1, 12, $1.58, 1),
    (2, 23, $2.57, 2),
    (3, 4, $10.00, 1)

    exec sp_InsertOrdersBatch @orders, @details

這個解決方案可讓每個批次使用一組從 1 開始的 OrderID 值。 這些暫時 OrderID 值描述批次中的關聯性，但實際 OrderID 值是在插入時才決定。 您可以重複執行上述範例中相同的陳述式，在資料庫中產生唯一的訂單。 因此，使用這種批次技術時，請考慮新增更多程式碼或資料庫邏輯，以防止訂單重複。

此範例示範即使是更複雜的資料庫作業，例如主要/詳細架構作業，也都可以透過資料表值參數進行批次處理。

### <a name="upsert"></a>UPSERT
另一個批次處理案例涉及同時更新現有的資料列和插入新資料列。 這項作業有時稱為 "UPSERT" (更新 + 插入) 作業。 MERGE 陳述式最適合這項作業，而不是分開呼叫 INSERT 和 UPDATE。 MERGE 陳述式可以在單一呼叫中同時執行插入和更新作業。

資料表值參數可以搭配 MERGE 陳述式來執行更新和插入。 例如，假設有一個簡化的 Employee 資料表，包含下列資料行：EmployeeID、FirstName、LastName、SocialSecurityNumber：

    CREATE TABLE [dbo].[Employee](
    [EmployeeID] [int] IDENTITY(1,1) NOT NULL,
    [FirstName] [nvarchar](50) NOT NULL,
    [LastName] [nvarchar](50) NOT NULL,
    [SocialSecurityNumber] [nvarchar](50) NOT NULL,
     CONSTRAINT [PrimaryKey_Employee] PRIMARY KEY CLUSTERED 
    ([EmployeeID] ASC ))

在此範例中，根據 SocialSecurityNumber 是唯一性的這項事實，您可以將多個員工 MERGE。 首先，建立使用者定義的資料表類型：

    CREATE TYPE EmployeeTableType AS TABLE 
    ( Employee_ID INT,
      FirstName NVARCHAR(50),
      LastName NVARCHAR(50),
      SocialSecurityNumber NVARCHAR(50) );
    GO

接下來，建立預存程序或撰寫程式碼，使用 MERGE 陳述式來執行更新和插入。 下列範例對 EmployeeTableType 類型的資料表值參數 @employees 使用 MERGE 陳述式。 此處未顯示 @employees 資料表的內容。

    MERGE Employee AS target
    USING (SELECT [FirstName], [LastName], [SocialSecurityNumber] FROM @employees) 
    AS source ([FirstName], [LastName], [SocialSecurityNumber])
    ON (target.[SocialSecurityNumber] = source.[SocialSecurityNumber])
    WHEN MATCHED THEN 
    UPDATE SET
    target.FirstName = source.FirstName, 
    target.LastName = source.LastName
    WHEN NOT MATCHED THEN
       INSERT ([FirstName], [LastName], [SocialSecurityNumber])
       VALUES (source.[FirstName], source.[LastName], source.[SocialSecurityNumber]);

如需詳細資訊，請參閱 MERGE 陳述式的文件和範例。 雖然在多步驟的預存程序呼叫中搭配個別的 INSERT 和 UPDATE 作業，也能執行相同的工作，但 MERGE 陳述式會更有效率。 資料庫程式碼也可以建構 Transact-SQL 呼叫，直接使用 MERGE 陳述式，而不需要為 INSERT 和 UPDATE 而分別執行兩次資料庫呼叫。

## <a name="recommendation-summary"></a>建議摘要
下列清單提供本文所討論批次處理建議的摘要：

* 使用緩衝處理和批次處理，提高 SQL Database 應用程式的效能和延展性。
* 了解批次處理/緩衝處理和恢復功能之間的權衡取捨。 在角色失敗期間，可能遺失一批尚未處理的商務關鍵資料，這種風險超過批次處理帶來的效能優點。
* 嘗試將所有資料庫呼叫納入單一資料中心以縮短延遲。
* 如果選擇單一批次處理技術，資料表值參數可以發揮最佳的效能與彈性。
* 為了獲得最快速的插入效能，請遵循下列一般指導方針，但要針對您的案例進行測試：
  * 如果 < 100 個資料列，使用單一參數化 INSERT 命令。
  * 如果 < 1000 個資料列，使用資料表值參數。
  * 如果 >= 1000 個資料列，使用 SqlBulkCopy。
* 針對更新和刪除作業，使用資料表值參數搭配預存程序邏輯，決定要在資料表參數中每個資料列上執行的正確作業。
* 批次大小的指導方針：
  * 使用符合您的應用程式和商務需求的最大批次大小。
  * 在大型批次的效能提升和暫時性或災難性失敗的風險之間取得平衡。 重試或在批次中遺失資料的後果是什麼？ 
  * 測試最大批次大小，確認 SQL Database 不會拒絕它。
  * 建立組態設定來控制批次處理，例如批次大小或緩衝時間範圍。 這些設定提供彈性。 您可以在生產環境中變更批次處理行為，不需重新部署雲端服務。
* 避免在一個資料庫的單一資料表上平行執行批次。 如果您選擇將單一批次分成多個背景工作角色執行緒，請執行測試來決定理想的執行緒數目。 超過未確定的臨界值之後，更多的執行緒只會降低效能，不會提高效能。
* 在更多案例下，考慮依大小和時間緩衝來實作批次處理。

## <a name="next-steps"></a>後續步驟
這篇文章著重於與批次處理相關的資料庫設計和程式碼撰寫技術，如何改善應用程式的效能和延展性。 但這只是整體策略中的一個因素。 關於其他可改善效能和延展性的方式，請參閱[單一資料庫的 Azure SQL Database 效能指引](sql-database-performance-guidance.md)和[彈性集區的價格和效能考量](sql-database-elastic-pool-guidance.md)。

