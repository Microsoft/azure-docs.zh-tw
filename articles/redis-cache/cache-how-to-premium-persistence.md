---
title: 如何設定進階 Azure Redis 快取的資料永續性
description: 了解如何設定和管理進階層 Azure Redis 快取執行個體的資料永續性
services: redis-cache
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.assetid: b01cf279-60a0-4711-8c5f-af22d9540d38
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: article
ms.date: 08/24/2017
ms.author: wesmc
ms.openlocfilehash: 270158bbf85a58a48a367a091ad2b09a9d114b2b
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38700878"
---
# <a name="how-to-configure-data-persistence-for-a-premium-azure-redis-cache"></a>如何設定進階 Azure Redis 快取的資料永續性
Azure Redis 快取有不同的快取供應項目，可讓您彈性選擇快取大小和功能，包括叢集、持續性和虛擬網路支援等進階層功能。 本文說明如何在進階 Azure Redis 快取執行個體中設定永續性。

如需其他進階快取功能的相關資訊，請參閱 [Azure Redis 快取進階層簡介](cache-premium-tier-intro.md)。

## <a name="what-is-data-persistence"></a>資料永續性是什麼？
[Redis 持續性](https://redis.io/topics/persistence)可讓您保存儲存在 Redis 中的資料。 您也可以擷取快照和備份資料，以在硬體失敗時載入。 這是優於基本或標準層的重大優勢，基本或標準層的所有資料是儲存在記憶體中，若發生快取節點故障的失敗，資料可能會遺失。 

Azure Redis 快取使用下列模型提供 Redis 持續性：

* **RDB 持續性** - 設定 RDB (Redis 資料庫) 持續性後，Azure Redis 快取會依據可設定的備份頻率，在磁碟中保存一份 Redis 二進位格式的 Redis 快取快照。 如果發生同時停用主要和複本快取的災難性事件，即可使用最新的快照重新建構快取。 深入了解 RDB 持續性的[優點](https://redis.io/topics/persistence#rdb-advantages)和[缺點](https://redis.io/topics/persistence#rdb-disadvantages)。
* **AOF 持續性** - 設定 AOF (僅附加檔案) 持續性後，Azure Redis 快取會將每個寫入作業儲存至記錄，而此記錄則會每秒至少一次儲存至 Azure 儲存體帳戶。 如果發生同時停用主要和複本快取的災難性事件，即可使用儲存的寫入作業重新建構快取。 深入了解 AOF 持續性的[優點](https://redis.io/topics/persistence#aof-advantages)和[缺點](https://redis.io/topics/persistence#aof-disadvantages)。

您可以在建立快取期間從 [新的 Redis 快取] 刀鋒視窗設定持續性，也可以在現有進階快取的 [資源] 功能表中設定。

[!INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]

選取進階定價層後，按一下 [Redis 永續性] 。

![Redis 永續性][redis-cache-persistence]

下節的步驟說明如何在您的新進階快取上設定 Redis 持續性。 Redis 永續性設定後，按一下 [ **建立** ] 以建立具有 Redis 永續性的新進階快取。

## <a name="enable-redis-persistence"></a>啟用 Redis 持續性

您可以在 [Redis 資料持續性] 刀鋒視窗中選擇 [RDB] 或 [AOF] 持續性，來啟用 Redis 持續性。 若為新的快取，則在快取建立程序期間存取此刀鋒視窗，如上節所述。 若為現有快取，則從快取的 [資源] 功能表存取 [Redis 資料永續性] 刀鋒視窗。

![Redis 設定][redis-cache-settings]


## <a name="configure-rdb-persistence"></a>設定 RDB 持續性

若要啟用 RDB 持續性，請按一下 [RDB]。 若要停用先前所啟用進階快取的 RDB 持續性，請按一下 [已停用]。

![Redis RDB 持續性][redis-cache-rdb-persistence]

若要設定備份間隔，請選取下拉式清單中的 [ **備份頻率** ]。 選項包括 [15 分鐘]、[30 分鐘]、[60 分鐘]、[6 小時]、[12 小時] 及 [24 小時]。 在先前的備份作業成功完成後，此間隔便會開始倒數計時，時間過後就會起始新的備份。

按一下 [儲存體帳戶] 選取要使用的儲存體帳戶，然後從 [儲存體金鑰] 下拉式清單中選擇 [主要金鑰] 或 [次要金鑰]。 您必須選擇與快取相同區域的儲存體帳戶，建議選取 [進階儲存體]  帳戶，因為進儲存體的輸送量較高。 

> [!IMPORTANT]
> 如果重新產生了永續性帳戶的儲存體金鑰，您必須從 [儲存體金鑰] 下拉式清單中重新設定所需的金鑰。
> 
> 

按一下 [確定]  以儲存持續性組態。

備份頻率間隔過後，會啟動下一個備份 (或新快取的第一個備份)。

## <a name="configure-aof-persistence"></a>設定 AOF 持續性

若要啟用 AOF 持續性，請按一下 [AOF]。 若要停用先前所啟用進階快取的 AOF 持續性，請按一下 [已停用]。

![Redis AOF 持續性][redis-cache-aof-persistence]

若要設定 AOF 持續性，請指定 [第一個儲存體帳戶]。 此儲存體帳戶必須與快取位於相同區域，建議選取 [進階儲存體] 帳戶，因為進階儲存體的輸送量較高。 您可以選擇性地設定額外的儲存體帳戶，並指定為 [第二個儲存體帳戶]。 如果設定第二個儲存體帳戶，複本快取的寫入內容會寫入至此第二個儲存體帳戶。 針對所設定的每個儲存體帳戶，請選擇要從 [儲存體金鑰] 下拉式清單中使用 [主要金鑰] 或 [次要金鑰]。 

> [!IMPORTANT]
> 如果重新產生了永續性帳戶的儲存體金鑰，您必須從 [儲存體金鑰] 下拉式清單中重新設定所需的金鑰。
> 
> 

啟用 AOF 持續性時，快取的寫入作業會儲存至指定的儲存體帳戶 (或設定為第二個儲存體帳戶的帳戶)。 如果發生使主要和複本快取離線的災難性失敗，則會使用儲存的 AOF 記錄來重建快取。

## <a name="persistence-faq"></a>永續性常見問題集
下列清單包含 Azure Redis 快取永續性常見問題的解答。

* [可以對先前建立的快取啟用永續性嗎？](#can-i-enable-persistence-on-a-previously-created-cache)
* [我可以同時啟用 AOF 和 RDB 持續性嗎？](#can-i-enable-aof-and-rdb-persistence-at-the-same-time)
* [我應該選擇哪一種持續性模型？](#which-persistence-model-should-i-choose)
* [如果我調整為不同大小，並還原為調整作業之前製作的備份時，會發生什麼事？](#what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation)


### <a name="rdb-persistence"></a>RDB 持續性
* [在建立快取之後，可以變更 RDB 備份頻率嗎？](#can-i-change-the-rdb-backup-frequency-after-i-create-the-cache)
* [為什麼我的 RDB 備份頻率是 60 分鐘，備份的間隔卻超過 60 分鐘？](#why-if-i-have-an-rdb-backup-frequency-of-60-minutes-there-is-more-than-60-minutes-between-backups)
* [建立新的備份時，舊的 RDB 備份會發生什麼事？](#what-happens-to-the-old-rdb-backups-when-a-new-backup-is-made)

### <a name="aof-persistence"></a>AOF 持續性
* [何時應該使用第二個儲存體帳戶？](#when-should-i-use-a-second-storage-account)
* [AOF 持續性是否會影響快取的輸送量、延遲或效能？](#does-aof-persistence-affect-throughout-latency-or-performance-of-my-cache)
* [我要如何移除第二個儲存體帳戶？](#how-can-i-remove-the-second-storage-account)
* [什麼是重寫，其對快取有何影響？](#what-is-a-rewrite-and-how-does-it-affect-my-cache)
* [在啟用 AOF 下調整快取預期會發生什麼事？](#what-should-i-expect-when-scaling-a-cache-with-aof-enabled)
* [我的 AOF 資料在儲存體中的組織方式為何？](#how-is-my-aof-data-organized-in-storage)


### <a name="can-i-enable-persistence-on-a-previously-created-cache"></a>可以對先前建立的快取啟用永續性嗎？
可以，可在建立快取時以及現有進階快取中設定 Redis 永續性。

### <a name="can-i-enable-aof-and-rdb-persistence-at-the-same-time"></a>我可以同時啟用 AOF 和 RDB 持續性嗎？

不可以，您只能啟用 RDB 或 AOF，但不能同時啟用。

### <a name="which-persistence-model-should-i-choose"></a>我應該選擇哪一種持續性模型？

AOF 持續性將每筆寫入內容儲存至記錄，因此會對輸送量造成一些影響；相較之下，RDB 持續性根據所設定的備份間隔儲存備份，因此對效能的影響很低。 如果您的主要目標在於減少資料遺失，而且您可以處理快取輸送量降低的情況，請選擇 AOF 持續性。 如果您想要維持快取的最佳輸送量，但仍需要資料復原的機制，請選擇 RDB 持續性。

* 深入了解 RDB 持續性的[優點](https://redis.io/topics/persistence#rdb-advantages)和[缺點](https://redis.io/topics/persistence#rdb-disadvantages)。
* 深入了解 AOF 持續性的[優點](https://redis.io/topics/persistence#aof-advantages)和[缺點](https://redis.io/topics/persistence#aof-disadvantages)。

如需使用 AOF 持續性時之效能的詳細資訊，請參閱 [AOF 持續性是否會影響快取的輸送量、延遲或效能？](#does-aof-persistence-affect-throughout-latency-or-performance-of-my-cache)

### <a name="what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation"></a>如果我調整為不同大小，並還原為調整作業之前製作的備份時，會發生什麼事？

針對 RDB 和 AOF 持續性：

* 如果您已調整為較大的大小則沒有任何影響。
* 如果已調整為較小的大小，而且您的自訂[資料庫](cache-configure.md#databases)設定大於新大小的[資料庫限制](cache-configure.md#databases)，則不會還原這些資料庫中的資料。 如需詳細資訊，請參閱[我的自訂資料庫設定在調整期間會受到影響嗎？](cache-how-to-scale.md#is-my-custom-databases-setting-affected-during-scaling)
* 如果已調整為較小的大小，而且較小的大小中沒有足夠的空間可保存來自最近備份的所有資料，將會在還原程序中收回金鑰，通常是使用 [allkeys-lru](http://redis.io/topics/lru-cache) 收回原則。

### <a name="can-i-change-the-rdb-backup-frequency-after-i-create-the-cache"></a>在建立快取之後，可以變更 RDB 備份頻率嗎？
可以，您可以在 [Redis 資料持續性] 刀鋒視窗上變更 RDB 持續性的備份頻率。 如需相關指示，請參閱 [設定 Redis 永續性](#configure-redis-persistence)。

### <a name="why-if-i-have-an-rdb-backup-frequency-of-60-minutes-there-is-more-than-60-minutes-between-backups"></a>為什麼我的 RDB 備份頻率是 60 分鐘，備份的間隔卻超過 60 分鐘？
在前一個備份程序順利完成後，RDB 持續性備份頻率間隔才會開始計算。 如果備份頻率是 60 分鐘，而備份程序要 15 分鐘才能順利完成，則下一次備份要在先前的備份開始的 75 分鐘後才會開始。

### <a name="what-happens-to-the-old-rdb-backups-when-a-new-backup-is-made"></a>建立新的備份時，舊的 RDB 備份會發生什麼事？
除了最新的備份外，所有 RDB 持續性備份都會自動刪除。 這項刪除作業可能不會立即發生，但較舊的備份不會無限期保存。


### <a name="when-should-i-use-a-second-storage-account"></a>何時應該使用第二個儲存體帳戶？

當您認為快取上有比預期更高的設定作業時，您應該針對 AOF 持續性使用第二個儲存體帳戶。  設定第二個儲存體帳戶有助於確保您的快取不會達到儲存體頻寬限制。

### <a name="does-aof-persistence-affect-throughout-latency-or-performance-of-my-cache"></a>AOF 持續性是否會影響快取的輸送量、延遲或效能？

當快取低於最大負載 (CPU 和伺服器負載皆低於 90%) 時，AOF 持續性會影響輸送量約 15%-20%。 當快取在這些限制範圍內時，不應該有延遲問題。 不過，啟用 AOF 時，快取會更快達到這些限制。

### <a name="how-can-i-remove-the-second-storage-account"></a>我要如何移除第二個儲存體帳戶？

您可以透過將 AOF 持續性的第二個儲存體帳戶設定為與第一個儲存體帳戶相同，來移除第二個儲存體帳戶。 如需相關指示，請參閱[設定 AOF 持續性](#configure-aof-persistence)。

### <a name="what-is-a-rewrite-and-how-does-it-affect-my-cache"></a>什麼是重寫，其對快取有何影響？

當 AOF 檔案變得夠大時，系統會自動將重寫作業排入快取佇列。 重寫作業會調整 AOF 檔案大小，只包含建立目前資料集所需的一組基本作業。 在重寫期間，預期會更快達到效能限制，特別是在處理大型資料集時。 隨著 AOF 檔案愈來愈大，重寫的發生頻率會減少，但發生時將需要大量時間。

### <a name="what-should-i-expect-when-scaling-a-cache-with-aof-enabled"></a>在啟用 AOF 下調整快取預期會發生什麼事？

如果 AOF 檔案在調整時相當大，調整作業所花的時間會超出預期，因為在調整完成之後會重新載入檔案。

如需調整的詳細資訊，請參閱[如果我調整為不同大小，並還原為調整作業之前製作的備份時，會發生什麼事？](#what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation)

### <a name="how-is-my-aof-data-organized-in-storage"></a>我的 AOF 資料在儲存體中的組織方式為何？

儲存在 AOF 檔案中的資料會根據節點分成多個分頁 Blob，以提升將資料儲存至儲存體的效能。 下表顯示針對每個定價層所使用的分頁 Blob 數量：

| 高階層 | Blob |
|--------------|-------|
| P1           | 每個分區 4 個    |
| P2           | 每個分區 8 個    |
| P3           | 每個分區 16 個   |
| P4           | 每個分區 20 個   |

啟用叢集時，快取中的每個分區會有一組專屬的分頁 Blob，如上表所示。 例如，具有三個分區的 P2 快取會將其 AOF 檔案散發到 24 個分頁 Blob (3 個分區，所以每個分區 8 個 Blob)。

重寫後，儲存體中會有兩組 AOF 檔案。 重寫會在背景進行並將內容附加至第一組檔案，而當重寫內容附加至第二組檔案期間，會將設定作業傳送至快取。 重寫期間會暫時儲存備份以備失敗之需，但在重寫完成之後則會立即刪除備份。


## <a name="next-steps"></a>後續步驟
了解如何使用更多進階快取功能。

* [Azure Redis Cache 高階層簡介](cache-premium-tier-intro.md)

<!-- IMAGES -->

[redis-cache-premium-pricing-tier]: ./media/cache-how-to-premium-persistence/redis-cache-premium-pricing-tier.png

[redis-cache-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-persistence.png

[redis-cache-rdb-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-rdb-persistence.png

[redis-cache-aof-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-aof-persistence.png

[redis-cache-settings]: ./media/cache-how-to-premium-persistence/redis-cache-settings.png
