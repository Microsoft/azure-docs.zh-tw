---
title: 將 Azure 虛擬機器容錯移轉至另一個 Azure 區域時保留 IP 位址 | Microsoft Docs
description: 說明如何在使用 Azure Site Recovery 進行 Azure 至 Azure 容錯移轉案例時保留 IP 位址
ms.service: site-recovery
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/16/2018
author: mayurigupta13
ms.topic: conceptual
ms.author: mayg
ms.openlocfilehash: 4260a4487480c42b518374965d8deda1e7803e62
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353317"
---
# <a name="ip-address-retention-for-azure-virtual-machine-failover"></a>針對 Azure 虛擬機器容錯移轉保留 IP 位址

Azure Site Recovery 能針對 Azure VM 啟用災害復原。 從一個 Azure 區域容錯移轉至另一個區域時，客戶通常需要保留其 IP 設定。 在目標區域上建立這些資源時，Site Recovery 預設會模擬來源的虛擬網路和子網路結構。 針對以靜態私人 IP 位址設定的 Azure VM，Site Recovery 也會盡力嘗試在目標 VM 上佈建相同的私人 IP，前提是該 IP 未被 Azure 資源或已複寫的 VM 封鎖。

針對簡單的應用程式，只需上述預設設定即可。 針對更為複雜的企業應用程式，客戶可能需要佈建其他網路資源，以確保在容錯移轉後能與其基礎結構的其他元件進行連線。 本文說明將 Azure VM 從一個區域容錯移轉至另一個區域，同時保留 VM IP 位址的網路需求。

## <a name="azure-to-azure-connectivity"></a>Azure 對 Azure 連線

針對第一個案例，我們假設**公司 A** 的所有應用程式基礎結構都是在 Azure 中執行。 基於商務持續性和合規性因素，**公司 A** 決定使用 Azure Site Recovery 來保護其應用程式。

基於 IP 保留的需求 (例如針對應用程式繫結)，公司 A 在目標區域上具有相同的虛擬網路和子網路結構。 為了進一步降低復原時間目標 (RTO)，**公司 A** 會運用針對 SQL Always ON、網域控制站等的複本節點，而這些節點都放置於目標區域上不同的虛擬網路中。 透過針對複本節點使用不同的位址空間，使得**公司 A** 能夠在來源和目標區域之間建立 VPN 站對站的連線能力；若不這麼做，就無法在兩端使用相同位址空間。

以下是網路架構在容錯移轉前的外觀：
- 應用程式 VM 裝載於 Azure 東亞，利用具有位址空間 10.1.0.0/16 的 Azure 虛擬網路。 此虛擬網路稱為 **Source VNet**。
- 應用程式工作負載會分散到三個子網路：10.1.1.0/24、10.1.2.0/24、10.1.3.0/24，分別名為 **Subnet 1**、**Subnet 2**、**Subnet 3**。
- Azure 東南亞為目標區域，並具有可模擬來源上位址空間和子網路設定的復原虛擬網路。 此虛擬網路稱為 **Recovery VNet**。
- 複本節點 (例如針對 Always On、網域控制站等所需的節點) 會放置於位址空間為 10.2.0.0/16 的虛擬網路中，位址為 10.2.4.0/24 的 Subnet 4 內。 此虛擬網路稱為 **Azure VNet**，且位於 Azure 東南亞。
- **Source VNet** 和 **Azure VNet** 是透過 VPN 站對站連線能力來連線。
- **Recovery VNet** 並未與任何其他虛擬網路連線。
- **公司 A** 會針已對複寫的項目指派/驗證目標 IP 位址。 針對此範例，每個 VM 的目標 IP 皆等同於來源 IP。

![容錯移轉前的 Azure 對 Azure 連線能力](./media/site-recovery-retain-ip-azure-vm-failover/azure-to-azure-connectivity-before-failover2.png)

### <a name="full-region-failover"></a>完整區域容錯移轉

發生區域中斷時，**公司 A** 可以使用 Azure Site Recovery 功能強大的[復原方案](site-recovery-create-recovery-plans.md)，快速且輕鬆地復原整個部署。 由於**公司 A**已經在容錯移轉之前針對每個 VM 設定目標 IP 位址，他們可以在 Recovery VNet 與 Azure Vnet 之間協調容錯移轉並自動化連線的建立，如下圖所示。

