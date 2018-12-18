---
title: 使用 Azure SQL Database 的資料相依路由 | Microsoft Docs
description: 如何在 .NET 應用程式中將 ShardMapManager 類別用於資料相依路由 (Azure SQL Database 中的分區化資料庫的一項功能)
services: sql-database
ms.service: sql-database
subservice: elastic-scale
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: 25bb665d9ea9166d099ab7f3f9696d92da8314e9
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161808"
---
# <a name="data-dependent-routing"></a>資料相依路由
**資料相依路由**是可使用查詢中的資料，將要求路由至適當的資料庫。 資料相依路由是使用分區化資料庫時的一種基本模式。 要求內容也可能會用於路由要求，特別是如果分區化索引鍵不是查詢的一部分。 在使用資料相依路由的應用程式中，每個特定的查詢或交易會限制每個要求只能存取單一資料庫。 針對 Azure SQL Database Elastic 工具，此路由會使用 **ShardMapManager**  ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager._shard_map_manager)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.aspx)) 類別來完成。

應用程式不需要在分區化環境中追蹤不同的連接字串或與不同資料片段相關聯的 DB 位置。 相反地， [分區對應管理員](sql-database-elastic-scale-shard-map-management.md) 會根據分區對應中的資料和分區化索引鍵的值 (應用程式要求的目標)，在必要時開啟正確資料庫的連接。 此索引鍵通常是 customer_id、tenant_id、date_key，或作為資料庫要求基本參數的其他一些特定的識別項。 

如需詳細資訊，請參閱 [Scaling Out SQL Server with Data Dependent Routing (使用資料相依路由相應放大 SQL Server)](https://technet.microsoft.com/library/cc966448.aspx)。

## <a name="download-the-client-library"></a>下載用戶端程式庫
若要下載：
* Java 版本的程式庫，請參閱 [Maven 中央存放庫](https://search.maven.org/#search%7Cga%7C1%7Celastic-db-tools)。
* .NET 版本的程式庫，請參閱 [NuGet](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)。

## <a name="using-a-shardmapmanager-in-a-data-dependent-routing-application"></a>在資料相依路由應用程式中使用 ShardMapManager
應用程式應該在初始化期間，使用 Factory 呼叫 **GetSQLShardMapManager** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager._shard_map_manager_factory.getsqlshardmapmanager)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager.aspx)) 來具現化 **ShardMapManager**。 此範例中會初始化 **ShardMapManager** 及其包含的特定 **ShardMap**。 這個範例示範 GetSqlShardMapManager 和 GetRangeShardMap ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager._shard_map_manager.getrangeshardmap)、[.NET](https://msdn.microsoft.com/library/azure/dn824173.aspx)) 方法。

```Java
ShardMapManager smm = ShardMapManagerFactory.getSqlShardMapManager(connectionString, ShardMapManagerLoadPolicy.Lazy);
RangeShardMap<int> rangeShardMap = smm.getRangeShardMap(Configuration.getRangeShardMapName(), ShardKeyType.Int32);
```

```csharp
ShardMapManager smm = ShardMapManagerFactory.GetSqlShardMapManager(smmConnnectionString, ShardMapManagerLoadPolicy.Lazy);
RangeShardMap<int> customerShardMap = smm.GetRangeShardMap<int>("customerMap"); 
```

### <a name="use-lowest-privilege-credentials-possible-for-getting-the-shard-map"></a>盡可能使用最低權限的認證來取得分區對應
如果應用程式不會自行操作分區對應，用於 Factory 方法中的認證在**全域分區對應**資料庫上應該有唯讀權限。 這些認證通常不同於用來對分區對應管理員開啟連接的認證。 另請參閱 [用來存取彈性資料庫用戶端程式庫的認證](sql-database-elastic-scale-manage-credentials.md)。 

## <a name="call-the-openconnectionforkey-method"></a>呼叫 OpenConnectionForKey 方法
**ShardMap.OpenConnectionForKey** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapper._list_shard_mapper.openconnectionforkey)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey.aspx)) 方法傳回的連線可根據 **key** 參數的值，對適當的資料庫發出命令。 **ShardMapManager** 會將分區資訊快取在應用程式中，因此這些要求通常不需要針對**全域分區對應**資料庫進行資料庫尋查。 

```Java
// Syntax: 
public Connection openConnectionForKey(Object key, String connectionString, ConnectionOptions options)
```

```csharp
// Syntax: 
public SqlConnection OpenConnectionForKey<TKey>(TKey key, string connectionString, ConnectionOptions options)
```
* **key** 參數做為分區對應中的查閱索引鍵，以決定要求的適當資料庫。 
* **connectionString** 只用來傳遞使用者認證給所需的連接。 此 *connectionString* 中不含任何資料庫名稱或伺服器名稱，因為此方法會使用 **ShardMap** 來決定資料庫和伺服器。 
* 若分區對應所在的環境可能變更，且資料列可能會移動到其他的資料庫成為分割或合併作業的結果，則 **connectionOptions** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapper._connection_options)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.connectionoptions.aspx)) 應設為 **ConnectionOptions.Validate**。 此驗證牽涉到在將連接傳遞至應用程式之前，對目標資料庫上的本機分區對應 (不是全域分區對應) 的簡短查詢。 

如果本機分區對應驗證失敗 (表示快取不正確)，分區對應管理員會查詢全域分區對應來取得查閱的新正確值、更新快取，然後取得並傳回適當的資料庫連線。 

