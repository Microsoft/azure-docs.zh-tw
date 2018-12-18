---
title: Azure SQL 資料倉儲備份與還原 - 快照集、異地備援 | Microsoft Docs
description: 了解「Azure SQL 資料倉儲」中備份與還原的運作方式。 使用資料倉儲備份來將您的資料倉儲還原至主要區域中的某一個還原點。 使用異地備援備份來還原至不同的地理區域。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 08/24/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: e9b5005fad1eeb13314e1fb6a5708bb02b96cbf9
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43248655"
---
# <a name="backup-and-restore-in-azure-sql-data-warehouse"></a>Azure SQL 資料倉儲中的備份與還原
了解「Azure SQL 資料倉儲」中備份與還原的運作方式。 使用資料倉儲快照集將您的資料倉儲還原或複製到主要區域中的先前還原點。 使用資料倉儲異地備援備份來還原至不同的地理區域。 

## <a name="what-is-a-data-warehouse-snapshot"></a>什麼是資料倉儲快照集？
「資料倉儲快照集」會建立還原點，您可以利用此還原點將資料倉儲還原或複製到先前的狀態。  由於 SQL 資料倉儲是一個分散式系統，所以一個資料倉儲快照集會由位於 Azure 儲存體中的許多檔案組成。 快照集會從資料倉儲中儲存的資料內擷取增量變更。

「資料倉儲還原」係指從現有或已刪除的資料倉儲還原點建立的新資料倉儲。 還原資料倉儲是所有商務持續性和災害復原策略中不可或缺的一部分，因為它可在您的資料發生意外損毀或刪除之後重新建立該資料。 資料倉儲也是功能強大的機制，可建立資料倉儲的副本，以用於測試或開發。  SQL 資料倉儲可在相同區域內對任何大小的資料使用快速還原機制，經過測量，還原時間皆少於 20 分鐘。 

## <a name="automatic-restore-points"></a>自動還原點
在會建立還原點的服務中，快照集是內建功能。 您不需啟用此功能。 使用者目前無法刪除自動還原點，因為服務會使用這些還原點來維護用於還原的 SLA。

SQL 資料倉儲可快照還原點建立那天一整天的資料倉儲，而此快照集可保留七天。 此保留期無法變更。 SQL 資料倉儲支援八小時的復原點目標 (RPO)。 您可以透過過去七天內所建立的任一個快照集，在主要區域中還原資料倉儲。

若要查看上一個快照集的開始時間，請在您的線上 SQL 資料倉儲上執行此查詢。 

```sql
select   top 1 *
from     sys.pdw_loader_backup_runs 
order by run_id desc
;
```

