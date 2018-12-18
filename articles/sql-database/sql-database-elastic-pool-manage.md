---
title: 建立和管理彈性集區 - Azure SQL Database | Microsoft Docs
description: 建立和管理 Azure SQL 彈性集區。
services: sql-database
ms.service: sql-database
subservice: elastic-pool
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: carlrab
manager: craigg
ms.date: 10/15/2018
ms.openlocfilehash: 6c43e5bf311d94ff2e5b5c31217c5071ea26746c
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49351857"
---
# <a name="create-and-manage-elastic-pools-in-azure-sql-database"></a>建立和管理 Azure SQL Database 中的彈性集區

使用彈性集區，您可以判斷彈性集區處理其資料庫工作負載所需的資源數量，以及適用於每個集區資料庫的資源數量。

## <a name="azure-portal-manage-elastic-pools-and-pooled-databases"></a>Azure 入口網站：管理彈性集區和集區資料庫

所有集區設定都可以在一個位置找到：[設定集區] 刀鋒視窗。 若要前往該位置，請在入口網站中尋找彈性集區，然後從刀鋒視窗頂端或左側的資源功能表按一下 [設定集區]。

您可以從這裡進行下列任何變更並將變更全部儲存在一個批次中：

1. 變更集區的服務層
2. 相應放大或縮小效能 (DTU 或虛擬核心) 和儲存體
3. 在集區中新增或移除資料庫
4. 為集區中的資料庫設定最小 (保證) 和最大效能限制
5. 檢閱成本摘要，以檢視因為您的新選擇所造成的任何帳單變更

![彈性集區組態刀鋒視窗](./media/sql-database-elastic-pool-manage-portal/configure-pool.png)

## <a name="powershell-manage-elastic-pools-and-pooled-databases"></a>PowerShell：管理彈性集區和集區資料庫

