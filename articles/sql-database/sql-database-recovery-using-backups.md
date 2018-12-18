---
title: 從備份還原 Azure SQL Database | Microsoft Docs
description: 了解還原時間點，其可讓您復原 Azure SQL Database 到先前的時間 (最多 35 天)。
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: carlrab
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: 4c9edd60ffa1cd9ed5d95b37592fa49f44117818
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161330"
---
# <a name="recover-an-azure-sql-database-using-automated-database-backups"></a>使用自動資料庫備份復原 Azure SQL Database
SQL Database 針對使用[自動資料庫備份](sql-database-automated-backups.md)和[長期保留備份](sql-database-long-term-retention.md)進行資料庫復原，提供以下選項。 您可從資料庫備份還原至︰

* 同一部邏輯伺服器上的新資料庫，復原到保留期限內指定的時間點。 
* 同一部邏輯伺服器上的資料庫，復原到已刪除資料庫的刪除時間。
* 任何區域中任何邏輯伺服器上的新資料庫，其中資料庫會復原到異地複寫 Blob 儲存體 (RA-GRS) 中最新每日備份的時間點。

> [!IMPORTANT]
> 在還原期間，您無法覆寫現有的資料庫。
>

還原的資料庫會在下列情況產生額外的儲存體成本： 
- 如果資料庫大小上限大於 500 GB，會將 P11–P15 還原至 S4-S12 或 P1–P6。
- 如果資料庫大小上限大於 250 GB，會將 P1–P6 還原至 S4-S12。

