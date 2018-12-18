---
title: 容錯移轉群組和主動式異地複寫 - Azure SQL Database | Microsoft Docs
description: 使用具有作用中異地複寫功能的自動容錯移轉群組，並在發生中斷時啟用自動容錯移轉。
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
ms.openlocfilehash: e3296430cd58c8ba33f7cc9acd4cded2925b5be9
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166277"
---
# <a name="overview-active-geo-replication-and-auto-failover-groups"></a>概觀：主動式異地複寫和自動容錯移轉群組

主動式異地複寫是 Azure SQL Database 的功能，可讓您在相同或不同資料中心 (區域) 中建立可讀取的資料庫複本。

![異地複寫](./media/sql-database-geo-replication-failover-portal/geo-replication.png )

作用中異地複寫是設計為商務持續性解決方案，可在資料中心擴展中斷時，允許應用程式執行快速的災害復原。 如果已啟用異地複寫，則應用程式可以在不同的 Azure 區域中起始容錯移轉至次要資料庫。 在相同或不同的區域中，最多可支援四個次要資料庫，次要資料庫也可以用於唯讀存取查詢。 容錯移轉必須由應用程式或使用者手動起始。 容錯移轉之後，新的主要資料庫會有不同的連接端點。 

> [!NOTE]
> 主動式異地複寫可供所有區域之所有服務層中的所有資料庫使用。
> 主動式異地複寫無法在受控執行個體中使用。
>  

自動容錯移轉群組是主動式異地複寫的延伸模組。 它的設計旨在同時管理多個異地複寫資料庫的容錯移轉，方法是使用應用程式起始的容錯移轉，或是根據使用者定義的準則，委派容錯移轉由 SQL Database 服務來完成。 後者可讓您在導致主要區域中完全或部分喪失 SQL Database 服務可用性的嚴重失敗或其他非計劃事件之後，自動復原次要地區中的多個相關資料庫。 此外，您可以使用可讀取次要資料庫來卸載唯讀查詢工作負載。 因為自動容錯移轉群組牽涉到多個資料庫，所以這些資料庫必須在主要伺服器上進行設定。 在容錯移轉群組中，資料庫的主要和次要伺服器都必須位於相同的訂用帳戶。 自動容錯移轉群組支援將群組中的所有資料庫都只複寫到不同區域的一部次要伺服器。

> [!NOTE]
> 如果需要多個次要伺服器，請使用主動式異地複寫。
>  

如果您使用主動式異地複寫，而且主要資料庫因為任何原因而失敗，或者只是需要離線，您可以起始容錯移轉至任何次要資料庫。 容錯移轉至其中一個次要資料庫啟動時，所有其他次要複本會自動連結至新的主要複本。 如果您使用自動容錯移轉群組管理資料庫復原，則影響群組中一或多個資料庫的任何中斷都會造自動容錯移轉。 您可以設定最符合您應用程式需求的自動容錯移轉原則，或者您可以選擇不設定而使用手動啟動。 此外，自動容錯移轉群組還提供在容錯移轉期間仍保持不變的讀寫和唯讀接聽程式端點。 無論您使用手動或自動啟動容錯移轉，容錯移轉都會將群組中所有次要資料庫切換到主要資料庫。 資料庫容錯移轉完成後，DNS 記錄會自動更新以將端點重新導向至新的區域。

您可以使用主動式異地複寫管理伺服器上或彈性集區中個別資料庫或一組資料庫的複寫和容錯移轉。 您可以使用 

- [Azure 入口網站](sql-database-geo-replication-portal.md)
- [PowerShell：單一資料庫](scripts/sql-database-setup-geodr-and-failover-database-powershell.md)
- [PowerShell：彈性集區](scripts/sql-database-setup-geodr-and-failover-pool-powershell.md)
- [PowerShell：容錯移轉群組](scripts/sql-database-setup-geodr-failover-database-failover-group-powershell.md)
- [Transact-SQL：單一資料庫或彈性集區](/sql/t-sql/statements/alter-database-azure-sql-database)
- [REST API：單一資料庫](/rest/api/sql/replicationlinks/failover)
- [REST API：容錯移轉群組](/rest/api/sql/failovergroups/failover) 
 
容錯移轉之後，請確認已在新的主要資料庫上設定伺服器和資料庫的驗證需求。 如需詳細資訊，請參閱 [災害復原後的 SQL Database 安全性](sql-database-geo-replication-security-config.md)。 這適用於主動式異地複寫和自動容錯移轉群組兩者。

