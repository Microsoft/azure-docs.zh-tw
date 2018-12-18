---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 09/24/2018
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: bb9a2a884439b00f52adfa9b7c1010a4610a77f7
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47401602"
---
# <a name="ultra-ssd-preview-managed-disks-for-azure-virtual-machine-workloads"></a>適用於 Azure 虛擬機器工作負載的Ultra SSD (預覽) 受控磁碟

Azure Ultra SSD (預覽) 可為 Azure IaaS VM 提供高輸送量、高 IOPS 以及一致的低延遲磁碟儲存體。 這個新的供應項目可提供絕佳的效能，同時保有我們現有磁碟供應項目的相同可用性層級。 Ultra SSD 的其他優點包括能夠以動態方式變更磁碟的效能和您的工作負載，而不需要重新啟動虛擬機器。 Ultra SSD 適用於資料密集的工作負載 (例如 SAP HANA)、最上層資料庫，以及高交易量的工作負載。

## <a name="ultra-ssd-features"></a>Ultra SSD 功能

**受控磁碟**：Ultra SSD 僅可供作為「受控磁碟」使用。 您無法將 Ultra SSD 部署為非受控磁碟或分頁 Blob。 建立「受控磁碟」時，您只要將磁碟 SKU 類型指定為 UltraSSD_LRS，並指定您所需的磁碟大小、IOPS 和輸送量，Azure 就會為您建立及管理該磁碟。  

**虛擬機器**：Ultra SSD 依設計可使用所有支援進階 SSD 的 Azure 虛擬機器 SKU，不過在預覽階段，VM 大小會限定為 ES/DS v3 VM 執行個體。

**動態效能組態**：Ultra SSD 可讓您以動態方式變更磁碟的效能 (IOPS 和輸送量) 以及您的工作負載需求，而不需要重新啟動虛擬機器。

## <a name="scalability-and-performance-targets"></a>擴充和效能目標

當您佈建 Ultra SSD 時，您將可選擇個別設定磁碟的容量和效能。 Ultra SSD 有 4 GiB 到 64 TiB 的幾種固定大小可供選擇，且具有彈性效能組態模型，可讓您個別設定 IOPS 和輸送量。 Ultra SSD 只能作為資料磁碟。 建議您以進階 SSD 作為作業系統磁碟。

Ultra SSD 的主要功能包括：

- 磁碟容量：Ultra SSD 為您提供從 4 GiB 到 64 TiB 的不同磁碟大小。
- 磁碟 IOPS：Ultra SSD 支援 300 IOPS/GiB 的 IOPS 限制，最多可達每個磁碟 160 K 的 IOPS。 若要達到您所佈建的 IOPS，請確定選取的磁碟 IOPS 小於 VM IOPS。 最小的磁碟 IOPS 為 100 IOPS。
- 磁碟輸送量：使用 Ultra SSD 時，對於每個佈建的 IOPS，單一磁碟的輸送量上限為 256 KiB/秒，最多可達每個磁碟 2000 MBps (其中，MBps = 10 ^6 位元組/秒)。 最小的磁碟輸送量為 1 MiB。

下表概述不同磁碟大小支援的不同設定：  

### <a name="ultra-ssd-managed-disk-offerings"></a>Ultra SSD 受控磁碟供應項目

|磁碟大小 (GiB)  |IOPS 上限  |輸送量上限 (MBps)  |
|---------|---------|---------|
|4     |1,200         |300         |
|8     |2,400         |600         |
|16     |4,800         |1,200         |
|32     |9,600         |2,000         |
|64     |19,200         |2,000         |
|128     |38,400         |2,000         |
|256     |76,800         |2,000         |
|512     |80,000         |2,000         |
|1,024-65,536 (此範圍中的大小會以 1 TiB 為單位遞增)     |160,000         |2,000         |

## <a name="pricing-and-billing"></a>價格和計費

使用 Ultra SSD 時，會有下列計費考量：

- 受控磁碟大小
- 受控磁碟佈建的 IOPS
- 受控磁碟佈建的輸送量
- Ultra SSD VM 保留費用

