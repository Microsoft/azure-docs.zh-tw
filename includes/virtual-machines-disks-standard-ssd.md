---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 08/14/2018
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: f8ca716f4ab991fecca52ca2d5fed080e6f4c177
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47060518"
---
# <a name="standard-ssd-managed-disks-for-azure-virtual-machine-workloads"></a>適用於 Azure 虛擬機器工作負載的標準 SSD 受控磁碟

「Azure 標準固態硬碟 (SSD) 受控磁碟」是一個已針對在較低 IOPS 層級需要一致效能的工作負載最佳化且符合成本效益的儲存體選項。 「標準 SSD」為想要移至雲端的使用者，尤其是在於內部部署 HDD 解決方案上執行之工作負載的變異方面遇到問題的使用者，提供一個良好的入門級體驗。 與「HDD 磁碟」相比，「標準 SSD」提供更佳的可用性、一致性、可靠性及延遲，適用於 Web 伺服器、低 IOPS 應用程式伺服器、輕量使用的企業應用程式，以及開發/測試工作負載。

## <a name="standard-ssd-features"></a>標準 SSD 功能

**受控磁碟**：「標準 SSD」僅可供作為「受控磁碟」使用。 「標準 SSD」不支援「非受控磁碟」和「分頁 Blob」。 建立「受控磁碟」時，您需將磁碟類型指定為 [標準 SSD] 並指示所需的磁碟大小，然後 Azure 就會為您建立及管理該磁碟。
「標準 SSD」支援「受控磁碟」所提供的所有服務管理作業。 例如，您可以建立、複製「標準 SSD 受控磁碟」或為其建立快照，方式與對「受控磁碟」所做的相同。

**虛擬機器**：「標準 SSD」可以與所有 Azure VM 搭配使用，包括不支援「進階磁碟」的 VM 類型。 例如，不論您使用 A 系列 VM、N 系列 VM、DS 系列還是任何其他 Azure VM 系列，都可以將「標準 SSD」與該 VM 搭配使用。 在導入「標準 SSD」之後，我們便能將先前使用 HDD 型磁碟的廣泛工作負載轉換至 SSD 型磁碟，並體驗一致的效能、更高的可用性、更佳的延遲，以及伴隨 SSD 而來整體更佳的體驗。

**高耐久性與可用性**：「標準 SSD」以相同的「Azure 磁碟」平台為基礎，此平台為磁碟一致地提供高可用性和耐久性。 「Azure 磁碟」已設計成可確保可用性達 99.999%。 與所有「受控磁碟」相同，「標準 SSD」也提供「本地備援儲存體」(LRS)。 有了 LRS，平台便可為每個磁碟保有多個資料複本，並為 IaaS 磁碟一致地提供領先業界、「年度失敗率」為零的企業級持久性。

**快照集**：與所有「受控磁碟」相同，「標準 SSD」也支援建立快照集。 快照集類型可以是「標準 (HDD)」或「進階 (SSD)」。 為了節省成本，建議您針對所有 Azure 磁碟類型都使用「標準 (HDD)」快照集類型。 這是因為當您從快照集建立受控磁碟時，一律可以選擇更高的階層，例如「標準 SSD」或「進階 SSD」。

## <a name="scalability-and-performance-targets"></a>擴充和效能目標

下表包含目前針對「標準 SSD」提供的磁碟大小。

|標準 SSD 磁碟類型  |磁碟大小  |每一磁碟的 IOPS  |每一磁碟的輸送量  |
|---------|---------|---------|---------|
|E10     |128 GB         |最多 500         |每秒最多 60 MiB         |
|E15     |256 GiB         |最多 500         |每秒最多 60 MiB         |
|E20     |512 GB         |最多 500         |每秒最多 60 MiB         |
|E30     |1,024 GiB       |最多 500         |每秒最多 60 MiB         |
|E40     |2,048 GiB       |最多 500         |每秒最多 60 MiB         |
|E50     |4,095 GiB       |最多 500         |每秒最多 60 MiB         |
|E60     |8,192 GiB       |最多 1,300       |每秒最多 300 MiB        |
|E70     |16,384 GiB      |最多 2,000       |每秒最多 500 MiB        |
|E80     |32,767 GiB      |最多 2,000       |每秒最多 500 MiB        |

