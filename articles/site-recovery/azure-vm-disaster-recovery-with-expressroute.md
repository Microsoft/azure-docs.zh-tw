---
title: 使用 ExpressRoute 來搭配 Azure 虛擬機器災害復原 | Microsoft Docs
description: 說明如何使用 Azure ExpressRoute 來搭配 Azure Site Recovery 以進行 Azure 虛擬機器災害復原
services: site-recovery
documentationcenter: ''
author: mayanknayar
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: manayar
ms.openlocfilehash: 73514b524f554affb9730ba63ccd608491497af2
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37920465"
---
# <a name="using-expressroute-with-azure-virtual-machine-disaster-recovery"></a>使用 ExpressRoute 來搭配 Azure 虛擬機器災害復原

Microsoft Azure ExpressRoute 可讓您透過連線提供者所提供的私人連線，將內部部署網路延伸至 Microsoft 雲端。 本文說明如何使用 ExpressRoute 來搭配 Site Recovery 以進行 Azure 虛擬機器的災害復原。

## <a name="prerequisites"></a>先決條件

開始之前，請確定您了解：
-   ExpressRoute [線路](../expressroute/expressroute-circuit-peerings.md)
-   ExpressRoute [路由網域](../expressroute/expressroute-circuit-peerings.md#expressroute-routing-domains)
-   Azure 虛擬機器[複寫架構](azure-to-azure-architecture.md)
-   針對 Azure 虛擬機器[設定複寫](azure-to-azure-tutorial-enable-replication.md)
-   [容錯移轉](azure-to-azure-tutorial-failover-failback.md) Azure 虛擬機器

## <a name="expressroute-and-azure-virtual-machine-replication"></a>ExpressRoute 和 Azure 虛擬機器複寫

使用 Site recovery 保護 Azure 虛擬機器時，視您的 Azure 虛擬機器是否使用 [Azure 受控磁碟](../virtual-machines/windows/managed-disks-overview.md)而定，複寫資料會被傳送到 Azure 儲存體帳戶或目標 Azure 區域上的複本受控磁碟。 儘管複寫端點是公用的，但不論來源虛擬網路存在於哪一個 Azure 區域，Azure VM 複寫的複寫流量預設都不會周遊網際網路。

針對 Azure VM 災害復原，由於複寫資料不會離開 Azure 界限，因此，不需要 ExpressRoute 就能進行複寫。 虛擬機器容錯移轉到目標 Azure 區域之後，您就可以使用[私用對等互連](../expressroute/expressroute-circuit-peerings.md#azure-private-peering)存取它們。

## <a name="replicating-azure-deployments"></a>複寫 Azure 部署

稍早的[文章](site-recovery-retain-ip-azure-vm-failover.md#on-premises-to-azure-connectivity)說明了一個簡單的設定過程，其中有一個透過 ExpressRoute 連線到客戶內部部署資料中心的 Azure 虛擬網路。 典型的企業部署會將工作負載分散到多個 Azure 虛擬網路，而且有一個中央連線中樞會建立外部連線，這會同時連至網際網路和內部部署。

這個範例說明企業部署中很常見的中樞和輪輻拓撲：
-   此部署位於 **Azure 東亞**區域，而內部部署資料中心有經由香港 Partner Edge 的 ExpressRoute 線路連線。
-   應用程式會跨兩個輪輻虛擬網路加以部署：位址空間為 10.1.0.0/24 的**來源 VNet1**，以及位址空間為 10.2.0.0/24 的**來源 VNet2**。
-   位址空間為 10.10.10.0/24 的中樞虛擬網路**來源中樞 VNet** 會用來作為閘道管理員。 所有跨子網路的通訊都會通過中樞。
-   中樞虛擬網路具有兩個子網路：位址空間為 10.10.10.0/25 的 **NVA 子網路**，以及位址空間為 10.10.10.128/25 的**閘道子網路**。
-   **NVA 子網路**具有 IP 位址為 10.10.10.10 的網路虛擬設備。
-   **閘道子網路**具備已連線到 ExpressRoute 連線的 ExpressRoute 閘道，其會透過私用對等互連路由網域路由傳送到客戶的內部部署資料中心。
-   每個輪輻虛擬網路都會連線到中樞虛擬網路，而且此網路拓樸內的所有路由都會透過 Azure 路由表 (UDR) 來控制。 從某一個 VNet 到另一個 VNet，或者到內部部署資料中心的所有連出流量都會透過 NVA 進行路由傳送。

![容錯移轉之前使用 ExpressRoute 的內部部署到 Azure](./media/azure-vm-disaster-recovery-with-expressroute/site-recovery-with-expressroute-before-failover.png)

### <a name="hub-and-spoke-peering"></a>中樞與輪輻對等互連

輪輻到中樞對等互連具有下列設定：
-   允許虛擬網路位址：已啟用
-   允許轉寄的流量：已啟用
-   允許閘道器傳輸：已停用
-   使用移除閘道器：已啟用

 ![輪輻到中樞對等互連設定](./media/azure-vm-disaster-recovery-with-expressroute/spoke-to-hub-peering-configuration.png)

中樞到輪輻對等互連具有下列設定：
-   允許虛擬網路位址：已啟用
-   允許轉寄的流量：已啟用
-   允許閘道器傳輸：已啟用
-   使用移除閘道器：已停用

 ![中樞到輪輻對等互連設定](./media/azure-vm-disaster-recovery-with-expressroute/hub-to-spoke-peering-configuration.png)

### <a name="enabling-replication-for-the-deployment"></a>針對部署啟用複寫

針對上述設定，先使用 Site Recovery 針對每部虛擬機器[設定災害復原](azure-to-azure-tutorial-enable-replication.md)。 Site Recovery 可以在目標區域上建立複本虛擬網路 (包括子網路和閘道子網路)，並在來源和目標虛擬網路之間建立必要的對應。 您也可以預先建立目標端的網路和子網路，並且在啟用複寫時使用相同的網路和子網路。

Site Recovery 不會複寫路由表、虛擬網路閘道、虛擬網路閘道連線、虛擬網路對等互連，或者任何其他網路資源或連線。 您必須在容錯移轉期間或之前建立這些項目和不屬於[複寫程序](azure-to-azure-architecture.md#replication-process)之一部分的其他資源，並連線到相關資源。 您可以使用 Azure Site Recovery 的強大[復原方案](site-recovery-create-recovery-plans.md)，使用自動化指令碼來自動建立並連線其他資源。

根據預設，複寫流量不會離開 Azure 界限。 通常，NVA 部署也會定義預設路由 (0.0.0.0/0)，強制連出網際網路流量流經 NVA。 在此情況下，如果所有複寫流量都會通過 NVA，可能就會對設備進行節流。 這同樣也適用於使用預設路由，將所有 Azure VM 流量路由傳送到內部部署時。 我們建議在您的虛擬網路中為「儲存體」[建立虛擬網路服務端點](azure-to-azure-about-networking.md#create-network-service-endpoint-for-storage)，這樣一來，複寫流量就不會離開 Azure 界限。

## <a name="failover-models-with-expressroute"></a>透過 ExpressRoute 的容錯移轉模型

Azure 虛擬機器容錯移轉到不同區域時，連至來源虛擬網路的現有 ExpressRoute 連線不會自動轉移到復原區域上的目標虛擬網路。 需要有新的連線，才能將 ExpressRoute 連線到目標虛擬網路。

您可以將 Azure 虛擬機器複寫到相同地理叢集內的任何 Azure 區域，如[這裡](azure-to-azure-support-matrix.md#region-support)所詳述。 如果選擇的目標 Azure 區域與來源位於不同的地緣政治區域內，若您針對來源和目標區域連線使用單一 ExpressRoute 線路，則需要啟用 ExpressRoute Premium。 如需詳細資訊，請參閱 [ExpressRoute 位置](../expressroute/expressroute-locations.md#azure-regions-to-expressroute-locations-within-a-geopolitical-region)和 [ExpressRoute 價格](https://azure.microsoft.com/pricing/details/expressroute/)。

### <a name="two-expressroute-circuits-in-two-different-expressroute-peering-locations"></a>位於兩個不同 ExpressRoute 對等互連位置的兩個 ExpressRoute 線路
-   如果您想要確保不會發生主要 ExpressRoute 線路失敗以及大規模的區域性災害，此設定很有用，其可能也會影響 ExpressRoute 對等互連位置並中斷您的主要 ExpressRoute 線路。
-   通常會使用連線到生產環境的線路作為主要線路，並使用次要線路來防止故障，通常適用於較低頻寬。 當次要線路必須接替主要線路的工作時，可在災害事件中提高次要線路的頻寬。
-   您可以使用此設定，建立從次要 ExpressRoute 線路到容錯移轉後目標虛擬網路的連線，或者建立連線並準備好進行災害宣告，進而減少整體復原時間。 透過同時連線到主要和目標區域虛擬網路，確保您的內部部署路由只有在容錯移轉之後才會使用次要線路和連線。
-   適用於使用 Site Recovery 保護之 VM 的來源和目標虛擬網路，在容錯移轉之後，可根據您的需求而擁有相同或不同的 IP 位址。 在這兩種情況下，可以在容錯移轉之前建立次要連線。

### <a name="two-expressroute-circuits-in-the-same-expressroute-peering-location"></a>位於同一個 ExpressRoute 對等互連位置的兩個 ExpressRoute 線路
-   使用此設定，您可以確保不會發生主要 ExpressRoute 線路失敗，但無法確保不會發生大規模的區域性災害，其可能也會影響 ExpressRoute 對等互連位置。 在後者的情況下，主要和次要線路都會受到影響。
-   對於 IP 位址和連線的其他條件則與先前案例維持一樣。 您可以使用主要線路同時從內部部署資料中心連線到來源虛擬網路，並使用次要線路連線到目標虛擬網路。 透過同時連線到主要和目標區域虛擬網路，確保您的內部部署路由只有在容錯移轉之後才會使用次要線路和連線。
-   若這兩個線路是在相同的對等互連位置建立，則您無法將這兩個線路連線到相同的虛擬網路。

### <a name="single-expressroute-circuit"></a>單一 ExpressRoute 線路
-   此設定無法確保不會發生大規模的區域性災害，其可能會影響 ExpressRoute 對等互連位置。
-   使用單一 ExpressRoute 線路時，如果在目標區域上使用相同的 IP 位址空間，您無法同時將來源和目標虛擬網路連線到線路。
-   在目標區域上使用相同的 IP 位址空間時，就應該將來源端連線中斷連線，之後會建立目標端連線。 您可以將此連線變更編寫為指令碼以作為復原方案的一部分。
-   發生區域性失敗時，如果無法存取主要區域，則中斷連線作業可能會失敗。 在目標虛擬網路上使用相同的 IP 位址空間時，這類中斷可能會影響對目標區域的連線建立。
-   如果在目標上順利建立連線且主要區域稍後復原，若兩個並行連線嘗試連線到相同的位址空間，則您可能面臨封包捨棄情況。 為了防止封包捨棄，應該立即終止主要連線。 將虛擬機器容錯回復到主要區域之後，將次要連線中斷連線之後，便可以再次建立主要連線。
-   如果在目標虛擬網路上使用不同的位址空間，則您可以從相同的 ExpressRoute 線路同時連線到來源和目標虛擬網路。

## <a name="recovering-azure-deployments"></a>復原 Azure 部署
假設容錯移轉模型具有兩個位於兩個不同對等互連位置的不同 ExpressRoute 線路，並會針對受保護的 Azure 虛擬機器保留私人 IP 位址。 目標復原區域是 Azure 東南亞，而次要 ExpressRoute 線路連線會透過位於新加坡的 Partner Edge 來建立。

若要將整個部署的復原自動化，除了複寫虛擬機器和虛擬網路之外，也必須建立其他相關的網路資源和連線。 針對先前的中樞和輪輻拓樸，需要在[容錯移轉](azure-to-azure-tutorial-failover-failback.md)作業期間或之後採取下列額外步驟：
1.  在目標區域中樞虛擬網路中建立 Azure ExpressRoute 閘道。 需要 ExpressRoute 閘道，才能將目標中樞虛擬網路連線到 ExpressRoute 線路。
2.  建立從目標中樞虛擬網路到目標 ExpressRoute 線路的虛擬網路連線。
3.  在目標區域的中樞和輪輻虛擬網路之間設定 VNet 對等互連。 目標區域上的對等互連屬性將會與來源區域上的相同。
4.  在中樞 VNet 中設定 UDR 和兩個輪輻 VNet。 使用相同的 IP 位址時，目標端 UDR 的屬性會與來源端上的一樣。 若要使用不同的目標 IP 位址，應該據以修改 UDR。

您可以將上述步驟編寫為指令碼以作為[復原方案](site-recovery-create-recovery-plans.md)的一部分。 視應用程式連線能力和復原時間需求而定，上述步驟也可以在開始容錯移轉之前完成。

復原虛擬機器並完成其他連線步驟之後，復原環境看起來如下：![容錯移轉之後使用 ExpressRoute 的內部部署到 Azure](./media/azure-vm-disaster-recovery-with-expressroute/site-recovery-with-expressroute-after-failover.png)

[這裡](site-recovery-retain-ip-azure-vm-failover.md#on-premises-to-azure-connectivity)詳細說明使用單一 ExpressRoute 線路進行 Azure VM 災害復原的簡單拓撲範例，其會在目標虛擬機器上使用相同 IP。

## <a name="recovery-time-objective-rto-considerations"></a>復原時間目標 (RTO) 考量
為了減少部署的整體復原時間，我們建議事先佈建及部署額外目標區域[網路元件](azure-vm-disaster-recovery-with-expressroute.md#enabling-replication-for-the-deployment)，例如虛擬網路閘道。 部署額外資源時會有短暫的停機時間，而且如果在規劃期間未考量到這個停機時間，則其可能會影響整體復原時間。

我們建議針對受保護的部署執行一般[災害復原演練](azure-to-azure-tutorial-dr-drill.md)。 此演練會在不會遺失資料也不需要停機的情況下驗證您的複寫策略，而且不會影響您的生產環境。 執行演練也可以避免在最後一刻發生設定問題，而對復原時間目標造成負面影響。 建議針對測試容錯移轉使用個別的 Azure VM 網路，而不是在您啟用複寫時所設定的預設網路。

如果您使用單一 ExpressRoute 線路，則建議針對目標虛擬網路使用不同的 IP 位址空間，以避免在發生區域性災害期間產生連線建立問題。 如果使用不同的 IP 位址不適用於您復原的生產環境，則應該在具有不同 IP 位址的個別測試網路上完成災害復原演練測試容錯移轉，因為您無法將兩個具有重疊 IP 位址空間的虛擬網路連線到相同 ExpressRoute 線路。

## <a name="next-steps"></a>後續步驟
- 深入了解 [ExpressRoute 線路](../expressroute/expressroute-circuit-peerings.md)。
- 深入了解 [ExpressRoute 路由網域](../expressroute/expressroute-circuit-peerings.md#expressroute-routing-domains)。
- 深入了解 [ExpressRoute 位置](../expressroute/expressroute-locations.md)。
- 深入了解自動執行應用程式容錯移轉的[復原計劃](site-recovery-create-recovery-plans.md)。
