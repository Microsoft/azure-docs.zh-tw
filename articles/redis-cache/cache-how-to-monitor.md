---
title: 如何監視 Azure Redis 快取 | Microsoft Docs
description: 了解如何監視您 Azure Redis 快取執行個體的健全狀況和效能
services: redis-cache
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.assetid: 7e70b153-9c87-4290-85af-2228f31df118
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.author: wesmc
ms.openlocfilehash: 03c6d45cb3a20244ddbb9c0aec693f7802a95b22
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46976353"
---
# <a name="how-to-monitor-azure-redis-cache"></a>如何監視 Azure Redis 快取
Azure Redis Cache 使用 [Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/)提供數個選項來監視您的快取執行個體。 您可以檢視度量、將度量圖表釘選到「開始面板」、自訂監視圖表的日期和時間範圍、新增和移除圖表中的度量，以及設定符合特定條件時的警示。 這些工具可讓您監視 Azure Redis 快取執行個體的健全狀況，並協助您管理快取應用程式。

Azure Redis Cache 執行個體的計量使用 Redis [INFO](http://redis.io/commands/info) 命令每分鐘收集兩次，自動儲存 30 天 (請參閱[匯出快取計量](#export-cache-metrics)來設定不同的保留原則)，以便其顯示在計量圖表中並依警示規則評估。 如需用於每個快取度量的不同 INFO 值詳細資訊，請參閱 [可用度量和報告間隔](#available-metrics-and-reporting-intervals)。

<a name="view-cache-metrics"></a>

若要檢視快取度量，請[瀏覽](cache-configure.md#configure-redis-cache-settings)至您在 [Azure 入口網站](https://portal.azure.com)中的快取執行個體。  Azure Redis Cache 會在 [概觀] 刀鋒視窗和 [Redis 計量] 刀鋒視窗中提供一些內建圖表。 新增或移除度量，以及變更報告間隔，即可自訂每個圖表。

![Redis 度量](./media/cache-how-to-monitor/redis-cache-redis-metrics-blade.png)

## <a name="view-pre-configured-metrics-charts"></a>檢視預先設定的計量圖表

[概觀] 刀鋒視窗具有下列預先設定的監視圖表。

* [監視圖表](#monitoring-charts)
* [使用量圖表](#usage-charts)

### <a name="monitoring-charts"></a>監視圖表
[概觀] 刀鋒視窗的 [監視] 區段具有 [點擊和遺漏]、[取得和設定]、[連線] 和 [總命令數] 圖表。

![監視圖表](./media/cache-how-to-monitor/redis-cache-monitoring-part.png)

### <a name="usage-charts"></a>使用量圖表
[概觀] 刀鋒視窗的 [使用量] 區段有 [Redis 伺服器負載]、[記憶體使用量]、[網路頻寬] 和 [CPU 使用量] 圖表，也會顯示快取執行個的 [定價層]。

![使用量圖表](./media/cache-how-to-monitor/redis-cache-usage-part.png)

[定價層]  會顯示快取定價層，而且可以用來將快取 [調整](cache-how-to-scale.md) 為不同的定價層。

## <a name="view-metrics-with-azure-monitor"></a>使用 Azure 監視器檢視計量
若要檢視 Redis 計量，並使用 Azure 監視器建立自訂圖表，請按一下 [資源] 功能表的 [計量]，使用所需的計量、報告間隔、圖表類型等來自訂圖表。

![Redis 度量](./media/cache-how-to-monitor/redis-cache-monitor.png)

如需使用 Azure 監視器處理計量的詳細資訊，請參閱 [Microsoft Azure 的計量概觀](../monitoring-and-diagnostics/monitoring-overview-metrics.md)。

<a name="how-to-view-metrics-and-customize-chart"></a>
<a name="enable-cache-diagnostics"></a>
## <a name="export-cache-metrics"></a>匯出快取計量
根據預設，Azure 監視器中的快取計量會[儲存 30 天](../monitoring/monitoring-data-collection.md#metrics)後刪除。 若要保留您的快取計量超過 30 天，您可以[指定儲存體帳戶](../monitoring-and-diagnostics/monitoring-archive-diagnostic-logs.md)，並指定快取計量的 [保留天數] 原則。 

設定快取計量的儲存體帳戶：

1. 按一下 [Redis Cache] 刀鋒視窗之 [資源] 功能表中的 [診斷]。
2. 按一下 [開啟]。
3. 勾選 [封存至儲存體帳戶]。
4. 選取要儲存快取計量的儲存體帳戶。
5. 勾選 [1 分鐘] 核取方塊，並指定 [保留天數] 原則。 如果您不想要套用任何保留期原則，也不想永久保留資料，請將 [保留天數] 設為 **0**。
6. 按一下 [檔案] 。

![Redis 診斷](./media/cache-how-to-monitor/redis-cache-diagnostics.png)

>[!NOTE]
>除了將快取計量封存至儲存體，您也可以[將它們串流處理至事件中樞或將它們傳送至 Log Analytics](../monitoring-and-diagnostics/monitoring-rest-api-walkthrough.md#retrieve-metric-values)。
>
>

若要存取計量，您可以如本文先前所述，在 Azure 入口網站中進行檢視，也可以使用 [Azure 監視器計量 REST API](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md) 存取計量。

> [!NOTE]
> 如果您變更儲存體帳戶，則先前設定之儲存體帳戶中的資料仍然可供下載，但不會顯示在 Azure 入口網站中。  
> 
> 

## <a name="available-metrics-and-reporting-intervals"></a>可用度量和報告間隔
快取度量是使用數個報告間隔所報告，包括 [過去一小時]、[今天]、[過去一週] 和 [自訂]。 每個度量圖的 [度量]  分頁都會顯示圖表中每個度量的平均值、最小值和最大值，而有些度量會顯示報告間隔的總計。 

每個度量都包含兩個版本。 有一個度量會測量整個快取的效能，以及使用[叢集](cache-how-to-premium-clustering.md)之快取的效能，名稱中包含 `(Shard 0-9)` 的第二個度量版本則會測量快取中單一分區的效能。 例如，如果快取有 4 個分區，`Cache Hits` 就是整個快取的點擊總數，而 `Cache Hits (Shard 3)` 就只是該快取分區的點擊數。

> [!NOTE]
> 即使快取閒置而沒有連線的作用中用戶端應用程式，您可能還是會看到某個快取活動 (例如連線的用戶端、記憶體使用量，以及正在執行的作業)。 在 Azure Redis 快取執行個體的作業期間，此活動正常。
> 
> 

| 計量 | 說明 |
| --- | --- |
| 快取點擊 |所指定報告間隔期間的成功金鑰查閱數目。 這會對應至 Redis [INFO](http://redis.io/commands/info) 命令的 `keyspace_hits` 。 |
| 快取延遲 (預覽) | 根據快取的內部節點延遲計算的快取延遲。 此計量會以微秒測量，並有三個維度：「平均」、「最小」和「最大」，分別代表在指定的報告間隔期間內快取的平均、最小和最大延遲。 |
| 快取遺漏 |所指定報告間隔期間的失敗金鑰查閱數目。 這會對應至 Redis INFO 命令的 `keyspace_misses` 。 快取遺漏不一定表示快取發生問題。 例如，使用另行快取程式設計模式時，應用程式會先在快取中尋找項目。 如果項目不存在 (快取遺漏)，項目會從資料庫中擷取，並在下次新增至快取中。 快取遺漏是另行快取程式設計模式的正常行為。 如果快取遺漏數目高於預期，請檢查可填入且讀取自快取的應用程式邏輯。 如果因記憶體壓力而正在收回快取中的項目，則可能會有一些快取遺漏，但監視記憶體壓力的較佳度量是 `Used Memory` 或 `Evicted Keys`。 |
| 快取讀取 |所指定報告間隔期間，從快取讀取的資料量 (以 MB/s 為單位)。 這個值衍生自網路介面卡，而網路介面卡支援裝載快取且非 Redis 特有的虛擬機器。 **這個值對應於此快取所使用的網路頻寬。如果您想要設定伺服器端網路頻寬限制的警示，則可使用此 `Cache Read` 計數器加以建立。如需所觀察到適用於各種快取定價層和大小的頻寬限制，請參閱[這個表格](cache-faq.md#cache-performance)。** |
| 快取寫入 |所指定報告間隔期間，寫入至快取的資料量 (以 MB/s 為單位)。 這個值衍生自網路介面卡，而網路介面卡支援裝載快取且非 Redis 特有的虛擬機器。 此值對應從用戶端傳送給快取之資料的網路頻寬。 |
| 連線的用戶端 |所指定報告間隔期間的快取用戶端連線數目。 這會對應至 Redis INFO 命令的 `connected_clients` 。 一旦達到 [連接限制](cache-configure.md#default-redis-server-configuration) 之後，後續對快取所做的連接嘗試都將失敗。 請注意，因為內部處理序和連線，所以即使沒有作用中用戶端應用程式，可能還是會有一些連線的用戶端執行個體。 |
| CPU |所指定報告間隔期間的 Azure Redis 快取伺服器 CPU 使用率 (百分比)。 這個值會對應至作業系統 `\Processor(_Total)\% Processor Time` 效能計數器。 |
| Errors | 在指定的報告間隔期間，快取可能遇到的特定失敗和效能問題。 此計量有八個代表不同錯誤類型的維度，但未來可能新增更多。 現在代表的錯誤類型如下所示： <br/><ul><li>**容錯移轉** – 當快取容錯移轉時 (從屬會提升至主要)</li><li>**損毀** – 當快取在任何一個節點意外損毀時</li><li>**資料遺失** – 當快取有資料遺失時</li><li>**UnresponsiveClients** – 當用戶端無法從速度夠快的伺服器讀取資料時</li><li>**AOF** – 有 AOF 持續性相關的問題時</li><li>**RDB** – 有 RDB 持續性相關的問題時</li><li>**匯入**– 有匯入 RDB 相關的問題時</li><li>**匯出**– 有匯出 RDB 相關的問題時</li></ul> |
| 收回的金鑰 |因 `maxmemory` 限制，在所指定報告間隔期間從快取收回的項目數。 這會對應至 Redis INFO 命令的 `evicted_keys` 。 |
| 到期的金鑰 |所指定報告間隔期間的快取到期項目數。 這個值會對應至 Redis INFO 命令的 `expired_keys` 。|
| 取得 |所指定報告間隔期間的快取 get 作業數目。 這個值是 Redis INFO all 命令的下列值總和：`cmdstat_get`、`cmdstat_hget``cmdstat_hgetall`、`cmdstat_hmget``cmdstat_mget`、`cmdstat_getbit` 和 `cmdstat_getrange`，而且等於報告期間的快取點擊和遺漏。 |
| 每秒的作業數 | 在指定的報告間隔期間，快取伺服器每秒所處理的命令總數。  這個值會對應至 Redis INFO 命令中的 "instantaneous_ops_per_sec"。 |
| Redis 伺服器負載 |Redis 伺服器忙著處理訊息且非訊息等候閒置之循環的百分比。 如果這個計數器達到 100，表示 Redis 伺服器已經達到效能上限，而且 CPU 處理工作的速度不能再更快。 如果您看到高「Redis 伺服器負載 (Redis Server Load)」，則會看到用戶端中的逾時例外狀況。 在此情況下，您應該考慮向上延展，或將資料分割成多個快取。 |
| 設定 |所指定報告間隔期間的快取 set 作業數目。 這個值是 Redis INFO all 命令的下列值總和：`cmdstat_set`、`cmdstat_hset``cmdstat_hmset`、`cmdstat_hsetnx`、`cmdstat_lset`、`cmdstat_mset`、`cmdstat_msetnx`、`cmdstat_setbit`、`cmdstat_setex`、`cmdstat_setrange` 和 `cmdstat_setnx` |
| 索引鍵總計  | 快取中的索引鍵在過去報告時段內的數目上限。 這會對應至 Redis INFO 命令的 `keyspace` 。 因為基礎計量系統的限制，凡是啟用叢集化的快取，[索引鍵總計] 都會傳回分區的索引鍵數目上限，該分區於報告間隔期間曾有過索引鍵的最大數目。  |
| 總作業數 |所指定報告間隔期間，快取伺服器所處理命令的總數。 這個值會對應至 Redis INFO 命令的 `total_commands_processed` 。 請注意，Azure Redis 快取純粹用於發佈/訂閱時，則沒有 `Cache Hits`、`Cache Misses`、`Gets` 或 `Sets` 的度量但是會有 `Total Operations` 度量可反映發佈/訂閱作業的快取使用量。 |
| 已使用的記憶體 |在指定的報告間隔期間，針對快取中金鑰/值組使用的快取記憶體數量 (MB)。 這個值會對應至 Redis INFO 命令的 `used_memory` 。 這不包括中繼資料或片段。 |
| 已用的記憶體百分比 | 在指定的報告間隔期間，正在使用的總記憶體百分比。  這會參照 Redis INFO 命令中的 "used_memory" 值來計算百分比。 |
| 已用的記憶體 RSS |在指定的報告間隔期間使用的快取記憶體數量 (MB)，包括片段和中繼資料。 這個值會對應至 Redis INFO 命令的 `used_memory_rss` 。 |

<a name="operations-and-alerts"></a>
## <a name="alerts"></a>警示
您可以進行設定來收到以計量和活動記錄為基礎的警示。 Azure 監視器可讓您將警示設定為在觸發時執行下列動作︰

* 傳送電子郵件通知
* 呼叫 Webhook
* 叫用 Azure 邏輯應用程式

若要設定快取警示規則，請按一下 [資源] 功能表的 [警示規則]。

![監視](./media/cache-how-to-monitor/redis-cache-monitoring.png)

如需設定和使用警示的詳細資訊，請參閱[警示概觀](../monitoring-and-diagnostics/insights-alerts-portal.md)。

## <a name="activity-logs"></a>活動記錄檔
活動記錄可讓您深入了解 Azure Redis Cache 執行個體上執行的作業。 此記錄以前稱為「稽核記錄」或「作業記錄」。 您可以使用活動記錄來判斷 Azure Redis Cache 執行個體上所執行之任何寫入作業 (PUT、POST、DELETE) 的「內容、對象和時間」。 

> [!NOTE]
> 活動記錄不包含讀取 (GET) 作業。
>
>

若要檢視快取的活動記錄，請按一下 [資源] 功能表的 [活動記錄]。

如需活動記錄的詳細資訊，請參閱 [Azure 活動記錄檔概觀](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)。