「標準 SSD」已設計成針對大多數 IO 作業均可提供個位數的毫秒延遲，並可提供高達上表所述限制的 IOPS 和輸送量。 實際的 IOPS 和輸送量有時可能會依流量模式而有所不同。 與 HDD 磁碟相比，「標準 SSD」將可提供延遲更低的更一致效能。

另一方面，「進階 SSD」則是效能比「標準 SSD」更佳，不僅提供低延遲、高 IOPS/輸送量，甚至也與所佈建的磁碟效能更一致。 就重要的生產環境工作負載而言，這是建議的磁碟類型。 如果您的工作負載需要高效能、低延遲磁碟支援，您應該考慮使用進階儲存體。

與「進階 SSD」相同，「標準 SSD」使用的「IO 單位」大小也是 256 KiB。 要傳送的資料如果少於 256 KiB，就會視為 1 個 I/O 單位。 更大的 I/O 大小則會視為大小是 256 KiB 的多個 I/O。 例如，一個 1,100 KiB 的 I/O 會視為 5 個 I/O 單位。

## <a name="pricing-and-billing"></a>價格和計費

使用「標準 SSD」時，會有下列計費考量：

- 受控磁碟大小
- 快照集
- 輸出資料傳輸
- 交易

**受控磁碟大小**：受控磁碟會依佈建大小計費。 Azure 會將佈建大小對應 (無條件進位) 至最接近的磁碟大小供應項目。 如需所提供磁碟大小的詳細資料，請參閱上述＜延展性和效能目標＞一節中的表格。 每個磁碟皆與一個支援的佈建磁碟大小對應，並據此計費。 例如，如果您佈建了 200 GiB 的「標準 SSD」，它將會與 E15 (256 GiB) 的磁碟大小供應項目對應。 任何已佈建的磁碟都是使用每月的進階儲存體供應項目價格，以每小時的方式計費。 例如，如果您佈建了 E10 磁碟並在 20 小時後刪除它，便會按 20 小時的比例向您收取 E10 供應項目的費用。 這與寫入磁碟的實際資料量無關。

**快照集**：計算「受控磁碟」的快照集費用時，會針對快照集在目標和來源使用的容量 (如果有的話) 計費。 如需有關快照集的詳細資訊，請參閱[受控磁碟快照集](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview#managed-disk-snapshots)。

**輸出資料傳輸**： [輸出資料傳輸](https://azure.microsoft.com/pricing/details/bandwidth/) (Azure 資料中心送出的資料) 會產生頻寬使用量費用。

**交易**：與「標準 HDD」類似，「標準 SSD」上的交易也會產生費用。 交易同時包含磁碟上的讀取和寫入作業。 「標準 SSD」上的交易所使用的會計 I/O 單位大小為 256 KiB。 更大的 I/O 大小則會視為大小是 256 KiB 的多個 I/O。

如需有關「虛擬機器」和「受控磁碟」的價格詳細資訊，請參閱：

- [虛擬機器定價](https://azure.microsoft.com/pricing/details/virtual-machines/linux/)
- [受控磁碟價格](https://azure.microsoft.com/pricing/details/managed-disks/)

## <a name="next-steps"></a>後續步驟

若要深入了解如何建立標準 SSD，請參考說明如何建立具有多個標準 SSD 之 VM 的範例。

> [!div class="nextstepaction"]
> [建立具有多個標準 SSD 的 VM](https://github.com/azure/azure-quickstart-templates/tree/master/101-vm-with-standardssd-disk/) \(英文\)