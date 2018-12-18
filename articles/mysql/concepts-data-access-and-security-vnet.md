---
title: 適用於 MySQL 的 Azure 資料庫伺服器 VNet 服務端點概觀 | Microsoft Docs
description: 說明 VNet 服務端點如何針對適用於 MySQL 的 Azure 資料庫伺服器運作。
services: mysql
author: mbolz
ms.author: mbolz
manager: jhubbard
editor: jasonwhowell
ms.service: mysql
ms.topic: conceptual
ms.date: 08/20/2018
ms.openlocfilehash: f18f52fc409df769d164607a128caaf02ead5e4b
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42145352"
---
# <a name="use-virtual-network-service-endpoints-and-rules-for-azure-database-for-mysql"></a>針對適用於 MySQL 的 Azure 資料庫使用虛擬網路服務端點和規則

*虛擬網路規則*是一項防火牆安全性功能，可控制適用於 MySQL 的 Azure 資料庫伺服器是否接受從虛擬網路中特定子網路所傳來的通訊。 本文說明為何虛擬網路規則功能有時是讓適用於 MySQL 的 Azure 資料庫伺服器安全接受通訊的最佳選項。

若要建立虛擬網路規則，必須先有[虛擬網路][vm-virtual-network-overview] (VNet) 和[虛擬網路服務端點][vm-virtual-network-service-endpoints-overview-649d]，以供規則參考。 下圖說明虛擬網路服務端點如何與適用於 MySQL 的 Azure 資料庫搭配運作：

![VNet 服務端點的運作方式範例](media/concepts-data-access-and-security-vnet/vnet-concept.png)

> [!NOTE]
> 如果適用於 MySQL 的 Azure 資料庫是針對「一般用途」和「記憶體最佳化」伺服器來部署的，則此功能可在所有 Azure 區域中使用。

<a name="anch-terminology-and-description-82f" />

## <a name="terminology-and-description"></a>術語和描述

**虛擬網路：** Azure 訂用帳戶可以有相關聯的虛擬網路。

**子網路：** 虛擬網路包含**子網路**。 您有的任何 Azure 虛擬機器 (VM) 會指派給子網路。 一個子網路可以包含多個 VM 或其他計算節點。 計算虛擬網路外部的節點無法存取虛擬網路，除非您設定安全性來允許存取。

**虛擬網路服務端點：**[虛擬網路服務端點][vm-virtual-network-service-endpoints-overview-649d]是一個子網路，其屬性值包含一或多個正式的 Azure 服務類型名稱。 本文中我們探討類型名稱 **Microsoft.Sql**，它參考名為 SQL Database 的 Azure 服務。 此服務標籤也會套用至適用於 MySQL 和 PostgreSQL 服務的 Azure 資料庫。 請務必注意，當您將 **Microsoft.Sql** 服務標籤套用到 VNet 服務端點時，它將會針對子網路上的所有 Azure SQL Database、適用於 MySQL 的 Azure 資料庫，以及適用於 PostgreSQL 的 Azure 資料庫伺服器設定服務端點流量。 

**虛擬網路規則：** 適用於 MySQL 的 Azure 資料庫伺服器的虛擬網路規則，是列於適用於 MySQL 的 Azure 資料庫伺服器之存取控制清單 (ACL) 中的子網路。 子網路必須包含 **Microsoft.Sql** 類型名稱，才能列在適用於 MySQL 的 Azure 資料庫伺服器的 ACL 中。

虛擬網路規則會指示適用於 MySQL 的 Azure 資料庫伺服器接受來自該子網路上每個節點的通訊。







<a name="anch-benefits-of-a-vnet-rule-68b" />

## <a name="benefits-of-a-virtual-network-rule"></a>虛擬網路規則的優點

除非您採取動作，否則子網路上的 VM 無法與適用於 MySQL 的 Azure 資料庫伺服器通訊。 建立通訊的一個動作，就是建立虛擬網路規則。 為了選擇虛擬網路 (VNet) 規則方法，基本上需要經過一番比較與對照討論，以確實了解防火牆所提供的各種只能擇其一的安全性選項。

### <a name="a-allow-access-to-azure-services"></a>A. 允許存取 Azure 服務

