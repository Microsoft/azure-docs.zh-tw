---
title: 如何針對 Azure Redis 快取進行疑難排解 | Microsoft Docs
description: 了解如何解決 Azure Redis 快取的常見問題。
services: redis-cache
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.assetid: 928b9b9c-d64f-4252-884f-af7ba8309af6
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: article
ms.date: 01/06/2017
ms.author: wesmc
ms.openlocfilehash: b41fc5c41b2e0d1e5d5ba3e39c7f6063cf57c6c2
ms.sourcegitcommit: 30221e77dd199ffe0f2e86f6e762df5a32cdbe5f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39205777"
---
# <a name="how-to-troubleshoot-azure-redis-cache"></a>如何針對 Azure Redis 快取進行疑難排解
本文提供以下類別的 Azure Redis 快取問題疑難排解指引。

* [用戶端疑難排解](#client-side-troubleshooting) - 本節提供連接到 Azure Redis 快取的應用程式所造成之問題的識別和解決指引。
* [伺服器端疑難排解](#server-side-troubleshooting) - 本節提供 Azure Redis 快取伺服器端上所造成之問題的識別和解決指引。
* [StackExchange.Redis 逾時例外狀況](#stackexchangeredis-timeout-exceptions) - 本節提供使用 StackExchange.Redis 用戶端時之問題的疑難排解資訊。

> [!NOTE]
> 本指南中的數個疑難排解步驟包含如何執行 Redis 命令和監視各種效能度量的指示。 如需詳細資訊和指示，請參閱 [其他資訊](#additional-information) 一節中的文章。
> 
> 

## <a name="client-side-troubleshooting"></a>用戶端疑難排解
本節討論因為用戶端應用程式的某種情況而發生之問題的疑難排解。

* [用戶端上的記憶體壓力](#memory-pressure-on-the-client)
* [流量暴增](#burst-of-traffic)
* [用戶端 CPU 使用量很高](#high-client-cpu-usage)
* [超過用戶端頻寬](#client-side-bandwidth-exceeded)
* [要求/回應大小很大](#large-requestresponse-size)
* [我在 Redis 中的資料怎麼了？](#what-happened-to-my-data-in-redis)

### <a name="memory-pressure-on-the-client"></a>用戶端上的記憶體壓力
#### <a name="problem"></a>問題
用戶端電腦上的記憶體壓力會導致各種效能問題，而可能延遲處理 Redis 執行個體所傳送的資料，但不會有任何延遲。 到達記憶體壓力時，系統通常必須將資料從實體記憶體分頁處理到磁碟上的虛擬記憶體。 此「分頁錯誤」  會使系統明顯變慢。

#### <a name="measurement"></a>測量
1. 監視電腦上的記憶體使用量，以確定它不會超過可用記憶體。 
2. 監視 `Page Faults/Sec` 效能計數器。 即使在正常作業期間，大部分的系統還是會有一些分頁錯誤，因此請留意這個與逾時對應的分頁錯誤效能計數器尖峰。

#### <a name="resolution"></a>解決方案
將用戶端升級至擁有更多記憶體的較大用戶端 VM 大小，或深入了解記憶體使用量模式以減少記憶體的耗用。

### <a name="burst-of-traffic"></a>流量暴增
#### <a name="problem"></a>問題
流量暴增加上不當的 `ThreadPool` 設定，可能會導致延遲處理 Redis 伺服器已傳送但用戶端上尚未使用的資料。

#### <a name="measurement"></a>測量
使用[如這裡所提供的](https://github.com/JonCole/SampleCode/blob/master/ThreadPoolMonitor/ThreadPoolLogger.cs)程式碼，監視 `ThreadPool` 統計資料如何隨著時間變更。 您也可以查看來自 StackExchange.Redis 的 `TimeoutException` 訊息。 下列是一個範例：

    System.TimeoutException: Timeout performing EVAL, inst: 8, mgr: Inactive, queue: 0, qu: 0, qs: 0, qc: 0, wr: 0, wq: 0, in: 64221, ar: 0, 
    IOCP: (Busy=6,Free=999,Min=2,Max=1000), WORKER: (Busy=7,Free=8184,Min=2,Max=8191)

在上述訊息中，有幾個有趣的問題︰

1. 請注意，在 `IOCP` 區段和 `WORKER` 區段中，`Busy` 值大於 `Min` 值。 有此差異表示 `ThreadPool` 設定需要調整。
2. 您也可以查看 `in: 64221`。 這個值表示已在核心通訊端層接收到 64,211 位元組，但應用程式 (例如 StackExchange.Redis) 尚未讀取。 有此差異通常表示應用程式從網路讀取資料的速度，比伺服器傳送資料給您的速度還慢。

#### <a name="resolution"></a>解決方案
設定 [ThreadPool 設定](https://gist.github.com/JonCole/e65411214030f0d823cb)以確保執行緒集區會在流量暴增的情況下相應增加。

### <a name="high-client-cpu-usage"></a>用戶端 CPU 使用量很高
#### <a name="problem"></a>問題
用戶端上的 CPU 使用量很高表示系統跟不上其應該執行的工作。 這種情況表示用戶端可能無法及時處理 Redis 傳來的回應，即使 Redis 快速地傳送回應也是一樣。

#### <a name="measurement"></a>測量
透過 Azure 入口網站或相關聯的效能計數器，監視全系統的 CPU 使用量。 請注意不要監視「處理程序」  CPU，因為有可能單一處理程序的 CPU 使用率很低，但同時間的整體系統 CPU 卻很高。 請監看與逾時對應的 CPU 使用量暴增。 由於高 CPU，您也可能會在 `TimeoutException` 錯誤訊息中看到高 `in: XXX` 值，如[流量暴增](#burst-of-traffic)一節所述。

> [!NOTE]
> StackExchange.Redis 1.1.603 和更新版本在 `TimeoutException` 錯誤訊息中包含 `local-cpu` 度量。 請確定您使用最新版的 [StackExchange.Redis NuGet 封裝](https://www.nuget.org/packages/StackExchange.Redis/)。 程式碼中的錯誤會經常修正，以便更能應付逾時問題，因此請務必要擁有最新版本。
> 
> 

#### <a name="resolution"></a>解決方案
升級為擁有更多 CPU 容量的較大 VM 大小，或調查導致 CPU 尖峰的原因。 

### <a name="client-side-bandwidth-exceeded"></a>超過用戶端頻寬
#### <a name="problem"></a>問題
用戶端機器可用的網路頻寬多寡可能會有限制 (視其架構而定)。 如果用戶端多載網路容量而超過可用的頻寬，則資料在用戶端上的處理速度不會和伺服器傳送資料的速度一樣快。 這種情況可能會導致逾時。

#### <a name="measurement"></a>測量
使用 [如這裡所提供的](https://github.com/JonCole/SampleCode/blob/master/BandWidthMonitor/BandwidthLogger.cs)程式碼，監視頻寬使用量如何隨著時間變更。 此程式碼可能無法在具有限制權限的某些環境 (例如 Azure 網站) 中順利執行。

#### <a name="resolution"></a>解決方案
增加用戶端 VM 大小或減少網路頻寬耗用。

### <a name="large-requestresponse-size"></a>要求/回應大小很大
#### <a name="problem"></a>問題
大型的要求/回應可能會導致逾時。 例如，假設您在用戶端上設定的逾時值為 1 秒。 應用程式同時 (使用相同的實體網路連線) 要求兩個金鑰 (例如 'A' 和 'B')。 大部分用戶端支援「管線傳送」要求，因此 'A' 和 'B' 這兩個要求沒有等待回應，就在網路上相繼傳送至伺服器。 伺服器會以相同順序傳回回應。 如果回應 'A' 夠大，它可能會用掉後續要求的大部分逾時。 

下列範例示範此案例。 在此案例中，要求 'A' 和 'B' 快速傳送，伺服器也開始快速傳送回應 'A' 和 'B'，但因為資料傳輸需要時間，'B' 被擋在其他要求後面，因此即使伺服器已迅速回應仍然會逾時。

    |-------- 1 Second Timeout (A)----------|
    |-Request A-|
         |-------- 1 Second Timeout (B) ----------|
         |-Request B-|
                |- Read Response A --------|
                                           |- Read Response B-| (**TIMEOUT**)



#### <a name="measurement"></a>測量
此要求/回應並不容易測量。 基本上，您必須檢測用戶端程式碼來追蹤大量要求和回應。 

#### <a name="resolution"></a>解決方案
1. Redis 最適合大量的較小值，而不是少數幾個較大值。 比較好的解決方案是將資料分割成相關的較小值。 請參閱 [Redis 的理想值大小範圍為何？100 KB 是否會太大？](https://groups.google.com/forum/#!searchin/redis-db/size/redis-db/n7aa2A4DZDs/3OeEPHSQBAAJ)文章，以取得為何建議使用較小值的詳細資料。
2. 增加 VM (適用於用戶端和 Redis 快取伺服器) 的大小，以取得更高的頻寬功能，減少較大回應的資料傳輸時間。 只在伺服器或用戶端上取得更多頻寬可能並不夠。 請測量頻寬使用量，並與您目前擁有的 VM 大小的功能做比較。
3. 增加您使用的 `ConnectionMultiplexer` 物件數目，並透過不同的連線循環配置要求。

### <a name="what-happened-to-my-data-in-redis"></a>我在 Redis 中的資料怎麼了？
#### <a name="problem"></a>問題
我預期 Azure Redis 快取執行個體中會有某些資料，但是裡面似乎沒有。

#### <a name="resolution"></a>解決方案
如需可能的原因和解決方案，請參閱 [我在 Redis 中的資料怎麼了？](https://gist.github.com/JonCole/b6354d92a2d51c141490f10142884ea4#file-whathappenedtomydatainredis-md) 。

## <a name="server-side-troubleshooting"></a>伺服器端疑難排解
本節討論因為快取伺服器的某種情況而發生之問題的疑難排解。

* [伺服器上的記憶體壓力](#memory-pressure-on-the-server)
* [高 CPU 使用率/伺服器負載](#high-cpu-usage-server-load)
* [超過伺服器端頻寬](#server-side-bandwidth-exceeded)

### <a name="memory-pressure-on-the-server"></a>伺服器上的記憶體壓力
#### <a name="problem"></a>問題
伺服器端上的記憶體壓力會導致各種效能問題，而可能延遲處理要求。 到達記憶體壓力時，系統通常必須將資料從實體記憶體分頁處理到磁碟上的虛擬記憶體。 此「分頁錯誤」  會使系統明顯變慢。 此記憶體壓力有幾個可能的原因︰ 

1. 您已在快取中填滿資料。 
2. Redis 遇到高度記憶體分散情形，儲存大型物件時最常造成此現象 (Redis 最適合小型物件，如需詳細資訊，請參閱 [Redis 的理想值大小範圍為何？100 KB 是否會太大？](https://groups.google.com/forum/#!searchin/redis-db/size/redis-db/n7aa2A4DZDs/3OeEPHSQBAAJ)文章)。 

#### <a name="measurement"></a>測量
Redis 會公開兩個可協助您識別此問題的度量。 第一個是 `used_memory`，另一個是 `used_memory_rss`。 [這些度量](cache-how-to-monitor.md#available-metrics-and-reporting-intervals)可在 Azure 入口網站中或透過 [Redis INFO](http://redis.io/commands/info) 命令來取得。

#### <a name="resolution"></a>解決方案
您可以進行幾項可能的變更，以協助讓記憶體使用量保持良好狀況︰

1. [設定記憶體原則](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) 並在金鑰上設定到期時間。 如果記憶體分散，此設定可能不足以應付。
2. [設定 maxmemory-reserved 值](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) ，讓此值大到足以彌補記憶體分散情形。
3. 將大型快取物件分割為較小的相關物件。
4. [調整](cache-how-to-scale.md) 為更大的快取大小。
5. 如果您使用[已啟用 Redis 叢集的高階快取](cache-how-to-premium-clustering.md)，則可以[增加分區數目](cache-how-to-premium-clustering.md#change-the-cluster-size-on-a-running-premium-cache)。

### <a name="high-cpu-usage--server-load"></a>高 CPU 使用率/伺服器負載
#### <a name="problem"></a>問題
高 CPU 使用率可能表示用戶端可能無法及時處理 Redis 傳來的回應，即使 Redis 快速地傳送回應也是一樣。

#### <a name="measurement"></a>測量
透過 Azure 入口網站或相關聯的效能計數器，監視全系統的 CPU 使用量。 請注意不要監視「處理程序」  CPU，因為有可能單一處理程序的 CPU 使用率很低，但同時間的整體系統 CPU 卻很高。 請監看與逾時對應的 CPU 使用量暴增。

#### <a name="resolution"></a>解決方案
* 檢閱 [Redis Cache Advisor](cache-configure.md#redis-cache-advisor) 中所提及的任何建議和警示。
* 另請檢閱本主題中的其他建議以及 [Azure Redis 的最佳做法](https://gist.github.com/JonCole/925630df72be1351b21440625ff2671f)，了解您是否已運用所有選項來將快取和用戶端進一步最佳化。 
* 檢閱 [Azure Redis 快取效能](cache-faq.md#azure-redis-cache-performance)圖表，了解您是否可能接近目前層級的上限閾值。 如有必要，[調整](cache-how-to-scale.md)為具有更多 CPU 容量的較大快取層。 如果您已使用進階層，建議您[利用叢集來相應放大](cache-how-to-premium-clustering.md)


### <a name="server-side-bandwidth-exceeded"></a>超過伺服器端頻寬
#### <a name="problem"></a>問題
快取執行個體可用的網路頻寬多寡可能會有限制 (視其大小而定)。 如果伺服器超過可用的頻寬，則資料不會一樣快地傳送至用戶端。 這種情況可能會導致逾時。

#### <a name="measurement"></a>測量
您可以監視 `Cache Read` 度量，這是所指定報告間隔期間，從快取讀取的資料量 (以 MB/s 為單位)。 這個值對應於此快取所使用的網路頻寬。 如果您想要設定伺服器端網路頻寬限制的警示，則可以使用此 `Cache Read` 計數器加以建立。 如需不同快取定價層和大小的觀察頻寬限制，請比較您的數據與 [本表](cache-faq.md#cache-performance) 中的值。

#### <a name="resolution"></a>解決方案
如果您一直逼近定價層和快取大小的最大觀察頻寬，請考慮使用[本表](cache-faq.md#cache-performance)中的值做為指南，來[調整](cache-how-to-scale.md)為具有更大網路頻寬的定價層或大小。

## <a name="stackexchangeredis-timeout-exceptions"></a>StackExchange.Redis 逾時例外狀況
StackExchange.Redis 使用名為 `synctimeout` 的組態設定來進行預設值為 1000 毫秒的同步作業。 如果同步呼叫未在約定的時間內完成，StackExchange.Redis 用戶端會擲回類似下列範例的逾時錯誤：

    System.TimeoutException: Timeout performing MGET 2728cc84-58ae-406b-8ec8-3f962419f641, inst: 1,mgr: Inactive, queue: 73, qu=6, qs=67, qc=0, wr=1/1, in=0/0 IOCP: (Busy=6, Free=999, Min=2,Max=1000), WORKER (Busy=7,Free=8184,Min=2,Max=8191)


此錯誤訊息包含的度量可協助指出問題的原因和可能的解決方案。 下表包含有關錯誤訊息度量的詳細資料。

| 錯誤訊息度量 | 詳細資料 |
| --- | --- |
| inst |在最後一個時間配量︰已發出 0 個命令 |
| mgr |通訊端管理員正在執行 `socket.select`，這表示它會要求 OS 指出有工作需要執行的通訊端；基本上︰讀取器不會主動從網路讀取，因為它不認為有工作需要執行 |
| 佇列 |總共有 73 個進行中的作業 |
| qu |在進行中的作業中，有 6 個位於未傳送的佇列，尚未寫入至輸出網路 |
| qs |在進行中的作業中，有 67 個已傳送至伺服器，但尚未取得回應。 回應可能是 `Not yet sent by the server` 或 `sent by the server but not yet processed by the client.` |
| qc |在進行中的作業中，有 0 個已看見回覆，但尚未標示為已完成，因為正在等候完成迴圈 |
| wr |有作用的寫入器 (亦即不會忽略 6 個未傳送的要求) 位元組/activewriter |
| 了嗎 |沒有作用中的讀取器，在 NIC 位元組/activereader 上可供讀取的位元組為零 |

### <a name="steps-to-investigate"></a>調查步驟
1. 最佳作法是確定您在使用 StackExchange.Redis 用戶端來連線時，使用的是下列模式。

    ```csharp
    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
    {
        return ConnectionMultiplexer.Connect("cachename.redis.cache.windows.net,abortConnect=false,ssl=true,password=...");
    
    });
    
    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }
    ````

    如需詳細資訊，請參閱 [使用 StackExchange.Redis 來連線到快取](cache-dotnet-how-to-use-azure-redis-cache.md#connect-to-the-cache)。

1. 確定 Azure Redis 快取和用戶端應用程式位在 Azure 中的相同區域。 例如，當快取位在美國東部，但用戶端位在美國西部，而要求未在 `synctimeout` 間隔內完成時，您可能會收到逾時，或者當您從本機開發電腦進行偵錯時，您可能會收到逾時。 
   
    強烈建議您讓快取和用戶端位在相同的 Azure 區域中。 如果您有包含跨區域呼叫的案例，您應該將 `synctimeout` 間隔設定為大於預設值 1000 毫秒間隔的值，方法是在連接字串中加入 `synctimeout` 屬性。 下列範例示範 `synctimeout` 為 2000 毫秒的 StackExchange.Redis 快取連接字串程式碼片段。
   
        synctimeout=2000,cachename.redis.cache.windows.net,abortConnect=false,ssl=true,password=...
2. 請確定您使用最新版的 [StackExchange.Redis NuGet 封裝](https://www.nuget.org/packages/StackExchange.Redis/)。 程式碼中的錯誤會經常修正，以便更能應付逾時問題，因此請務必要擁有最新版本。
3. 如果伺服器或用戶端上有要求受到頻寬限制所侷限，則可能需要較長時間才能完成，因而會導致逾時。 若要確認逾時是不是因為伺服器上的網路頻寬所導致，請參閱 [超過伺服器端頻寬](#server-side-bandwidth-exceeded)。 若要確認逾時是不是因為用戶端網路頻寬所導致，請參閱[超過用戶端頻寬](#client-side-bandwidth-exceeded)。
4. 您是否在伺服器或用戶端上受到 CPU 限制？
   
   * 確認您是否在用戶端上受到 CPU 限制，而可能造成要求未在 `synctimeout` 間隔內進行處理，進而導致逾時。 改用較大的用戶端或分散負載有助於控制此問題。 
   * 確認您是否在伺服器上受到 CPU 限制，方法是監視 `CPU` [快取效能度量](cache-how-to-monitor.md#available-metrics-and-reporting-intervals)。 如果要求在 Redis 受到 CPU 限制時傳入，則可能會造成這些要求逾時。 若要解決此狀況，您可以將負載分散在高階快取的多個分區中，或升級至較大的大小或定價層。 如需詳細資訊，請參閱 [超過伺服器端頻寬](#server-side-bandwidth-exceeded)。
5. 伺服器上是否有命令需要很長的處理時間？ Redis 伺服器上處理時間很長的長時間執行命令可能會導致逾時。 長時間執行之命令的部分範例包括有大量金鑰的 `mget`、`keys *` 或編寫得不好的 lua 指令碼。 您可以使用 redis-cli 用戶端連線到 Azure Redis 快取執行個體，或使用 [Redis 主控台](cache-configure.md#redis-console)並執行 [SlowLog](http://redis.io/commands/slowlog) 命令，以查看是否有要求花費的時間超出預期。 Redis 伺服器和 StackExchange.Redis 最適合許多小型要求，而非少數幾個大型要求。 將資料分割成較小的區塊可以改善這些問題。 
   
    如需使用 redis-cli 和 stunnel 連線到 Azure Redis 快取 SSL 端點的相關資訊，請參閱 [宣佈 Redis 預覽版本的 ASP.NET 工作階段狀態提供者](http://blogs.msdn.com/b/webdev/archive/2014/05/12/announcing-asp-net-session-state-provider-for-redis-preview-release.aspx) 部落格文章。 如需詳細資訊，請參閱 [SlowLog](http://redis.io/commands/slowlog)。
6. 高 Redis 伺服器負載可能會導致逾時。 您可以藉由監視 `Redis Server Load` [快取效能度量](cache-how-to-monitor.md#available-metrics-and-reporting-intervals)來監視伺服器負載。 伺服器負載為 100 (最大值) 表示 Redis 伺服器正忙碌處理要求，並沒有閒置的時間。 若要確認特定要求是否會佔用所有伺服器功能，請執行 SlowLog 命令，如上一段所述。 如需詳細資訊，請參閱 [高 CPU 使用率/伺服器負載](#high-cpu-usage-server-load)。
7. 用戶端上是否有其他任何事件可能導致網路問題？ 檢查用戶端 (Web、背景工作角色或 Iaas VM) 上是否有用戶端執行個體數目相應增加或相應減少，或部署新版用戶端或已啟用自動調整之類的事件？我們在測試中發現，自動調整或相應增加/相應減少可能會導致輸出網路連線中斷幾秒鐘。 StackExchange.Redis 程式碼對於這類事件具有復原能力，因此將會重新連線。 在重新連線的這段時間內，佇列中的任何要求都會逾時。
8. 針對 Redis 快取，在幾個小型要求之前是否有大型要求已逾時？ 錯誤訊息中的參數 `qs` 會告訴您有多少要求已從用戶端傳送到伺服器，但尚未處理回應。 這個值會不斷成長，因為 StackExchange.Redis 使用單一 TCP 連線，而且一次只能讀取一個回應。 即使第一項作業已逾時，它也不會停止傳送資料到伺服器/從伺服器傳出資料，在大型要求完成之前，其他要求都會遭到封鎖，因而導致逾時。 有一個解決方案是確保快取足以容納工作負載，並將較大的值分割成較小的區塊，以將逾時的機會降到最低。 另一個可能的解決方案是在用戶端中使用 `ConnectionMultiplexer` 物件集區，並在傳送新要求時選擇最少負載的 `ConnectionMultiplexer`。 這應該就能防止單一逾時造成其他要求也逾時。
9. 如果您使用 `RedisSessionStateprovider`，請確定您已正確設定重試逾時。 `retrytimeoutInMilliseconds` 應高於 `operationTimeoutinMilliseonds`，否則不會發生任何重試。 在下列範例中， `retrytimeoutInMilliseconds` 已設定為 3000。 如需詳細資訊，請參閱 [Azure Redis 快取的 ASP.NET 工作階段狀態供應器](cache-aspnet-session-state-provider.md)和[如何使用工作階段狀態供應器與輸出快取提供者的組態參數](https://github.com/Azure/aspnet-redis-providers/wiki/Configuration)。

    <add
      name="AFRedisCacheSessionStateProvider"
      type="Microsoft.Web.Redis.RedisSessionStateProvider"
      host="enbwcache.redis.cache.windows.net"
      port="6380"
      accessKey="…"
      ssl="true"
      databaseId="0"
      applicationName="AFRedisCacheSessionState"
      connectionTimeoutInMilliseconds = "5000"
      operationTimeoutInMilliseconds = "1000"
      retryTimeoutInMilliseconds="3000" />


1. 請透過[監視](cache-how-to-monitor.md#available-metrics-and-reporting-intervals) `Used Memory RSS` 和 `Used Memory`，檢查 Azure Redis 快取伺服器上的記憶體使用量。 如果已備有收回原則，Redis 就會在 `Used_Memory` 達到快取大小時開始收回金鑰。 理想情況下，`Used Memory RSS` 應該只稍微高於 `Used memory`。 差異很大表示記憶體分散 (內部或外部)。 當 `Used Memory RSS` 小於 `Used Memory` 時，則表示作業系統已交換部分快取記憶體。 如果發生此交換情況，您應該就會遇到顯著的延遲。 因為 Redis 無法控制其配置對應到記憶體分頁的方式，高 `Used Memory RSS` 通常是記憶體使用量突然增加的結果。 當 Redis 釋放記憶體時，記憶體會回到配置器，但配置器不一定會重新提供記憶體給系統。 `Used Memory` 值和作業系統報告的記憶體耗用量可能會有差異。 原因可能是 Redis 已使用並釋放記憶體，但未交回給系統。 若要降低記憶體問題，您可以執行下列步驟：
   
   * 將快取升級為更大的大小，以免達到系統的記憶體限制。
   * 設定金鑰的到期時間，以便主動收回較舊的值。
   * 監視 `used_memory_rss` 快取計量。 當這個值接近其快取的大小時，您可能會開始看到效能問題。 請將資料分散到多個分區 (如果您使用高階快取)，或升級至較大的快取大小。
   
   如需詳細資訊，請參閱 [伺服器上的記憶體壓力](#memory-pressure-on-the-server)。

## <a name="additional-information"></a>其他資訊
* [應該使用哪個 Redis 快取供應項目和大小？](cache-faq.md#what-redis-cache-offering-and-size-should-i-use)
* [如何效能評定和測試我快取的效能？](cache-faq.md#how-can-i-benchmark-and-test-the-performance-of-my-cache)
* [如何執行 Redis 命令？](cache-faq.md#how-can-i-run-redis-commands)
* [如何監視 Azure Redis 快取](cache-how-to-monitor.md)

