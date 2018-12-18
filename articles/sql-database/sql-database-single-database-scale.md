---
title: 調整單一資料庫資源 - Azure SQL Database | Microsoft Docs
description: 本文說明如何在 Azure SQL Database 中調整單一資料庫可用的計算和儲存資源。
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 09/28/2018
ms.openlocfilehash: 2c4f772bbf4f0a5bb6eb2323eefa906fba0e5653
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451404"
---
# <a name="scale-single-database-resources-in-azure-sql-database"></a>在 Azure SQL Database 中調整單一資料庫資源

本文說明如何在 Azure SQL Database 中調整單一資料庫可用的計算和儲存資源。 

## <a name="vcore-based-purchasing-model-change-storage-size"></a>以虛擬核心為基礎的購買模型：變更儲存體大小

- 可以使用 1GB 為增量單位，將儲存體佈建到大小上限。 資料儲存體的最小可設定值是 5 GB 
- 藉由使用 [Azure 入口網站](https://portal.azure.com)、[Transact-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current#examples-1)、[PowerShell](/powershell/module/azurerm.sql/set-azurermsqldatabase)、[Azure CLI](/cli/azure/sql/db#az-sql-db-update) 或 [REST API](/rest/api/sql/databases/update) 增加或減少其大小上限，即可佈建單一資料庫的儲存體。
- SQL Database 會自動為記錄檔配置 30% 的額外儲存體，為 TempDB 的每個虛擬核心配置 32GB 儲存體，但不超過 384GB。 TempDB 位於所有服務層中的已連結 SSD 上。
- 單一資料庫的儲存體價格為資料儲存體和記錄儲存體數量的總和乘以服務層的儲存體單價。 TempDB 的成本包含在虛擬核心價格內。 如需有關額外儲存體的價格詳細資訊，請參閱 [SQL Database 定價](https://azure.microsoft.com/pricing/details/sql-database/)。

> [!IMPORTANT]
> 在某些情況下，您可能需要壓縮資料庫來回收未使用的空間。 如需詳細資訊，請參閱[管理 Azure SQL Database 中的檔案空間](sql-database-file-space-management.md)。

## <a name="vcore-based-purchasing-model-change-compute-resources"></a>以虛擬核心為基礎的購買模型：變更計算資源

一開始選取虛擬核心數目之後，您可以根據實際經驗，使用 [Azure 入口網站](sql-database-single-databases-manage.md#manage-an-existing-sql-server)、[Transact-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current#examples-1)、[PowerShell](/powershell/module/azurerm.sql/set-azurermsqldatabase)、[Azure CLI](/cli/azure/sql/db#az-sql-db-update) 或 [REST API](/rest/api/sql/databases/update) 來動態地相應增加或相應減少單一資料庫。 

變更資料庫的服務層和/或運算大小會以新的運算大小建立原始資料庫的複本，然後將連線切換到複本。 此程序期間不會遺失任何資料，但在我們切換到複本的短暫期間，資料庫的連接會停用，因此執行中的某些交易可能會回復。 切換的時間長度會有所不同，但通常是在 4 秒以下，而 99% 的時間會小於 30 秒。 如果在連線停用時正在執行大型交易，則切換的時間長度可能會更長。 

整個向上調整程序的期間取決於資料庫變更前後的大小和服務層。 例如，在一般用途服務層內進行變更的 250 GB 資料庫應在六小時內完成。 對於業務關鍵服務層內變更計算大小的相同大小資料庫，應該在三小時內完成相應增加。

> [!TIP]
> 若要監視進行中的作業，請參閱：[使用 SQL REST API 管理作業](/rest/api/sql/Operations/List)、[使用 CLI 管理作業](/cli/azure/sql/db/op)、[使用 T-SQL 監視作業](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database)，以及下列兩個 PowerShell 命令：[Get-AzureRmSqlDatabaseActivity](/powershell/module/azurerm.sql/get-azurermsqldatabaseactivity) 和 [Stop-AzureRmSqlDatabaseActivity](/powershell/module/azurerm.sql/stop-azurermsqldatabaseactivity)。

* 如果您要升級到較高服務層或計算大小，除非明確指定較大的大小 (大小上限)，否則資料庫大小上限不會增加。
* 若要將資料庫降級，資料庫已用的空間必須小於目標服務層和運算大小允許的大小上限。 
* 升級資料庫時，若已啟用[異地複寫](sql-database-geo-replication-portal.md)，您必須先將其次要資料庫升級為所需的服務層與計算大小，然後再升級主要資料庫 (最佳效能的一般指引)。 升級至不同的版本時，必須先升級次要資料庫。
* 當您在啟用[異地複寫](sql-database-geo-replication-portal.md)的情況下將資料庫降級時，必須先將其主要資料庫降級為所需的服務層與計算大小，然後再將次要資料庫降級 (最佳效能的一般指引)。 降級至不同的版本時，必須先降級主要資料庫。
* 完成變更之前，不會將新屬性套用至資料庫。

## <a name="dtu-based-purchasing-model-change-storage-size"></a>以 DTU 為基礎的購買模型：變更儲存體大小

- 單一資料庫的 DTU 價格包含一定數量不額外收費的儲存體。 佈建超過內含量的額外儲存體會產生額外費用，以 250 GB 為單位最多增加到大小上限 1 TB，超過 1 TB 則以 256 GB 為單位增加。 如需了解內含儲存體數量和大小上限，請參閱[單一資料庫：儲存體大小與計算大小](sql-database-dtu-resource-limits-single-databases.md#single-database-storage-sizes-and-compute-sizes)。
- 可以透過使用 Azure 入口網站、[Transact-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current#examples-1)、[PowerShell](/powershell/module/azurerm.sql/set-azurermsqldatabase)、[Azure CLI](/cli/azure/sql/db#az-sql-db-update) 或 [REST API](/rest/api/sql/databases/update) 增加其大小上限，以佈建單一資料庫的額外儲存體。
- 單一資料庫之額外儲存體的價格為額外儲存體數量乘以服務層的額外儲存體單價。 如需有關額外儲存體的價格詳細資訊，請參閱 [SQL Database 定價](https://azure.microsoft.com/pricing/details/sql-database/)。

> [!IMPORTANT]
> 在某些情況下，您可能需要壓縮資料庫來回收未使用的空間。 如需詳細資訊，請參閱[管理 Azure SQL Database 中的檔案空間](sql-database-file-space-management.md)。

## <a name="dtu-based-purchasing-model-change-compute-resources-dtus"></a>以 DTU 為基礎的購買模型：變更計算資源 (DTU)

一開始選取服務層、計算大小和儲存體數量之後，您可以根據實際經驗，使用 Azure 入口網站、[Transact-SQL](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current#examples-1)、[PowerShell](/powershell/module/azurerm.sql/set-azurermsqldatabase)、[Azure CLI](/cli/azure/sql/db#az-sql-db-update) 或 [REST API](/rest/api/sql/databases/update) 來動態地相應增加或相應減少單一資料庫。 

下列影片示範了如何動態變更服務層與計算大小，以增加單一資料庫的可用 DTU。

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Database-dynamically-scale-up-or-scale-down/player]
>

變更資料庫的服務層和/或運算大小會以新的運算大小建立原始資料庫的複本，然後將連線切換到複本。 此程序期間不會遺失任何資料，但在我們切換到複本的短暫期間，資料庫的連接會停用，因此執行中的某些交易可能會回復。 切換的時間長度會有所不同，但 99% 的時間會小於 30 秒。 如果在連線停用時正在執行大型交易，則切換的時間長度可能會更長。 

整個向上調整程序的期間取決於資料庫變更前後的大小和服務層。 例如，在標準服務層內進行變更的 250 GB 資料庫應在六小時內完成。 對於進階服務層內變更計算大小的相同大小資料庫，應該在三小時內完成相應增加。

> [!TIP]
> 若要監視進行中的作業，請參閱：[使用 SQL REST API 管理作業](/rest/api/sql/Operations/List)、[使用 CLI 管理作業](/cli/azure/sql/db/op)、[使用 T-SQL 監視作業](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database)，以及下列兩個 PowerShell 命令：[Get-AzureRmSqlDatabaseActivity](/powershell/module/azurerm.sql/get-azurermsqldatabaseactivity) 和 [Stop-AzureRmSqlDatabaseActivity](/powershell/module/azurerm.sql/stop-azurermsqldatabaseactivity)。

* 如果您要升級到較高服務層或計算大小，除非明確指定較大的大小 (大小上限)，否則資料庫大小上限不會增加。
* 若要將資料庫降級，資料庫已用的空間必須小於目標服務層和運算大小允許的大小上限。 
* 從**進階**降級至**標準**層時，如果發生下列情況，將會產生額外的儲存體成本：(1) 在目標計算大小中以資料庫的大小上限進行支援作業，而且 (2) 大小上限超過目標計算大小的內含儲存體數量。 例如，如果大小上限為 500 GB 的 P1 資料庫縮減為 S3，即會產生額外的儲存體成本，因為 S3 可支援 500 GB 的大小上限，而且其內含儲存體數量只有 250 GB。 因此，額外的儲存體數量為 500 GB – 250 GB = 250 GB。 如需額外儲存體的價格詳細資訊，請參閱 [SQL Database 定價](https://azure.microsoft.com/pricing/details/sql-database/)。 如果實際的使用空間量小於內含的儲存體數量，則可將資料庫大小上限降低至內含量，以避免造成額外成本。 
* 升級資料庫時，若已啟用[異地複寫](sql-database-geo-replication-portal.md)，您必須先將其次要資料庫升級為所需的服務層與計算大小，然後再升級主要資料庫 (最佳效能的一般指引)。 升級至不同的版本時，必須先升級次要資料庫。
* 當您在啟用[異地複寫](sql-database-geo-replication-portal.md)的情況下將資料庫降級時，必須先將其主要資料庫降級為所需的服務層與計算大小，然後再將次要資料庫降級 (最佳效能的一般指引)。 降級至不同的版本時，必須先降級主要資料庫。
* 還原服務會針對各種服務層提供不同的供應項目。 如果降級至**基本**層，會有較短的備份保留期 - 請參閱 [Azure SQL Database 備份](sql-database-automated-backups.md)。
* 完成變更之前，不會將新屬性套用至資料庫。

## <a name="dtu-based-purchasing-model-limitations-of-p11-and-p15-when-the-maximum-size-greater-than-1-tb"></a>以 DTU 為基礎的購買模型：當大小上限大於 1 TB 時，P11 和 P15 的限制

下列區域支援大小上限大於 1 TB 的 P11 和 P15 資料庫：澳大利亞東部、澳大利亞東南部、巴西南部、加拿大中部、加拿大東部、美國中部、法國中部、德國中部、日本東部、日本西部、南韓中部、美國中北部、北歐、美國中南部、東南亞、英國南部、英國西部、美國東部 2、美國西部、美國維吉尼亞州政府及西歐。 下列考量與限制適用於大小上限大於 1 TB 的 P11 和 P15 資料庫：

- 如果您在建立資料庫時選擇的大小上限大於 1 TB (使用的值為 4 TB 或 4096 GB)，但資料庫佈建在不受支援的區域，create 命令會失敗並產生錯誤。
- 若現有的 P11 和 P15 資料庫位於其中一個受支援區域，則能以 256 GB 為單位，將儲存體上限提高到 1 TB 以上，最多可增加到 4 TB。 若要查看您所在的區域是否支援較大的大小，請使用 [DATABASEPROPERTYEX](/sql/t-sql/functions/databasepropertyex-transact-sql) 函數或在 Azure 入口網站中檢查資料庫大小。 只有伺服器層級的主要登入或 dbmanager 資料庫角色的成員，才能執行現有的 P11 或 P15 資料庫升級。 
- 如果是在支援的區域中執行升級作業，則系統會立即更新組態。 在升級過程中，資料庫會保持離線狀態。 但是，在將實際的資料庫檔案升級至新的大小上限前，您無法完整地使用超過 1 TB 的儲存體。 所需的時間長短依進行升級的資料庫大小而定。 
- 建立或更新 P11 或 P15 資料庫時，您只能以 256 GB 為增加單位，在 1 TB 和 4 TB 的大小上限之間選擇。 在建立 P11/P15 時，系統會預先選取預設的 1 TB 儲存體選項。 若資料庫位於其中一個受支援區域，您可以將新的或現有單一資料庫的儲存體上限增加為 4 TB。 在其他所有區域中，無法將大小上限增加至 1 TB 以上。 當您選取 4 TB 的內含儲存體時，價格不會變更。
- 如果資料庫的大小上限設為大於 1 TB，即使使用的實際儲存體未達 1 TB，也無法變更為 1 TB。 因此，您無法將大小上限大於 1 TB 的 P11 或 P15 降級至 1 TB P11 或 1 TB P15，也無法降至較低的計算大小，例如 P1-P6。 這項限制也適用於包括時間點、異地還原、長期備份保留期和資料庫備份等還原和複製案例。 一旦將資料庫的大小上限設定為 1 TB 以上，此資料庫的所有還原作業都必須以大於 1 TB 大小上限的 P11/P15 執行。
- 若是作用中異地複寫案例：
   - 設定異地複寫關聯性：如果 P11 或 P15 為主要資料庫，則次要資料庫也必須是 P11 或 P15；系統會拒絕使用較低的計算大小作為次要資料庫，因為這些資料庫無法支援 1 TB 以上的大小。
   - 在異地複寫關聯性中升級主要資料庫：將主要資料庫的大小上限變更為 1 TB 以上會在次要資料庫中觸發相同的變更。 這兩種升級方式都必須成功，在主要資料庫中的變更才會生效。 適用於大於 1-TB 選項的區域限制。 如果次要資料庫位於不支援大於 1 TB 的區域，就無法升級主要資料庫。
- 不支援使用匯入/匯出服務載入具有 1 TB 以上之 P11/P15 資料庫的功能。 使用 SqlPackage.exe 來[匯入](sql-database-import.md)和[匯出](sql-database-export.md)資料。


## <a name="next-steps"></a>後續步驟

如需整體資源限制，請參閱[SQL Database 以虛擬核心為基礎的資源限制 - 單一資料庫](sql-database-vcore-resource-limits-single-databases.md)和[SQL Database 以 DTU 為基礎的資源限制 - 彈性集區](sql-database-dtu-resource-limits-single-databases.md)。