連線安全性窗格有一個標示為 [允許存取 Azure 服務] 的 [開啟/關閉] 按鈕。 [開啟] 設定允許來自所有 Azure IP 位址和所有 Azure 子網路的通訊。 這些 Azure IP 或子網路可能不是您所擁有。 此 [開啟] 設定可能會超出您想要開啟適用於 MySQL 資料庫的 Azure 資料庫的幅度。 虛擬網路規則功能提供更細微的控制。

### <a name="b-ip-rules"></a>B. IP 規則

適用於 MySQL 的 Azure 資料庫防火牆可讓您指定 IP 位址範圍，以接受來自此範圍內的通訊進入適用於 MySQL 資料庫的 Azure 資料庫。 此方法對 Azure 私人網路外部的穩定 IP 位址很適合。 但 Azure 私人網路內部的許多節點都以「動態」IP 位址設定。 動態 IP 位址可能變更，例如當 VM 重新啟動時。 在生產環境中，請勿在防火牆規則中指定動態 IP 位址。

您可以取得 VM 的「靜態」IP 位址，以挽回 IP 選項。 如需詳細資料，請參閱[使用 Azure 入口網站設定虛擬機器的私人 IP 位址][vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w]。

不過，靜態 IP 方法可能變得難以管理，在大規模使用時成本很高。 虛擬網路規則較容易建立和管理。

### <a name="c-cannot-yet-have-azure-database-for-mysql-on-a-subnet-without-defining-a-service-endpoint"></a>C. 不能在尚未定義服務端點的情況下在子網路上具備適用於 MySQL 的 Azure 資料庫

如果 **Microsoft.Sql** 伺服器是虛擬網路中某個子網路上的節點，則該虛擬網路內的所有節點都可以與適用於 MySQL 的 Azure 資料庫伺服器通訊。 在此情況下，您的 VM 可在不需要任何虛擬網路規則或 IP 規則的情況下，與適用於 MySQL 的 Azure 資料庫通訊。

不過，截至 2018 年 8 月，適用於 MySQL 的 Azure 資料庫服務還不是可直接指派給子網路的服務。

<a name="anch-details-about-vnet-rules-38q" />

## <a name="details-about-virtual-network-rules"></a>虛擬網路規則的詳細資料

本節描述虛擬網路規則的一些詳細資料。

### <a name="only-one-geographic-region"></a>只有一個地理區域

每個虛擬網路服務端點只套用至一個 Azure 區域。 端點無法讓其他區域接受來自子網路的通訊。

任何虛擬網路規則只以套用其基礎端點的區域為對象。

### <a name="server-level-not-database-level"></a>伺服器層級，非資料庫層級

每個虛擬網路規則都會套用到整個適用於 MySQL 的 Azure 資料庫伺服器，而不只是伺服器上的特定資料庫。 換句話說，虛擬網路規則是在伺服器層級套用，而不是資料庫層級。

### <a name="security-administration-roles"></a>安全性管理角色

有一組獨立的安全性角色負責管理虛擬網路服務端點。 下列每個角色都需要採取動作：

- **網路管理員：** &nbsp;開啟端點。
- **資料庫管理員：**&nbsp;更新存取控制清單 (ACL) 以將指定的子網路新增至適用於 MySQL 的 Azure 資料庫伺服器。

RBAC 替代方案：

「網路管理員」和「資料庫管理員」角色的能力已超過管理虛擬網路規則所需。 只需要其中一部分能力。

在 Azure 中，您可以選擇使用[角色型存取控制 (RBAC)][ rbac-what-is-813s] 來建立單一自訂安全性角色，而且只給予一部分必要的能力。 可使用此自訂角色來代替，不必動用到「網路管理員」或「資料庫管理員」。將使用者新增至自訂角色，而不要新增至其他兩個主要的系統管理員角色，就可縮小安全性曝露面。

> [!NOTE]
> 在某些案例中，適用於 MySQL 的 Azure 資料庫和 VNet 子網路是位於不同的訂用帳戶。 在這些情況下，您必須確保下列設定：
> - 兩個訂用帳戶在相同的 Azure Active Directory 租用戶中。
> - 使用者具備啟動作業的必要權限，例如啟用服務端點、將 VNet 子網路新增至指定的伺服器。

