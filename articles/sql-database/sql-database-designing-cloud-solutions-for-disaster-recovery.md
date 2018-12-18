---
title: 使用 Azure SQL Database 設計全域可用的服務 | Microsoft Docs
description: 了解使用 Azure SQL Database 對高可用性服務的應用程式設計。
keywords: cloud disaster recovery,disaster recovery solutions,app data backup,geo-replication,business continuity planning,雲端災害復原,災害復原方案,應用程式資料備份,異地複寫,商務持續性計劃
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: carlrab
manager: craigg
ms.date: 07/26/2018
ms.openlocfilehash: 8522fea10a4ec8f85d20e5a9ec04712c77bb6b94
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47064263"
---
# <a name="designing-globally-available-services-using-azure-sql-database"></a>使用 Azure SQL Database 設計全域可用的服務

使用 Azure SQL Database 建置和部署雲端服務時，您可以使用[容錯移轉群組和主動式異地複寫](sql-database-geo-replication-overview.md)來為區域性中斷和嚴重的故障提供恢復能力。 相同的功能可讓您建立已針對本機資料存取進行最佳化的全域分散式應用程式。 本文會討論常見的應用程式模式，包括每個選項的優缺點。 

> [!NOTE]
> 如果您目前使用進階或業務關鍵資料庫和彈性集區，您可以將它們轉換成區域備援部署組態，使它們具備區域中斷復原能力。 請參閱[區域備援資料庫](sql-database-high-availability.md)。  

## <a name="scenario-1-using-two-azure-regions-for-business-continuity-with-minimal-downtime"></a>情節 1：使用兩個 Azure 區域以獲得最少停機時間的業務持續性
在此情節中，應用程式具有下列特性： 
*   應用程式在一個 Azure 區域中為作用中
*   所有的資料庫工作階段都需要資料的讀取和寫入權限 (RW)
*   必須共置 web 層和資料層才能減少延遲和流量成本 
*   基本上，停機時間對於這些應用程式的商業風險高於資料遺失

在此情況下，當所有應用程式元件需要共同容錯移轉時，應用程式部署拓撲最適合用來處理區域性災害。 下圖顯示此拓撲。 如需地理備援，會將應用程式的資源部署至區域 A 和 B。不過，並不會使用區域 B 中的資源，直到區域 A 失敗為止。 會在兩個區域之間容錯移轉群組，以管理資料庫連線、複寫和容錯移轉。 兩個區域中的 web 服務都會設定為透過讀寫接聽程式 **&lt;failover-group-name&gt;.database.windows.net** (1) 存取資料庫。 流量管理員設為使用[優先順序路由方法](../traffic-manager/traffic-manager-configure-priority-routing-method.md) (2)。  

> [!NOTE]
> 這整篇文章使用 [Azure 流量管理員](../traffic-manager/traffic-manager-overview.md)僅供說明用途。 您可以使用任何支援優先順序路由方法的負載平衡方案。    
>

下圖顯示此設定在運作中斷之前的情形：

![案例 1. 在中斷之前的設定。](./media/sql-database-designing-cloud-solutions-for-disaster-recovery/scenario1-a.png)

在主要區域中斷之後，SQL Database 服務會偵測到主要資料庫無法存取，並以自動容錯移轉原則的參數作為基礎，觸發容錯移轉至次要區域 (1)。 根據您的應用程式 SLA，可以在中斷偵測和容錯移轉本身之間設定可控制時間的寬限期。 流量管理員可能會在容錯移轉群組觸發資料庫之前，起始端點容錯移轉。 在此情況下，web 應用程式無法立即重新連線至資料庫。 但是，資料庫容錯移轉一旦完成，重新連線就會自動成功。 當失敗的區域還原並再度上線時，舊的主要區域會自動重新連線作為新的次要區域。 下圖說明容錯移轉後的設定。
 
> [!NOTE]
> 在容錯移轉之後認可的所有交易會在重新連線時遺失。 容錯移轉完成之後，區域 B 中的應用程式就能夠重新連線，然後重新開始處理使用者要求。 Web 應用程式與主要資料庫現在都位於區域 B，且仍維持共置。 n>

