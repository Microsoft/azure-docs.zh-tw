---
title: 轉換現有的資料庫以相應放大 | Microsoft Docs
description: 建立分區對應管理員來轉換分區化資料庫，以使用彈性資料庫工具
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
ms.date: 04/01/2018
ms.openlocfilehash: e5039e299df30df4d49f24430af4b44837d65c44
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166352"
---
# <a name="migrate-existing-databases-to-scale-out"></a>將現有的資料庫移轉到相應放大的資料庫
使用 Azure SQL Database 資料庫工具 (例如 [彈性資料庫用戶端程式庫](sql-database-elastic-database-client-library.md))，輕鬆地管理現有相應放大的分區化資料庫。 請先轉換現有的資料庫，才能使用[分區對應管理員](sql-database-elastic-scale-shard-map-management.md)。 

## <a name="overview"></a>概觀
若要移轉現有的分區化資料庫︰ 

1. 準備 [分區對應管理員資料庫](sql-database-elastic-scale-shard-map-management.md)。
2. 建立分區對應。
3. 準備個別分區。  
4. 將對應新增至分區對應。

您可以使用 [.NET Framework 用戶端程式庫](http://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)或在 [Azure SQL DB - 彈性資料庫工具指令碼](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db)中找到的 PowerShell 指令碼，來實作這些技巧。 以下範例會使用 PowerShell 指令碼。

如需 ShardMapManager 的詳細資訊，請參閱 [分區對應管理](sql-database-elastic-scale-shard-map-management.md)。 如需彈性資料庫工具的概觀，請參閱 [彈性資料庫功能概觀](sql-database-elastic-scale-introduction.md)。

## <a name="prepare-the-shard-map-manager-database"></a>準備分區對應管理員資料庫
分區對應管理員是一個特殊的資料庫，其中包含用以管理相應放大之資料庫的資料。 您可以使用現有的資料庫，或建立新的資料庫。 作為分區對應管理員的資料庫不應該是與分區相同的資料庫。 PowerShell 指令碼不會為您建立資料庫。 

## <a name="step-1-create-a-shard-map-manager"></a>步驟 1︰建立分區對應管理員
    # Create a shard map manager. 
    New-ShardMapManager -UserName '<user_name>' 
    -Password '<password>' 
    -SqlServerName '<server_name>' 
    -SqlDatabaseName '<smm_db_name>' 
    #<server_name> and <smm_db_name> are the server name and database name 
    # for the new or existing database that should be used for storing 
    # tenant-database mapping information.

### <a name="to-retrieve-the-shard-map-manager"></a>擷取分區對應管理員
在建立之後，您可以使用 Cmdlet 擷取分區對應管理員。 每次需要使用 ShardMapManager 物件，則需要此步驟。

    # Try to get a reference to the Shard Map Manager  
    $ShardMapManager = Get-ShardMapManager -UserName '<user_name>' 
    -Password '<password>' 
    -SqlServerName '<server_name>' 
    -SqlDatabaseName '<smm_db_name>' 


## <a name="step-2-create-the-shard-map"></a>步驟 2︰建立分區對應
選取要建立的分區對應類型。 請依據資料庫結構進行選擇︰ 

1. 每個資料庫有一個租用戶 (相關詞彙，請參閱 [詞彙](sql-database-elastic-scale-glossary.md))。 
2. 每個資料庫有多個租用戶 (兩種類型)︰
   1. 清單對應
   2. 範圍對應

針對單一租用戶模型，建立 **清單對應** 分區對應。 單一租用戶模型會指派每個租用戶一個資料庫。 這是適用於 SaaS 開發人員的有效模式，因為它會簡化管理。

![清單對應][1]

多租用戶模型會將數個租用戶指派給單一資料庫 (而且您可以跨多個資料庫散發租用戶的群組)。 預期每個租用戶有小型資料需求時，請使用此模型。 在此模型中，使用**範圍對應**將某範圍的租用戶指派給資料庫。 

![範圍對應][2]

或者，您可以使用「清單對應」  來實作多租用戶資料庫模型，以將多個租用戶指派給單一資料庫。 例如，DB1 是用來儲存租用戶 ID 1 和 5 的相關資訊，而 DB2 是用來儲存租用戶 7 和租用戶 10 的資料。 

![單一資料庫上的多個租用戶][3] 

**根據您的選擇，選擇下列其中一個選項︰**

### <a name="option-1-create-a-shard-map-for-a-list-mapping"></a>選項 1︰建立清單對應的分區對應
使用 ShardMapManager 物件建立分區對應。 

    # $ShardMapManager is the shard map manager object. 
    $ShardMap = New-ListShardMap -KeyType $([int]) 
    -ListShardMapName 'ListShardMap' 
    -ShardMapManager $ShardMapManager 


### <a name="option-2-create-a-shard-map-for-a-range-mapping"></a>選項 2︰建立範圍對應的分區對應
若要利用此對應模式，租用戶 ID 值需要是連續的範圍，而且可接受範圍中有間距，方法為在建立資料庫時略過範圍。

    # $ShardMapManager is the shard map manager object 
    # 'RangeShardMap' is the unique identifier for the range shard map.  
    $ShardMap = New-RangeShardMap 
    -KeyType $([int]) 
    -RangeShardMapName 'RangeShardMap' 
    -ShardMapManager $ShardMapManager 

### <a name="option-3-list-mappings-on-a-single-database"></a>選項 3︰單一資料庫上的清單對應
設定此模式也需要建立清單對應，如步驟 2，選項 1 所示。

## <a name="step-3-prepare-individual-shards"></a>步驟 3︰準備個別分區
將每個分區 (資料庫) 新增至分區對應管理員。 這會準備個別資料庫以儲存對應資訊。 在每個分區上執行此方法。

    Add-Shard 
    -ShardMap $ShardMap 
    -SqlServerName '<shard_server_name>' 
    -SqlDatabaseName '<shard_database_name>'
    # The $ShardMap is the shard map created in step 2.


## <a name="step-4-add-mappings"></a>步驟 4︰新增對應
新增對應取決於您所建立的分區對應種類。 如果已建立清單對應，則會新增清單對應。 如果已建立範圍對應，則會新增範圍對應。

### <a name="option-1-map-the-data-for-a-list-mapping"></a>選項 1︰對應清單對應的資料
新增清單對應來對應每個租用戶的資料。  

    # Create the mappings and associate it with the new shards 
    Add-ListMapping 
    -KeyType $([int]) 
    -ListPoint '<tenant_id>' 
    -ListShardMap $ShardMap 
    -SqlServerName '<shard_server_name>' 
    -SqlDatabaseName '<shard_database_name>' 

### <a name="option-2-map-the-data-for-a-range-mapping"></a>選項 2︰對應範圍對應的資料
新增所有租用戶識別碼範圍的範圍對應 - 資料庫關聯︰

    # Create the mappings and associate it with the new shards 
    Add-RangeMapping 
    -KeyType $([int]) 
    -RangeHigh '5' 
    -RangeLow '1' 
    -RangeShardMap $ShardMap 
    -SqlServerName '<shard_server_name>' 
    -SqlDatabaseName '<shard_database_name>' 


### <a name="step-4-option-3-map-the-data-for-multiple-tenants-on-a-single-database"></a>步驟 4 選項 3︰對應單一資料庫上多個租用戶的資料
對於每個租用戶，執行 Add-ListMapping (選項 1)。 

## <a name="checking-the-mappings"></a>檢查對應
您可以使用下列命令來查詢現有分區以及與其相關聯之對應的相關資訊 ︰  

    # List the shards and mappings 
    Get-Shards -ShardMap $ShardMap 
    Get-Mappings -ShardMap $ShardMap 

## <a name="summary"></a>總結
一旦完成安裝後，您就可以開始使用彈性資料庫用戶端程式庫。 您也可以使用[資料相依路由](sql-database-elastic-scale-data-dependent-routing.md)和[多分區查詢](sql-database-elastic-scale-multishard-querying.md)。

## <a name="next-steps"></a>後續步驟
從 [Azure SQL DB 彈性資料庫工具指令碼](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db)取得 PowerShell 指令碼。

這些工具也會在 GitHub 上︰ [Azure/elastic-db-tools](https://github.com/Azure/elastic-db-tools)。

使用分割合併工具，在多租用戶模型與單一租用戶模型之間來回移動資料。 請參閱 [分割合併工具](sql-database-elastic-scale-get-started.md)。

## <a name="additional-resources"></a>其他資源
如需多租用戶型軟體即服務 (SaaS) 資料庫應用程式的常見資料架構模式的資訊，請參閱 [多租用戶 SaaS 應用程式與 Azure SQL Database 的設計模式](sql-database-design-patterns-multi-tenancy-saas-applications.md)。

## <a name="questions-and-feature-requests"></a>問題和功能要求
如有問題，請使用 [SQL Database 論壇](http://social.msdn.microsoft.com/forums/azure/home?forum=ssdsgetstarted)，如需要求增加功能，請將這些功能新增至 [SQL Database 意見反應論壇](https://feedback.azure.com/forums/217321-sql-database/)。

<!--Image references-->
[1]: ./media/sql-database-elastic-convert-to-use-elastic-tools/listmapping.png
[2]: ./media/sql-database-elastic-convert-to-use-elastic-tools/rangemapping.png
[3]: ./media/sql-database-elastic-convert-to-use-elastic-tools/multipleonsingledb.png

