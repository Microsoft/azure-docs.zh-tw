---
title: 搭配使用彈性資料庫用戶端程式庫與 Dapper | Microsoft Docs
description: 搭配使用彈性資料庫用戶端程式庫與 Dapper。
services: sql-database
manager: craigg
author: stevestein
ms.service: sql-database
ms.custom: scale out apps
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: sstein
ms.openlocfilehash: 6619f2dfe5f58cd23dbd0ffe6e2b545b803f3acc
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34647923"
---
# <a name="using-elastic-database-client-library-with-dapper"></a>搭配使用彈性資料庫用戶端程式庫與 Dapper
本文件適用於下列開發人員︰依賴 Dapper 建置應用程式，但也想利用[彈性資料庫工具](sql-database-elastic-scale-introduction.md)建立應用程式，經由實作分區化來相應放大資料層。  這份文件說明為了與彈性資料庫工具整合，Dapper 應用程式中需要做的變更。 重點在於使用 Dapper 撰寫彈性資料庫分區管理和資料相依路由。 

**範例程式碼**： [Azure SQL Database - Dapper 整合的彈性資料庫工具](https://code.msdn.microsoft.com/Elastic-Scale-with-Azure-e19fc77f)。

整合 **Dapper** 和 **DapperExtensions** 與 Azure SQL Database 的彈性資料庫用戶端程式庫很容易。 將新的 [SqlConnection](http://msdn.microsoft.com/library/system.data.sqlclient.sqlconnection.aspx) 物件的建立和開啟作業變更為使用[用戶端程式庫](http://msdn.microsoft.com/library/azure/dn765902.aspx)的 [OpenConnectionForKey](http://msdn.microsoft.com/library/azure/dn807226.aspx) 呼叫，您的應用程式即可使用資料相依路由。 這會使應用程式中的變更侷限於建立和開啟新連線的位置。 

## <a name="dapper-overview"></a>Dapper 概觀
**Dapper** 是物件關聯式對應程式。 它可將應用程式的 .NET 物件對應到關聯式資料庫 (反之亦然) 。 範例程式碼的第一部分說明如何整合彈性資料庫用戶端程式庫與 Dapper 應用程式。 範例程式碼的第二部分說明如何在同時使用 Dapper 和 DapperExtensions 時進行整合。  

Dapper 的對應程式功能提供資料庫連線的擴充方法，以簡化提交 T-SQL 陳述式以便執行或查詢資料庫。 比方說，Dapper 可輕鬆對應 .NET 物件與 **Execute** 呼叫的 SQL 陳述式參數，或取用您從 Dapper 使用 **Query** 呼叫對 .NET 物件進行 SQL 查詢的結果。 

使用 DapperExtensions，您不再需要提供 SQL 陳述式。 透過資料庫連接的延伸方法 (如 **GetList** 或 **Insert**) 可在幕後建立 SQL 陳述式。

Dapper 以及 DapperExtensions 的另一個好處就是應用程式可控制資料庫連線的建立。 這有助於與彈性資料庫用戶端程式庫互動，該程式庫會根據 Shardlet 與資料庫的對應來代理資料庫連接。

若要取得 Dapper 組件，請參閱 [Dapper dot net](http://www.nuget.org/packages/Dapper/)。 如需 Dapper 延伸模組，請參閱 [DapperExtensions](http://www.nuget.org/packages/DapperExtensions)。

## <a name="a-quick-look-at-the-elastic-database-client-library"></a>彈性資料庫用戶端程式庫的快速概覽
透過彈性資料庫用戶端程式庫，您可以定義應用程式資料的資料分割 (稱為 *Shardlet*)、將它們對應到資料庫，以及利用*分區化索引鍵*加以識別。 您可以擁有所需數量的資料庫，並將 Shardlet 分散於這些資料庫。 分區化索引鍵值到資料庫的對應，由程式庫的 API 所提供的分區對應來儲存。 這項功能稱為 **分區對應管理**。 對於攜帶分區化索引鍵的要求，分區對應也充當資料庫連接的代理人。 這項功能稱為 **資料相依路由**。

![分區對應和資料相依路由][1]

分區對應管理員可防止使用者檢視 Shardlet 資料時出現不一致，這種情況發生於資料庫正在進行並行 Shardlet 管理作業時。 在作法上，分區對應會代理以程式庫建置的應用程式的資料庫連接。 當分區管理作業可能影響 Shardlet 時，即可允許分區對應功能自動終止資料庫連線。 

您需要使用 [OpenConnectionForKey 方法](http://msdn.microsoft.com/library/azure/dn824099.aspx)來為 Dapper 建立連線，而不使用傳統的方式。 這可確保所有驗證都會發生，而且在分區之間移動任何資料時會適當地管理連線。

### <a name="requirements-for-dapper-integration"></a>Dapper 整合需求
當使用彈性資料庫用戶端程式庫和 Dapper API 時，您需要保留下列屬性：

* **相應放大**：我們要依需要在分區化應用程式資料層中新增或移除資料庫，以滿足應用程式的容量需求。 
* **一致性**：由於使用分區化相應放大應用程式，所以您必須執行資料相依路由。 我們需要使用程式庫的資料相依路由功能來執行此作業。 尤其是，您需要保留透過分區對應管理員代理的連線所提供的驗證和一致性保證，以避免查詢結果損毀或錯誤。 這可確保如果 (例如) Shardlet 目前已使用分割/合併 API 移至不同的分區，則會拒絕或停止對特定 Shardlet 的連線。
* **物件對應**：我們想要保留 Dapper 所提供的對應便利性，以轉換應用程式和基礎資料庫結構中的類別。 

下一節針對以 **Dapper** 和 **DapperExtensions** 為基礎的應用程式，提供這些需求的指導。

## <a name="technical-guidance"></a>技術指導
### <a name="data-dependent-routing-with-dapper"></a>Dapper 的資料相依路由
有了 Dapper，應用程式通常負責建立和開啟對基礎資料庫的連線。 如果應用程式指定型別 T，Dapper 會將查詢結果傳回為型別 T 的 .NET 集合。Dapper 會執行從 T-SQL 結果資料列到型別 T 物件的對應。同樣地，Dapper 會將 .NET 物件對應到資料操作語言 (DML) 陳述式的 SQL 值或參數。 Dapper 透過延伸方法，在 ADO.NET SQL 用戶端程式庫中的標準 [SqlConnection](http://msdn.microsoft.com/library/system.data.sqlclient.sqlconnection.aspx) 物件上提供這項功能。 Elastic Scale DDR API 所傳回的 SQL 連接也是標準 [SqlConnection](http://msdn.microsoft.com/library/system.data.sqlclient.sqlconnection.aspx) 物件。 如此一來，我們即可透過用戶端程式庫的 DDR API 所傳回的型別，直接使用 Dapper 延伸模組，因為這也是簡單的 SQL 用戶端連接。

經過這些觀察，請直接使用 Dapper 的彈性資料庫用戶端程式庫所代理的連接。

此程式碼範例 (來自隨附的範例) 說明如何由應用程式提供分區化索引鍵給程式庫，以代理對適當分區的連接。   

    using (SqlConnection sqlconn = shardingLayer.ShardMap.OpenConnectionForKey(
                     key: tenantId1, 
                     connectionString: connStrBldr.ConnectionString, 
                     options: ConnectionOptions.Validate))
    {
        var blog = new Blog { Name = name };
        sqlconn.Execute(@"
                      INSERT INTO
                            Blog (Name)
                            VALUES (@name)", new { name = blog.Name }
                        );
    }

呼叫 [OpenConnectionForKey](http://msdn.microsoft.com/library/azure/dn807226.aspx) API 可取代 SQL 用戶端連接的預設建立和開啟作業。 [OpenConnectionForKey](http://msdn.microsoft.com/library/azure/dn807226.aspx) 呼叫接受資料相依路由所需的引數： 

* 用來存取資料相依路由介面的分區對應
* 用來識別 Shardlet 的分區化索引鍵
* 用來連接到分區的認證 (使用者名稱和密碼)

分區對應物件會建立分區的連線，而此分區保留給定分區化索引鍵的 Shardlet。 彈性資料庫用戶端 API 也會標記此連接以履行其一致性保證。 由於呼叫 [OpenConnectionForKey](http://msdn.microsoft.com/library/azure/dn807226.aspx) 會傳回標準 SQL 用戶端連接物件，所以後續從 Dapper 呼叫 **Execute** 延伸方法時會遵循標準 Dapper 作法。

查詢的運作方式幾乎完全相同 - 首先使用 [OpenConnectionForKey](http://msdn.microsoft.com/library/azure/dn807226.aspx) 從用戶端 API 開啟連線。 然後使用標準 Dapper 延伸方法，將 SQL 查詢的結果對應至 .NET 物件：

    using (SqlConnection sqlconn = shardingLayer.ShardMap.OpenConnectionForKey(
                    key: tenantId1, 
                    connectionString: connStrBldr.ConnectionString, 
                    options: ConnectionOptions.Validate ))
    {    
           // Display all Blogs for tenant 1
           IEnumerable<Blog> result = sqlconn.Query<Blog>(@"
                                SELECT * 
                                FROM Blog
                                ORDER BY Name");

           Console.WriteLine("All blogs for tenant id {0}:", tenantId1);
           foreach (var item in result)
           {
                Console.WriteLine(item.Name);
            }
    }

請注意，包含 DDR 連接的 **using** 區段，將區塊內的所有資料庫作業都放在保有 tenantId1 的一個分區中。 查詢只會傳回目前分區上儲存的部落格，但不會傳回任何其他分區上儲存的部落格。 

## <a name="data-dependent-routing-with-dapper-and-dapperextensions"></a>Dapper 和 DapperExtensions 的資料相依路由
Dapper 隨附其他延伸模組的生態系統，可在開發資料庫應用程式時從資料庫提供進一步的便利性和抽象概念。 DapperExtensions 就是一個範例。 

在您的應用程式中使用 DapperExtensions，不會改變建立和管理資料庫連線的方式。 仍然由應用程式負責開啟連線，而延伸方法預期會有標準 SQL 用戶端連線物件。 我們可以如上所述依賴 [OpenConnectionForKey](http://msdn.microsoft.com/library/azure/dn807226.aspx) 。 如下列程式碼範例所示，唯一的改變就是您不再需要撰寫 T-SQL 陳述式：

    using (SqlConnection sqlconn = shardingLayer.ShardMap.OpenConnectionForKey(
                    key: tenantId2, 
                    connectionString: connStrBldr.ConnectionString, 
                    options: ConnectionOptions.Validate))
    {
           var blog = new Blog { Name = name2 };
           sqlconn.Insert(blog);
    }

以下是查詢的程式碼範例： 

    using (SqlConnection sqlconn = shardingLayer.ShardMap.OpenConnectionForKey(
                    key: tenantId2, 
                    connectionString: connStrBldr.ConnectionString, 
                    options: ConnectionOptions.Validate))
    {
           // Display all Blogs for tenant 2
           IEnumerable<Blog> result = sqlconn.GetList<Blog>();
           Console.WriteLine("All blogs for tenant id {0}:", tenantId2);
           foreach (var item in result)
           {
               Console.WriteLine(item.Name);
           }
    }

### <a name="handling-transient-faults"></a>處理暫時性錯誤
Microsoft 模式和作法小組已發佈[暫時性錯誤處理應用程式區塊](http://msdn.microsoft.com/library/hh680934.aspx)，以協助應用程式開發人員緩和在雲端執行時所遇到的常見暫時性錯誤狀況。 如需詳細資訊，請參閱 [堅持，是所有成功的秘方：使用暫時性錯誤處理應用程式區塊](http://msdn.microsoft.com/library/dn440719.aspx)。

此程式碼範例依賴暫時性錯誤程式庫來防止暫時性錯誤。 

    SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
    {
       using (SqlConnection sqlconn = 
          shardingLayer.ShardMap.OpenConnectionForKey(tenantId2, connStrBldr.ConnectionString, ConnectionOptions.Validate))
          {
              var blog = new Blog { Name = name2 };
              sqlconn.Insert(blog);
          }
    });

上述程式碼中的 **SqlDatabaseUtils.SqlRetryPolicy** 定義為 **SqlDatabaseTransientErrorDetectionStrategy**，並指定重試計數 10，重試之間等待時間為 5 秒。 如果您正在使用交易，請確定您的重試範圍會回到暫時性錯誤情況下的交易開頭。

## <a name="limitations"></a>限制
這份文件中所述的方法有幾個限制：

* 這份文件的範例程式碼不示範如何管理跨分區的結構描述。
* 提出要求時，我們假定所有資料庫處理都包含在要求所提供的分區化索引鍵所識別的單一分區內。 不過，這項假設並未永遠維持，例如，不可能使用分區化索引鍵時。 為了解決這個問題，彈性資料庫用戶端程式庫中提供 [MultiShardQuery 類別](http://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multishardexception.aspx)。 此類別可實作透過數個分區查詢的連線抽象概念。 使用 MultiShardQuery 搭配 Dapper 已超出本文的範圍

## <a name="conclusion"></a>結論
使用 Dapper 與 DapperExtensions 的應用程式可以輕易地受益於 Azure SQL Database 的彈性資料庫工具。 透過這份文件中所述的步驟，將新的 [SqlConnection](http://msdn.microsoft.com/library/system.data.sqlclient.sqlconnection.aspx) 物件的建立和開啟作業變更為使用彈性資料庫用戶端程式庫的 [OpenConnectionForKey](http://msdn.microsoft.com/library/azure/dn807226.aspx) 呼叫，這些應用程式即可使用工具的資料相依路由功能。 這會將所需的應用程式變更限制為建立及開啟新連線的位置。 

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-working-with-dapper/dapperimage1.png