若要使用 Azure PowerShell 建立和管理 SQL Database 彈性集區和集區資料庫，請使用下列 PowerShell Cmdlet。 如果您需要安裝或升級 PowerShell，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 若要建立及管理彈性集區的邏輯伺服器，請參閱[建立及管理邏輯伺服器](sql-database-logical-servers.md)。 若要建立及管理防火牆規則，請參閱[使用 PowerShell 建立及管理防火牆規則](sql-database-firewall-configure.md#manage-firewall-rules-using-azure-powershell)。

> [!TIP]
> 如需 PowerShell 範例指令碼，請參閱[使用 PowerShell 建立彈性集區並在集區之間移動資料庫以及將其移出集區](scripts/sql-database-move-database-between-pools-powershell.md)和[使用 PowerShell 在 Azure SQL Database 中監視和調整 SQL 彈性集區](scripts/sql-database-monitor-and-scale-pool-powershell.md)。
>

| Cmdlet | 說明 |
| --- | --- |
|[New-AzureRmSqlElasticPool](/powershell/module/azurerm.sql/new-azurermsqlelasticpool)|在邏輯 SQL Server 建立彈性資料庫集區。|
|[Get-AzureRmSqlElasticPool](/powershell/module/azurerm.sql/get-azurermsqlelasticpool)|取得邏輯 SQL Server 上的彈性集區及其屬性值。|
|[Set-AzureRmSqlElasticPool](/powershell/module/azurerm.sql/set-azurermsqlelasticpool)|修改邏輯 SQL Server 上的彈性資料庫集區屬性。 例如，使用 **StorageMB** 屬性可修改彈性集區的最大儲存體。|
|[Remove-AzureRmSqlElasticPool](/powershell/module/azurerm.sql/remove-azurermsqlelasticpool)|刪除邏輯 SQL Server 上的彈性資料庫集區。|
|[Get-AzureRmSqlElasticPoolActivity](/powershell/module/azurerm.sql/get-azurermsqlelasticpoolactivity)|取得邏輯 SQL server 上的彈性集區作業狀態。|
|[New-AzureRmSqlDatabase](/powershell/module/azurerm.sql/new-azurermsqldatabase)|在現有的集區建立新的資料庫，或建立新的資料庫做為單一資料庫。 |
|[Get-AzureRmSqlDatabase](/powershell/module/azurerm.sql/get-azurermsqldatabase)|取得一或多個資料庫。|
|[Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase)|設定資料庫的屬性，或將現有資料庫移入彈性集區、移出彈性集區，或在彈性集區之間移動。|
|[Remove-AzureRmSqlDatabase](/powershell/module/azurerm.sql/remove-azurermsqldatabase)|移除資料庫。|

> [!TIP]
> 使用入口網站或一次只建立單一資料庫的 PowerShell Cmdlet 在彈性集區中建立許多資料庫可能需要花費一些時間。 若要自動建立成彈性集區，請參閱 [CreateOrUpdateElasticPoolAndPopulate](https://gist.github.com/billgib/d80c7687b17355d3c2ec8042323819ae)。

## <a name="azure-cli-manage-elastic-pools-and-pooled-databases"></a>Azure CLI：管理彈性集區和集區資料庫

若要使用 [Azure CLI](/cli/azure) 建立和管理 SQL Database 彈性集區，請使用下列 [Azure CLI SQL Database](/cli/azure/sql/db) 命令。 使用 [Cloud Shell](/azure/cloud-shell/overview) 在您的瀏覽器中執行 CLI，或在 macOS、Linux 或 Windows 中[安裝](/cli/azure/install-azure-cli)。

> [!TIP]
> 關於 Azure CLI 範例指令碼，請參閱[使用 CLI 移動 SQL 彈性集區中的 Azure SQL Database](scripts/sql-database-move-database-between-pools-cli.md)和[使用 Azure CLI 在 Azure SQL Database 中調整 SQL 彈性集區](scripts/sql-database-scale-pool-cli.md)。
>

| Cmdlet | 說明 |
| --- | --- |
|[az sql elastic-pool create](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-create)|建立彈性集區。|
|[az sql elastic-pool list](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-list)|傳回將伺服器中的彈性集區列出的清單。|
|[az sql elastic-pool list-dbs](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-list-dbs)|傳回將彈性集區中的資料庫列出的清單。|
|[az sql elastic-pool list-editions](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-list-editions)|也包含可用的集區 DTU 設定、儲存體限制，以及個別資料庫設定。 為了減少繁複度，額外的儲存空間限制和個別資料庫設定預設為隱藏。|
|[az sql elastic-pool update](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-update)|更新彈性集區。|
|[az sql elastic-pool delete](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-delete)|刪除彈性集區。|

## <a name="transact-sql-manage-pooled-databases"></a>Transact-SQL：管理集區資料庫

若要在現有彈性集區中建立並移動資料庫，或傳回 SQL Database 彈性集區與 Transact-SQL 的資訊，請使用下列 T-SQL 命令。 您可以使用 Azure 入口網站、[SQL Server Management Studio](/sql/ssms/use-sql-server-management-studio)、[Visual Studio Code](https://code.visualstudio.com/docs)，或任何可連線到 Azure SQL Database 伺服器並傳遞 Transact-SQL 命令的其他程式來發出這些命令。 若要使用 T-SQL 建立及管理防火牆規則，請參閱[使用 Transact-SQL 管理防火牆規則](sql-database-firewall-configure.md#manage-firewall-rules-using-transact-sql)。

> [!IMPORTANT]
> 您無法使用 Transact-SQL 建立、更新或刪除 Azure SQL Database 彈性集區。 您可以新增或移除彈性集區中的資料庫，也可以使用 DMV 傳回現有彈性集區的資訊。
>

| 命令 | 說明 |
| --- | --- |
|[CREATE DATABASE (Azure SQL Database)](/sql/t-sql/statements/create-database-azure-sql-database)|在現有的集區建立新的資料庫，或建立新的資料庫做為單一資料庫。 您必須連線到 master 資料庫才能建立新的資料庫。|
| [ALTER DATABASE (Azure SQL Database)](/sql/t-sql/statements/alter-database-azure-sql-database) |將資料庫移入彈性集區、將資料庫移出彈性集區，或在彈性集區之間移動資料庫。|
|[DROP DATABASE (Transact-SQL)](/sql/t-sql/statements/drop-database-transact-sql)|刪除資料庫。|
|[sys.elastic_pool_resource_stats (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database)|傳回邏輯伺服器中的所有彈性資料庫集區的資源使用量統計資料。 每個彈性資料庫集區，每 15 秒報告時間範圍會傳回一列 (每分鐘四列)。 包括集區中所有資料庫的 CPU、IO、記錄、儲存體使用情況和並行的要求/工作階段使用量。|
|[sys.database_service_objectives (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-database-service-objectives-azure-sql-database)|傳回 Azure SQL 資料庫或 Azure SQL 資料倉儲的版本 (服務層)、服務目標 (定價層) 和彈性集區名稱 (如果有的話)。 若已登入 Azure SQL Database 伺服器中的 master 資料庫，則傳回所有資料庫的相關資訊。 對於 Azure SQL 資料倉儲，您必須連線到 master 資料庫。|

## <a name="rest-api-manage-elastic-pools-and-pooled-databases"></a>REST API：管理彈性集區和集區資料庫

若要建立和管理 SQL Database 彈性集區和集區資料庫，請使用這些 REST API 要求。

| 命令 | 說明 |
| --- | --- |
|[彈性集區 - 建立或更新](https://docs.microsoft.com/rest/api/sql/elasticpools/elasticpools_createorupdate)|建立新的彈性集區或更新現有的彈性集區。|
|[彈性集區 - 刪除](https://docs.microsoft.com/rest/api/sql/elasticpools/elasticpools_delete)|刪除彈性集區。|
|[彈性集區 - 取得](https://docs.microsoft.com/rest/api/sql/elasticpools/elasticpools_get)|取得彈性集區。|
|[彈性集區 - 依伺服器列出](https://docs.microsoft.com/rest/api/sql/elasticpools/elasticpools_listbyserver)|傳回將伺服器中的彈性集區列出的清單。|
|[彈性集區 - 更新](https://docs.microsoft.com/rest/api/sql/elasticpools/elasticpools_listbyserver)|更新現有的彈性集區。|
|[彈性集區活動](https://docs.microsoft.com/rest/api/sql/elasticpoolactivities)|傳回彈性集區活動。|
|[彈性集區資料庫活動](https://docs.microsoft.com/rest/api/sql/elasticpooldatabaseactivities)|傳回資料庫內彈性集區上的活動。|
|[資料庫 - 建立或更新](https://docs.microsoft.com/rest/api/sql/databases/databases_createorupdate)|建立新的資料庫或更新現有資料庫。|
|[資料庫 - 取得](https://docs.microsoft.com/rest/api/sql/databases/databases_get)|取得資料庫。|
|[資料庫 - 依彈性集區列出](https://docs.microsoft.com/rest/api/sql/databases/databases_listbyelasticpool)|傳回將彈性集區中的資料庫列出的清單。|
|[資料庫 - 依伺服器列出](https://docs.microsoft.com/rest/api/sql/databases/databases_listbyserver)|傳回伺服器中的資料庫清單。|
|[資料庫 - 更新](https://docs.microsoft.com/rest/api/sql/databases/databases_update)|更新現有的資料庫。|

## <a name="next-steps"></a>後續步驟

* 若要觀賞影片，請參閱[有關 Azure SQL Database 彈性功能的 Microsoft Virtual Academy 視訊課程](https://mva.microsoft.com/training-courses/elastic-database-capabilities-with-azure-sql-db-16554)
* 若要深入了解使用彈性集區的 SaaS 應用程式的設計模式，請參閱 [採用 Azure SQL Database 的多租用戶 SaaS 應用程式的設計模式](sql-database-design-patterns-multi-tenancy-saas-applications.md)。
* 如需使用彈性集區的 SaaS 教學課程，請參閱 [Wingtip SaaS 應用程式簡介](sql-database-wtp-overview.md)。
