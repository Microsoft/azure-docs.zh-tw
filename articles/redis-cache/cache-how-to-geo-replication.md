---
title: 如何設定 Azure Redis 快取的異地複寫 | Microsoft Docs
description: 了解如何跨地理區域複寫您的 Azure Redis 快取執行個體。
services: redis-cache
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.assetid: 375643dc-dbac-4bab-8004-d9ae9570440d
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: article
ms.date: 09/15/2017
ms.author: wesmc
ms.openlocfilehash: 644a7414365ded53acb20bb2f9ae9d086f263765
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "42141471"
---
# <a name="how-to-configure-geo-replication-for-azure-redis-cache"></a>如何設定 Azure Redis 快取的異地複寫

異地複寫提供一個機制，可供連結兩個高階層 Azure Redis 快取執行個體。 其中一個快取被指定為主要連結快取，而另一個則為次要連結快取。 次要連結快取會變成唯讀，而寫入主要快取的資料會複寫至次要連結快取。 這項功能可用來跨 Azure 區域複寫快取。 本文提供的指南說明如何設定高階層 Azure Redis 快取執行個體的異地複寫。

## <a name="geo-replication-prerequisites"></a>異地複寫的必要條件

若要設定兩個快取之間的異地複寫，必須符合下列必要條件：

- 這兩個快取必須是[高階層](cache-premium-tier-intro.md)快取。
- 這兩個快取必須位於相同的 Azure 訂用帳戶。
- 次要連結快取必須為相同的定價層，或超過主要連結快取的定價層。
- 如果主要連結快取已啟用叢集，次要連結快取就必須啟用具有相同分區數目的叢集作為主要連結快取。
- 必須建立兩個快取並同時處於執行中狀態。
- 任何一個快取上皆不能啟用持續性。
- 支援在相同 VNET 中多個快取之間的異地複寫。 
- 在相同區域內對等互連 VNet 中的快取之間進行異地複寫，目前為預覽功能。 兩個 VNet 必須設定成能讓 VNet 中的資源透過 TCP 連線觸達彼此。
- 目前還不支援在不同區域內對等互連 VNet 中的快取之間進行異地複寫，但很快就會推出其預覽功能。

在設定異地複寫之後，下列限制適用於連結的快取組：

