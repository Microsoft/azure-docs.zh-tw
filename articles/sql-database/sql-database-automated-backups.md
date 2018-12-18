---
title: Azure SQL Database 自動、異地備援備份 | Microsoft Docs
description: SQL Database 每隔幾鐘會自動建立一個本機資料庫備份，並使用 Azure 讀取權限異地備援儲存體來進行異地備援。
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
ms.date: 09/25/2018
ms.openlocfilehash: 5c6ebfcb7eae52915af24fc67e9b3c774656149d
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47181136"
---
# <a name="learn-about-automatic-sql-database-backups"></a>了解自動 SQL Database 備份

SQL Database 會自動建立資料庫備份，並使用 Azure 讀取權限異地備援儲存體 (RA-GRS) 來提供異地備援。 這些備份是自動建立的，且不需額外付費。 您不需要執行任何動作來建立備份。 資料庫備份可保護資料免於意外損毀或刪除，是商務持續性和災害復原策略中不可或缺的一部分。 如果您的安全性規則需要備份可供使用一段時間，您可以設定長期備份保留原則。 如需詳細資訊，請參閱[長期保存](sql-database-long-term-retention.md)。

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## <a name="what-is-a-sql-database-backup"></a>什麼是 SQL Database 備份？

針對還原時間點 (PITR) 目的，SQL Database 會使用 SQL Server 技術來建立[完整](https://msdn.microsoft.com/library/ms186289.aspx)、[差異](https://docs.microsoft.com/sql/relational-databases/backup-restore/differential-backups-sql-server)及[交易記錄](https://msdn.microsoft.com/library/ms191429.aspx)備份。 根據計算大小和資料庫活動量的頻率，交易記錄備份通常每隔 5-10 分鐘會進行一次，而差異備份通常每隔 12 小時進行一次。 具有完整和差異備份的交易記錄備份可讓您將資料庫還原到特定的時間點，至裝載資料庫相同的伺服器。 儲存在 RA-GRS 儲存體 Blob 中的備份會複寫到[配對的資料中心](../best-practices-availability-paired-regions.md)，以防止發生資料中心中斷的情況。 在您還原資料庫時，服務會判斷需要還原的完整、差異及交易記錄備份。


您可以使用這些備份︰

* 將資料庫還原至保留期限內的某一個時間點。 這項作業將會在與原始資料庫相同的伺服器中建立新的資料庫。
* 將已刪除的資料庫還原至其被刪除的時間，或保留期限內的任何時間。 已刪除的資料庫只能在當中已建立原始資料庫的相同伺服器中還原。
* 將資料庫還原到另一個地理區域。 這可讓您在無法存取您的伺服器和資料庫時，從地理災害中復原。 在世界各地任何現有的伺服器中建立新的資料庫。 
* 如果已使用長期保留原則 (LTR) 設定資料庫，請從特定長期備份還原資料庫。 這可讓您還原舊版的資料庫來符合規範要求，或執行舊版的應用程式。 請參閱[長期保留](sql-database-long-term-retention.md)。
* 若要執行還原，請參閱[從備份還原資料庫](sql-database-recovery-using-backups.md)。

> [!NOTE]
> 在 Azure 儲存體中，「複寫」一詞指的是將檔案從某個位置複製到另一個位置。 SQL 的「資料庫複寫」指的是保持多個次要資料庫與主要資料庫同步。 
> 

## <a name="how-long-are-backups-kept"></a>備份會保留多久的時間？
每個 SQL Database 備份都具有預設保留週期，此週期是依據資料庫的服務層，而且在 [DTU 購買模型](sql-database-service-tiers-dtu.md)與 [vCore 購買模型](sql-database-service-tiers-vcore.md) 之間有所不同。 您可以更新資料庫的備份保留週期。 如需詳細資料，請參閱[變更備份保留週期](#how-to-change-backup-retention-period)。

如果您刪除資料庫，則 SQL Database 會以保存線上資料庫備份的相同方式保存備份。 例如，如果您刪除保留期間為七天的基本資料庫，則為期四天的備份還會再儲存三天。

如果您需要保留備份的時間超過 PITR 保留週期上限，則可以修改備份的屬性，以將一或多個長期保留週期新增至資料庫。 如需詳細資料，請參閱[長期備份保留](sql-database-long-term-retention.md)。

> [!IMPORTANT]
> 如果您刪除裝載 SQL 資料庫的 Azure SQL Server，則也會一併刪除所有屬於該伺服器的彈性集區和資料庫，且無法復原。 您無法還原已刪除的伺服器。 但是，如果您已設定長期保留，則不會刪除具有 LTR 之資料庫的備份，而且可以還原這些資料庫。

### <a name="pitr-retention-period"></a>PITR 保留期限
使用 DTU 購買模型所建立之資料庫的預設保留週期取決於服務層：

* 基本服務層為 1 週。
* 標準服務層為 5 週。
* 進階服務層為 5 週。

如果您使用[虛擬核心形式的購買模型](sql-database-service-tiers-vcore.md)，備份保留期可以設定為最多 35 天。 

如果您降低目前 PITR 保留期間，則比新保留期間還要舊的所有現有備份將不再可用。 

如果您延長目前 PITR 保留期間，則 SQL Database 會保留現有備份，直到達到較長的保留期間為止。

## <a name="how-often-do-backups-happen"></a>備份頻率是？
### <a name="backups-for-point-in-time-restore"></a>時間點還原的備份
SQL Database 透過自動建立完整備份、差異備份和交易記錄備份，以支援自助式時間點還原 (PITR)。 根據計算大小和資料庫活動量的頻率，完整資料庫備份會每週建立，差異資料庫備份通常每隔 12 小時建立，而交易記錄備份通常每隔 5-10 分鐘建立。 建立資料庫之後，會立即排程第一次完整備份。 通常會在 30 分鐘內完成，但如果資料庫很大，則時間可能更久。 比方說，在還原的資料庫或資料庫複本上，初始備份可能需要較長的時間。 第一次完整備份之後，將會自動排程進一步的備份，並在背景中以無訊息方式管理。 資料庫備份的確切時間，依 SQL Database 服務整體系統工作負載維持平衡而決定。

PITR 備份為異地備援，並受到 [Azure 儲存體跨區域複寫](../storage/common/storage-redundancy-grs.md#read-access-geo-redundant-storage)保護

如需詳細資訊，請參閱[還原時間點](sql-database-recovery-using-backups.md#point-in-time-restore)

### <a name="backups-for-long-term-retention"></a>長期保留的備份
SQL Database 提供選項讓您在 Azure Blob 儲存體中設定完整備份的長期保留 (LTR)，最長可達 10 年。 如果啟用 LTR 原則，則會將每週完整備份自動複製到不同的 RA-GRS 儲存體容器。 為了符合不同的合規性需求，您可以針對每週、每月和/或每年備份選取不同的保留期限。 儲存體耗用量取決於選取的備份頻率和保留期間。 您可以使用 [LTR 定價計算機](https://azure.microsoft.com/pricing/calculator/?service=sql-database)來估算 LTR 儲存體的成本。 

與 PITR 類似，LTR 備份為異地備援，並受到 [Azure 儲存體跨區域複寫](../storage/common/storage-redundancy-grs.md#read-access-geo-redundant-storage)保護。

如需詳細資訊，請參閱[長期備份保留](sql-database-long-term-retention.md)。

## <a name="are-backups-encrypted"></a>備份經過加密？

如果您的資料庫使用 TDE 加密，則備份會在靜止時自動加密 (包括 LTR 備份)。 Azure SQL 資料庫啟用 TDE 時，也會加密備份。 所有新的 Azure SQL Database 預設都會設定為啟用 TDE。 如需 TDE 的詳細資訊，請參閱 [Azure SQL Database 的透明資料加密](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)。

## <a name="how-does-microsoft-ensure-backup-integrity"></a>Microsoft 如何確保備份完整性

Azure SQL Database 工程小組會持續自動地對服務上所有資料庫，進行自動資料庫備份的還原測試。 一旦還原，資料庫也會收到使用 DBCC CHECKDB 的完整性檢查。 在完整性檢查期間找到的任何問題都會對工程小組發出警示。 如需有關 Azure SQL Database 中資料完整性的詳細資訊，請參閱 [Azure SQL Database 中的資料完整性](https://azure.microsoft.com/blog/data-integrity-in-azure-sql-database/)。

## <a name="how-do-automated-backups-impact-my-compliance"></a>自動化備份對合規性的影響為何？

當您將資料庫從預設 PITR 保留 35 天的 DTU 型服務層遷移至 vCore 型服務層時，會保留 PITR 保留，確保不會危害您應用程式的資料復原原則。 如果預設保留週期不符合合規性需求，您可以使用 PowerShell 或 REST API 變更 PITR 保留週期。 如需詳細資料，請參閱[變更備份保留週期](#how-to-change-backup-retention-period)。

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## <a name="how-to-change-backup-retention-period"></a>如何變更備份保留期間
您可以使用 REST API 或 PowerShell 變更預設保留。 支援的值為：7、14、21、28 或 35 天。 下列範例說明如何將 PITR 保留變更為 28 天。 

> [!NOTE]
> 這些 API 只會影響 PITR 保留期間。 如果您已將資料庫設定為 LTR，則它不受影響。 如需如何變更 LTR 保留期間的詳細資料，請參閱[長期備份保留](sql-database-long-term-retention.md)。

### <a name="change-pitr-backup-retention-period-using-powershell"></a>使用 PowerShell 變更 PITR 備份保留期間
```powershell
Set-AzureRmSqlDatabaseBackupShortTermRetentionPolicy -ResourceGroupName resourceGroup -ServerName testserver -DatabaseName testDatabase -RetentionDays 28
```
> [!IMPORTANT]
> 從 [4.7.0-preview](https://www.powershellgallery.com/packages/AzureRM.Sql/4.7.0-preview) 版開始，此 API 包含在 AzureRM.Sql PowerShell 模組中。 

### <a name="change-pitr-retention-period-using-rest-api"></a>使用 REST API 變更 PITR 保留期間
**範例要求**
```http
PUT https://management.azure.com/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/resourceGroup/providers/Microsoft.Sql/servers/testserver/databases/testDatabase/backupShortTermRetentionPolicies/default?api-version=2017-10-01-preview
```
**要求本文**
```json
{
  "properties":{  
      "retentionDays":28
   }
}
```
**範例回應**

狀態碼：200
```json
{
  "id": "/subscriptions/00000000-1111-2222-3333-444444444444/providers/Microsoft.Sql/resourceGroups/resourceGroup/servers/testserver/databases/testDatabase/backupShortTermRetentionPolicies/default",
  "name": "default",
  "type": "Microsoft.Sql/resourceGroups/servers/databases/backupShortTermRetentionPolicies",
  "properties": {
    "retentionDays": 28
  }
}
```
如需詳細資料，請參閱[備份保留 REST API](https://docs.microsoft.com/rest/api/sql/backupshorttermretentionpolicies)。

## <a name="next-steps"></a>後續步驟

- 資料庫備份可保護資料免於意外損毀或刪除，是商務持續性和災害復原策略中不可或缺的一部分。 若要深入了解其他 Azure SQL Database 業務持續性解決方案，請參閱[業務持續性概觀](sql-database-business-continuity.md)。
- 若要使用 Azure 入口網站還原至某個時間點，請參閱[使用 Azure 入口網站將資料庫還原至時間點](sql-database-recovery-using-backups.md)。
- 若要使用 PowerShell 還原至某個時間點，請參閱[使用 PowerShell 將資料庫還原至時間點](scripts/sql-database-restore-database-powershell.md)。
- 若要使用 Azure 入口網站在 Azure Blob 儲存體中設定、管理自動備份的長期保留及從該保留還原，請參閱[使用 Azure 入口網站來管理長期備份保留 (英文)](sql-database-long-term-backup-retention-configure.md)。
- 若要使用 Azure PowerShell 在 Azure Blob 儲存體中設定、管理自動備份的長期保留及從該保留還原，請參閱[使用 Azure PowerShell 來管理長期備份保留 (英文)](sql-database-long-term-backup-retention-configure.md)。