主動式異地複寫會利用 SQL Server 的 [Always On](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) 技術，使用快照隔離，以非同步方式將主要資料庫上認可的交易複寫到次要資料庫。 自動容錯移轉群組在主動式異地複寫之上提供群組語意，但使用相同的非同步複寫機制。 雖然次要資料可能會在任何指定時間點稍微落後主要資料庫，但是次要資料保證絕對不會含有部分交易。 跨區域備援可讓應用程式從天然災害、災難性人為錯誤或惡意行為所造成的全部或部分資料中心永久遺失快速復原。 在 [商務持續性概觀](sql-database-business-continuity.md)中可以找到特定的 RPO 資料。

下圖顯示以美國中北部區域的主要資料庫和美國中南部區域的次要資料庫所設定的作用中異地複寫範例。

![異地複寫關聯性](./media/sql-database-active-geo-replication/geo-replication-relationship.png)

因為次要資料庫可讀取，所以可用來卸載唯讀工作負載，例如報告作業。 如果您使用主動式異地複寫，可以在具有主要資料庫的相同區域中建立次要資料庫，但這樣不會增加應用程式發生嚴重失敗時的恢復能力。 如果您使用自動容錯移轉群組，則一律要在不同區域中建立次要資料庫。

除了災害復原之外，主動式異地複寫還可用於下列案例︰

* **資料庫移轉**：您可以使用主動式異地複寫，以最少的停機時間將資料庫從一部伺服器移轉到另一個線上伺服器。
* **應用程式升級**：您可以在應用程式升級期間建立額外的次要資料庫做為容錯回復複本。

若要達到真正的業務持續性，新增資料中心之間的資料庫備援只是解決方案的一部分。 在災難性失敗後要端對端復原應用程式 (服務) 需要復原構成服務的所有元件和任何相依的服務。 這些元件的範例包括用戶端軟體 (例如自訂 JavaScript 的瀏覽器)、web 前端、儲存體和 DNS。 所有元件都必須對相同的失敗具有恢復功能，並且在應用程式的復原時間目標 (RTO) 內可供使用。 因此，您需要識別所有相依服務並了解其提供的保證與功能。 然後，您必須採取適當步驟以確保服務功能在它所依賴的服務容錯移轉期間都正常。 如需有關設計災害復原解決方案的詳細資訊，請參閱[使用主動式異地複寫設計災害復原的雲端解決方案](sql-database-designing-cloud-solutions-for-disaster-recovery.md)。

## <a name="active-geo-replication-capabilities"></a>主動式異地複寫功能
主動式異地複寫功能提供下列基本功能：
* **自動非同步複寫**︰您只能藉由加入至現有資料庫來建立次要資料庫。 您可以在任何 Azure SQL Database 伺服器建立次要資料庫。 一旦建立之後，次要資料庫就要填入從主要資料庫複製的資料。 這個程序稱為植入。 建立並植入次要資料庫之後，主要資料庫的更新會以非同步方式自動複製到次要資料庫。 非同步複寫表示交易會先在主要資料庫上受到認可，才會複製到次要資料庫。 
* **可讀取的次要資料庫**：應用程式可以存取次要資料庫，使用用於存取主要資料庫的相同安全性主體或不同安全性主體進行唯讀作業。 在快照集隔離模式中執行次要資料庫，以確保主要資料庫更新的複寫 (記錄重播) 不會被次要資料庫上執行的查詢延遲。

> [!NOTE]
> 如果主要資料庫上有結構描述更新，次要資料庫上的記錄重播會延遲， 因為後者需要次要資料庫上的結構描述鎖定。 
> 

* **多個可讀取次要資料庫**：兩個或以上的次要資料庫可增加主要資料庫和應用程式的備援和保護層級。 如果有多個次要資料庫存在，即使其中一個次要資料庫失敗，應用程式仍會受到保護。 如果只有一個次要資料庫卻失敗了，應用程式會暴露在更高的風險中，直到建立新的次要資料庫。

> [!NOTE]
> 如果您使用主動式異地複寫建置分散在世界各地的應用程式，而且必須在四個以上的區域中提供唯讀的資料存取，您可以建立次要資料庫的次要資料庫 (稱為鏈結的程序)。 如此一來您就可以達到幾乎無限擴充的資料庫複寫。 此外，鏈結可減少來自主要資料庫的複寫額外負荷。 缺點是會增加分葉最尾端之次要資料庫上的複寫延遲。 
>