## <a name="limitations"></a>限制

針對適用於 MySQL 的 Azure 資料庫，虛擬網路規則功能具有下列限制：

- 在適用於 MySQL 的 Azure 資料庫防火牆中，每個虛擬網路規則都會參考一個子網路。 裝載所有這些參考子網路的地理區域，必須和裝載適用於 MySQL 的 Azure 資料庫的地理區域相同。

- 在任何指定的虛擬網路中，每個適用於 MySQL 的 Azure 資料庫伺服器最多只能有 128 個 ACL 項目。

- 虛擬網路規則只套用至 Azure Resource Manager 虛擬網路，而不是[傳統部署模型][arm-deployment-model-568f]網路。

- 使用 **Microsoft.Sql** 服務標籤對適用於 MySQL 的 Azure 資料庫開啟虛擬網路服務端點，也會對所有 Azure 資料庫服務啟用端點：適用於 MySQL 的 Azure 資料庫、適用於 PostgreSQL 的 Azure 資料庫、Azure SQL Database，以及 Azure SQL 資料倉儲。

- VNet 服務端點的支援僅適用於一般用途伺服器和記憶體最佳化伺服器。

- 在防火牆上，IP 位址範圍會套用到下列網路項目，但虛擬網路規則不這麼做：
    - [站對站 (S2S) 虛擬私人網路 (VPN)][vpn-gateway-indexmd-608y]
    - 透過 [ExpressRoute][expressroute-indexmd-744v] 的內部部署

## <a name="expressroute"></a>ExpressRoute

如果您的網路使用 [ExpressRoute][expressroute-indexmd-744v] 連線至 Azure 網路，則每個線路在 Microsoft Edge 會設定兩個公用 IP 位址。 兩個 IP 位址可使用 Azure 公用對等互連來連線至 Microsoft 服務，例如 Azure 儲存體。

若要允許從您的線路針對適用於 MySQL 的 Azure 資料庫進行通訊，您必須為線路的公用 IP 位址建立 IP 網路規則。 若要尋找您 ExpressRoute 線路的公用 IP 位址，請使用 Azure 入口網站開啟具有 ExpressRoute 的支援票證。

## <a name="adding-a-vnet-firewall-rule-to-your-server-without-turning-on-vnet-service-endpoints"></a>在不開啟 VNET 服務端點的情況下將 VNET 防火牆規則新增至伺服器

只是設定防火牆規則不能協助伺服器防禦 VNet。 您也必須**開啟** VNet 服務端點，安全性才會生效。 當您**開啟**服務端點時，您的 VNet 子網路會停機，直到完成**關閉**到**開啟**的轉換。 特別是大型的 VNet，這會更明顯。 您可以使用 **IgnoreMissingServiceEndpoint** 旗標來減少或排除在轉換期間的停機時間。

您可以使用 Azure CLI 或入口網站設定 **IgnoreMissingServiceEndpoint** 旗標。

## <a name="related-articles"></a>相關文章
- [Azure 虛擬網路][vm-virtual-network-overview]
- [Azure 虛擬網路服務端點][vm-virtual-network-service-endpoints-overview-649d]

## <a name="next-steps"></a>後續步驟
如需建立 VNet 規則的文章，請參閱：
- [使用 Azure 入口網站建立及管理適用於 MySQL 的 Azure 資料庫的 VNet 規則](howto-manage-vnet-using-portal.md)
- [使用 Azure CLI 建立及管理適用於 MySQL 的 Azure 資料庫的 VNet 規則](howto-manage-vnet-using-cli.md)

<!-- Link references, to text, Within this same Github repo. -->
[arm-deployment-model-568f]: ../azure-resource-manager/resource-manager-deployment-model.md

[vm-virtual-network-overview]: ../virtual-network/virtual-networks-overview.md

[vm-virtual-network-service-endpoints-overview-649d]: ../virtual-network/virtual-network-service-endpoints-overview.md

[vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w]: ../virtual-network/virtual-networks-static-private-ip-arm-pportal.md

[rbac-what-is-813s]: ../active-directory/role-based-access-control-what-is.md

[vpn-gateway-indexmd-608y]: ../vpn-gateway/index.yml

[expressroute-indexmd-744v]: ../expressroute/index.yml