## <a name="user-defined-restore-points"></a>使用者定義的還原點
這項功能可讓您在大規模修改資料之前和之後，手動觸發快照來建立資料倉儲的還原點。 這項功能可確保還原點在邏輯上保持一致，並提供額外的資料保護，以避免在進行快速復原期間發生任何工作負載中斷或使用者錯誤。 使用者定義的還原點可保留七天，而且會自動刪除。 您無法變更使用者定義還原點的保留期。 因為只支援在任何時間點設定 42 個使用者定義的還原點，因此必須先[刪除](https://go.microsoft.com/fwlink/?linkid=875299)還原點，才能再建立另一個還原點。 您可以透過 [PowerShell](https://docs.microsoft.com/powershell/module/azurerm.sql/new-azurermsqldatabaserestorepoint?view=azurermps-6.2.0#examples) 或 Azure 入口網站觸發快照集，以建立使用者定義的還原點。


> [!NOTE]
> 如果您需要保留還原點超過 7 天，請[在此](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/35114410-user-defined-retention-periods-for-restore-points)投票給這項功能。 您也可以建立使用者定義的還原點，並從新建立的還原點還原到新資料倉儲。 還原之後，資料倉儲會上線，而您可以無限期地暫停它來節省計算成本。 暫停的資料庫會產生儲存體費用，以「Azure 進階儲存體」費率計費。 如果您需要作用中的已還原資料倉儲副本，您可以讓其恢復上線 (只需幾分鐘的時間)。
>

### <a name="snapshot-retention-when-a-data-warehouse-is-paused"></a>資料倉儲暫停時的快照集保留期
暫停資料倉儲時，SQL 資料倉儲不會建立快照集，也不會使還原點過期。 暫停資料倉儲時，還原點不會改變。 還原點保留期是以資料倉儲上線的天數為依據，而不是以行事曆天數為依據。

例如，如果快照集是在 10 月 1 日下午 4 點開始，而資料倉儲是在 10 月 3 日下午 4 點暫停，則還原點最多是兩天前的。 當資料倉儲恢復上線時，還原點是兩天前的。 如果資料倉儲在 10 月 5 日下午 4 點恢復上線，還原點的存在時間為 2 天，並會繼續保留 5 天。

當資料倉儲恢復上線時，SQL 資料倉儲會繼續建立新的還原點，並在快照集包含超過 7 天的資料時讓快照集過期。

### <a name="snapshot-retention-when-a-data-warehouse-is-dropped"></a>資料倉儲被卸除時的快照集保留期
當您卸除資料倉儲時，SQL 資料倉儲會建立最後的快照集，並將它儲存七天。 您可以將資料倉儲還原到刪除期間建立的最後一個還原點。 

> [!IMPORTANT]
> 如果您刪除邏輯 SQL Server 執行個體，所有屬於該執行個體的資料庫也會一併刪除，且無法復原。 您無法還原已刪除的伺服器。
>

## <a name="geo-backups"></a>異地備份
SQL 資料倉儲每天都會對[配對的資料中心](../best-practices-availability-paired-regions.md)執行一次異地備份。 異地還原的 RPO 為 24 小時。 您可以在任何 SQL 資料倉儲支援的區域中將異地備份還原到伺服器。 萬一您無法存取主要地區中的還原點，異地備份可以確保您能夠還原資料倉儲。

預設會開啟異地備份。 如果您的資料倉儲是 Gen1，則可以視需要[選擇退出](/powershell/module/azurerm.sql/set-azurermsqldatabasegeobackuppolicy)。 您無法選擇退出 Gen2 的異地備份，因為資料保護是既定的保證。

> [!NOTE]
> 如果您的異地備份需要較短的 RPO，請[在此](https://feedback.azure.com/forums/307516-sql-data-warehouse)投票給這項功能。 您也可以建立使用者定義的還原點，並從新建立的還原點還原到不同區域中的新資料倉儲。 還原之後，資料倉儲會上線，而您可以無限期地暫停它來節省計算成本。 暫停的資料庫會產生儲存體費用，以「Azure 進階儲存體」費率計費。 然後暫停。 如果您需要作用中的資料倉儲副本，您可以讓其恢復運上線 (只需幾分鐘的時間)。
>


## <a name="backup-and-restore-costs"></a>備份與還原成本
您將發現 Azure 帳單含有儲存體的明細項目以及災害復原儲存體的明細項目。 儲存體費用是在主要區域中儲存資料，加上快照集擷取增量變更的總費用。 如需如何建立目前快照集的詳細說明，請參閱此[文件](https://docs.microsoft.com/rest/api/storageservices/Understanding-How-Snapshots-Accrue-Charges?redirectedfrom=MSDN#snapshot-billing-scenarios)。 異地備援的費用則涵蓋儲存異地備份的成本。  

您主要資料倉儲和 7 天快照集變更的總成本會四捨五入到最接近的 TB。 例如，如果您的資料倉儲為 1.5 TB，且快照集取用 100 GB，就會以 Azure 進階儲存體費率向您收取 2 TB 資料費用。 

如果您正在使用異地備援儲存體，您會收到個別的儲存體收費項目。 異地備援儲存體是依據標準讀取權限異地備援儲存體 (RA-GRS) 費率收費。

如需 SQL 資料倉儲價格的相關詳細資訊，請參閱 [SQL 資料倉儲價格](https://azure.microsoft.com/pricing/details/sql-data-warehouse/)，以及用於跨區域還原的[輸出費用](https://azure.microsoft.com/pricing/details/bandwidth/)。

## <a name="restoring-from-restore-points"></a>從還原點還原
每個快照都會建立一個代表快照開始時間的還原點。 若要還原資料倉儲，您需選擇一個還原點並發出還原命令。  

您可以保留已還原的資料倉儲和目前的資料倉儲，或是刪除它們其中之一。 如果您想要以已還原的資料倉儲取代目前的資料倉儲，便可以使用 [ALTER DATABASE (Azure SQL 資料倉儲)](/sql/t-sql/statements/alter-database-azure-sql-data-warehouse) 搭配 MODIFY NAME 選項來將它重新命名。 

若要還原資料倉儲，請參閱[使用 Azure 入口網站來還原資料倉儲](sql-data-warehouse-restore-database-portal.md)、[使用 PowerShell 來還原資料倉儲](sql-data-warehouse-restore-database-powershell.md)或[使用 T-SQL 來還原資料倉儲](sql-data-warehouse-restore-database-rest-api.md)。

若要還原已刪除或暫停的資料倉儲，您可以[建立支援票證](sql-data-warehouse-get-started-create-support-ticket.md)。 


## <a name="geo-redundant-restore"></a>異地備援還原
您可以[將資料倉儲還原](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-restore-database-powershell#restore-from-an-azure-geographical-region)到在所選效能等級上支援 SQL 資料倉儲的任何區域。 

> [!NOTE]
> 若要執行異地備援還原，您不可選擇退出這項功能。
>

## <a name="next-steps"></a>後續步驟
如需有關災害規劃的詳細資訊，請參閱[商務持續性概觀](../sql-database/sql-database-business-continuity.md)
