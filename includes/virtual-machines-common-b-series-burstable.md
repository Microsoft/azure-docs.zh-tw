---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: jonbeck7
ms.service: virtual-machines
ms.topic: include
ms.date: 03/09/2018
ms.author: azcspmt;jonbeck;cynthn
ms.custom: include file
ms.openlocfilehash: 14feb7ad09a24904034f9ae90cf4a54cf786a44c
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44369507"
---
B 系列 VM 系列可讓您選擇能為您的工作負載提供所需基本層級效能的 VM 大小，並且能夠使 CPU 效能高載高達 Intel® Broadwell E5-2673 v4 2.3 GHz 或 Intel® Haswell 2.4 GHz E5-2673 v3 處理器 vCPU 的 100%。

B 系列 VM 非常適合不需要持續性完整 CPU 效能的工作負載，例如 Web 伺服器、概念證明、小型資料庫以及開發建置環境。 這些工作負載通常具有高載的效能需求。 B 系列可讓您購買基準效能的 VM 大小，並在 VM 執行個體使用低於其基準時再向上建立點數。 當 VM 累積點數時，VM 可以在您的應用程式需要較高的 CPU 效能時，使用最多 100% 的 vCPU 來高載高於基準。

B 系列隨附下列六個 VM 大小：

| 大小          | vCPU | 記憶體：GiB | 暫存儲存體 (SSD) GiB | VM 的基礎 CPU 效能 | VM 的最大 CPU 效能 | 點數累積 / 小時 | 最大累積點數 |
|---------------|--------|-------------|----------------|--------------------------------|---------------------------|-----------------------|--------------------|
| Standard_B1s  | 1      | 1           | 4              | 10%                            | 100%                      | 6                     | 144                |
| Standard_B1ms | 1      | 2           | 4              | 20%                            | 100%                      | 12                    | 288                |
| Standard_B2s  | 2      | 4           | 8              | 40%                            | 200%                      | 24                    | 576                |
| Standard_B2ms | 2      | 8           | 16             | 60%                            | 200%                      | 36                    | 864                |
| Standard_B4ms | 4      | 16          | 32             | 90%                            | 400%                      | 54                    | 1296               |
| Standard_B8ms | 8      | 32          | 64             | 135%                           | 800%                      | 81                    | 1944               |




## <a name="q--a"></a>問答集 

### <a name="q-how-do-you-get-135-baseline-performance-from-a-vm"></a>問：如何從 VM 取得 135% 的基準效能？
**答**：會在構成 VM 大小的 8 個 vCPU 之間共用 135%。 例如，如果您的應用程式將 8 個核心其中 4 個用於批次處理，而這 4 個 vCPU 每個都以 30% 的使用率執行，VM CPU 效能的總數就會等於 120%。  這表示 VM 會以您基準效能的 15% 差異作為基礎來建置點數時間。  但這也表示，當您有可用點數時，該相同 VM 可以 100% 使用所有 8 個 vCPU，使該 VM 擁有 800% 的最大 CPU 效能。


### <a name="q-how-can-i-monitor-my-credit-balance-and-consumption"></a>問：如何監視我的點數結餘和耗用量
**答**：我們在未來幾週會推出 2 個新的計量，**點數**計量可讓您檢視 VM 累積了多少點數，而**已耗用點數**計量會顯示 VM 從撲滿中耗用了多少 CPU 點數。    您可以在入口網站中的 [計量] 窗格檢視這些計量，或透過 Azure 監視器 API 以程式設計方式檢視。

如需如何存取 Azure 計量資料的詳細資訊，請參閱 [Microsoft Azure 的計量概觀](../articles/monitoring-and-diagnostics/monitoring-overview-metrics.md)。

### <a name="q-how-are-credits-accumulated"></a>問：如何累積點數？
**答**：VM 累積與消耗率設定的方式，是以基礎效能層級執行的 VM 會具有高載點數的淨累積或耗用量。  每當 VM 在其基礎效能層級下執行時，點數淨值都會增加，且每當 VM 利用 CPU 超過其基礎效能層級時，點數淨值就會降低。

**範例**：我針對短暫的時間使用 B1ms 大小及現身資料庫應用程式來部署 VM。 這個大小可讓我的應用程式使用最多 20% 的 vCPU 作為基準，也就是我每分鐘可以使用或累積 0.2 個點數。 

我的應用程式從我員工的工作日開始運作到員工下班，時間是上午 7:00-9:00 和下午 4:00 - 6:00 之間。 一天的其他 20 小時內，我的應用程式通常為閒置狀態，只會使用 10% 的 vCPU。 在非尖峰時間，我每分鐘獲得 0.2 個點數，但每分鐘只使用 0.1 個點數，因此我的 VM 每小時會累積 0.1 x 60 = 6 個點數。  在離峰的 20 小時內，我會累積 120 個點數。  

在尖峰時間內，我的應用程式平均會有 60% 的 vCPU 使用率，我每分鐘仍可獲得 0.2 個點數，但我每分鐘使用 0.6 個點數，一分鐘的淨成本是 0.4 個點數，也就是每小時 0.4 x 60 = 24 個點數。 我每日有 4 小時的尖峰使用量，因此尖峰使用量的成本是 4 x 24 = 96 個點數。

如果我用離峰時間所獲得的 120 個點數減去尖峰時間使用的 96 個點數，每天就可以額外累積 24 個點數，能用於其他高載的活動。


### <a name="q-does-the-b-series-support-premium-storage-data-disks"></a>問：B 系列是否支援進階儲存體資料磁碟？
**答**：是的，所有 B 系列大小都支援進階儲存體資料磁碟。   
    
### <a name="q-why-is-my-remaining-credit-set-to-0-after-a-redeploy-or-a-stopstart"></a>問：為何在重新部署或停止/啟動之後，剩餘信用額度會設定為 0？
**答**：當虛擬機器「重新部署」且虛擬機器移到另一個節點時，累積的信用額度會遺失。 如果虛擬機器停止/啟動，但仍留在相同節點上，則虛擬機器會保留累積的信用額度。 每當虛擬機器在節點上以全新方式啟動時，它會取得初始信用額度，如果是 Standard_B8ms，額度為 240 分鐘。

    

    