- 次要連結快取是唯讀的；您可以從中讀取，但無法寫入任何資料。 
- 任何將連結新增之前就已在次要連結快取中的資料都會加以移除。 不過，如果後續將異地複寫移除，複寫的資料就會保留在次要連結快取中。
- 如果快取已啟用叢集，就無法在任一個快取上起始[調整大小作業](cache-how-to-scale.md)，或是[變更分區數目](cache-how-to-premium-clustering.md)。
- 您無法啟用任一個快取的持續性。
- 您可以對任一個快取使用[匯出](cache-how-to-import-export-data.md#export)，但是您只可以[匯入](cache-how-to-import-export-data.md#import)主要連結快取中。
- 您無法刪除任一個連結快取或包含它們的資源群組，直到您移除異地複寫連結為止。 如需詳細資訊，請參閱[當我嘗試刪除連結快取時，作業失敗的原因？](#why-did-the-operation-fail-when-i-tried-to-delete-my-linked-cache)
- 如果兩個快取位於不同的區域，網路輸出費用將會套用至跨區域複寫到次要連結快取的資料。 如需詳細資訊，請參閱[跨 Azure 區域複寫我的資料需要多少費用？](#how-much-does-it-cost-to-replicate-my-data-across-azure-regions)
- 如果主要快取 (和其複本) 關閉，沒有任何自動容錯移轉至次要連結快取。 為了容錯移轉用戶端應用程式，您必須將異地複寫連結手動移除，並將用戶端應用程式指向先前為次要連結快取的快取。 如需詳細資訊，請參閱[容錯移轉至次要連結快取如何運作？](#how-does-failing-over-to-the-secondary-linked-cache-work)

## <a name="add-a-geo-replication-link"></a>新增異地複寫連結

1. 若要將兩個高階快取連結在一起以進行異地備援，請從要作為主要連結快取之快取的 [資源] 功能表按一下 [異地複寫]，然後從 [異地複寫] 刀鋒視窗按一下 [新增快取複寫連結]。

    ![新增連結](./media/cache-how-to-geo-replication/cache-geo-location-menu.png)

2. 從 [相容的快取] 清單中按一下所需次要快取的名稱。 如果您所需的快取未顯示在清單中，請確認符合所需次要快取的[異地複寫必要條件](#geo-replication-prerequisites)。 若要依區域篩選快取，按一下地圖中的所需區域，可在 [相容的快取] 清單中僅顯示這些快取。

    ![異地複寫相容的快取](./media/cache-how-to-geo-replication/cache-geo-location-select-link.png)
    
    您也可以使用操作功能表來起始連結的流程，或檢視有關次要快取的詳細資料。

    ![異地複寫操作功能表](./media/cache-how-to-geo-replication/cache-geo-location-select-link-context-menu.png)

3. 按一下 [連結] 可將兩個快取連結在一起，並開始複寫流程。

    ![連結快取](./media/cache-how-to-geo-replication/cache-geo-location-confirm-link.png)

4. 您可以在 [異地複寫] 刀鋒視窗上檢視複寫流程的進度。

    ![連結狀態](./media/cache-how-to-geo-replication/cache-geo-location-linking.png)

    您也可以在 [概觀]刀鋒視窗上檢視主要和次要快取的連結狀態。

    ![快取狀態](./media/cache-how-to-geo-replication/cache-geo-location-link-status.png)

    一旦複寫程序完成之後，[連結狀態] 會變為 [成功]。

    ![快取狀態](./media/cache-how-to-geo-replication/cache-geo-location-link-successful.png)

    在連結流程期間，主要連結快取仍然可供使用，但連結流程完成之前，沒有可用的次要連結快取。

## <a name="remove-a-geo-replication-link"></a>移除異地複寫連結

1. 若要將兩個快取之間的連結移除並停止異地複寫，請從 [異地複寫] 刀鋒視窗按一下 [取消連結快取]。
    
    ![取消連結快取](./media/cache-how-to-geo-replication/cache-geo-location-unlink.png)

    取消連結流程完成時，次要快取就可供讀取和寫入。

>[!NOTE]
>將異地複寫連結移除時，從主要連結快取複寫的資料就會保留在次要快取中。
>
>

## <a name="geo-replication-faq"></a>異地複寫常見問題集

- [可以使用異地複寫搭配標準或基本層快取嗎？](#can-i-use-geo-replication-with-a-standard-or-basic-tier-cache)
- [在連結或取消連結流程期間，快取是否可供使用？](#is-my-cache-available-for-use-during-the-linking-or-unlinking-process)
- [可以同時連結兩個以上的快取嗎？](#can-i-link-more-than-two-caches-together)
- [可以將不同 Azure 訂用帳戶中的兩個快取加以連結嗎？](#can-i-link-two-caches-from-different-azure-subscriptions)
- [可以將兩個不同大小的快取加以連結嗎？](#can-i-link-two-caches-with-different-sizes)
- [啟用叢集時可以使用異地複寫嗎？](#can-i-use-geo-replication-with-clustering-enabled)
- [可以使用異地複寫搭配 VNET 中的快取嗎？](#can-i-use-geo-replication-with-my-caches-in-a-vnet)
- [哪個複寫排程適用於 Redis 異地複寫？](#what-is-the-replication-schedule-for-redis-geo-replication)
- [異地複寫需要花費多久的時間？](#how-long-does-geo-replication-replication-take)
- [是否保證複寫復原點？](#is-the-replication-recovery-point-guaranteed)
- [可以使用 PowerShell 或 Azure CLI 管理異地複寫嗎？](#can-i-use-powershell-or-azure-cli-to-manage-geo-replication)
- [跨 Azure 區域複寫我的資料需要多少費用？](#how-much-does-it-cost-to-replicate-my-data-across-azure-regions)
- [當我嘗試刪除連結快取時，作業失敗的原因？](#why-did-the-operation-fail-when-i-tried-to-delete-my-linked-cache)
- [要將次要連結快取用於哪個區域？](#what-region-should-i-use-for-my-secondary-linked-cache)
- [容錯移轉至次要連結快取如何運作？](#how-does-failing-over-to-the-secondary-linked-cache-work)

### <a name="can-i-use-geo-replication-with-a-standard-or-basic-tier-cache"></a>可以使用異地複寫搭配標準或基本層快取嗎？

否，異地複寫僅適用於高階層快取。

### <a name="is-my-cache-available-for-use-during-the-linking-or-unlinking-process"></a>在連結或取消連結流程期間，快取是否可供使用？

- 同時連結兩個快取進行異地複寫時，主要連結快取仍然可供使用，但連結流程完成之前，沒有可用的次要連結快取。
- 將兩個快取之間的異地複寫連結移除時，這兩個快取都保持可供使用。

### <a name="can-i-link-more-than-two-caches-together"></a>可以同時連結兩個以上的快取嗎？

否，使用異地複寫時，您只能將兩個快取連結在一起。

### <a name="can-i-link-two-caches-from-different-azure-subscriptions"></a>可以將不同 Azure 訂用帳戶中的兩個快取加以連結嗎？

否，這兩個快取必須位於相同的 Azure 訂用帳戶。

### <a name="can-i-link-two-caches-with-different-sizes"></a>可以將兩個不同大小的快取加以連結嗎？

是，前提是次要連結快取超過主要連結快取。

### <a name="can-i-use-geo-replication-with-clustering-enabled"></a>啟用叢集時可以使用異地複寫嗎？

是，前提是這兩個快取具有相同的分區數目。

### <a name="can-i-use-geo-replication-with-my-caches-in-a-vnet"></a>可以使用異地複寫搭配 VNET 中的快取嗎？

是，支援 VNET 中快取的異地複寫。 

- 支援在相同 VNET 中多個快取之間的異地複寫。
- 只要將兩個 VNET 設定為 VNET 中的資源能透過 TCP 連線觸達彼此的方式，就也可支援在不同 VNET 中多個快取之間的異地複寫。

### <a name="what-is-the-replication-schedule-for-redis-geo-replication"></a>哪個複寫排程適用於 Redis 異地複寫？

複寫不會在特定排程發生，它是連續且非同步的，亦即， 已在主要複本上完成的所有寫入，都會在次要複本上立即以非同步方式加以複寫。

### <a name="how-long-does-geo-replication-replication-take"></a>異地複寫需要花費多久的時間？

複寫是累加式、非同步且連續的，而所花費的時間與跨區域的延遲通常不會有很大的差異。 在某些情況下，在特定時間，可能需要次要複本對來自主要複本的資料進行完整同步處理。 在此情況下，複寫時間會取決於許多因素，例如：主要快取上的負載、快取電腦上的可用頻寬、區域間的延遲等。例如，根據某些測試，我們發現在美國東部與美國西部中完整 53 GB 異地複寫對組的複寫時間會介於 5 到 10 分鐘。

### <a name="is-the-replication-recovery-point-guaranteed"></a>是否保證複寫復原點？

目前，已針對異地複寫模式中的快取，停用持續性和匯入/匯出功能。 因此，萬一客戶起始了容錯移轉，或是異地複寫對組之間的複寫連結已中斷，則次要複本將會保留記憶體中的資料，該資料是它到該時點為止已與主要複本進行同步處理的資料。 在這類情況下並未提供任何復原點保證。

### <a name="can-i-use-powershell-or-azure-cli-to-manage-geo-replication"></a>可以使用 PowerShell 或 Azure CLI 管理異地複寫嗎？

目前，您只能使用 Azure 入口網站管理異地複寫。

### <a name="how-much-does-it-cost-to-replicate-my-data-across-azure-regions"></a>跨 Azure 區域複寫我的資料需要多少費用？

使用異地複寫時，主要連結快取的資料會複寫到次要連結快取。 如果兩個連結快取位於相同的 Azure 區域中，資料轉送就不需付費。 如果兩個連結快取位於不同的 Azure 區域中，異地複寫資料轉送費用就是將該資料複寫到其他 Azure 區域的頻寬費用。 如需詳細資訊，請參閱[頻寬定價詳細資料](https://azure.microsoft.com/pricing/details/bandwidth/)。

### <a name="why-did-the-operation-fail-when-i-tried-to-delete-my-linked-cache"></a>當我嘗試刪除連結快取時，作業失敗的原因？

當兩個快取連結在一起時，您無法刪除任一個快取或包含它們的資源群組，直到您移除異地複寫連結為止。 如果您嘗試將包含一個或兩個連結快取的資源群組刪除，就會將資源群組中的其他資源刪除，但資源群組會保留在 `deleting` 狀態，而資源群組中的任何連結快取會維持 `running` 狀態。 若要完成刪除資源群組和其中的連結快取，請如[移除異地複寫連結](#remove-a-geo-replication-link)中所述，將異地複寫連結中斷。

### <a name="what-region-should-i-use-for-my-secondary-linked-cache"></a>要將次要連結快取用於哪個區域？

一般情況下，建議您的快取與存取它的應用程式位於相同的 Azure 區域中。 如果您的應用程式有主要和後援區域，您的主要和次要快取就應該位於這些相同的區域。 如需配對區域的詳細資訊，請參閱[最佳做法 – Azure 配對的區域](../best-practices-availability-paired-regions.md)。

### <a name="how-does-failing-over-to-the-secondary-linked-cache-work"></a>容錯移轉至次要連結快取如何運作？

在異地複寫的初始版本中，Azure Redis 快取不支援跨 Azure 區域自動容錯移轉。 異地複寫主要用於災害復原情節。 在災害復原情節中，客戶需以協調的方式，在備份的區域中提出整個應用程式堆疊，而不是讓個別應用程式元件決定何時要自行切換至其備份。 這與 Redis 特別相關。 Redis 的主要優點之一是，它是非常低延遲的存放區。 如果應用程式所使用的 Redis 容錯移轉至不同的 Azure 區域，但計算層並未容錯移轉，新增的來回時間就會對效能產生明顯的影響。 基於這個理由，我們將需要避免 Redis 因暫時性的可用性問題而自動容錯移轉。

目前，若要起始容錯移轉，您必須將 Azure 入口網站中的異地複寫連結移除，然後將 Redis 用戶端中的連線端點從主要連結快取變更為 (先前稱為連結) 次要快取。 當兩個快取解除關聯時，複本會再次變成一般的讀寫快取，並接受直接來自 Redis 用戶端的要求。


## <a name="next-steps"></a>後續步驟

深入了解 [Azure Redis Cache 高階層](cache-premium-tier-intro.md)。

