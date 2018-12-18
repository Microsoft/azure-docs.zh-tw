---
title: 雲端商務持續性 - 資料庫復原 - SQL Database | Microsoft Docs
description: 了解 Azure SQL Database 如何支援雲端商務持續性和資料庫復原，以及如何協助維持任務關鍵性雲端應用程式持續執行。
keywords: business continuity,cloud business continuity,database disaster recovery,database recovery,商務持續性,雲端商務持續性,資料庫災害復原,資料庫復原
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
ms.date: 09/19/2018
ms.openlocfilehash: e18b637ee583757e040ef6fd5c2d52cff14cb4fc
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221141"
---
# <a name="overview-of-business-continuity-with-azure-sql-database"></a>使用 Azure SQL Database 的商務持續性概觀

Azure SQL Database 是針對 Azure 雲端環境設定及最佳化的最新穩定 SQL Server Database Engine 實作，可對會影響您商務程序的錯誤提供[高可用性](sql-database-high-availability.md)和復原功能。 Azure SQL Database 中的**商務持續性**是指在面臨中斷時可讓企業持續運作的機制、原則和程序 (特別是其運算基礎結構)。  在大部分情況下，Azure SQL Database 會處理雲端環境中可能發生的干擾性事件，並且讓您的商務程序持續執行。 不過，SQL Database 有一些無法處理的干擾性事件，例如：
 - 使用者不小心刪除或更新了資料表中的資料列。
 - 惡意攻擊者接著刪除資料或刪除資料庫。
 - 地震導致電源中斷，而且資料中心暫時停用。
 
Azure SQL Database 無法控制這些情況，因此您必須在 SQL Database 使用商務持續性功能，這些功能可讓您復原您的資料並且讓應用程式持續執行。

本概觀說明 Azure SQL Database 針對商務持續性和災害復原所提供的功能。 了解選項、建議和教學課程，以從可能導致資料遺失或造成資料庫和應用程式無法使用的干擾性事件中復原。 了解當使用者或應用程式錯誤影響資料完整性、Azure 區域中斷，或您的應用程式需要維護時該如何處理。

## <a name="sql-database-features-that-you-can-use-to-provide-business-continuity"></a>您可用來提供商務持續性的 SQL Database 功能

就資料庫的觀點而言，有四個主要可能中斷案例：
- 會影響資料庫節點的**本機硬體或軟體失敗**，例如磁碟機失敗。
- **資料損毀或刪除** - 通常是由應用程式錯誤或人為錯誤所造成。  這類失敗本質上為應用程式專屬的，而且通常無法由基礎結構自動偵測或緩和。
- **資料中心中斷**，可能由天然災害造成。  此案例需要某種程度的異地備援，讓應用程式能容錯移轉到替代資料中心。
- **升級或維護錯誤** – 在計劃性升級或維護期間內，應用程式或資料庫發生的非預期問題可能需要快速復原到先前的資料庫狀態。

