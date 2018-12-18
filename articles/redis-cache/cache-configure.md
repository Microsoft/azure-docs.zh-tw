---
title: 如何設定 Azure Redis 快取 | Microsoft Docs
description: 了解 Azure Redis 快取的預設 Redis 組態，以及了解如何設定您的 Azure Redis 快取執行個體
services: redis-cache
documentationcenter: na
author: wesmc7777
manager: cfowler
editor: tysonn
ms.assetid: d0bf2e1f-6a26-4e62-85ba-d82b35fc5aa6
ms.service: cache
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: cache-redis
ms.workload: tbd
ms.date: 08/22/2017
ms.author: wesmc
ms.openlocfilehash: 58e913aa2b4a89a573f6d901803979e662c27af2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954036"
---
# <a name="how-to-configure-azure-redis-cache"></a>如何設定 Azure Redis 快取
本主題說明適用於您 Azure Redis 快取執行個體的設定。 本主題也會涵蓋適用於 Azure Redis 快取執行個體的預設 Redis 伺服器設定。

> [!NOTE]
> 如需設定及使用進階快取功能的詳細資訊，請參閱[如何設定持續性](cache-how-to-premium-persistence.md)、[如何設定叢集](cache-how-to-premium-clustering.md)，以及[如何設定虛擬網路支援](cache-how-to-premium-vnet.md)。
> 
> 

## <a name="configure-redis-cache-settings"></a>設定 Redis 快取設定
[!INCLUDE [redis-cache-create](../../includes/redis-cache-browse.md)]

您可以在 [Redis 快取] 刀鋒視窗中使用 [資源功能表] 檢視及設定 Azure Redis 快取設定。

![Redis 快取設定](./media/cache-configure/redis-cache-settings.png)

您可以使用 [資源功能表] 檢視及設定下列設定。