![Azure 對 Azure 連線能力完整區域容錯移轉](./media/site-recovery-retain-ip-azure-vm-failover/azure-to-azure-connectivity-full-region-failover2.png)

根據應用程式需求，可在容錯移轉之前、期間 (作為中繼步驟) 或之後於目標區域上建立兩個 VNet 之間的連線。 使用[復原方案](site-recovery-create-recovery-plans.md)來新增指令碼，並定義容錯移轉順序。

公司 A 也可以選擇使用 VNet 對等互連或站對站 VPN，以在 Recovery VNet 和 Azure VNet 之間建立連線能力。 VNet 對等互連不會使用 VPN 閘道，且具有不同的條件約束。 此外，[VNet 對等互連價格](https://azure.microsoft.com/pricing/details/virtual-network)與 [VNet 對 VNet VPN 閘道價格](https://azure.microsoft.com/pricing/details/vpn-gateway)的計算方式不同。 針對容錯移轉，通常建議模擬來源連線能力 (包括連線類型)，以將網路變更所造成之無法預期的事件降至最少。

### <a name="isolated-application-failover"></a>隔離的應用程式容錯移轉

在某些情況下，使用者可能需要對其應用程式基礎結構的某些部分進行容錯移轉。 其中一個例子為針對位於專用子網路內的特定應用程式或階層進行容錯移轉。 雖然可以搭配 IP 保留進行子網路容錯移轉，但在大多數情況下均不建議這麼做，因為它會大幅增加連線能力的不一致性。 您也會遺失針對位於相同 Azure 虛擬網路內其他子網路的子網路連線能力。

處理子網路層級應用程式容錯移轉需求的更好方式，是使用不同的目標 IP 位址進行容錯移轉 (如果需要與來源虛擬網路上的其他子網路連線)，或是將每個應用程式隔離於其位於來源上的專屬虛擬網路中。 使用第二種方法時，您可以在來源上建立網際網路連線能力，並在容錯移轉至目標區域時模擬相同情況。

若要建構個別應用程式以提供恢復功能，建議先將應用程式置於其專屬的虛擬網路中，並視需要在這些虛擬網路之間建立連線能力。 這允許進行隔離的應用程式容錯移轉，同時保留原始的私人 IP 位址。

容錯移轉前設定的外觀將如下所示：
- 應用程式 VM 裝載於 Azure 東亞，會針對第一個應用程式運用位址空間為 10.1.0.0/16 的 Azure 虛擬網路，針對第二個應用程式則是搭配 10.2.0.0/16。 虛擬網路針對第一個和第二個應用程式分別名為 **Source VNet1** 和 **Source VNet2**。
- 每個 VNet 都會再進一步分成兩個子網路。
- Azure 東南亞為目標區域，且具有復原虛擬網路 Recovery VNet1 和 Recovery VNet2。
- 複本節點 (例如針對 Always On、網域控制站等所需的節點) 會放置於位址空間為 10.3.0.0/16 的虛擬網路中，位址為 10.3.4.0/24 的 **Subnet 4** 內。 此虛擬網路稱為 Azure VNet，且位於 Azure 東南亞。
- **Source VNet1** 和 **Azure VNet** 是透過 VPN 站對站連線能力來連線。 同樣地，**Source VNet2** 和 **Azure VNet** 也是透過 VPN 站對站連線能力來連線。
- 在此範例中，**Source VNet1** 和 **Source VNet2** 也是過 S2S VPN 來連線。 由於這兩個 VNet 位於相同區域，因此也可以不使用 S2S VPN 而改用 VNet 對等互連。
- **Recovery VNet1** 和 **Recovery VNet2** 並未與任何其他虛擬網路連線。
- 為了降低復原時間目標 (RTO)，會在容錯移轉之前在 **Recovery VNet1** 和 **Recovery VNet2** 上設定 VPN 閘道。

![容錯移轉前的 Azure 對 Azure 連線能力隔離應用程式](./media/site-recovery-retain-ip-azure-vm-failover/azure-to-azure-connectivity-isolated-application-before-failover2.png)

在發生只會影響單一應用程式 (在此範例中是位於 Source VNet2) 的災害狀況下，公司 A 可以依下列方式復原受影響的應用程式：
- 中斷 **Source VNet1** 和 **Source VNet2** 之間的 VPN 連線，以及 **Source VNet2** 和 **Azure VNet** 之間的 VPN 連線。
- 於 **Source VNet1** 和 **Recovery VNet2** 之間，以及 **Recovery VNet2** 和 **Azure VNet** 之間建立 VPN 連線。
- 來自 **Source VNet2** 的 VM 會容轉移轉至 **Recovery VNet2**。

![容錯移轉後的 Azure 對 Azure 連線能力隔離應用程式](./media/site-recovery-retain-ip-azure-vm-failover/azure-to-azure-connectivity-isolated-application-after-failover2.png)

您可以擴充上述隔離容錯移轉範例，以包含更多應用程式和網路連線。 從來源容錯移轉至目標時，建議盡可能遵循群聚連線模型。

### <a name="further-considerations"></a>進一步考量

VPN 閘道會利用公用 IP 位址與閘道躍點來建立連線。 如果您不想使用公用 IP 及/或想要避免額外的躍點，則可以使用 [Azure 虛擬網路對等互連](../virtual-network/virtual-network-peering-overview.md)，跨[支援的 Azure 區域](../virtual-network/virtual-network-manage-peering.md#cross-region)針對虛擬網路進行對等互連。

## <a name="on-premises-to-azure-connectivity"></a>內部部署對 Azure 連線能力

針對第二個案例，我們假設**公司 B** 有一部分的應用程式基礎架構是在 Azure 上執行，其餘則是在內部部署中執行。 基於商務持續性和合規性因素，**公司 B** 決定使用 Azure Site Recovery 來保護其在 Azure 中執行的應用程式。

以下是網路架構在容錯移轉前的外觀：
- 應用程式 VM 裝載於 Azure 東亞，利用具有位址空間 10.1.0.0/16 的 Azure 虛擬網路。 此虛擬網路稱為 **Source VNet**。
- 應用程式工作負載會分散到三個子網路：10.1.1.0/24、10.1.2.0/24、10.1.3.0/24，分別名為 **Subnet 1**、**Subnet 2**、**Subnet 3**。
- Azure 東南亞為目標區域，並具有可模擬來源上位址空間和子網路設定的復原虛擬網路。 此虛擬網路稱為 **Recovery VNet**。
- 位於 Azure 東亞的 VM 會透過 ExpressRoute 或站對站 VPN 連線到內部部署資料中心。
- 為了降低復原時間目標 (RTO)，公司 B 在容錯移轉之前於 Azure 東南亞的復原 VNet 上佈建閘道。
- **公司 B** 會針對已複寫的項目指派/驗證目標 IP 位址。 針對此範例，每個 VM 的目標 IP 皆等同於來源 IP

![容錯移轉前的內部部署對 Azure 連線能力](./media/site-recovery-retain-ip-azure-vm-failover/on-premises-to-azure-connectivity-before-failover2.png)

### <a name="full-region-failover"></a>完整區域容錯移轉

發生區域中斷時，**公司 B** 可以使用 Azure Site Recovery 功能強大的[復原方案](site-recovery-create-recovery-plans.md)，快速且輕鬆地復原整個部署。 由於**公司 B** 已經在容錯移轉之前針對每個 VM 設定目標 IP 位址，他們可以在 Recovery VNet 與內部部署資料中心之間協調容錯移轉並自動化連線的建立，如下圖所示。

介於 Azure 東亞與內部部署資料中心之間的原始連線，應該要在建立 Azure 東南亞與內部部署資料中心之間的連線前先中斷連線。 內部部署路由也會在容錯移轉之後，重新設定為指向目標區域和閘道。

![容錯移轉後的內部部署對 Azure 連線能力](./media/site-recovery-retain-ip-azure-vm-failover/on-premises-to-azure-connectivity-after-failover2.png)

### <a name="subnet-failover"></a>子網路容錯移轉

不同於針對**公司 A** 所述的 Azure 對 Azure 案例，在此情況中，**公司 B** 並不適用子網路層級的容錯移轉。這是因為來源和復原虛擬網路上的位址空間是一樣的，且原始來源對內部部署的連線是處於作用中狀態。

若要實現應用程式恢復功能，建議將每個應用程式置於其專屬的 Azure 虛擬網路中。 應用程式接著能以隔離方式進行容錯移轉，並且可將所需的內部部署對來源連線路由傳送至目標區域，如上所述。

## <a name="next-steps"></a>後續步驟
- 深入了解[復原方案](site-recovery-create-recovery-plans.md)。