### <a name="managed-disk-size"></a>受控磁碟大小

受控磁碟依據佈建的大小計費。 Azure 會將佈建大小對應 (無條件進位) 至最接近的磁碟大小供應項目。 如需所提供磁碟大小的詳細資料，請參閱上述＜延展性和效能目標＞一節中的表格。 每個磁碟皆與一個支援的佈建磁碟大小對應，並據此依小時計費。 例如，若您佈建了一個 200 GiB 的 Ultra SSD 磁碟，並在 20 小時後刪除了這個磁碟，這個磁碟將會對應至 256 GiB 的磁碟大小方案，並且以 256 GiB 的基準向您收取 20 小時的費用。 這與寫入磁碟的實際資料量無關。

### <a name="managed-disk-provisioned-iops"></a>受控磁碟佈建的 IOPS

IOPS 是指您的應用程式在一秒內傳送到磁碟的要求數。 輸入/輸出作業可以是循序或隨機、讀取或寫入作業。 和磁碟大小相同，佈建的 IOPS 也會依小時計費。 如需所提供 IOPS 的詳細資訊，請參閱上述「延展性和效能目標」一節中的表格。

### <a name="managed-disk-provisioned-throughput"></a>受控磁碟佈建的輸送量

輸送量是指應用程式在指定的間隔內傳送到磁碟的資料量，以位元組/秒為計算單位。 如果您的應用程式執行較大的輸入/輸出作業，則需要較高輸送量。  

輸送量和 IOPS 之間存在某種關聯，如下列公式所示：IOPS x IO 大小 = 輸送量

因此，務必判斷應用程式所需的最佳輸送量和 IOPS 值。 嘗試最佳化其中一個時，另一個也會受影響。 建議一開始先使用對應於 16 KiB IO 大小的輸送量，等到需要更多輸送量時再進行調整。

如需與 Ultra SSD 支援的磁碟輸送量有關的詳細資訊，請參閱上述「延展性和效能目標」一節中的表格。 如同磁碟大小和 IOPS，佈建的輸送量也會根據佈建的 MBps 依小時計費。

### <a name="ultra-ssd-vm-reservation-fee"></a>Ultra SSD VM 保留費用

我們導入了一項 VM 上的功能，可將您的 VM 指定為與 Ultra SSD 相容。 與 Ultra SSD 相容的 VM 會在計算 VM 執行個體與區塊儲存體的縮放單位之間配置專用頻寬容量，以最佳化效能並減少延遲。 若在 VM 上新增此功能，只有在未將 Ultra SSD 磁碟連結至該 VM 的情況下於 VM 上啟用 Ultra SSD 功能時，才會產生保留費用。 當 Ultra SSD 磁碟連結至與 Ultra SSD 相容的 VM 時，就不會收取此費用。 此費用會根據在 VM 上佈建的 vCPU 計費。

請參閱 [Azure 磁碟定價頁面](https://azure.microsoft.com/pricing/details/managed-disks/)，以了解有限預覽提供的最新 Ultra SSD 磁碟價格明細。

### <a name="ultra-ssd-preview-scope-and-limitations"></a>Ultra SSD 預覽版的範圍和限制

在預覽期間，Ultra SSD 磁碟：

- 最初可在美國東部 2 的單一可用性區域中受到支援  
- 只能用於可用性區域 (區域以外的可用性設定組和單一 VM 部署將無法連結 Ultra SSD 磁碟)
- 僅在 ES/DS v3 VM 上受到支援
- 只能作為資料磁碟，且僅支援 4k 實體磁區大小  
- 只能建立為空磁碟  
- 目前只能使用 Resource Manager 範本、CLI 和 Python SDK 進行部署。
- 尚不支援磁碟快照集、VM 映像、可用性設定組、虛擬機器擴展集和 Azure 磁碟加密。
- 尚不支援與 Azure 備份或 Azure Site Recovery 整合。
- 如同 [大部分的預覽](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)，這項功能在正式運作 (GA) 之前不應該用於生產工作負載。