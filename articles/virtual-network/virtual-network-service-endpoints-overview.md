---
title: Azure 虛擬網路服務端點 | Microsoft Docs
description: 了解如何使用服務端點，啟用對虛擬網路中的 Azure 資源直接存取。
services: virtual-network
documentationcenter: na
author: sumeetmittal
manager: narayan
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/15/2018
ms.author: sumeet.mittal
ms.custom: ''
ms.openlocfilehash: 77fad7b0035a9ba21d71e6c493a4f1a5bd9a2111
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49395202"
---
# <a name="virtual-network-service-endpoints"></a>虛擬網路服務端點

虛擬網路 (VNet) 服務端點可透過直接連線，將您的虛擬網路私人位址空間和 VNet 的身分識別延伸至 Azure 服務。 端點可讓您將重要的 Azure 服務資源只放到您的虛擬網路保護。 從您的 VNet 到 Azure 服務的流量一定會保留在 Microsoft Azure 骨幹網路上。

這項功能會在下列 Azure 服務和區域中提供：

**正式推出**

- **[Azure 儲存體](../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network)**：已在所有 Azure 區域正式推出。
- **[Azure SQL Database](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)**：已在所有 Azure 區域正式推出。
- **[適用於 PostgreSQL 伺服器的 Azure 資料庫](../postgresql/howto-manage-vnet-using-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)**：在有提供資料庫服務的 Azure 區域中正式推出。
- **[適用於 MySQL 伺服器的 Azure 資料庫](../mysql/howto-manage-vnet-using-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)**：在有提供資料庫服務的 Azure 區域中正式推出。
- **[Azure Cosmos DB](../cosmos-db/vnet-service-endpoint.md?toc=%2fazure%2fvirtual-network%2ftoc.json)**：已在所有 Azure 公用雲端區域正式推出。
- **[Azure Key Vault](https://blogs.technet.microsoft.com/kv/2018/06/25/announcing-virtual-network-service-endpoints-for-key-vault-preview/)** (英文)：已在所有 Azure 公用雲端區域正式推出。

<bpt id="p1">**</bpt>Preview<ept id="p1">**</ept>

- **[Azure SQL 資料倉儲](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)**：在所有 Azure 公用雲端區域以預覽版提供。
- **[Azure 服務匯流排](../service-bus-messaging/service-bus-service-endpoints.md?toc=%2fazure%2fvirtual-network%2ftoc.json)**：在預覽版中提供。
- **[Azure 事件中樞](../event-hubs/event-hubs-service-endpoints.md?toc=%2fazure%2fvirtual-network%2ftoc.json)**：在預覽版中提供。
- **[Azure Data Lake Store Gen 1](../data-lake-store/data-lake-store-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json)**：可供預覽。

如需最新通知，請查看 [Azure 虛擬網路更新](https://azure.microsoft.com/updates/?product=virtual-network)頁面。

## <a name="key-benefits"></a>主要權益

服務端點可提供下列優點：

- **改善您 Azure 服務資源的安全性**：VNet 私人位址空間可以重疊，因此無法只用此項目來識別從您 VNet 輸出的流量。 服務端點藉由將 VNet 身分識別延伸至服務，提供能在虛擬網路中保護 Azure 服務資源的能力。 一旦在您的虛擬網路中啟用服務端點，即可將虛擬網路規則新增至資源，以將 Azure 服務資源放到虛擬網路保護。 這樣可以藉由完全移除這些資源的公用網際網路存取，而且只允許來自您虛擬網路的流量，來改善安全性。
- **來自虛擬網路之 Azure 服務流量的最佳路由**：現在，虛擬網路中強制網際網路流量通過內部部署和 (或) 虛擬設備的任何路由 (也稱為強制通道)，也會強制 Azure 服務流量採用與網際網路流量相同的路由。 服務端點可提供 Azure 流量的最佳路由。 

  端點一律會直接採用從虛擬網路到 Microsoft Azure 骨幹網路上服務的服務流量。 Keeping traffic on the Azure backbone network 可讓您透過強制通道，繼續稽核和監視來自虛擬網路的輸出網際網路流量，而不會影響服務流量。 深入了解[使用者定義的路由和強制通道](virtual-networks-udr-overview.md)。
- **設定簡單且管理額外負荷較小**：虛擬網路中不再需要保留的公用 IP 位址，就可以透過 IP 防火牆保護 Azure 資源。 設定服務端點時不需要 NAT 或閘道裝置。 只要在子網路上按一下，即可設定服務端點。 維護端點沒有額外的負荷。

## <a name="limitations"></a>限制

- 這項功能僅適用於透過 Azure Resource Manager 部署模型所部署的虛擬網路。
- 端點會在 Azure 虛擬網路中設定的子網路上啟用。 無法將端點使用於從內部部署環境到 Azure 服務的流量。 如需詳細資訊，請參閱[保護來自內部部署環境的 Azure 服務存取](#securing-azure-services-to-virtual-networks)。
- 針對 Azure SQL，服務端點只適用於虛擬網路區域內的 Azure 服務流量。 針對 Azure 儲存體，為了支援 RA-GRS 和 GRS 流量，端點也會擴充為包含虛擬網路部署所在的配對區域。 深入了解 [Azure 配對區域](../best-practices-availability-paired-regions.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-paired-regions)。

## <a name="securing-azure-services-to-virtual-networks"></a>將 Azure 服務放到虛擬網路保護

- 虛擬網路服務端點可將您虛擬網路的身分識別提供給 Azure 服務。 一旦在您的虛擬網路中啟用服務端點，即可將虛擬網路規則新增至資源，以將 Azure 服務資源放到虛擬網路保護。
- 現在，來自虛擬網路的 Azure 服務流量會使用公用 IP 位址作為來源 IP 位址。 透過服務端點，服務流量會在從虛擬網路存取 Azure 服務時，切換為使用虛擬網路私人位址作為來源 IP 位址。 此切換讓您不需要 IP 防火牆中使用的保留公用 IP 位址，即可存取服務。

>[!NOTE]
> 使用服務端點，服務流量子網路中虛擬機器的來源 IP 位址會從使用公用 IPv4 位址切換為使用私人 IPv4 位址。 使用 Azure 公用 IP 位址的現有 Azure 服務防火牆規則會停止使用此參數。 請先確定 Azure 服務防火牆規則允許這個參數，再設定服務端點。 設定服務端點時，您也可能遇到來自此子網路的服務流量暫時中斷。 
 
- __保護來自內部部署環境的 Azure 服務存取__：

  根據預設，從內部部署網路無法觸達放到虛擬網路保護的 Azure 服務資源。 如果需要允許來自內部部署的流量，您也必須允許內部部署或 ExpressRoute 中的 NAT IP 位址。 透過 Azure 服務資源的 IP 防火牆設定，可以新增這些 IP 位址。

  ExpressRoute：如果您使用來自內部部署環境的 [ExpressRoute](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 進行公用對等互連或 Microsoft 對等互連，您必須識別所用的 NAT IP 位址。 在公用對等互連中，每個 Expressroute 線路預設都會使用兩個 NAT IP 位址，而這兩個位址會在流量進入 Microsoft Azure 網路骨幹時套用至 Azure 服務流量。 在 Microsoft 對等互連中，所用的 NAT IP 位址是由客戶提供或由服務提供者提供。 若要允許存取您的服務資源，就必須在資源 IP 防火牆設定中允許這些公用 IP 位址。 若要尋找您的公用對等互連 ExpressRoute 線路 IP 位址，請透過 Azure 入口網站[開啟有 ExpressRoute 的支援票證](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)。 深入了解 [ExpressRoute 公用與 Microsoft 對等互連的 NAT。](../expressroute/expressroute-nat.md?toc=%2fazure%2fvirtual-network%2ftoc.json#nat-requirements-for-azure-public-peering)

![將 Azure 服務放到虛擬網路保護](./media/virtual-network-service-endpoints-overview/VNet_Service_Endpoints_Overview.png)

### <a name="configuration"></a>組態

- 服務端點設定於虛擬網路的子網路上。 端點會使用在子網路內執行之任何類型的計算執行個體。
- 您可以對子網路上所有支援的 Azure 服務 (例如 Azure 儲存體或 Azure SQL Database) 設定多個服務端點。
- 針對 Azure SQL Database，虛擬網路必須位於與 Azure 服務資源相同的區域中。 如果使用 GRS 和 RA-GRS Azure 儲存體帳戶，則主要帳戶必須位在與虛擬網路相同的區域中。 針對其他所有服務，Azure 服務資源可以安全地放置在任何區域中的虛擬網路。 
- 端點設定所在的虛擬網路可以位於與 Azure 服務資源相同或不同的訂用帳戶中。 如需設定端點和保護 Azure 服務所需權限的詳細資訊，請參閱[佈建](#Provisioning)。
- 對於支援的服務，您可以使用服務端點，將新的或現有資源放到虛擬網路保護。

### <a name="considerations"></a>考量

- 啟用服務端點之後，子網路中虛擬機器的來源 IP 位址會在與該子網路中的服務進行通訊時，從使用公用 IPv4 位址切換為使用其私人 IPv4 位址。 在此切換期間，會關閉該服務的任何現有開放 TCP 連線。 在啟用或停用子網路服務的服務端點時，確定沒有任何重要工作正在執行中。 此外，確保在 IP 位址切換之後，您的應用程式可以自動連線到 Azure 服務。

  IP 位址切換只會影響來自您虛擬網路的服務流量。 不會影響任何從指派給虛擬機器的公用 IPV4 位址傳送或傳送至其中的其他流量。 對於 Azure 服務，如果您有使用 Azure 公用 IP 位址的現有防火牆規則，則這些規則會停止使用切換至虛擬網路私人位址。
- 透過服務端點，Azure 服務的 DNS 項目會保留現狀，並繼續解析為指派給 Azure 服務的公用 IP 位址。

- 具有服務端點的網路安全性群組 (NSG)：
  - 根據預設，NSG 允許輸出網際網路流量，因此，也可允許從您的 VNet 至 Azure 服務的流量。 透過服務端點，這會繼續照常運作。 
  - 如果您要拒絕所有輸出網際網路流量，只允許對特定 Azure 服務的流量，您可以使用 NSG 中的 [服務標籤](security-overview.md#service-tags)。 您可以將支援的 Azure 服務指定為 NSG 規則中的目的地，而作為每個標籤基礎之 IP 位址的維護則是由 Azure 提供。 如需詳細資訊，請參閱 [Azure 服務標籤](security-overview.md#service-tags)。 

### <a name="scenarios"></a>案例

- **對等互連、已連線或多個虛擬網路**：若要將 Azure 服務放到一個虛擬網路或多個虛擬網路內的多個子網路保護，您可以獨立啟用每個子網路上的服務端點，並將 Azure 服務資源放到所有子網路保護。
- **Filtering outbound traffic from a virtual network to Azure services**：如果您想檢查或篩選從虛擬網路送到 Azure 服務的流量，則可以在虛擬網路內部署網路虛擬設備。 接著，可以將服務端點套用到網路虛擬設備部署所在的子網路，只將 Azure 服務資源放到此子網路保護。 如果您想要使用網路虛擬設備篩選，讓來自虛擬網路的 Azure 服務存取只限於特定 Azure 資源，則此案例可能有幫助。 如需詳細資訊，請參閱[使用網路虛擬設備輸出](/azure/architecture/reference-architectures/dmz/nva-ha#egress-with-layer-7-nvas.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
- **將 Azure 資源放到直接部署至虛擬網路的服務保護**：可以將各種 Azure 資源直接部署至虛擬網路中的特定子網路。 在受控服務子網路上設定服務端點，即可將 Azure 服務資源放到[受控服務](virtual-network-for-azure-services.md)子網路保護。
- **來自 Azure 虛擬機器的磁碟流量**：對受控/非受控磁碟而言，虛擬機器磁碟流量 (包括掛接和卸載、diskIO) 不會因為 Azure 儲存體的服務端點路徑變更而受到影響。 您可以透過服務端點和 [Azure 儲存體網路規則](../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，將分頁 Blob 的 REST 存取限制為選取網路。 

### <a name="logging-and-troubleshooting"></a>記錄和疑難排解

將服務端點設定為特定服務後，請驗證服務端點路由是否有作用，方法如下： 
 
- 驗證服務診斷中任何服務要求的來源 IP 位址。 所有透過服務端點的新要求都會將要求的來源 IP 位址顯示為虛擬網路私人 IP 位址 (已指派給從您的虛擬網路提出要求的用戶端)。 未透過端點，此位址是 Azure 公用 IP 位址。
- 檢視子網路的任何網路介面上的有效路由。 服務的路由：
  - 顯示每個服務之位址前置詞範圍的更特定預設路由
  - 的 nextHopType 為 *VirtualNetworkServiceEndpoint*
  - 指出服務的更直接連線已作用 (相較於任何強制通道路由)

>[!NOTE]
> 服務端點路由會覆寫 Azure 服務之位址前置詞相符項目的任何 BGP 或 UDR 路由。 深入了解[有效路由的疑難排解](diagnose-network-routing-problem.md)

## <a name="provisioning"></a>佈建

擁有虛擬網路寫入權的使用者可以任意地在虛擬網路上設定服務端點。 若要將 Azure 服務資源放到 VNet 保護，使用者必須擁有所要新增之子網路的 *Microsoft.Network/JoinServicetoaSubnet* 權限。 此權限預設會隨附在內建的服務管理員角色中，可藉由建立自訂角色加以修改。

深入了解[內建角色](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)以及如何將特定權限指派給[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

虛擬網路和 Azure 服務資源可以位在相同或不同的訂用帳戶中。 如果虛擬網路和 Azure 服務資源位在不同的訂用帳戶中，資源必須位在相同的 Active Directory (AD) 租用戶下。 

## <a name="pricing-and-limits"></a>價格和限制

使用服務端點不會額外收費。 Azure 服務 (Azure 儲存體、Azure SQL Database 等等) 的目前定價模型照常適用。

虛擬網路中的服務端點總數沒有限制。

對於 Azure 服務資源 (例如，Azure 儲存體帳戶)，服務可能會強制執行用來保護資源的子網路數目限制。 如需詳細資料，請參閱[後續步驟](#next-steps)中各種服務的文件。

## <a name="next-steps"></a>後續步驟

- 了解如何[設定虛擬網路服務端點](tutorial-restrict-network-access-to-resources.md)
- 了解如何[將 Azure 儲存體帳戶放到虛擬網路保護](../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- 了解如何[將 Azure SQL Database 放到虛擬網路保護](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- 了解[虛擬網路中的 Azure 服務整合](virtual-network-for-azure-services.md)
-  快速入門：[Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/201-vnet-2subnets-service-endpoints-storage-integration)可設定 VNet 子網路上的服務端點，並保護該子網路的 Azure 儲存體帳戶。

