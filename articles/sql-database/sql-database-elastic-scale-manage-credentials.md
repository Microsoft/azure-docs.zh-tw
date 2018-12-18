---
title: 在彈性資料庫用戶端程式庫中管理認證 | Microsoft Docs
description: 如何設定彈性資料庫應用程式的正確認證層級 (管理員到唯讀)
services: sql-database
manager: craigg
author: stevestein
ms.service: sql-database
ms.custom: scale out apps
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: sstein
ms.openlocfilehash: 3a371a2c055ed2d5c3c5c2ddf825bea4ad7e33f0
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34646317"
---
# <a name="credentials-used-to-access-the-elastic-database-client-library"></a>用來存取彈性資料庫用戶端程式庫的認證
[彈性資料庫用戶端程式庫](sql-database-elastic-database-client-library.md)使用三種不同的認證存取[分區對應管理員](sql-database-elastic-scale-shard-map-management.md)。 視需要使用盡可能低的存取層級認證。

* **管理認證**：建立或操作分區對應管理員。 (請參閱[詞彙](sql-database-elastic-scale-glossary.md)。) 
* **存取認證**：存取現有的分區對應管理員，以取得分區的相關資訊。
* **連接認證**：連接到分區。 

另請參閱 [在 Azure SQL Database 中管理資料庫與登入](sql-database-manage-logins.md)。 

## <a name="about-management-credentials"></a>關於管理認證
管理認證會用來建立操作分區對應之應用程式的 **ShardMapManager** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager._shard_map_manager)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.aspx)) 物件。 (例如，請參閱[使用彈性資料庫工具新增分區](sql-database-elastic-scale-add-a-shard.md)和[資料相依路由](sql-database-elastic-scale-data-dependent-routing.md))。 彈性級別用戶端程式庫的使用者可建立 SQL 使用者和 SQL 登入，也要確定將全域分區對應資料庫及所有分區資料庫的讀取/寫入權限授與給每個人。 在分區對應上執行變更時，這些認證用來維護全域分區對應和本機分區對應。 例如，使用管理認證來建立分區對應管理員物件 (使用 **GetSqlShardMapManager** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager._shard_map_manager_factory.getsqlshardmapmanager)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager.aspx))： 

```
// Obtain a shard map manager. 
ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(smmAdminConnectionString,ShardMapManagerLoadPolicy.Lazy); 
```

變數 **smmAdminConnectionString** 是包含管理認證的連接字串。 使用者識別碼和密碼提供分區對應資料庫，以及個別分區的讀取/寫入存取權。 管理連接字串也包含伺服器名稱和資料庫名稱，以識別全域分區對應資料庫。 以下是針對該用途的一般連接字串：

```
"Server=<yourserver>.database.windows.net;Database=<yourdatabase>;User ID=<yourmgmtusername>;Password=<yourmgmtpassword>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30;” 
```

請勿使用 "username@server" 格式的值，而只要使用 "username" 值。  這是因為認證必須同時適用於入分區對應管理員資料庫和個別分區 (可能位於不同的伺服器上)。

## <a name="access-credentials"></a>存取認證
在不管理分區對應的應用程式中建立分區對應管理員時，請使用在全域分區對應上具有唯讀權限的認證。 在這些認證下，從全域分區對應所擷取的資訊用於[資料相依路由](sql-database-elastic-scale-data-dependent-routing.md)，以及填入用戶端的分區對應快取。 認證是透過 **GetSqlShardMapManager** 的相同呼叫模式來提供： 

```
// Obtain shard map manager. 
ShardMapManager shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(smmReadOnlyConnectionString, ShardMapManagerLoadPolicy.Lazy);  
```

請注意，我們使用 **smmReadOnlyConnectionString**，以反映使用不同的認證以代表**非管理員**使用者進行此存取：這些認證不應提供全域分區對應上的寫入權限。 

## <a name="connection-credentials"></a>連接認證
使用 **OpenConnectionForKey** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapper._list_shard_mapper.openconnectionforkey)、[.NET](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey.aspx)) 方法來存取與分區索引鍵相關聯的分區時，需要額外的認證。 這些認證必須提供唯讀權限來存取分區上的本機分區對應資料表。 在分區上執行資料相依路由的連線驗證時，這是必要的。 此程式碼片段允許資料相依路由內容中的資料存取： 

```csharp
using (SqlConnection conn = rangeMap.OpenConnectionForKey<int>(targetWarehouse, smmUserConnectionString, ConnectionOptions.Validate)) 
```

在此範例中， **smmUserConnectionString** 保留使用者認證的連接字串。 在 Azure SQL DB 中，以下是使用者認證的一般連接字串： 

```
"User ID=<yourusername>; Password=<youruserpassword>; Trusted_Connection=False; Encrypt=True; Connection Timeout=30;”  
```

與系統管理員認證情況相同，請勿使用 "username@server" 格式的值。 請改為只使用 "username"。  另請注意，連接字串不包含伺服器名稱和資料庫名稱。 這是因為 **OpenConnectionForKey** 呼叫會以索引鍵作為基礎，自動將連線導向正確的分區。 因此，不會提供資料庫名稱和伺服器名稱。 

## <a name="see-also"></a>另請參閱
[管理 Azure SQL Database 的資料庫和登入](sql-database-manage-logins.md)

[保護您的 SQL Database](sql-database-security-overview.md)

[開始使用彈性資料庫工作](sql-database-elastic-jobs-getting-started.md)

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