額外成本是因為還原的資料庫大小上限大於該計算大小所含的儲存體數量，而佈建的任何額外儲存體若大於內含量則會產生額外費用。  如需有關額外儲存體的價格詳細資訊，請參閱 [SQL Database 價格頁面](https://azure.microsoft.com/pricing/details/sql-database/)。  如果實際的使用空間量小於內含的儲存體數量，則可將資料庫大小上限降低至內含量，以避免造成額外成本。  

> [!NOTE]
> 當您建立[資料庫複本](sql-database-copy.md)時，會使用[自動資料庫備份](sql-database-automated-backups.md)。 
>


## <a name="recovery-time"></a>復原時間
使用自動資料庫備份還原資料庫的復原時間會受到一些因素所影響： 

* 資料庫的大小
* 資料庫的計算大小
* 相關的交易記錄檔數目
* 需要重新執行以復原到還原點的活動數目
* 還原到不同區域的網路頻寬 
* 要在目標區域中處理的並行還原要求數目。 
  
  針對非常大型及/或作用中的資料庫，還原可能需要數小時。 如果某區域中的中斷延長，其他區域可能要處理大量的異地還原要求。 如果有許多要求，則該區域中資料庫的復原時間可能會增加。 大多數資料庫還原會在 12 小時內完成。

針對單一訂用帳戶，有關正在提交及處理的並行還原要求 (包含時間點還原、異地還原與自長期保留備份還原) 數目有一些限制：
|  | **正在處理的並行要求的最大數目** | **正在提交的並行要求的最大數目** |
| :--- | --: | --: |
|單一資料庫 (每個訂閱)|10|60|
|彈性集區 (每個集區)|4|200|
||||

沒有可執行大量還原的內建功能。 [Azure SQL Database: Full Server Recovery](https://gallery.technet.microsoft.com/Azure-SQL-Database-Full-82941666) 指令碼是其中一種完成這項工作的範例。

> [!IMPORTANT]
> 若要使用自動備份復原，您必須是訂用帳戶中 SQL Server 參與者角色的成員或訂用帳戶擁有者 - 請參閱 [RBAC：內建角色](../role-based-access-control/built-in-roles.md)。 您可以使用 Azure 入口網站、PowerShell 或 REST API 來進行復原。 您無法使用 Transact-SQL。 
> 

## <a name="point-in-time-restore"></a>還原時間點

您可以使用 Azure 入口網站、[PowerShell](https://docs.microsoft.com/powershell/module/azurerm.sql/restore-azurermsqldatabase) 或 [REST API](https://msdn.microsoft.com/library/azure/mt163685.aspx)，將現有的資料庫還原到先前的時間點，以做為相同邏輯伺服器上的新資料庫。 

> [!TIP]
> 如需示範如何執行資料庫還原時間點的 PowerShell 指令碼範例，請參閱[使用 PowerShell 還原 SQL 資料庫](scripts/sql-database-restore-database-powershell.md)。
>

資料庫可以還原到任何服務層或計算大小，以及以單一資料庫的形式還原或還原到彈性集區中。 請確定在作為您資料庫還原目的地的邏輯伺服器上或彈性集區中有足夠的資源。 完成之後，還原的資料庫會是一般、完全可供存取的線上資料庫。 還原的資料庫會根據其服務層和計算大小依一般費率計費。 在完成資料庫還原之前，不會產生任何費用。

基於復原目的，您通常會將資料庫還原到較早的時間點。 當您執行此動作時，可以將還原的資料庫視為原始資料庫的替代品，或用它來從原始資料庫擷取資料並將其更新。 

* ***資料庫取代：*** 如果要使用已還原的資料庫作為原始資料庫的替代品，您應該確認計算大小和/或服務層是適當的，並在必要時調整資料庫大小。 您可以重新命名原始資料庫，然後使用 T-SQL 中的 [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-database) 命令提供原始名稱給還原的資料庫。 
* ***資料復原︰*** 如果您打算從還原的資料庫擷取資料，以便從使用者或應用程式錯誤中復原，您就必須撰寫並執行所需的資料復原指令碼，以從還原的資料庫中擷取資料到原始資料庫。 雖然還原作業可能要花很長的時間才能完成，但還原中的資料庫在整個還原過程中都會顯示在資料庫清單上。 如果您在還原期間刪除該資料庫，系統便會取消還原作業，而且不會針對未完成還原的資料庫向您收費。 

### <a name="azure-portal"></a>Azure 入口網站

若要使用 Azure 入口網站復原至某個時間點，請開啟資料庫頁面，然後按一下工具列上的 [還原]。

![point-in-time-restore](./media/sql-database-recovery-using-backups/point-in-time-recovery.png)

## <a name="deleted-database-restore"></a>還原已刪除的資料庫
您可以使用 Azure 入口網站、[PowerShell](https://docs.microsoft.com/powershell/module/azurerm.sql/restore-azurermsqldatabase) 或 [REST (createMode=Restore)](https://msdn.microsoft.com/library/azure/mt163685.aspx)，將已刪除的資料庫還原到相同邏輯伺服器上已刪除資料庫的刪除時間。 您可以使用 [PowerShell](https://docs.microsoft.com/powershell/module/azurerm.sql/restore-azurermsqldatabase)，將已刪除的資料庫還原到保留期間內的較早時間點。

> [!TIP]
> 如需示範如何還原已刪除資料庫的 PowerShell 指令碼範例，請參閱[使用 PowerShell 還原 SQL 資料庫](scripts/sql-database-restore-database-powershell.md)。
>

> [!IMPORTANT]
> 如果您刪除 Azure SQL Database 伺服器執行個體，其所有資料庫也會一併刪除且無法加以復原。 目前不支援還原已刪除的伺服器。
> 

### <a name="azure-portal"></a>Azure 入口網站

若要使用 Azure 入口網站復原處於其[以 DTU 為基礎的模型保留期限](sql-database-service-tiers-dtu.md)或[以虛擬核心為基礎的模型保留期限](sql-database-service-tiers-vcore.md)的已刪除資料庫，請開啟伺服器頁面，然後在 [作業] 區域中按一下 [已刪除的資料庫]。

![deleted-database-restore-1](./media/sql-database-recovery-using-backups/deleted-database-restore-1.png)


![deleted-database-restore-2](./media/sql-database-recovery-using-backups/deleted-database-restore-2.png)

## <a name="geo-restore"></a>異地還原
您可以從最新的異地複寫完整備份和差異備份，在任何 Azure 區域中的任何伺服器上還原 SQL Database。 異地還原使用異地備援備份做為其來源，即使因為中斷而無法存取資料庫或資料中心，也能用來復原資料庫。 

當您的資料庫因為裝載資料庫區域中的事件而無法使用時，異地還原就是預設的復原選項。 如果區域中的大規模意外導致您無法使用資料庫應用程式，則您可以從異地複寫備份，將資料庫還原到任何其他區域中的伺服器。 在建立差異備份與將它以異地方式複寫至不同區域中的 Azure Blob 之間會有延遲。 此延遲可能最長達一小時，因此當發生災害時，最多可能會遺失最長達一小時的資料。 下圖顯示從另一個區域中的上次可用備份來還原資料庫。

![異地還原](./media/sql-database-geo-restore/geo-restore-2.png)

> [!TIP]
> 如需示範如何執行異地還原的 PowerShell 指令碼範例，請參閱[使用 PowerShell 還原 SQL 資料庫](scripts/sql-database-restore-database-powershell.md)。
> 

目前不支援異地次要資料庫上的時間點復原。 只有在主要資料庫上才可進行時間點復原。 如需使用異地還原來從中斷復原的詳細資訊，請參閱[從中斷復原](sql-database-disaster-recovery.md)。

> [!IMPORTANT]
> 從備份復原是 SQL Database 中最基本的災害復原解決方案，具備最長的復原點目標 (RPO) 和預估復原時間 (ERT)。 如果使用小型資料庫 (例如，基本服務層或彈性集區中的小型租用戶資料庫)，常見且合理的 DR 解決方案是異地還原，且這個解決方案具有 12 小時的 ERT。 若使用大型資料庫，且需要較短的復原時間，您應該考慮使用[容錯移轉群組和主動式異地複寫](sql-database-geo-replication-overview.md)的解決方案。 主動式異地複寫提供明顯較低的 RPO 和 ERT，因為它只需要您起始對連續複寫次要資料庫的容錯移轉。 如需商務持續性選項的詳細資訊，請參閱[商務持續性概觀](sql-database-business-continuity.md)。
> 

### <a name="azure-portal"></a>Azure 入口網站

若要使用 Azure 入口網站異地還原處於其[以 DTU 為基礎的模型保留期限](sql-database-service-tiers-dtu.md)或[以虛擬核心為基礎的模型保留期限](sql-database-service-tiers-vcore.md)的資料庫，請開啟 [SQL Database] 頁面，然後按一下 [新增]。 在 [選取來源] 文字方塊中，選取 [備份]。 指定要在區域和您選擇的伺服器上，執行復原的來源備份。 

## <a name="programmatically-performing-recovery-using-automated-backups"></a>使用自動備份以程式設計方式執行復原
如先前所述，除了 Azure 入口網站之外，還可使用 Azure PowerShell 或 REST API，以程式設計方式執行資料庫復原。 下表描述可用的命令集。

### <a name="powershell"></a>PowerShell
| Cmdlet | 說明 |
| --- | --- |
| [Get-AzureRmSqlDatabase](/powershell/module/azurerm.sql/get-azurermsqldatabase) |取得一或多個資料庫。 |
| [Get-AzureRMSqlDeletedDatabaseBackup](/powershell/module/azurerm.sql/get-azurermsqldeleteddatabasebackup) | 取得可還原的已刪除資料庫。 |
| [Get-AzureRmSqlDatabaseGeoBackup](/powershell/module/azurerm.sql/get-azurermsqldatabasegeobackup) |取得資料庫的異地備援備份。 |
| [Restore-AzureRmSqlDatabase](/powershell/module/azurerm.sql/restore-azurermsqldatabase) |還原 SQL Database。 |
|  | |

### <a name="rest-api"></a>REST API
| API | 說明 |
| --- | --- |
| [REST (createMode=Recovery)](https://msdn.microsoft.com/library/azure/mt163685.aspx) |還原資料庫 |
| [取得建立或更新資料庫狀態](https://msdn.microsoft.com/library/azure/mt643934.aspx) |在還原作業期間傳回狀態 |
|  | |

## <a name="summary"></a>總結
自動備份可在發生使用者和應用程式錯誤、意外刪除資料庫和長時間中斷時保護您的資料庫。 所有服務層和計算大小都可以取得此內建功能。 

## <a name="next-steps"></a>後續步驟
* 如需商務持續性概觀和案例，請參閱[商務持續性概觀](sql-database-business-continuity.md)。
* 若要了解 Azure SQL Database 自動備份，請參閱 [SQL Database 自動備份](sql-database-automated-backups.md)。
* 若要深入了解長期保留，請參閱[長期保留](sql-database-long-term-retention.md)。
* 若要了解更快速的復原選項，請參閱[容錯移轉群組和主動式異地複寫](sql-database-geo-replication-overview.md)。  