![案例 1. 容錯移轉之後的設定](./media/sql-database-designing-cloud-solutions-for-disaster-recovery/scenario1-b.png)

如果在區域 B 中發生中斷情況，主要與次要資料庫之間的複寫程序就會暫停，但兩者之間的連結會維持不變 (1)。 受控流量會偵測到區域 B 的連線已中斷，並且將端點 web 應用程式 2 標示為已降級 (2)。 在此情況下，不會影響應用程式的效能，但是資料庫會變成公開，因此當區域 A 失敗時，資料遺失的風險會較高。

> [!NOTE]
> 災害復原的應用程式部署設定，建議限制在兩個區域。 這是因為大部分的 Azure 地理位置只有兩個區域。 這些設定不會保護應用程式免於遭受兩個區域同時發生的重大失敗。 在這種罕見的失敗情況下，您可以使用[異地復原作業](sql-database-disaster-recovery.md#recover-using-geo-restore)，在第三個區域復原資料庫。
>

 一旦運作中斷情況趨緩後，次要資料庫會自動與主要資料庫同步處理。 同步處理期間，可能會影響主要區域的效能。 特定的影響取決於容錯移轉之後，新的主要區域取得的資料量。 下圖說明次要地區發生運作中斷的情形：

![案例 1. 次要區域中發生中斷後的設定。](./media/sql-database-designing-cloud-solutions-for-disaster-recovery/scenario1-c.png)

此設計模式的主要 **優點** 包括：

* 這兩個區域會部署相同的 Web 應用程式，不需要任何特定地區設定和任何其他邏輯來回應容錯移轉。 
* 應用程式的效能不會受到容錯移轉所影響，因為 Web 應用程式和資料庫一律為共置。

主要的**取捨**是區域 B 中的應用程式資源在大部分的時間使用量過低。

## <a name="scenario-2-azure-regions-for-business-continuity-with-maximum-data-preservation"></a>情節 2：達到最大資料保留之業務持續性的 Azure 區域
這個選項最適合具有下列特性的應用程式：

* 任何資料遺失都會帶來極高的商業風險。 資料庫容錯移轉只能當作嚴重失敗造成之中斷情況時的最後手段。
* 應用程式支援唯讀和讀寫模式的作業，而且可以「唯讀模式」運作一段時間。

在此模式中，應用程式會在讀寫連接開始收到逾時錯誤時切換到唯讀模式。 Web 應用程式要部署到這兩個區域，並且包含讀寫接聽程式端點的連線，和唯讀接聽程式端點的不同連線 (1)。 流量管理員設定檔應該使用[優先順序路由](../traffic-manager/traffic-manager-configure-priority-routing-method.md)。 應在每個區域中啟用應用程式端點的[結束點監視](../traffic-manager/traffic-manager-monitoring.md) (2)。

下圖說明此設定在運作中斷之前的情形：

![案例 2. 在中斷之前的設定。](./media/sql-database-designing-cloud-solutions-for-disaster-recovery/scenario2-a.png)

當流量管理員偵測到區域 A 的連線失敗時，會自動將使用者流量切換到區域 B 中的應用程式執行個體。使用此模式時，請務必將資料遺失的寬限期設為夠高的值，例如 24 小時。 如果在該時間內中斷情況已趨緩，這可以確保避免遺失資料。 當區域 B 的 Web 應用程式啟動時，讀寫作業將會開始失敗。 此時應該會切換到唯讀模式 (1)。 在此模式中，要求將會自動路由到次要資料庫。 如果中斷是由重大錯誤所導致，就很有可能無法在寬限期內減輕。 到期時，容錯移轉群組就會觸發容錯移轉。 之後就可以使用讀寫接聽程式，而連線不會再失敗 (2)。 下圖說明復原程序的兩個階段。

> [!NOTE]
> 如果主要區域的運作中斷情況在寬限期內趨緩，流量管理員就會偵測到主要區域中的連線恢復，然後將使用者流量切換回到區域 A 中的應用程式執行個體。該應用程式執行個體會繼續使用區域 A 的主要資料庫以讀寫模式運作，如先前圖表中所示。
>

![案例 2. 災難復原階段。](./media/sql-database-designing-cloud-solutions-for-disaster-recovery/scenario2-b.png)

如果是在區域 B 中發生中斷情況，流量管理員會在區域 B 中偵測到失敗的結束點 web-app-2，並將它標記為效能降低 (1)。 在此同時，容錯移轉群組會將唯讀接聽程式切換至區域 A (2)。 此中斷不會影響使用者體驗，但會在中斷期間公開主要資料庫。 下圖說明次要地區發生失敗的情形：

![案例 2. 次要地區中斷。](./media/sql-database-designing-cloud-solutions-for-disaster-recovery/scenario2-c.png)

一旦中斷情況趨緩，次要資料庫會立即與主要資料庫同步處理，唯讀接聽程式會切換回區域 B 中的次要資料庫。在同步處理期間，主要資料庫的效能可能會根據需要進行同步處理的資料量而稍微受到影響。

此設計模式有幾個 **優點**：

* 在暫時性運作中斷期間避免資料遺失。
* 停機時間僅取決於流量管理員多快偵測到連線失敗，而這是可設定的。

**取捨**是指應用程式必須能夠在唯讀模式下運作。

## <a name="scenario-3-application-relocation-to-a-different-geography-without-data-loss-and-near-zero-downtime"></a>情節 3：應用程式重新配置到不同的地理位置而不會遺失資料且幾乎零停機時間 
在此情節中，應用程式具有下列特性： 
* 終端使用者從不同的地理位置存取應用程式
* 應用程式包括唯讀工作負載，並非根據包含最新更新的完整同步處理
* 針對多數使用者，必須在相同的地理位置支援寫入資料的存取權 
* 讀取延遲對終端使用者經驗很重要 


為了符合這些需求，您必須保證使用者裝置**一律**會連線到部署在相同地理位置中進行唯讀作業的應用程式，例如瀏覽資料、分析等。而**大部分時間**會在相同的地理位置中處理 OLTP 作業。 例如，在日間時間內，會在相同的地理位置中處理 OLTP 作業，但在下班時間內，它們會在不同的地理位置中進行處理。 如果終端使用者活動通常是發生在工作時間內，就可以保證大部分使用者在大多數情況下都能獲得最佳的效能。 下圖顯示此拓撲。 
 
應該將應用程式的資源部署在您有大量使用需求的每個地理位置。 例如，如果您是在美國境內、歐盟和東南亞主動使用應用程式，就應該將應用程式部署在這所有的地理位置。 在工作時間結束時，應該以動態方式將主要資料庫從一個地理位置切換到下一個地理位置。 此方法稱為「追日」。 OLTP 工作負載一律會透過讀寫接聽程式 **&lt;failover-group-name&gt;.database.windows.net** 連線到資料庫 (1)。 唯讀工作負載會使用資料庫伺服器端點 **&lt;server-name&gt;.database.windows.net**，直接連線至本機資料庫 (2)。 已使用[效能路由方法](../traffic-manager/traffic-manager-configure-performance-routing-method.md)設定流量管理員。 它會確保使用者的裝置連線到最接近區域中的 web 服務。 應使用針對每個 web 服務端點啟用的端點監視來設定流量管理員 (3)。

> [!NOTE]
> 容錯移轉群組設定會定義哪些區域會用於容錯移轉。 因為新的主要區域位於不同的地理位置中，容錯移轉會導致 OLTP 和唯讀工作負載較長的延遲，直到受影響區域再度上線為止。
>

![案例 3. 美國東部主要區域的設定。](./media/sql-database-designing-cloud-solutions-for-disaster-recovery/scenario3-a.png)

在每天結束時 (例如本地時間晚上 11 點)，作用中的資料庫應該切換到下一個區域 (北歐)。 可以使用 [Azure 排程服務](../scheduler/scheduler-intro.md)將這項工作全面自動化。  此工作涉及下列步驟：
* 使用好記的容錯移轉，將容錯移轉群組中的主要伺服器切換至北歐 (1)
* 將美國東部和北歐之間的容錯移轉群組移除
* 使用相同名稱但在北歐和東亞之間建立新的容錯移轉群組 (2)。 
* 在北歐將主要伺服器及在東亞將次要伺服器新增至此容錯移轉群組 (3)。


下圖說明規劃容錯移轉之後的新設定：

![案例 3. 將主要伺服器轉換至北歐。](./media/sql-database-designing-cloud-solutions-for-disaster-recovery/scenario3-b.png)

例如，如果在北歐發生中斷情況，容錯移轉群組就會起始自動資料庫容錯移轉，從而將應用程式在排程之前有效地移動到下一個區域 (1)。  在此情況下，北歐再度上線之前，美國東部是唯一剩餘的次要區域。 其餘的兩個區域會藉由切換角色來服務所有三個地理位置的客戶。 Azure 排程器必須隨之調整。 因為其餘的區域會從歐洲取得額外的使用者流量，應用程式的效能不只會受到額外延遲時間的影響，還會受到終端使用者連線遽增的影響。 一旦北歐的運作中斷情況趨緩，該處的次要資料庫會立即與目前的主要資料庫進行同步處理。 下圖說明北歐發生運作中斷的情形：

![案例 3. 北歐發生中斷。](./media/sql-database-designing-cloud-solutions-for-disaster-recovery/scenario3-c.png)

> [!NOTE]
> 在歐洲的終端使用者體驗由於很長的延遲而降級時，您可以減少時間。 若要這麼做，您應該主動部署應用程式複本，並在另一個本地區域 (西歐) 中建立次要資料庫，以取代北歐的離線應用程式執行個體。 後者再度上線時，您可以決定是否要使用西歐繼續進行，還是要移除該處的應用程式複本，然後切換回使用北歐。
>

這項設計的主要**優點**如下：
* 唯讀應用程式工作負載隨時都能存取最接近區域中的資料。 
* 在每個地理位置中的最高活動期間，讀寫應用程式工作負載會存取最接近區域中的資料
* 因為應用程式是部署到多個區域，它可以免於遺失其中一個區域，而不需要任何重大停機。 

但會有一些**取捨**：
* 地區中斷會導致地理位置受到較長延遲的影響。 讀寫和唯讀工作負載都是由不同地理位置中的應用程式提供服務。 
* 唯讀工作負載必須連線到每個區域中的不同結束點。 


## <a name="business-continuity-planning-choose-an-application-design-for-cloud-disaster-recovery"></a>商務持續性計劃：針對雲端災害復原選擇應用程式設計
特定的雲端災害復原策略可以合併或擴充這些設計模式，以完全滿足應用程式的需求。  如先前所述，您選擇的策略取決於您想要提供給客戶的 SLA 和應用程式部署拓樸。 為了協助指導您做決定，下表是以復原點目標 (RPO) 和預估復原時間 (ERT) 作為基礎來比較各種選擇。

| 模式 | RPO | ERT |
|:--- |:--- |:--- |
| 災害復原的主動-被動部署，使用共置資料庫存取 |讀寫存取 < 5 秒 |失敗偵測時間 + DNS TTL |
| 應用程式負載平衡的主動-主動部署 |讀寫存取 < 5 秒 |失敗偵測時間 + DNS TTL |
| 資料保留的主動-被動部署 |唯讀存取 < 5 秒 | 唯讀存取 = 0 |
||讀寫存取 = 0 | 讀寫存取 = 失敗偵測時間 + 遺失資料的寬限期 |
|||

## <a name="next-steps"></a>後續步驟
* 如需商務持續性概觀和案例，請參閱 [商務持續性概觀](sql-database-business-continuity.md)
* 若要深入了解異地複寫和容錯移轉群組，請參閱[作用中異地複寫](sql-database-geo-replication-overview.md)  
* 如需主動式異地複寫與彈性集區的相關資訊，請參閱[彈性集區災害復原策略](sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool.md)。