* [概觀](#overview)
* [活動記錄檔](#activity-log)
* [存取控制 (IAM)](#access-control-iam)
* [標記](#tags)
* [診斷並解決問題](#diagnose-and-solve-problems)
* [設定](#settings)
    * [存取金鑰](#access-keys)
    * [進階設定](#advanced-settings)
    * [Redis 快取顧問](#redis-cache-advisor)
    * [調整](#scale)
    * [Redis 叢集大小](#cluster-size)
    * [Redis 資料永續性](#redis-data-persistence)
    * [排程更新](#schedule-updates)
    * [異地複寫](#geo-replication)
    * [虛擬網路](#virtual-network)
    * [防火牆](#firewall)
    * [屬性](#properties)
    * [鎖定](#locks)
    * [自動化指令碼](#automation-script)
* [系統管理](#administration)
    * [匯入資料](#importexport)
    * [匯出資料](#importexport)
    * [重新啟動](#reboot)
* [監視](#monitoring)
    * [Redis 度量](#redis-metrics)
    * [警示規則](#alert-rules)
    * [診斷](#diagnostics)
* [支援和疑難排解設定](#support-amp-troubleshooting-settings)
    * [資源健康情況](#resource-health)
    * [新的支援要求](#new-support-request)


## <a name="overview"></a>概觀

**概觀**提供您快取的基本資訊，例如名稱、連接埠、定價層，以及選取的快取度量。

### <a name="activity-log"></a>活動記錄檔

按一下 [活動記錄]  ，以檢視在快取上執行的動作。 您也可以使用篩選，來展開此檢視以包含其他資源。 如需使用稽核記錄的詳細資訊，請參閱[使用 Resource Manager 來稽核作業](../azure-resource-manager/resource-group-audit.md)。 如需有關監視 Azure Redis 快取事件的詳細資訊，請參閱 [作業和警示](cache-how-to-monitor.md#operations-and-alerts)。

### <a name="access-control-iam"></a>存取控制 (IAM)

**存取控制 (IAM)** 區段提供 Azure 入口網站中對於角色型存取控制 (RBAC) 的支援。 此設定可協助組織簡單而精準地符合其存取管理需求。 如需詳細資訊，請參閱 [Azure 入口網站中的角色型存取控制](../role-based-access-control/role-assignments-portal.md)。

### <a name="tags"></a>標記

[標記]  區段有助於您組織資源。 如需詳細資訊，請參閱 [使用標記組織您的 Azure 資源](../azure-resource-manager/resource-group-using-tags.md)。


### <a name="diagnose-and-solve-problems"></a>診斷並解決問題

按一下 [診斷並解決問題]  將提供您這些常見問題和解決問題的策略。



## <a name="settings"></a>設定
[設定] 區段中的設定可讓您存取和設定下列快取設定。

* [存取金鑰](#access-keys)
* [進階設定](#advanced-settings)
* [Redis 快取顧問](#redis-cache-advisor)
* [調整](#scale)
* [Redis 叢集大小](#cluster-size)
* [Redis 資料永續性](#redis-data-persistence)
* [排程更新](#schedule-updates)
* [異地複寫](#geo-replication)
* [虛擬網路](#virtual-network)
* [防火牆](#firewall)
* [屬性](#properties)
* [鎖定](#locks)
* [自動化指令碼](#automation-script)



### <a name="access-keys"></a>存取金鑰
按一下 [存取金鑰]  以檢視或重新產生快取的存取金鑰。 這些金鑰是由連線到您的快取的用戶端所使用。

![Redis 快取存取金鑰](./media/cache-configure/redis-cache-manage-keys.png)

### <a name="advanced-settings"></a>進階設定
下列設定是在 [進階設定] 刀鋒視窗上進行設定。

* [存取連接埠](#access-ports)
* [記憶體原則](#memory-policies)
* [Keyspace 通知 (進階設定)](#keyspace-notifications-advanced-settings)

#### <a name="access-ports"></a>存取連接埠
根據預設，新的快取會停用非 SSL 存取。 若要啟用非 SSL 連接埠，請針對 [進階設定] 刀鋒視窗上的 [只允許透過 SSL 存取]，按一下 [否]，然後按一下 [儲存]。

![Redis 快取存取連接埠](./media/cache-configure/redis-cache-access-ports.png)

<a name="maxmemory-policy-and-maxmemory-reserved"></a>
#### <a name="memory-policies"></a>記憶體原則
[進階設定] 刀鋒視窗中的 [Maxmemory 原則]、[maxmemory-reserved] 和 [maxfragmentationmemory-reserved] 設定，會設定快取的記憶體原則。

![Redis 快取 Maxmemory 原則](./media/cache-configure/redis-cache-maxmemory-policy.png)

[Maxmemory 原則] 設定快取的收回原則，並讓您從下列收回原則中選擇：

* `volatile-lru`：這是預設的收回原則。
* `allkeys-lru`
* `volatile-random`
* `allkeys-random`
* `volatile-ttl`
* `noeviction`

如需 `maxmemory` 原則的詳細資訊，請參閱[收回原則](http://redis.io/topics/lru-cache#eviction-policies)。

**maxmemory-reserved** 設定會設定保留給非快取作業 (例如容錯移轉期間的複寫) 的記憶體量 (MB)。 設定此值可讓您在負載變動時具有更一致的 Redis 伺服器體驗。 對於頻繁寫入的工作負載，此值應該設定為更高的值。 當記憶體保留給這類作業時，無法用於儲存快取的資料。

[maxfragmentationmemory-reserved] 設定會以 MB 為單位設定保留的記憶體數量，以容納過於分散的記憶體。 設定此值可讓您在快取已滿或接近全滿，且片段比率很高時，擁有更為一致的 Redis 伺服器體驗。 當記憶體保留給這類作業時，無法用於儲存快取的資料。

選擇新的記憶體保留值 (**maxmemory-reserved** 或 **maxfragmentationmemory-reserved**) 時，需要考慮的一件事是，這項變更對已有大量資料在執行的快取會有怎麼樣的影響。 例如，如果您有容量為 53 GB 的快取，而其中有 49 GB 的資料，則將保留值變更為 8 GB，此變更會將系統的最大可用記憶體降至 45 GB。 如果目前的 `used_memory` 或 `used_memory_rss` 值高於 45 GB 的新限制，則等 `used_memory` 和 `used_memory_rss` 都低於 45 GB 後，系統必須收回資料。 收回會增加伺服器負載並讓記憶體過於分散。 如需快取計量的詳細資訊，例如 `used_memory` 和 `used_memory_rss`，請參閱[可用計量和報告間隔](cache-how-to-monitor.md#available-metrics-and-reporting-intervals)。

> [!IMPORTANT]
> 只有標準和高階快取提供 **maxmemory-reserved** 和 **maxfragmentationmemory-reserved** 設定。
> 
> 

#### <a name="keyspace-notifications-advanced-settings"></a>Keyspace 通知 (進階設定)
Redis Keyspace 通知是在 [進階設定]  刀鋒視窗上進行設定。 Keyspace 通知可讓用戶端在特定事件發生時收到通知。

![Redis 快取進階設定](./media/cache-configure/redis-cache-advanced-settings.png)

> [!IMPORTANT]
> Keyspace 通知和 **notify-keyspace-events** 設定只適用於標準和高階快取。
> 
> 

如需詳細資訊，請參閱 [Redis Keyspace 通知](http://redis.io/topics/notifications)(英文)。 如需範例程式碼，請參閱 [Hello world](https://github.com/rustd/RedisSamples/tree/master/HelloWorld) 範例中的 [KeySpaceNotifications.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/KeySpaceNotifications.cs) 檔案。


<a name="recommendations"></a>
## <a name="redis-cache-advisor"></a>Redis 快取顧問
[Redis 快取建議程式] 刀鋒視窗會顯示適用於快取的建議。 在一般作業期間，不會顯示任何建議。 

![建議](./media/cache-configure/redis-cache-no-recommendations.png)

如果在快取作業期間發生任何狀況 (例如，高記憶體使用量、網路頻寬或伺服器負載)，即會在 [Redis 快取]  刀鋒視窗中顯示警示。

![建議](./media/cache-configure/redis-cache-recommendations-alert.png)

您可以在 [建議]  刀鋒視窗中找到進一步資訊。

![建議](./media/cache-configure/redis-cache-recommendations.png)

您可以在 [Redis 快取] 刀鋒視窗的[監視圖表](cache-how-to-monitor.md#monitoring-charts)和[使用量圖表](cache-how-to-monitor.md#usage-charts)區段上監視這些計量。

每個定價層都有不同的用戶端連線、記憶體和頻寬的限制。 如果您的快取持續一段時間接近這些計量的最大容量，即會提供建議。 如需透過**建議**工具檢閱的計量和限制詳細資訊，請參閱下表：

| Redis 快取計量 | 詳細資訊 |
| --- | --- |
| 網路頻寬使用量 |[快取效能 - 可用的頻寬](cache-faq.md#cache-performance) |
| 連線的用戶端 |[預設 Redis 伺服器組態 - maxclients](#maxclients) |
| 伺服器負載 |[使用量圖表 - Redis 伺服器負載](cache-how-to-monitor.md#usage-charts) |
| 記憶體使用量 |[快取效能 - 大小](cache-faq.md#cache-performance) |

若要升級快取，按一下 [立即升級]  以變更定價層及[調整](#scale)您的快取。 如需選擇定價層的詳細資訊，請參閱 [應該使用哪個 Redis 快取供應項目和大小？](cache-faq.md#what-redis-cache-offering-and-size-should-i-use)


### <a name="scale"></a>調整
按一下 [調整] 以檢視或變更快取的定價層。 如需調整的詳細資訊，請參閱 [如何調整 Azure Redis Cache](cache-how-to-scale.md)。

![Redis 快取定價層](./media/cache-configure/pricing-tier.png)

<a name="cluster-size"></a>

### <a name="redis-cluster-size"></a>Redis 叢集大小
按一下 [Redis 叢集大小 (預覽)]  ，針對已啟用叢集且目前執行中的進階快取，變更叢集大小。

> [!NOTE]
> 請注意，雖然 Azure Redis 快取進階層已發行正式上市版，但 Redis 叢集大小功能目前為預覽狀態。
> 
> 

![Redis 叢集大小](./media/cache-configure/redis-cache-redis-cluster-size.png)

若要變更叢集大小，請使用滑桿，或在 [分區計數] 文字方塊中輸入 1 到 10 之間的數字，然後按一下 [確定] 加以儲存。

> [!IMPORTANT]
> Redis 叢集只適用於進階快取。 如需詳細資訊，請參閱 [如何設定進階 Azure Redis 快取叢集](cache-how-to-premium-clustering.md)。
> 
> 


### <a name="redis-data-persistence"></a>Redis 資料永續性
按一下 [Redis 資料持續性]  可加以啟用、停用，或設定高階快取的資料持續性。 Azure Redis 快取使用 [RDB 持續性](cache-how-to-premium-persistence.md#configure-rdb-persistence)或 [AOF 持續性](cache-how-to-premium-persistence.md#configure-aof-persistence)來提供 Redis 持續性。

如需詳細資訊，請參閱 [如何設定進階 Azure Redis 快取的持續性](cache-how-to-premium-persistence.md)。


> [!IMPORTANT]
> Redis 資料持續性僅適用於進階快取。 
> 
> 

### <a name="schedule-updates"></a>更新排程
[排程更新]  刀鋒視窗可讓您指定適用於快取的 Redis 伺服器更新維護期間。 

> [!IMPORTANT]
> 維護期間僅適用於 Redis 伺服器更新，不適用於任何 Azure 更新，或是在裝載快取的 VM 上更新作業系統。
> 
> 

![更新排程](./media/cache-configure/redis-schedule-updates.png)

若要指定維護期間，請檢查所需的天數，並指定每一天的維護期間開始小時，然後按一下 [確定] 。 請注意，維護期間時間是 UTC。 

> [!IMPORTANT]
> 只有進階層快取提供**排程更新**功能。 如需詳細資訊和指示，請參閱 [Azure Redis 快取管理 - 排程更新](cache-administration.md#schedule-updates)。
> 
> 

### <a name="geo-replication"></a>異地複寫

[異地複寫] 刀鋒視窗提供一個機制，可供連結兩個進階層 Azure Redis 快取執行個體。 其中一個快取被指定為主要連結快取，而另一個則為次要連結快取。 次要連結快取會變成唯讀，而寫入主要快取的資料會複寫至次要連結快取。 這項功能可用來跨 Azure 區域複寫快取。

> [!IMPORTANT]
> **異地複寫**僅適用於進階層快取。 如需詳細資訊和指示，請參閱[如何設定 Azure Redis 快取的異地複寫](cache-how-to-geo-replication.md)。
> 
> 

### <a name="virtual-network"></a>虛擬網路
[虛擬網路] 區段可讓您為快取設定虛擬網路。 如需使用 VNET 支援建立進階快取並更新其設定的相關資訊，請參閱 [如何設定進階 Azure Redis 快取的虛擬網路支援](cache-how-to-premium-vnet.md)。

> [!IMPORTANT]
> 虛擬網路設定只適用於快取建立期間利用 VNET 支援設定的進階快取。 
> 
> 

### <a name="firewall"></a>防火牆

防火牆規則設定適用於所有 Azure Redis 快取層。

按一下 [防火牆] 以檢視和設定快取的防火牆規則。

![防火牆](./media/cache-configure/redis-firewall-rules.png)

您可以利用開始和結束 IP 位址範圍來指定防火牆規則。 設定防火牆規則時，只有來自指定 IP 位址範圍的用戶端連線可以連接至快取。 儲存防火牆規則時，在規則生效之前，會有短暫的延遲。 此延遲通常不超過一分鐘。

> [!IMPORTANT]
> 一律允許來自 Azure Redis 快取監視系統的連線，即使設定了防火牆規則也一樣。
> 
> 

### <a name="properties"></a>properties
按一下 [屬性]  以檢視快取的相關資訊，包括快取端點和連接埠。

![Redis 快取屬性](./media/cache-configure/redis-cache-properties.png)

### <a name="locks"></a>鎖定
[鎖定] 區段可讓您鎖定訂用帳戶、資源群組或資源，以防止組織中的其他使用者不小心刪除或修改重要資源。 如需詳細資訊，請參閱 [使用 Azure 資源管理員來鎖定資源](../azure-resource-manager/resource-group-lock-resources.md)。

### <a name="automation-script"></a>自動化指令碼

按一下 [自動化指令碼] 可建置並匯出您所部署資源的範本，供未來部署使用。 如需使用範本的詳細資訊，請參閱 [使用 Azure Resource Manager 範本部署資源](../azure-resource-manager/resource-group-template-deploy.md)。

## <a name="administration-settings"></a>管理設定
[管理] 區段中的設定可讓您針對快取執行下列管理工作。 

![系統管理](./media/cache-configure/redis-cache-administration.png)

* [匯入資料](#importexport)
* [匯出資料](#importexport)
* [重新啟動](#reboot)


### <a name="importexport"></a>匯入/匯出
匯入/匯出是 Azure Redis 快取的資料管理作業，可讓您匯入或匯出快取中的資料，方法是從進階快取將 Redis 快取資料庫 (RDB) 快照集匯入和匯出至 Azure 儲存體帳戶中的分頁 Blob。 匯入/匯出可讓您在不同的 Azure Redis 快取執行個體之間移轉，或在使用前將資料填入快取。

匯入可以用來從任何雲端或環境中執行的 Redis 伺服器 (包含在 Linux、Windows 上執行的 Redis，或任何雲端提供者，例如 Amazon Web Services 等) 引入 Redis 相容 RDB 檔案。 匯入資料是使用預先填入資料建立快取的輕鬆方式。 在匯入程序期間，Azure Redis 快取會從 Azure 儲存體將 RDB 檔案載入記憶體，然後將金鑰插入快取。

匯出可讓您將儲存在 Azure Redis 快取中的資料匯出至 Redis 相容 RDB 檔案。 您可以使用這項功能，將資料從一個 Azure Redis 快取執行個體移到另一個或其他 Redis 伺服器。 在匯出程序期間，會在裝載 Azure Redis 快取伺服器執行個體的 VM 上建立站存檔案，並將檔案上傳至指定的儲存體帳戶。 當匯出作業完成時的狀態為成功或失敗時，都會刪除暫存檔案。

> [!IMPORTANT]
> 匯入/匯出僅供進階層快取使用。 如需詳細資訊和指示，請參閱 [在 Azure Redis 快取中匯入與匯出資料](cache-how-to-import-export-data.md)。
> 
> 

### <a name="reboot"></a>重新啟動
[重新啟動] 刀鋒視窗可讓您重新啟動快取的節點。 這個重新啟動的能力可讓您測試應用程式在快取節點失敗時的恢復功能。

![重新啟動](./media/cache-configure/redis-cache-reboot.png)

如果您的進階快取已啟用叢集，您可以選取要重新啟動的快取分區。

![重新啟動](./media/cache-configure/redis-cache-reboot-cluster.png)

若要重新啟動快取的一或多個節點，選取所需的節點，然後按一下 [重新啟動] 。 如果您的進階快取已啟用叢集，選取要重新啟動的分區，然後按一下 [重新啟動] 。 稍候幾分鐘之後，選取的節點會重新啟動，並在幾分鐘之後重新上線。

> [!IMPORTANT]
> 重新啟動現在適用於所有定價層。 如需詳細資訊和指示，請參閱 [Azure Redis Cache administration - Reboot (Azure Redis 快取管理 - 重新啟動)](cache-administration.md#reboot)。
> 
> 


## <a name="monitoring"></a>監視

[監視] 區段可讓您設定診斷並監視您的 Redis 快取。 如需有關 Azure Redis 快取監視和診斷的詳細資訊，請參閱[如何監視 Azure Redis 快取](cache-how-to-monitor.md)。

![診斷](./media/cache-configure/redis-cache-diagnostics.png)

* [Redis 度量](#redis-metrics)
* [警示規則](#alert-rules)
* [診斷](#diagnostics)

### <a name="redis-metrics"></a>Redis 度量
按一下 [Redis 度量] 以針對您的快取[檢視度量](cache-how-to-monitor.md#view-cache-metrics)。

### <a name="alert-rules"></a>警示規則

按一下 [警示規則] 以根據 Redis 快取度量設定警示。 如需詳細資訊，請參閱[警示](cache-how-to-monitor.md#alerts)。

### <a name="diagnostics"></a>診斷

根據預設，Azure 監視器中的快取計量會[儲存 30 天](../monitoring/monitoring-data-collection.md#metrics)，而後刪除。 若要保存您的快取計量超過 30 天，按一下 [診斷] 以[設定用來儲存快取診斷的儲存體帳戶](cache-how-to-monitor.md#export-cache-metrics)。

>[!NOTE]
>除了將快取計量封存至儲存體，您也可以[將它們串流處理至事件中樞或將它們傳送至 Log Analytics](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md)。
>
>

## <a name="support--troubleshooting-settings"></a>支援和疑難排解設定
**支援 + 疑難排解** 區段中的設定提供選項，讓您解決快取的問題。

![支援 + 疑難排解](./media/cache-configure/redis-cache-support-troubleshooting.png)

* [資源健康情況](#resource-health)
* [新的支援要求](#new-support-request)

### <a name="resource-health"></a>資源健康情況
**資源健康狀態** 會監看您的資源，並告知您資源是否正如預期般執行。 如需 Azure 資源健康狀態服務的詳細資訊，請參閱 [Azure 資源健康狀態概觀](../resource-health/resource-health-overview.md)。

> [!NOTE]
> 資源健全狀況目前無法回報裝載於虛擬網路上的 Azure Redis 快取執行個體健全狀況。 如需詳細資訊，請參閱 [將快取裝載於 VNET 時，所有快取功能都可以正常運作嗎？](cache-how-to-premium-vnet.md#do-all-cache-features-work-when-hosting-a-cache-in-a-vnet)
> 
> 

### <a name="new-support-request"></a>新增支援要求
按一下 [新增支援要求]  以開啟快取的支援要求。





## <a name="default-redis-server-configuration"></a>預設 Redis 伺服器組態
新的 Azure Redis 快取執行個體會以下列預設 Redis 設定值來設定：

> [!NOTE]
> 您無法使用 `StackExchange.Redis.IServer.ConfigSet` 方法變更本區段中的設定。 如果使用本區段中的其中一個命令來呼叫此方法，則會擲回類似下列範例的例外狀況：  
> 
> `StackExchange.Redis.RedisServerException: ERR unknown command 'CONFIG'`
> 
> 任何可設定的值 (例如 **max-memory-policy**) 都可以透過 Azure 入口網站或命令列管理工具 (例如 Azure CLI 或 PowerShell) 加以設定。
> 
> 

| 設定 | 預設值 | 說明 |
| --- | --- | --- |
| `databases` |16 |資料庫的預設數目為 16，但是您可以根據定價層設定不同的數字。<sup>1</sup> 預設資料庫為 DB 0，您可以根據每個連線使用 `connection.GetDatabase(dbid)` 選取一個不同的資料庫，其中 `dbid` 是介於 `0` 與 `databases - 1` 之間的數字。 |
| `maxclients` |取決於定價層<sup>2</sup> |此值是允許同時連線的用戶端數目上限。 一旦達到限制，Redis 會關閉所有新的連線，並傳送「達到用戶端的數目上限」錯誤。 |
| `maxmemory-policy` |`volatile-lru` |maxmemory 原則可設定當達到 `maxmemory` (建立快取時所選取之快取供應項目的大小) 時 Redis 將如何選取要移除的具目。 Azure Redis 快取的預設設定為 `volatile-lru`，其會移除使用 LRU 演算法設定到期日的金鑰。 此設定可以在 Azure 入口網站中設定。 如需詳細資訊，請參閱[記憶體原則](#memory-policies)。 |
| `maxmemory-samples` |3 |為了節省記憶體，LRU 和最小 TTL 演算法是近似的演算法而不是精確的演算法。 依預設 Redis 將檢查三個金鑰，並挑選最近較少使用的金鑰。 |
| `lua-time-limit` |5,000 |Lua 指令碼的最大執行時間 (以毫秒為單位)。 如果已到達最大執行時間，Redis 會記錄指令碼在最大允許的時間之後仍在執行中，並開始回覆查詢發生錯誤。 |
| `lua-event-limit` |500 |指令碼事件佇列的大小上限。 |
| `client-output-buffer-limit` `normalclient-output-buffer-limit` `pubsub` |0 0 032mb 8mb 60 |用戶端輸出緩衝區限制可用來強制中斷基於某些原因而無法足夠快地從伺服器讀取資料之用戶端的連線 (常見的原因是 Pub/Sub 用戶端使用訊息的速度無法與發佈者產生這些訊息的速度一樣快)。 如需詳細資訊，請參閱 [http://redis.io/topics/clients](http://redis.io/topics/clients)。 |

<a name="databases"></a>
<sup>1</sup>適用於每個 Azure Redis 快取定價層的 `databases` 限制皆不相同，可在快取建立時加以設定。 如果快取建立期間未指定 `databases` 設定，則預設值為 16。

* 基本和標準的快取
  * C0 (250 MB) 快取 - 最多 16 個資料庫
  * C1 (1 GB) 快取 - 最多 16 個資料庫
  * C2 (2.5 GB) 快取 - 最多 16 個資料庫
  * C3 (6 GB) 快取 - 最多 16 個資料庫
  * C4 (13 GB) 快取 - 最多 32 個資料庫
  * C5 (26 GB) 快取 - 最多 48 個資料庫
  * C6 (53 GB) 快取 - 最多 64 個資料庫
* 進階快取
  * P1 (6 GB - 60 GB) - 最多 16 個資料庫
  * P2 (13 GB - 130 GB) - 最多 32 個資料庫
  * P3 (26 GB - 260 GB) - 最多 48 個資料庫
  * P4 (53 GB - 530 GB) - 最多 64 個資料庫
  * 所有進階快取均已啟用 Redis 叢集 - Redis 叢集僅支援使用資料庫 0，因此對於已啟用 Redis 叢集的任何進階快取， `databases` 限制實際上是 1，並且不允許 [Select](http://redis.io/commands/select) 命令。 如需詳細資訊，請參閱 [我需要對用戶端應用程式進行任何變更才能使用叢集嗎？](cache-how-to-premium-clustering.md#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)

如需資料庫的詳細資訊，請參閱[什麼是 Redis 資料庫？](cache-faq.md#what-are-redis-databases)

> [!NOTE]
> `databases` 設定，而且只能使用 PowerShell、CLI 或其他管理用戶端。 如需在快取建立期間使用 PowerShell 設定 `databases` 的範例，請參閱 [New-AzureRmRedisCache](cache-howto-manage-redis-cache-powershell.md#databases)。
> 
> 

<a name="maxclients"></a>
<sup>2</sup>適用於每個 Azure Redis 快取定價層的 `maxclients` 皆不相同。

* 基本和標準的快取
  * C0 (250 MB) 快取 - 最多 256 個連接
  * C1 (1 GB) 快取 - 最多 1,000 個連接
  * C2 (2.5GB) 快取 - 最多 2,000 個連接
  * C3 (6 GB) 快取 - 最多 5,000 個連接
  * C4 (13 GB) 快取 - 最多 10,000 個連接
  * C5 (26 GB) 快取 - 最多 15,000 個連接
  * C6 (53 GB) 快取 - 最多 20,000 個連接
* 進階快取
  * P1 (6 GB - 60 GB) - 最多 7,500 個連接
  * P2 (13 GB - 130 GB) - 最多 15,000 個連接
  * P3 (26 GB - 260 GB) - 最多 30,000 個連接
  * P4 (53 GB - 530 GB) - 最多 40,000 個連接

> [!NOTE]
> 雖然每種大小的快取皆允許連接數可*多達*某個數目，但對 Redis 的每個連接都有相關聯的負荷。 因 TLS/SSL 加密而產生的 CPU 與記憶體使用量即是這類額外負荷的其中一例。 所指定快取大小的連線數上限是假設快取負載情況為輕度。 如果連接負荷的負擔*加上*用戶端作業的負擔，超過系統的容量，則快取會碰到容量問題，即使您尚未超過目前快取大小的連接限制。
> 
> 



## <a name="redis-commands-not-supported-in-azure-redis-cache"></a>Azure Redis 快取中不支援的 Redis 命令
> [!IMPORTANT]
> 因為 Azure Redis 快取執行個體的設定與管理是由 Microsoft 管理，所以會停用下列命令。 如果您嘗試叫用它們，會收到類似 `"(error) ERR unknown command"` 的錯誤訊息。
> 
> * BGREWRITEAOF
> * BGSAVE
> * CONFIG
> * DEBUG
> * MIGRATE
> * SAVE
> * SHUTDOWN
> * SLAVEOF
> * CLUSTER - 叢集寫入命令已停用，但允許唯讀的叢集命令。
> 
> 

如需 Redis 命令的詳細資訊，請參閱 [http://redis.io/commands](http://redis.io/commands)。

## <a name="redis-console"></a>Redis 主控台
您可以使用 [Redis 主控台] (適用於 Azure 入口網站中的所有快取層) 安全地發出命令給 Azure Redis 快取執行個體。

> [!IMPORTANT]
> - Redis 主控台不使用 [VNET](cache-how-to-premium-vnet.md)。 如果您的快取是 VNET 的一部分，只有在 VNET 中的用戶端可以存取快取。 由於 Redis 主控台在您的本機瀏覽器 (位於 VNET 之外) 中執行，因此無法連接到您的快取。
> - Azure Redis 快取並未支援所有的 Redis 命令。 如需對 Azure Redis 快取所停用之 Redis 命令的清單，請參閱先前的 [Azure Redis 快取中不支援的 Redis 命令](#redis-commands-not-supported-in-azure-redis-cache)一節。 如需 Redis 命令的詳細資訊，請參閱 [http://redis.io/commands](http://redis.io/commands)。
> 
> 

若要存取 Redis 主控台，請從 [Redis 快取] 刀鋒視窗按一下 [主控台]。

![Redis 主控台](./media/cache-configure/redis-console-menu.png)

只需在主控台中輸入想要的命令，即可對您的快取執行個體發出命令。

![Redis 主控台](./media/cache-configure/redis-console.png)


### <a name="using-the-redis-console-with-a-premium-clustered-cache"></a>使用 Redis 主控台搭配進階叢集快取

使用 Redis 主控台搭配進階叢集快取時，您可以向單一快取分區發出命令。 若要向特定的分區發出命令，請先按一下分區選擇器上所需的分區進行連線。

![Redis 主控台](./media/cache-configure/redis-console-premium-cluster.png)

如果您嘗試存取儲存在與已連線的分區不同分區中的金鑰，您會收到類似下列訊息的錯誤訊息：

```
shard1>get myKey
(error) MOVED 866 13.90.202.154:13000 (shard 0)
```

在上一個範例中，分區 1 是所選的分區，但 `myKey` 位於分區 0，如錯誤訊息的 `(shard 0)` 部分所指出。 在此範例中，若要存取 `myKey`，使用分區選擇器選取分區 0，然後發出所需的命令。


## <a name="move-your-cache-to-a-new-subscription"></a>將您的快取移動到新的訂用帳戶
您也可以按一下 [移動] ，將快取移到新的訂用帳戶。

![移動 Redis 快取](./media/cache-configure/redis-cache-move.png)

如需將資源從某個資源群組移到另一個資源群組，以及從某個訂用帳戶移到另一個訂用帳戶的相關資訊，請參閱 [將資源移動到新的資源群組或訂用帳戶](../azure-resource-manager/resource-group-move-resources.md)。

## <a name="next-steps"></a>後續步驟
* 如需使用 Redis 命令的詳細資訊，請參閱[如何執行 Redis 命令？](cache-faq.md#how-can-i-run-redis-commands)

