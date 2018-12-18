---
title: 查詢分區化 Azure SQL Database | Microsoft Docs
description: 使用彈性資料庫用戶端程式跨分區執行查詢。
services: sql-database
manager: craigg
author: stevestein
ms.service: sql-database
ms.custom: scale out apps
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: sstein
ms.openlocfilehash: 759ef7bfca118434c36044ff490ff3d2735b11c9
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44719159"
---
# <a name="multi-shard-querying"></a>多分區查詢
## <a name="overview"></a>概觀
使用[彈性資料庫工具](sql-database-elastic-scale-introduction.md)，您可以建立分區化資料庫解決方案。 **多分區查詢**用於資料收集/報告等工作，這些工作需要跨越數個分區執行查詢。 (這與在單一分區上執行所有工作的[資料相依路由](sql-database-elastic-scale-data-dependent-routing.md)相反。) 

1. 使用 **TryGetRangeShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager._shard_map_manager.trygetrangeshardmap)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetrangeshardmap.aspx))、**TryGetListShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager._shard_map_manager.trygetlistshardmap), [.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetlistshardmap.aspx)) 或 **GetShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager._shard_map_manager.getshardmap)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getshardmap.aspx)) 方法，取得 **RangeShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map._range_shard_map)、[.NET](https://msdn.microsoft.com/library/azure/dn807318.aspx)) 或 **ListShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map._list_shard_map)、[.NET](https://msdn.microsoft.com/library/azure/dn807370.aspx))。 請參閱**[建構 ShardMapManager](sql-database-elastic-scale-shard-map-management.md#constructing-a-shardmapmanager)** 和**[取得 RangeShardMap 或 ListShardMap](sql-database-elastic-scale-shard-map-management.md#get-a-rangeshardmap-or-listshardmap)**。
2. 建立 **MultiShardConnection** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard._multi_shard_connection)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multishardconnection.aspx)) 物件。
3. 建立 **MultiShardStatement 或 MultiShardCommand** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard._multi_shard_statement)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand.aspx))。 
4. 將 **CommandText 屬性** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard._multi_shard_statement)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand.commandtext.aspx#P:Microsoft.Azure.SqlDatabase.ElasticScale.Query.MultiShardCommand.CommandText)) 設定為 T-SQL 命令。
5. 呼叫 **ExecuteQueryAsync 或 ExecuteReader** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard._multi_shard_statement.executeQueryAsync)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand.executereader.aspx)) 方法，以執行命令。
6. 使用 **MultiShardResultSet 或 MultiShardDataReader** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard._multi_shard_result_set)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multisharddatareader.aspx)) 類別，以檢視結果。 

## <a name="example"></a>範例
下列程式碼說明如何使用給定的 **ShardMap** (名為 *myShardMap*) 來使用多分區查詢。 

```csharp
using (MultiShardConnection conn = new MultiShardConnection(myShardMap.GetShards(), myShardConnectionString)) 
{ 
    using (MultiShardCommand cmd = conn.CreateCommand())
    { 
        cmd.CommandText = "SELECT c1, c2, c3 FROM ShardedTable"; 
        cmd.CommandType = CommandType.Text; 
        cmd.ExecutionOptions = MultiShardExecutionOptions.IncludeShardNameColumn; 
        cmd.ExecutionPolicy = MultiShardExecutionPolicy.PartialResults; 

        using (MultiShardDataReader sdr = cmd.ExecuteReader()) 
        { 
            while (sdr.Read())
            { 
                var c1Field = sdr.GetString(0); 
                var c2Field = sdr.GetFieldValue<int>(1); 
                var c3Field = sdr.GetFieldValue<Int64>(2);
            } 
        } 
    } 
} 
```

主要差異是多分區連接的建構方式。 其中，**SqlConnection** 在單一資料庫上運作，**MultiShardConnection** 接受分區集合做為輸入。 從分區對應填入分區集合。 然後，使用 **UNION ALL** 語意在分區集合上執行查詢，以組成單一的整體結果。 (選擇性) 在命令上使用 **ExecutionOptions** 屬性，可將資料列的來源分區名稱加入至輸出。 

請注意 **myShardMap.GetShards()** 的呼叫。 這個方法會從分區對應擷取所有分區，並提供簡單的方式跨所有相關的資料庫執行查詢。 在呼叫 **myShardMap.GetShards()** 所傳回的集合分區上執行 LINQ 查詢，可以進一步調整多分區查詢的分區集合。 結合部分結果原則，多分區查詢目前的功能已設計成可適當處理數十個到數百個分區。

多分區查詢目前的限制在於無法驗證所查詢的分區和 Shardlet。 資料相依路由可在查詢時驗證給定的分區是屬於分區對應，但多分區查詢不會執行這項檢查。 這可能會導致在已從分區對應中移除的資料庫上執行多分區查詢。

## <a name="multi-shard-queries-and-split-merge-operations"></a>多分區查詢與分割合併作業
多分區查詢不會驗證所查詢資料庫上的 Shardlet 是否參與進行中的分割/合併作業。 (請參閱[使用彈性資料庫分割合併工具來縮放](sql-database-elastic-scale-overview-split-and-merge.md)。)這可能會導致不一致的情況，亦即在相同的多分區查詢中，多個資料庫顯示相同 Shardlet 的資料列。 請注意這些限制，在執行多分區查詢時，請考慮清空進行中的分割/合併作業和分區對應的變更。

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]