唯有當應用程式在線上，分區對應的變更是非預期之時，才使用 **ConnectionOptions.None** 。 在此情況下，快取的值可假定為永遠正確，可放心地略過對於目標資料庫的額外往返驗證呼叫。 這會減少資料庫流量。 也可透過組態檔中的值來設定 **connectionOptions** ，以指出在一段時間內是否預期有分區化變更。  

此範例會利用名為 **customerShardMap** 的 **ShardMap** 物件，使用整數索引鍵值 **CustomerID**。  

```Java 
int customerId = 12345; 
int productId = 4321; 
// Looks up the key in the shard map and opens a connection to the shard
try (Connection conn = shardMap.openConnectionForKey(customerId, Configuration.getCredentialsConnectionString())) {
    // Create a simple command that will insert or update the customer information
    PreparedStatement ps = conn.prepareStatement("UPDATE Sales.Customer SET PersonID = ? WHERE CustomerID = ?");

    ps.setInt(1, productId);
    ps.setInt(2, customerId);
    ps.executeUpdate();
} catch (SQLException e) {
    e.printStackTrace();
}
```

```csharp
int customerId = 12345; 
int newPersonId = 4321; 

// Connect to the shard for that customer ID. No need to call a SqlConnection 
// constructor followed by the Open method.
using (SqlConnection conn = customerShardMap.OpenConnectionForKey(customerId, Configuration.GetCredentialsConnectionString(), ConnectionOptions.Validate)) 
{ 
    // Execute a simple command. 
    SqlCommand cmd = conn.CreateCommand(); 
    cmd.CommandText = @"UPDATE Sales.Customer 
                        SET PersonID = @newPersonID WHERE CustomerID = @customerID"; 

    cmd.Parameters.AddWithValue("@customerID", customerId);cmd.Parameters.AddWithValue("@newPersonID", newPersonId); 
    cmd.ExecuteNonQuery(); 
}  
```

**OpenConnectionForKey** 方法會傳回新的已開啟連接至正確的資料庫。 以這種方式使用連線仍可充分利用連線共用。 

如果您的應用程式利用非同步程式設計，則也可以使用 **OpenConnectionForKeyAsync method** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapper._list_shard_mapper.openconnectionforkeyasync)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkeyasync.aspx))。

## <a name="integrating-with-transient-fault-handling"></a>與暫時性錯誤處理整合
在雲端中開發資料存取應用程式時，最佳做法就是確保應用程式會攔截暫時性錯誤，並且將作業重試數次之後才會擲回錯誤。 暫時性錯誤處理中討論雲端應用程式的暫時性錯誤處理 ([Java](/java/api/com.microsoft.azure.elasticdb.core.commons.transientfaulthandling)、[.NET](https://msdn.microsoft.com/library/dn440719\(v=pandp.60\).aspx))。 

暫時性錯誤處理可以自然地與資料相依路由模式並存。 主要需求是重試整個資料存取要求，包括用以取得資料相依路由連接的 **using** 區塊。 上述範例可以改寫如下。 

### <a name="example---data-dependent-routing-with-transient-fault-handling"></a>範例 - 資料相依路由與暫時性錯誤處理
```Java 
int customerId = 12345; 
int productId = 4321; 
try {
    SqlDatabaseUtils.getSqlRetryPolicy().executeAction(() -> {
        // Looks up the key in the shard map and opens a connection to the shard
        try (Connection conn = shardMap.openConnectionForKey(customerId, Configuration.getCredentialsConnectionString())) {
            // Create a simple command that will insert or update the customer information
            PreparedStatement ps = conn.prepareStatement("UPDATE Sales.Customer SET PersonID = ? WHERE CustomerID = ?");

            ps.setInt(1, productId);
            ps.setInt(2, customerId);
            ps.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    });
} catch (Exception e) {
    throw new StoreException(e.getMessage(), e);
}
```

```csharp
int customerId = 12345; 
int newPersonId = 4321; 

Configuration.SqlRetryPolicy.ExecuteAction(() =&gt; 
{
    // Connect to the shard for a customer ID. 
    using (SqlConnection conn = customerShardMap.OpenConnectionForKey(customerId, Configuration.GetCredentialsConnectionString(), ConnectionOptions.Validate)) 
    { 
        // Execute a simple command 
        SqlCommand cmd = conn.CreateCommand(); 

        cmd.CommandText = @"UPDATE Sales.Customer 
                            SET PersonID = @newPersonID 
                            WHERE CustomerID = @customerID"; 

        cmd.Parameters.AddWithValue("@customerID", customerId); 
        cmd.Parameters.AddWithValue("@newPersonID", newPersonId); 
        cmd.ExecuteNonQuery(); 

        Console.WriteLine("Update completed"); 
    } 
}); 
```

當您建置彈性資料庫範例應用程式時，自動會下載實作暫時性錯誤處理所需的封裝。 

## <a name="transactional-consistency"></a>交易一致性
對於分區範圍內的所有作業，都保證交易式屬性。 例如，透過資料相依路由提交的交易，都在連接的目標分區範圍內執行。 目前無法將多個連接編列到交易中，因此對於跨分區執行的作業，不提供交易式保證。

## <a name="next-steps"></a>後續步驟
若要中斷連結分區或重新附加分區，請參閱 [使用 RecoveryManager 類別來修正分區對應問題](sql-database-elastic-database-recovery-manager.md)

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