* **支援彈性集區資料庫**：每個複本可以分別參與彈性集區，或完全不在任何彈性集區中。 每個複本的集區選擇都不同，而且不相依於任何其他複本 (不論是主要還是次要) 的設定。 每個彈性集區都包含在單一區域內，因此相同拓撲中的多個複本永遠不會共用彈性集區。
* **次要資料庫的可設定計算大小**：主要與次要資料庫都必須有相同的服務層。 此外也強烈建議您使用與主要資料庫相同的計算大小 (DTU 或虛擬核心) 來建立次要資料庫。 計算大小較低的次要資料庫可能會有複寫延遲增加和無法使用的風險，進而有在容錯移轉之後遺失重要資料的風險。 因此，將無法保證已發佈的 RPO = 5 秒。 另一個風險是，在容錯移轉之後，應用程式的效能會因為新主要資料庫的計算容量不足而受到影響，除非該資料庫升級至較高的計算大小。 升級所需時間視資料庫大小而定。 此外，目前此類升級要求主要與次要資料庫都必須在線上，因此在中斷情況趨緩之前無法完成。 如果您決定建立具有較低計算大小的次要資料庫，您可以利用 Azure 入口網站上的記錄 IO 百分比圖表，來預估次要資料庫承受複寫負載所需的最低計算大小。 例如，如果您的主要資料庫是 P6 (1000 DTU) 和其記錄 IO 百分比為 50%，則次要資料庫必須至少是 P4 (500 DTU)。 您也可以使用 [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database) 或 [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) 資料庫檢視來擷取記錄 IO 資料。  如需 SQL Database 計算大小的詳細資訊，請參閱 [SQL Database 服務層是什麼](sql-database-service-tiers.md)。 
* **使用者控制的容錯移轉和容錯回復**：應用程式或使用者可以隨時明確也將次要資料庫切換到主要角色。 在實際的中斷期間，應該使用「非計劃性」選項，這樣會立即將次要升級為主要。 當失敗的主要資料庫復原，並且可再次使用時，系統會自動將復原的主要資料庫標示為次要資料庫，並讓它與新的主要資料庫保持更新。 由於複寫的非同步本質，如果主要資料庫在將最新的變更複寫至次要資料庫之前失敗，則非計劃的容錯移轉期間會有少量的資料遺失。 當具有多個次要資料庫的主要資料庫容錯移轉時，系統會自動重新設定複寫關聯性，並且將剩餘的次要資料庫連結至新升級的主要資料庫，而不需要任何使用者介入。 解決造成容錯移轉的中斷之後，可能想要讓應用程式返回主要區域。 若要這麼做，應該使用「計劃」選項叫用容錯移轉命令。 
* **保持認證和防火牆規則同步**︰我們建議針對異地複寫資料庫使用 [資料庫防火牆規則](sql-database-firewall-configure.md) ，這樣一來，這些規則可以使用資料庫複寫，以確保所有次要資料庫具有與主要資料庫相同的防火牆規則。 此方法不需要客戶手動設定及維護同時裝載主要和次要資料庫之伺服器上的防火牆規則。 同樣地，針對資料存取使用 [自主資料庫使用者](sql-database-manage-logins.md) ，確保主要與次要資料庫永遠具有相同的使用者認證，在容錯移轉時不會因為登入和密碼不相符而有任何中斷。 使用額外的 [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md)，客戶可以管理主要和次要資料庫的使用者存取，並且排除在資料庫中管理認證的需求。

## <a name="auto-failover-group-capabilities"></a>自動容錯移轉群組功能

自動容錯移轉群組功能支援群組層級的複寫和自動容錯移轉，提供強大的主動式異地複寫抽象概念。 此外，它提供額外的接聽程式端點，不需要在容錯移轉之後變更 SQL 連接字串。 

* **容錯移轉群組**︰可以在不同區域 (主要和次要伺服器) 的兩部伺服器之間建立一或多個容錯移轉群組。 每個群組可包含一或多個作為復原單位的資料庫，以防所有或部分的主要資料庫因為主要區域服務中斷而變成無法使用。  
* **主要伺服器**︰容錯移轉群組中裝載主要資料庫的伺服器。
* **次要伺服器**︰容錯移轉群組中裝載次要資料庫的伺服器。 次要伺服器和主要伺服器不能位於相同區域。
* **在容錯移轉群組中新增資料庫**︰您可以將伺服器或彈性集區內的多個資料庫放在同一個容錯移轉群組中。 如果您在群組中新增單一資料庫，它會使用相同的版本和計算大小自動建立次要資料庫。 如果主要資料庫在彈性集區中，則會使用相同名稱在該彈性集區中建立次要資料庫。 如果您新增在次要伺服器中已經有次要資料庫的資料庫，群組就會繼承該異地複寫。

