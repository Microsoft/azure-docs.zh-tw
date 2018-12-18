---
title: 分區對應管理員的效能計數器
description: ShardMapManager 類別與資料相依路由效能計數器
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 03/31/2018
ms.openlocfilehash: d4ecfe700c90beb94455e3607cee4ea30227bd0e
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166212"
---
# <a name="performance-counters-for-shard-map-manager"></a>分區對應管理員的效能計數器
您可以擷取[分區對應管理員](sql-database-elastic-scale-shard-map-management.md)的效能，特別是在使用[資料相依路由](sql-database-elastic-scale-data-dependent-routing.md)時。 計數器是使用 Microsoft.Azure.SqlDatabase.ElasticScale.Client 類別的方法建立。  

計數器是用來追蹤 [資料相依路由](sql-database-elastic-scale-data-dependent-routing.md) 作業的效能。 這些計數器可在「彈性資料庫：分區管理」類別下的「效能監視器」中存取。

**如需最新版本** ：請瀏覽 [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)。 另請參閱 [將應用程式升級以使用最新的彈性資料庫用戶端程式庫](sql-database-elastic-scale-upgrade-client-library.md)。

## <a name="prerequisites"></a>必要條件
* 若要建立效能類別和計數器，使用者必須屬於裝載應用程式的電腦上的本機 **Administrators** 群組。  
* 若要建立效能計數器執行個體和更新計數器，使用者必須是 **Administrators** 或 **Performance Monitor Users** 群組的成員。 

## <a name="create-performance-category-and-counters"></a>建立效能類別和計數器
若要建立計數器，請呼叫 [ShardMapManagmentFactory 類別](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.aspx)的 CreatePeformanceCategoryAndCounters 方法。 只有系統管理員可以執行這個方法︰ 

    ShardMapManagerFactory.CreatePerformanceCategoryAndCounters()  

您也可以使用 [這個](https://gallery.technet.microsoft.com/scriptcenter/Elastic-DB-Tools-for-Azure-17e3d283) PowerShell 指令碼來執行方法。 這個方法會建立下列效能計數器︰  

* **快取對應**︰針對分區對應快取的對應數目。
* **DDR 作業數/秒**︰分區對應的資料相依路由作業速率。 對 [OpenConnectionForKey()](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey.aspx) 的呼叫導致順利連接到目的地分區時，就會更新此計數器。 
* **對應查閱快取點擊/秒**︰分區對應中對應的成功快取查閱作業的速率。 
* **對應查閱快取遺失/秒**︰分區對應中對應的失敗快取查閱作業的速率。
* **快取中的對應新增或更新/秒**︰分區對應的快取中新增或更新對應的速率。 
* **從快取中移除對應/秒**︰從分區對應的快取中移除對應的速率。 

效能計數器會在每個程序針對每個快取的分區對應建立。  

## <a name="notes"></a>注意
下列事件會觸發效能計數器的建立︰  

* 在 ShardMapManager 包含任何分區對應的情況下，使用[積極式載入](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerloadpolicy.aspx)將 [ShardMapManager](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.aspx) 初始化。 這些包括 [GetSqlShardMapManager](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager.aspx?f=255&MSPPError=-2147217396#M:Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.ShardMapManagerFactory.GetSqlShardMapManager%28System.String,Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.ShardMapManagerLoadPolicy%29) 和 [TryGetSqlShardMapManager](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager.aspx) 方法。
* 成功查閱分區對應 (使用 [GetShardMap()](https://msdn.microsoft.com/library/azure/dn824215.aspx)、[GetListShardMap()](https://msdn.microsoft.com/library/azure/dn824212.aspx) 或 [GetRangeShardMap()](https://msdn.microsoft.com/library/azure/dn824173.aspx))。 
* 使用 CreateShardMap() 成功建立分區對應。

效能計數器會由分區對應和對應上執行的所有快取作業更新。 使用 DeleteShardMap() 導致刪除效能計數器執行個體，成功移除分區對應。  

## <a name="best-practices"></a>最佳作法
* 建立效能類別和計數器應該僅在建立 ShardMapManager 物件之前執行一次。 每次執行命令 CreatePerformanceCategoryAndCounters() 都會清除先前的計數器 (遺失所有執行個體報告的資料)，並建立新的計數器。  
* 每個程序都會建立效能計數器執行個體。 任何應用程式當機或從快取移除分區對應都會導致刪除效能計數器執行個體。  

### <a name="see-also"></a>另請參閱
[彈性資料庫功能概觀](sql-database-elastic-scale-introduction.md)  

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Anchors-->
<!--Image references-->