SQL Database 提供幾種商務持續性功能，包括可以緩和這些案例的自動備份和選用的料庫複寫。 首先，您必須了解 SQL Database [高可用性架構](sql-database-high-availability.md)如何將 99.99% 的可用性和復原功能提供給某些會影響您商務程序的干擾性事件。
然後，您可以了解您可用來從 SQL Database 高可用性架構無法處理的干擾性事件復原的其他機制，例如：
 - [時態表](sql-database-temporal-tables.md)可讓您從任何時間點還原資料列版本。
 - [內建自動備份](sql-database-automated-backups.md)和[還原時間點](sql-database-recovery-using-backups.md#point-in-time-restore)可讓您將整個資料庫還原至過去 35 天內的某個時間點。
 - 如果[邏輯伺服器尚未刪除](sql-database-recovery-using-backups.md#deleted-database-restore)，您可以**將已刪除的資料庫還原**到它被刪除的時間點。
 - [長期備份保留](sql-database-long-term-retention.md)可讓您最多保留備份 10 年。
 - [異地複寫](sql-database-geo-replication-overview.md)可在資料中心擴展中斷時，允許應用程式執行快速災害復原。

每個功能對於預估的復原時間 (ERT) 都有不同的特性，最近的交易都有可能遺失資料。 一旦您了解這些選項，就可以在其中選擇，而在大部分情況下，可以針對不同情況一起搭配使用。 當您開發商務持續性計劃時，您必須了解應用程式在干擾性事件之後完全復原所需的最大可接受時間。 完全復原應用程式所需的時間，也稱為復原時間目標 (RTO)。 您也必須了解在干擾性事件之後復原時，應用程式可忍受遺失的最近資料更新 (時間間隔) 最大期間。 您可能經得起遺失的更新時間週期，也稱為復原點目標 (RPO)。

下表針對三種最常見的案例，為每個服務層提供 ERT 與 RPO 的比較。

| 功能 | 基本 | 標準 | 進階  | 一般用途 | 業務關鍵
| --- | --- | --- | --- |--- |--- |
| 從備份進行時間點還原 |7 天內的任何還原點 |35 天內的任何還原點 |35 天內的任何還原點 |設定期間內的任何還原點 (最多 35 天)|設定期間內的任何還原點 (最多 35 天)|
| 從異地複寫備份進行異地還原 |ERT < 12 h，RPO < 1 h |ERT < 12 h，RPO < 1 h |ERT < 12 h，RPO < 1 h |ERT < 12 h，RPO < 1 h|ERT < 12 h，RPO < 1 h|
| 從 SQL 長期保留還原 |ERT < 12 h，RPO < 1 wk |ERT < 12 h，RPO < 1 wk |ERT < 12 h，RPO < 1 wk |ERT < 12 h，RPO < 1 wk|ERT < 12 h，RPO < 1 wk|
| 主動式異地複寫 |ERT < 30 秒，RPO < 5 秒 |ERT < 30 s，RPO < 5 s |ERT < 30 s，RPO < 5 s |ERT < 30 s，RPO < 5 s|ERT < 30 s，RPO < 5 s|

## <a name="recover-a-database-to-the-existing-server"></a>將資料庫復原到現有的伺服器

SQL Database 會每週自動執行完整資料庫備份、通常每 12 小時自動執行差異資料庫備份，且每 5 - 10 分鐘自動執行交易記錄備份，透過這樣的備份組合來防止您的企業遺失資料。 所有服務層的備份都會儲存在 RA-GRS 儲存體中 35 天，但基本 DTU 服務層的備份儲存 7 天除外。 如需詳細資訊，請參閱[自動資料庫備份](sql-database-automated-backups.md)。 您可以使用 Azure 入口網站、PowerShell 或 REST API，將現有的資料庫從自動備份還原到先前的時間點，以做為相同邏輯伺服器上的新資料庫。 如需詳細資訊，請參閱[還原時間點](sql-database-recovery-using-backups.md#point-in-time-restore)。

如果應用程式的最大支援時間點還原 (PITR) 保留期限不夠，可以藉由針對資料庫設定長期保留 (LTR) 原則來延長。 如需詳細資訊，請參閱[長期備份保留](sql-database-long-term-retention.md)。

您可以使用這些自動資料庫備份，將資料庫從各種干擾性事件復原，不論是在您的資料中心內復原，還是復原到另一個資料中心，都可以。 使用自動資料庫備份時，預估的復原時間取決於數個因素，包括在相同區域中同時進行復原的資料庫總數、資料庫大小、交易記錄大小，以及網路頻寬。 復原時間通常不到 12 小時。 復原非常大型的作用中資料庫可能需要比較長的時間。 如需復原時間的詳細資訊，請參閱[資料庫復原時間](sql-database-recovery-using-backups.md#recovery-time)。 當復原到另一個資料區時，會使用異地備援備份將可能的資料遺失限制為 1 小時。

如果您的應用程式有下列狀況，可以使用自動備份和[時間點還原](sql-database-recovery-using-backups.md#point-in-time-restore)作為您的商務持續性和復原機制︰

* 非關鍵性應用程式。
* 沒有繫結 SLA - 停機 24 小時或更長的時間不會衍生財務責任。
* 資料變更率低 (每小時的交易次數低)，並且最多可接受遺失一小時的資料變更。
* 成本有限。

如果您需要更快速的復原，請使用[主動式異地複寫](sql-database-geo-replication-overview.md) (會接著討論)。 如果您必須能夠復原 35 天之前的資料，請使用[長期保留](sql-database-long-term-retention.md)。 

## <a name="recover-a-database-to-another-region"></a>將資料庫復原到另一個區域
<!-- Explain this scenario -->

雖然很罕見，但 Azure 資料中心也可能會有中斷的時候。 發生中斷時，可能只會讓業務中斷幾分鐘，也可能會持續幾小時。

* 其中一個選項是在資料中心中斷結束時等待您的資料庫重新上線。 這適用於可以容忍資料庫離線的應用程式。 例如，您不需要不斷處理的開發專案或免費試用版。 當資料中心中斷時，您不會知道中斷會持續多久，因此這個選項僅適用於您可以一段時間暫時不需要資料庫。
* 另一個選項是使用[異地備援資料庫備份](sql-database-recovery-using-backups.md#geo-restore) (異地還原)，在任何 Azure 區域中的任何伺服器上還原資料庫。 異地還原使用異地備援備份做為其來源，即使因為中斷而無法存取資料庫或資料中心，也能用來復原資料庫。
* 最後，如果您使用主動式異地複寫，您可以快速將另一個資料區域上的次要升級成主要 (也稱為容錯移轉)，並將應用程式設定為連線到已升級的主要資料庫。 由於非同步複寫的性質緣故，近期交易可能會有一些少量的資料遺失。 您可使用自動容錯移轉群組來自訂容錯移轉原則，以將潛在資料遺失風險降至最低。 在所有情況下，使用者都會經歷短暫的停機時間，而需要重新連線。 容錯移轉只需要幾秒鐘的時間，而從備份復原資料庫則需要幾個小時。

若要容錯移轉到另一個區域，您可以使用[主動式異地複寫](sql-database-geo-replication-overview.md)將資料庫設定為在您所選的區域中最多可有 4 個可讀取的次要資料庫。 這些次要資料庫會使用非同步複寫機制與主要資料庫保持同步。 

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Database-protecting-important-DBs-from-regional-disasters-is-easy/player]
>


> [!IMPORTANT]
> 若要使用主動式異地複寫和自動容錯移轉群組，您必須是訂用帳戶擁有者，或是在 SQL Server 中擁有系統管理權限。 您可以使用 Azure 入口網站、PowerShell 或 REST API 並透過 Azure 訂用帳戶的權限來進行設定和容錯移轉，也可以使用 Transact-SQL 並透過 SQL Server 權限來進行。
> 

此功能可用來防範資料中心中斷或應用程式升級期間的業務中斷。 若要啟用自動透明容錯移轉，您應使用 SQL Database [自動容錯移轉群組](sql-database-geo-replication-overview.md)功能，將異地複寫資料庫分組。 如果您的應用程式符合下列任何準則，請使用主動式異地複寫和自動容錯移轉群組：

* 是關鍵性應用程式。
* 具有服務等級協定 (SLA)，不允許 24 小時以上的停機時間。
* 停機可能會衍生財務責任。
* 具有很高的資料變更率，而且不接受遺失一個小時的資料。
* 與潛在的財務責任和相關企業損失相較下，使用主動式異地複寫的額外成本較低。

當您採取行動時，復原所需的時間以及會遺失多少資料，取決於您如何決定在應用程式中使用這些商務持續性功能。 事實上，您可以根據應用程式需求，選擇使用資料庫備份和主動式異地複寫的組合。 若要探討使用這些商務持續性功能針對獨立資料庫和彈性集區進行應用程式設計時的考量，請參閱[設計雲端災害復原應用程式](sql-database-designing-cloud-solutions-for-disaster-recovery.md)和[彈性集區災害復原策略](sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool.md)。

下列各節概述使用資料庫備份或主動式異地複寫來進行復原的步驟。 如需包括規劃需求的詳細步驟、復原後步驟，以及有關如何模擬中斷以執行災害復原演練的資訊，請參閱[從中斷復原 SQL Database](sql-database-disaster-recovery.md)。

### <a name="prepare-for-an-outage"></a>準備中斷
無論您要使用何種商務持續性功能，您都必須︰

* 識別並準備目標伺服器，包括伺服器層級防火牆規則、登入和 master 資料庫層級權限。
* 決定如何將用戶端和用戶端應用程式重新導向到新的伺服器
* 記錄其他相依性，例如稽核設定和警示

如果您沒有適當地準備，在容錯移轉或資料庫復原後讓應用程式上線將會多花費時間，而且也可能需要在有壓力的情況下進行疑難排解 - 這是不良的情況組合。

### <a name="fail-over-to-a-geo-replicated-secondary-database"></a>容錯移轉至異地複寫的次要資料庫
若您使用主動式異地複寫和自動容錯移轉群組做為復原機制，則可設定自動容錯移轉原則或使用[手動容錯移轉](sql-database-disaster-recovery.md#fail-over-to-geo-replicated-secondary-server-in-the-failover-group)。 啟動容錯移轉後，次要資料庫就會成為新的主要資料庫，並準備好記錄新的交易以及回應查詢 - 只會遺失尚未複寫的資料。 如需關於設計容錯移轉程序的資訊，請參閱[設計雲端災害復原應用程式](sql-database-designing-cloud-solutions-for-disaster-recovery.md)。

> [!NOTE]
> 資料中心恢復連線時，舊的主要複本會自動重新連線至新的主要複本，且會成為次要資料庫。 若您需要將主要複本重新定位至原始區域，可手動啟動規劃的容錯移轉 (容錯回復)。 
> 

### <a name="perform-a-geo-restore"></a>執行異地還原
如果您使用自動備份搭配異地備援儲存體 (預設為啟用)，您可以使用[異地還原](sql-database-disaster-recovery.md#recover-using-geo-restore)來復原資料庫。 復原通常會在 12 小時內進行，且可能遺失最多 1 小時的資料 (視最後一次記錄備份的建立並複寫時間而定)。 在復原完成之前，資料庫無法記錄任何交易或回應任何查詢。 請注意，異地還原只會將資料庫還原至最後一個可用的時間點。

> [!NOTE]
> 如果資料中心在您的應用程式切換到復原的資料庫之前就再次上線，您可以取消復原。  

### <a name="perform-post-failover--recovery-tasks"></a>執行容錯移轉後/復原後工作
從其中任何一種復原機制復原之後，您都必須執行下列額外的工作，您的使用者和應用程式才能回復正常執行狀態︰

* 重新導向用戶端與用戶端應用程式到新的伺服器與還原的資料庫
* 確定有適當的伺服器層級防火牆規則供使用者連線 (或使用 [資料庫層級防火牆](sql-database-firewall-configure.md#creating-and-managing-firewall-rules))
* 確定有適當的登入和 master 資料庫層級權限 (或使用 [自主的使用者](https://msdn.microsoft.com/library/ff929188.aspx))
* 依適當情況設定稽核
* 依適當情況設定警示

## <a name="upgrade-an-application-with-minimal-downtime"></a>在最少停機時間的情況下升級應用程式
有時，應用程式會因為計劃性維護 (例如應用程式升級) 而必須離線。 [管理應用程式升級](sql-database-manage-application-rolling-upgrade.md) 說明如何使用「主動式異地複寫」來輪流升級雲端應用程式，以將升級時的停機時間縮到最短，並提供發生錯誤時的復原路徑。 

## <a name="next-steps"></a>後續步驟
如需獨立資料庫和彈性集區的應用程式設計考量探討，請參閱[設計雲端災害復原應用程式](sql-database-designing-cloud-solutions-for-disaster-recovery.md)和[彈性集區災害復原策略](sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool.md)。