> [!NOTE]
> 新增在伺服器中已經有次要資料庫但不屬於容錯移轉群組一部分的資料庫時，會在次要伺服器中建立新的次要資料庫。 
>

* **容錯移轉群組讀寫接聽程式**︰指向目前主要伺服器 URL 且格式為 **&lt;容錯移轉群組名稱&gt;.database.windows.net** 的 DNS CNAME 記錄。 它可讓讀寫 SQL 應用程式在容錯移轉之後，當主要資料庫變更時毫無障礙地重新連線至主要資料庫。 
* **容錯移轉群組唯讀接聽程式**︰指向目前次要伺服器 URL 且格式為 **&lt;容錯移轉群組名稱&gt;.secondary.database.windows.net** 的 DNS CNAME 記錄。 它可讓唯讀 SQL 應用程式使用指定的負載平衡規則毫無障礙地連線到次要資料庫。 
* **自動容錯移轉原則**︰容錯移轉群組預設是利用自動容錯移轉原則設定。 系統在偵測到失敗且寬限期已過期之後，會觸發容錯移轉。 系統必須就影響級別，藉由 SQL Database 服務的內建高可用性基礎結構，來驗證中斷情況不會趨緩。 如果您想要控制應用程式的容錯移轉工作流程，可以關閉自動容錯移轉。 
* **唯讀的容錯移轉原則**：根據預設，唯讀接聽程式的容錯移轉已停用。 它可確保在次要伺服器離線時不會影響主要伺服器的性能。 不過，這也表示在次要伺服器恢復之前，唯讀的工作階段將無法連線。 如果您無法容忍唯讀工作階段的停機時間，並且可以暫時將主要伺服器用於唯讀和讀寫流量，但代價是主要伺服器潛在的效能降低，則可以為唯讀接聽程式啟用容錯移轉。 在這種情況下，如果次要伺服器無法使用時，唯讀流量將自動重新導向到主要伺服器。  
* **手動容錯移轉**︰無論自動容錯移轉設定為何，您都可以在任何時候手動起始容錯移轉。 如果未設定自動容錯移轉原則，就需要手動容錯移轉才能復原容錯移轉群組中的資料庫。 您可以起始強制或易用容錯移轉 (具有完整的資料同步處理)。 後者可用來將作用中伺服器重新放置到主要區域。 容錯移轉完成時，DNS 記錄會自動更新以確保能連線到正確的伺服器。
* **資料遺失的寬限期**︰因為主要和次要資料庫是使用非同步複寫進行同步處理，容錯移轉可能會導致資料遺失。 您可以自訂自動容錯移轉原則，以反映應用程式對資料遺失的容錯程度。 設定 **GracePeriodWithDataLossHours**，可以控制系統在起始可能造成資料遺失的容錯移轉之前等候的時間長度。 

> [!NOTE]
> 當系統偵測到群組中的資料庫仍保持線上狀態 (例如，中斷只影響了服務控制平面) 時，它會立即啟動具完整資料同步處理的容錯移轉 (易用容錯移轉)，無論 **GracePeriodWithDataLossHours** 所設定的值為何。 此行為可確保復原期間不會遺失任何資料。 寬限期只有在無法使用易用容錯移轉時才有效。 如果中斷情況在寬限期到期之前就趨緩，則不會啟動容錯移轉。
>

* **多個容錯移轉群組**︰您可以為相同的兩部伺服器設定多個容錯移轉群組來控制容錯移轉的規模。 每個群組分別進行容錯移轉。 如果您的多租用戶應用程式使用彈性集區，可以使用這項功能來混合每個集區中的主要和次要資料庫。 如此一來，可以讓中斷只影響一半的租用戶。

## <a name="best-practices-of-using-failover-groups-for-business-continuity"></a>使用容錯移轉群組獲得商務持續性的最佳做法
在商務持續性前提下設計服務時，您應該遵循這些指導方針：

- **使用一或多個容錯移轉群組來管理多個資料庫的容錯移轉**︰可以在位於不同區域 (主要和次要伺服器) 的兩部伺服器之間，建立一或多個容錯移轉群組。 每個群組可包含一或多個作為復原單位的資料庫，以防所有或部分的主要資料庫因為主要區域服務中斷而變成無法使用。 容錯移轉群組會使用和主要資料庫相同的服務目標，來建立異地次要資料庫。 如果您在容錯移轉群組中新增現有的異地複寫關聯性，請確定異地次要資料庫所設定的服務層與計算大小和主要資料庫相同。
- **針對 OLTP 工作負載使用讀取寫入接聽程式**：在執行 OLTP 作業時使用 **&lt;容錯移轉群組名稱&gt;.database.windows.net** 做為伺服器 URL，連線會自動導向至主要伺服器。 在容錯移轉之後，不會變更此 URL。 請注意，容錯移轉牽涉到更新 DNS 記錄，因此只有在重新整理用戶端 DNS 快取之後，才會將用戶端連線重新導向至新的主要伺服器。
- **針對唯讀工作負載使用唯讀接聽程式**：如果您有容忍某些過時資料的邏輯隔離唯讀工作負載，則可以使用應用程式中的次要資料庫。 針對唯讀工作階段，請使用 **&lt;容錯移轉群組名稱&gt;.secondary.database.windows.net** 做為伺服器 URL，連線會自動導向至次要伺服器。 也建議您使用 **ApplicationIntent=ReadOnly**，在連接字串中表示讀取意圖。 
- **對效能降低做好心理準備**：應用程式的其餘部分或所使用的其他服務並不會影響 SQL 的容錯移轉決策。 應用程式可能會「混用」某個區域的某些元件和另一個區域的某些元件。 若要避免效能降低，請務必要在 DR 區域中部署備援應用程式，並遵循本文中的指導方針。 請注意，DR 區域中的應用程式不一定非得使用不同的連接字串。  
- **對資料遺失做好心理準備**：如果系統偵測到服務中斷，SQL 會在就我們所知並無資料遺失時，自動觸發讀寫容錯移轉。 否則，它會等候您透過 **GracePeriodWithDataLossHours** 所指定的這段時間。 如果您指定了 **GracePeriodWithDataLossHours**，請做好可能會遺失資料的心理準備。 服務中斷期間，Azure 一般會傾向維持可用性。 如果您無法承擔資料遺失情況，請務必將 **GracePeriodWithDataLossHours** 設定為足夠大的數字，例如 24 小時。 

> [!IMPORTANT]
> 具有 800 個或更少 DTU 且使用異地複寫的資料庫超過 250 個的彈性集區，可能會遇到的問題包括規劃的容錯移轉時間較久與效能降低。  當地理複寫端點依地理位置廣泛相隔，或當每個資料庫使用多個次要端點時，較可能會發生這些寫入大量工作負載的問題。  異地複寫延遲隨時間而增加時，就可看出這些問題的徵兆。  您可以使用 [sys.dm_geo_replication_link_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database) 來監視這個延遲。  如果發生這些問題，則風險降低方式包括增加集區 DTU 數目，或減少相同集區內異地複寫的資料庫數目。

## <a name="failover-groups-and-network-security"></a>容錯移轉群組和網路安全性 

對於某些應用程式，安全性規則會要求對資料層的網路存取需限制為特定元件，例如 VM、Web 服務等等。這項需求會出現一些商務持續性設計、以及使用容錯移轉群組的挑戰。 實作這類限制的存取權時，您應該考慮下列選項。

### <a name="using-failover-groups-and-virtual-network-rules"></a>使用容錯移轉群組和虛擬網路規則

如果您使用[虛擬網路服務端點和規則](sql-database-vnet-service-endpoint-rule-overview.md)來限制存取您的 SQL 資料庫，請留意，每個虛擬網路服務端點只適用於一個 Azure 區域。 端點無法讓其他區域接受來自子網路的通訊。 因此，只有部署到相同區域中的用戶端應用程式可以連線到主要資料庫。 因為容錯移轉會導致 SQL 用戶端工作階段重新路由至不同 (次要) 區域中的伺服器，所以如果這些工作階段是來自該區域外的用戶端，就會失敗。 基於這個理由，如果參與的伺服器包含在虛擬網路規則中，則無法啟用自動容錯移轉原則。 若要支援手動容錯移轉，請遵循下列步驟：

1.  在次要地區中佈建您應用程式 (Web 服務、虛擬機器等) 前端元件的備援副本
2.  分別設定主要和次要伺服器的[虛擬網路規則](sql-database-vnet-service-endpoint-rule-overview.md)
3.  啟用[使用流量管理員設定的前端容錯移轉](sql-database-designing-cloud-solutions-for-disaster-recovery.md#scenario-1-using-two-azure-regions-for-business-continuity-with-minimal-downtime)
4.  偵測到中斷情況時，起始手動容錯移轉

這個選項最適合用於在前端和資料層之間需要一致延遲的應用程式，且支援當前端、資料層或兩者受到中斷影響時進行復原。 

> [!NOTE]
> 如果您使用**唯讀接聽程式**對唯讀工作負載進行負載平衡，請確保此工作負載會在 VM 或是次要地區的其他資源中執行，使其得以連線到次要資料庫。
>

 ### <a name="using-failover-groups-and-sql-database-firewall-rules"></a>使用容錯移轉群組和 SQL 資料庫防火牆規則

如果您的商務持續性計劃需要使用具有自動容錯移轉的群組，可以使用傳統的防火牆規則，來限制對您 SQL 資料庫的存取。  若要支援自動容錯移轉，請遵循下列步驟：

1.  [建立公用 IP](../virtual-network/virtual-network-public-ip-address.md#create-a-public-ip-address) 
2.  [建立公用負載平衡器](../load-balancer/quickstart-create-basic-load-balancer-portal.md#create-a-basic-load-balancer)並為其指派公用 IP。 
3.  [建立虛擬網路和虛擬機器](../load-balancer/quickstart-create-basic-load-balancer-portal.md#create-back-end-servers)以用於前端元件 
4.  [建立網路安全性群組](../virtual-network/security-overview.md)並設定輸入連線。 
5. 請使用 'Sql' [服務標籤](../virtual-network/security-overview.md#service-tags)確定會開啟到 Azure SQL 資料庫的輸出連線。 
5.  建立 [SQL 資料庫防火牆規則](sql-database-firewall-configure.md)以允許您在步驟 1 中所建立公用 IP 位址的輸入流量。 

如需如何設定輸出存取，以及在防火牆規則中使用哪些 IP 的相關詳細資訊，請參閱[負載平衡器連出連線](../load-balancer/load-balancer-outbound-connections.md)。

上述組態可確保自動容錯移轉不會封鎖來自前端元件的連線，並假設應用程式可以容忍前端與資料層之間較長的延遲。

> [!IMPORTANT]
> 若要保證區域性中斷時的商務持續性，您必須確定前端元件與資料庫的異地備援。 
>

## <a name="upgrading-or-downgrading-a-primary-database"></a>升級或降級主要資料庫
您可以將主要資料庫升級或降級至不同的計算大小 (在相同的服務層內)，而不需要將任何次要資料庫中斷連線。 升級時，我們建議您先升級次要資料庫，然後再升級主要資料庫。 降級時，順序相反︰先降級主要資料庫，然後再降級次要資料庫。 當您將資料庫升級或降級到不同的服務層時，會強制執行這項建議。 

> [!NOTE]
> 如果您已在容錯移轉群組設定中建立次要資料庫，則不建議降級次要資料庫。 這是為了確保您的資料層在容錯移轉啟動之後有足夠的容量來處理一般工作負載。 
>

## <a name="preventing-the-loss-of-critical-data"></a>防止重要資料遺失
由於廣域網路的高度延遲，連續複製採用非同步複寫機制。 如果發生失敗，非同步複寫導致部分資料遺失是無法避免的。 不過，有些應用程式可能會要求資料不能遺失。 若要保護這些重大更新，應用程式開發人員可以在認可交易後立即呼叫 [sp_wait_for_database_copy_sync](/sql/relational-databases/system-stored-procedures/active-geo-replication-sp-wait-for-database-copy-sync) 系統程序。 呼叫 **sp_wait_for_database_copy_sync** 會封鎖呼叫執行緒，直到最後認可的交易傳輸到次要資料庫。 不過，它不會等候在次要資料庫上重新執行和認可傳輸的交易。 **sp_wait_for_database_copy_sync** 以特定的連續複製連結為範圍。 任何具備主要資料庫連接權限的使用者都可以呼叫此程序。

> [!NOTE]
> **sp_wait_for_database_copy_sync** 可避免在容錯移轉之後資料遺失，但是不保證讀取權限會完整同步。 **sp_wait_for_database_copy_sync** 程序呼叫所造成的延遲可能會相當明顯，且取決於呼叫時的交易記錄大小。 
> 

## <a name="programmatically-managing-failover-groups-and-active-geo-replication"></a>以程式設計方式管理容錯移轉群組和主動式異地複寫
如前所述，自動容錯移轉群組和主動式異地複寫也可以使用 Azure PowerShell 和 REST API，以程式設計的方式管理。 下表描述可用的命令集。

**Azure Resource Manager API 和以角色為基礎的安全性**︰主動式異地複寫包含一組可管理的 Azure Resource Manager API，包括 [Azure SQL Database REST API](https://docs.microsoft.com/rest/api/sql/) 和 [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview)。 這些 API 需要使用資源群組，並支援以角色為基礎的安全性 (RBAC)。 如需如何實作存取角色的詳細資訊，請參閱 [Azure 角色型存取控制](../role-based-access-control/overview.md)。

## <a name="manage-sql-database-failover-using-transact-sql"></a>使用 Transact-SQL 管理 SQL 資料庫容錯移轉

| 命令 | 說明 |
| --- | --- |
| [ALTER DATABASE (Azure SQL Database)](/sql/t-sql/statements/alter-database-azure-sql-database) |使用 ADD SECONDARY ON SERVER 引數，針對現有資料庫建立次要資料庫並開始資料複寫 |
| [ALTER DATABASE (Azure SQL Database)](/sql/t-sql/statements/alter-database-azure-sql-database) |使用 FAILOVER 或 FORCE_FAILOVER_ALLOW_DATA_LOSS，將次要資料庫切換為主要資料庫以便開始容錯移轉 |
| [ALTER DATABASE (Azure SQL Database)](/sql/t-sql/statements/alter-database-azure-sql-database) |使用 REMOVE SECONDARY ON SERVER，來終止 SQL Database 和指定次要資料庫間的資料複寫。 |
| [sys.geo_replication_links (Azure SQL Database)](/sql/relational-databases/system-dynamic-management-views/sys-geo-replication-links-azure-sql-database) |針對 Azure SQL Database 邏輯伺服器上的每個資料庫，傳回所有結束複寫連結的相關資訊。 |
| [sys.dm_geo_replication_link_status (Azure SQL Database)](/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database) |針對指定的 SQL Database，取得上次複寫時間、上次複寫延遲，以及複寫連結的其他相關資訊。 |
| [sys.dm_operation_status (Azure SQL Database)](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database) |顯示所有資料庫作業的狀態，包括複寫連結的狀態。 |
| [sp_wait_for_database_copy_sync (Azure SQL Database)](/sql/relational-databases/system-stored-procedures/active-geo-replication-sp-wait-for-database-copy-sync) |導致應用程式等候，直到作用中次要資料庫複寫並認可所有認可的交易為止。 |
|  | |

## <a name="manage-sql-database-failover-using-powershell"></a>使用 PowerShell 管理 SQL 資料庫容錯移轉

| Cmdlet | 說明 |
| --- | --- |
| [Get-AzureRmSqlDatabase](/powershell/module/azurerm.sql/get-azurermsqldatabase) |取得一或多個資料庫。 |
| [New-AzureRmSqlDatabaseSecondary](/powershell/module/azurerm.sql/new-azurermsqldatabasesecondary) |針對現有資料庫建立次要資料庫並開始資料複寫。 |
| [Set-AzureRmSqlDatabaseSecondary](/powershell/module/azurerm.sql/set-azurermsqldatabasesecondary) |將次要資料庫切換為主要資料庫以開始容錯移轉。 |
| [Remove-AzureRmSqlDatabaseSecondary](/powershell/module/azurerm.sql/remove-azurermsqldatabasesecondary) |終止 SQL Database 和指定次要資料庫間的資料複寫。 |
| [Get-AzureRmSqlDatabaseReplicationLink](/powershell/module/azurerm.sql/get-azurermsqldatabasereplicationlink) |取得 Azure SQL Database 和資源群組或 SQL Server 之間的異地複寫連結。 |
| [New-AzureRmSqlDatabaseFailoverGroup](/powershell/module/azurerm.sql/set-azurermsqldatabasefailovergroup) |   此命令會建立容錯移轉群組，並同時在主要和次要伺服器上註冊|
| [Remove-AzureRmSqlDatabaseFailoverGroup](/powershell/module/azurerm.sql/remove-azurermsqldatabasefailovergroup) | 從伺服器移除容錯移轉群組，並刪除包含群組的所有次要資料庫 |
| [Get-AzureRmSqlDatabaseFailoverGroup](/powershell/module/azurerm.sql/get-azurermsqldatabasefailovergroup) | 擷取容錯移轉群組設定 |
| [Set-AzureRmSqlDatabaseFailoverGroup](/powershell/module/azurerm.sql/set-azurermsqldatabasefailovergroup) |   修改容錯移轉群組的設定 |
| [Switch-AzureRMSqlDatabaseFailoverGroup](/powershell/module/azurerm.sql/switch-azurermsqldatabasefailovergroup) | 觸發容錯移轉群組的容錯移轉到次要伺服器 |
|  | |

> [!IMPORTANT]
> 如需範例指令碼，請參閱[使用作用中異地複寫設定單一資料庫並進行容錯移轉](scripts/sql-database-setup-geodr-and-failover-database-powershell.md)、[使用作用中異地複寫設定集區資料庫並進行容錯移轉](scripts/sql-database-setup-geodr-and-failover-pool-powershell.md)、[設定單一資料庫的容錯移轉群組並進行容錯移轉](scripts/sql-database-setup-geodr-failover-database-failover-group-powershell.md)。
>

## <a name="manage-sql-database-failover-using-the-rest-api"></a>使用 REST API 管理 SQL 資料庫容錯移轉
| API | 說明 |
| --- | --- |
| [Create or Update Database (createMode=Restore)](/rest/api/sql/Databases/CreateOrUpdate) |建立、更新或還原主要或次要資料庫。 |
| [取得建立或更新資料庫狀態](/rest/api/sql/Databases/CreateOrUpdate) |在建立作業期間傳回狀態。 |
| [將次要資料庫設定為主要資料庫 (計劃性容錯移轉)](/rest/api/sql/replicationlinks/failover) |從目前主要複本資料庫進行容錯移轉，以設定主要的複本資料庫。 |
| [將次要資料庫設定為主要資料庫 (非計劃的容錯移轉)](/rest/api/sql/replicationlinks/failoverallowdataloss) |從目前主要複本資料庫進行容錯移轉，以設定主要的複本資料庫。 這項作業可能會導致資料遺失。 |
| [取得複寫連結](/rest/api/sql/replicationlinks/get) |取得異地複寫關聯性中指定 SQL Database 的特定複寫連結。 它會擷取 sys.geo_replication_links 目錄檢視中顯示的資訊。 |
| [複寫連結 - 依資料庫列示](/rest/api/sql/replicationlinks/listbydatabase) | 取得異地複寫關聯性中指定 SQL Database 的所有複寫連結。 它會擷取 sys.geo_replication_links 目錄檢視中顯示的資訊。 |
| [刪除複寫連結](/rest/api/sql/databases/delete) | 刪除資料庫複寫連結。 無法在容錯移轉期間進行。 |
| [建立或更新容錯移轉群組](/rest/api/sql/failovergroups/createorupdate) | 建立或更新容錯移轉群組 |
| [刪除容錯移轉群組](/rest/api/sql/failovergroups/delete) | 從伺服器中移除容錯移轉群組 |
| [容錯移轉 (計劃性)](/rest/api/sql/failovergroups/failover) | 從目前主要伺服器容錯移轉到此伺服器。 |
| [強制容錯移轉允許資料遺失](/rest/api/sql/failovergroups/forcefailoverallowdataloss) |從目前主要伺服器容錯移轉到此伺服器。 這項作業可能會導致資料遺失。 |
| [取得容錯移轉群組](/rest/api/sql/failovergroups/get) | 取得容錯移轉群組。 |
| [依伺服器列出容錯移轉群組](/rest/api/sql/failovergroups/listbyserver) | 列出伺服器中的容錯移轉群組。 |
| [更新容錯移轉群組](/rest/api/sql/failovergroups/update) | 更新容錯移轉群組。 |
|  | |

## <a name="next-steps"></a>後續步驟
* 如需範例指令碼，請參閱：
   - [使用作用中異地複寫設定單一資料庫並進行容錯移轉](scripts/sql-database-setup-geodr-and-failover-database-powershell.md)
   - [使用作用中異地複寫設定集區資料庫並進行容錯移轉](scripts/sql-database-setup-geodr-and-failover-pool-powershell.md)
   - [設定單一資料庫的容錯移轉群組並進行容錯移轉](scripts/sql-database-setup-geodr-failover-database-failover-group-powershell.md)
* 如需商務持續性概觀和案例，請參閱 [商務持續性概觀](sql-database-business-continuity.md)
* 若要了解 Azure SQL Database 自動備份，請參閱 [SQL Database 自動備份](sql-database-automated-backups.md)。
* 若要了解如何使用自動備份進行復原，請參閱 [從服務起始的備份還原資料庫](sql-database-recovery-using-backups.md)。
* 若要深入了解新的主要伺服器和資料庫的驗證需求，請參閱 [災害復原後的 SQL Database 安全性](sql-database-geo-replication-security-config.md)。

