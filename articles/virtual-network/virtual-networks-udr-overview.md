---
title: Azure 虛擬網路流量路由 | Microsoft Docs
description: 了解 Azure 如何路由虛擬網路流量，以及您可以如何自訂 Azure 的路由。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/26/2017
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: 97e192312619455c0055a917df880cc48eb082dd
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978898"
---
# <a name="virtual-network-traffic-routing"></a>虛擬網路流量路由

了解 Azure 如何在 Azure、內部部署和網際網路資源間路由流量。 Azure 會在 Azure 虛擬網路內自動為每個子網路建立路由表，並將系統的預設路由新增至該表格。 若要深入了解虛擬網路和子網路，請參閱[虛擬網路概觀](virtual-networks-overview.md)。 您可以使用[自訂路由](#custom-routes)覆寫某些 Azure 的系統路由，並將其他自訂路由新增至路由表。 Azure 會依據子網路路由表中的路由，來路由子網路的輸出流量。

## <a name="system-routes"></a>系統路由

Azure 會自動建立系統路由，並將路由指派給虛擬網路中的每個子網路。 您無法建立系統路由，也無法移除系統路由，但是您可以使用[自訂路由](#custom-routes)覆寫某些系統路由。 當您使用特定 Azure 功能時，Azure 會為每個子網路建立預設的系統路由，並將其他[選擇性預設路由](#optional-default-routes)新增至特定子網路或每個子網路。

### <a name="default"></a>預設值

每個路由會包含位址首碼和下一個躍點類型。 當流量離開子網路並傳送至具有路由位址首碼的 IP 位址時，包含該首碼的路由就是 Azure 使用的路由。 了解有多個路由都包含相同或重疊的首碼時，[Azure 如何選取路由](#how-azure-selects-a-route)。 每次建立虛擬網路時，Azure 會在虛擬網路內自動為每個子網路建立下列預設系統路由：


|來源 |位址首碼                                        |下一個躍點類型  |
|-------|---------                                               |---------      |
|預設值|虛擬網路獨有                           |虛擬網路|
|預設值|0.0.0.0/0                                               |Internet       |
|預設值|10.0.0.0/8                                              |None           |
|預設值|172.16.0.0/12                                           |None           |
|預設值|192.168.0.0/16                                          |None           |
|預設值|100.64.0.0/10                                           |None           |

上表列出的下一個躍點類型，代表 Azure 如何路由上述位址首碼指定的流量。 下一個躍點類型的說明如下：

- **虛擬網路**：在虛擬網路[位址空間](manage-virtual-network.md#add-or-remove-an-address-range)內的位址範圍間路由流量。 Azure 建立路由所用的位址首碼，會與每個虛擬網路位址空間中定義的位址範圍相對應。 如果虛擬網路位址空間有多個定義的位址範圍，則 Azure 會為每個位址範圍建立個別路由。 Azure 會使用針對每個位址範圍建立的路由，來自動路由子網路之間的流量。 您不需要為 Azure 定義閘道來路由子網路之間的流量。 雖然虛擬網路包含子網路，而且每個子網路都有定義的位址範圍，但 Azure 不會為子網路位址範圍建立預設路由，因為每一個子網路位址範圍都在虛擬網路位址空間的位址範圍內。

- **網際網路**：將位址首碼所指定的流量路由到網際網路。 系統預設路由會指定 0.0.0.0/0 位址首碼。 如果您不覆寫 Azure 的預設路由，Azure 會將所有不是由虛擬網路內位址範圍指定的流量路由至網際網路，但有一個例外。 如果目的地位址適用於 Azure 的其中一個服務，Azure 會透過 Azure 的骨幹網路直接將流量路由至該服務，而不是將流量路由至網際網路。 不論虛擬網路存在哪一個 Azure 區域，或 Azure 服務執行個體部署在哪一個 Azure 區域，Azure 服務之間的流量都不會周遊網際網路。 您可以使用[自訂路由](#custom-routes)，來覆寫位址首碼為 0.0.0.0/0 的 Azure 預設系統路由。

- **無**：路由至下一個躍點類型為**無**的流量會遭到捨棄，而不是路由至子網路以外的地方。 Azure 會為下列位址首碼自動建立預設路由：
    - **10.0.0.0/8、172.16.0.0/12 和 192.168.0.0/16**：在 RFC 1918 中保留以作為私人用途。
    - **100.64.0.0/10**：在 RFC 6598 中保留。

    如果您在虛擬網路位址空間內指派上述任何位址範圍時，Azure 會自動將路由的下一個躍點類型從**無**變更至**虛擬網路**。 如果您對其指派位址範圍的虛擬網路位址空間中，包含四個保留位址首碼的其中一個 (但不是完全相同) 時，Azure 會移除該首碼的路由，並針對您新增的位址首碼來新增路由 (使用**虛擬網路**作為下一個躍點類型)。

### <a name="optional-default-routes"></a>選擇性預設路由

Azure 會針對不同的 Azure 功能，新增其他預設系統路由，但只有當您啟用這些功能時才會新增。 根據不同功能，Azure 會將選擇性預設路徑新增至該虛擬網路內的特定子網路，或新增至一個虛擬網路內的所有子網路。 當您啟用不同功能時，Azure 可能會新增的其他系統路由和下一個躍點類型如下：

|來源                 |位址首碼                       |下一個躍點類型|虛擬網路中新增路由的子網路|
|-----                  |----                                   |---------                    |--------|
|預設值                |虛擬網路獨有，例如：10.1.0.0/16|VNet 對等互連                 |全部|
|虛擬網路閘道|透過 BGP 從內部部署公佈的首碼，或在本機網路閘道中設定的首碼     |虛擬網路閘道      |全部|
|預設值                |多個                               |VirtualNetworkServiceEndpoint|僅限服務端點已啟用的子網路。|

- **虛擬網路 (VNet) 對等互連**：當您在兩個虛擬網路間建立虛擬網路對等互連時，系統會在每個建立對等連線的虛擬網路位址空間中，為每個位址範圍新增路由。 深入了解[虛擬網路對等互連](virtual-network-peering-overview.md)。  
- **虛擬網路閘道**：將虛擬網路閘道新增到虛擬網路時，系統會新增一個或多個下一個躍點類型列為*虛擬網路閘道*的路由。 來源也是*虛擬網路閘道*，因為閘道會將路由新增至子網路。 如果您的內部部署網路閘道會交換邊界閘道協定 ([BGP](#border-gateway-protocol)) 路由與 Azure 虛擬網路閘道，則系統會針對每個從內部部署網路閘道散佈的每個路由新增路由。 建議您盡可能將內部部署路由彙總至最大的位址範圍，那麼散佈至 Azure 虛擬網路閘道的路由就會最少。 您可以散佈至 Azure 虛擬網路閘道的路由數目有限。 如需詳細資訊，請參閱 [Azure 限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits)。
- **VirtualNetworkServiceEndpoint**：當您對特定服務啟用服務端點時，Azure 會將該服務的公用 IP 位址新增至路由表。 服務端點是針對虛擬網路內的個別子網路啟用，因此路由只會新增至服務端點已啟用的子網路路由表。 Azure 服務的公用 IP 位址會定期變更。 當位址變更時，Azure 會自動管理路由表中的位址。 深入了解[虛擬網路服務端點](virtual-network-service-endpoints-overview.md)，以及可以對其建立服務端點的服務。 

> [!NOTE]
> **VNet 對等互連**和 **VirtualNetworkServiceEndpoint** 下一個躍點類型只會新增至透過 Azure Resource Manager 部署模型建立的虛擬網路子網路路由表。 如果路由表是與透過傳統部署模型建立的虛擬網路子網路相關聯，則下一個躍點類型不會新增至該路由表。 深入了解 Azure [部署模型](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="custom-routes"></a>自訂路由

您可以透過建立[使用者定義](#user-defined)路由，或藉由交換[邊界閘道協定](#border-gateway-protocol) (BGP) 路由，在內部網路閘道和 Azure 虛擬網路閘道間建立自訂路由。 
 
### <a name="user-defined"></a>使用者定義

您可以在 Azure 中建立自訂或使用者定義路由來覆寫 Azure 的預設系統路由，或將其他路由新增至子網路的路由表。 在 Azure 中，您可以建立路由表，然後使路由表與零個或多個虛擬網路子網路產生關聯。 每個子網路可以有零個或一個與其相關聯的路由表。 若要了解您可以新增至路由表的最大路由數目，以及您可以為每個 Azure 訂用帳戶建立的最大使用者定義路由表數目，請參閱 [Azure 限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits)。 如果您建立路由表並讓它與子網路產生關聯，其中的路由會結合或覆寫 Azure 新增至子網路的預設路由。

建立使用者定義路由時，您可以指定下列的下一個躍點類型：

- **虛擬設備**：一般執行網路應用程式 (例如防火牆) 的虛擬機器。 若要了解您可以在虛擬網路中部署之多種預先設定的網路虛擬設備，請參閱 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?page=1&subcategories=appliances)。 當您使用**虛擬設備**躍點類型建立路由時，您也會指定下一個躍點 IP 位址。 IP 位址可以是：

    - 連接至虛擬機器的網路介面[私人 IP 位址](virtual-network-ip-addresses-overview-arm.md#private-ip-addresses)。 連接至虛擬機器，並將網路流量轉送至本身以外其他位址的任何網路介面，都必須啟用 Azure *啟用 IP 轉送*選項。 該設定會使 Azure 停止檢查網路介面的來源和目的地。 深入了解如何[啟用網路介面的 IP 轉送](virtual-network-network-interface.md#enable-or-disable-ip-forwarding)。 雖然 [啟用 IP 轉送] 是 Azure 的設定，但您可能也需要啟用虛擬機器作業系統內的 IP 轉送，以便設備轉送已指派給 Azure 網路介面的私人 IP 位址間的流量。 如果設備必須將流量路由傳送至公用 IP 位址，則設備必須代理流量，或將來源的私人 IP 位址進行網路位址轉譯為其自有的私人 IP 位址，然後 Azure 會將該位址進行網路位址轉譯為公用 IP 位址，再將流量傳送到網際網路。 若要決定虛擬機器內的必要設定，請參閱您作業系統或網路應用程式的文件。 若要了解 Azure 中的輸出連線，請參閱[了解輸出連線](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

    > [!NOTE]
    > 將虛擬設備部署至不同子網路，則透過該虛擬設備路由的資源就會在其中部署。 如果將虛擬設備部署至相同子網路，然後將路由表套用至透過虛擬設備路由流量的子網路，就會造成路由迴圈，使得流量不會離開子網路。

    - Azure [內部負載平衡器](../load-balancer/load-balancer-get-started-ilb-arm-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)的私人 IP 位址。 負載平衡器通常作為[網路虛擬設備高可用性策略](/azure/architecture/reference-architectures/dmz/nva-ha?toc=%2fazure%2fvirtual-network%2ftoc.json)的一部分使用。

    您可以定義位址首碼為 0.0.0.0/0 的路由和虛擬設備的下一個躍點類型，讓設備檢查流量並判斷是否要轉送或捨棄流量。 如果您想要建立包含 0.0.0.0/0 位址首碼的使用者定義路由，請先了解 [0.0.0.0/0 位址首碼](#default-route)。

- **虛擬網路閘道**：當您要將特定位址首碼指定的流量路由至虛擬網路閘道時指定。 虛擬網路閘道必須使用 **VPN** 類型建立。 您無法在使用者定義路由中指定建立為 **ExpressRoute** 類型的虛擬網路閘道，因為若使用 ExpressRoute，您必須使用適用於自訂路由的 [BGP](#border-gateway-protocol-routes)。 您可以將路由定義為將 0.0.0.0/0 位址首碼指定流量導向[路由式](../vpn-gateway/vpn-gateway-plan-design.md?toc=%2fazure%2fvirtual-network%2ftoc.json#vpntype)虛擬網路閘道。 在內部部署上，您可能有裝置可檢查流量並決定是否要轉送或捨棄流量。 如果您想要建立 0.0.0.0/0 位址首碼的使用者定義路由，請先了解 [0.0.0.0/0 位址首碼](#default-route)。 您不需要設定 0.0.0.0/0 位址首碼的使用者定義路由，如果您已[啟用 VPN 虛擬網路閘道的 BGP](../vpn-gateway/vpn-gateway-bgp-resource-manager-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，則可以透過 BGP 公佈具有 0.0.0.0/0 首碼的路由。
- **無**：當您要捨棄位址首碼的流量，而不是將流量轉送至目的地時指定。 如果有功能尚未設定完畢，Azure 可能會針對某些選擇性系統路徑列出「無」。 例如，如果您看到**下一個躍點類型**為「虛擬網路閘道」或「虛擬設備」的**下一個躍點 IP 位址**列為「無」，這可能是因為裝置未執行或未完整設定。 Azure 會針對下一個躍點類型為**無**的保留位址首碼，建立系統[預設路由](#default)。
- **虛擬網路**：當您想要覆寫虛擬網路中的預設路由時指定。 請參閱[路由範例](#routing-example)，以針對您想以**虛擬網路**躍點類型建立路由的原因取得範例。
- **網際網路**：當您想要明確地將位址首碼指定的流量路由至網際網路時指定，或您想要 Azure 服務所用的流量 (具有公用 IP 位址) 保留在 Azure 骨幹網路中時指定。

您無法在使用者定義路由中指定 **VNet 對等互連**或 **VirtualNetworkServiceEndpoint** 作為下一個躍點類型。 當您設定虛擬網路對等互連或服務端點時，具有 **VNet 對等互連**或 **VirtualNetworkServiceEndpoint** 下一個躍點類型的路由只可由 Azure 建立。

**Azure 工具間的下一個躍點類型**

下一個躍點類型的顯示和參照名稱在 Azure 入口網站和命令列工具之間是不同的，以及在 Azure Resource Manager 和傳統部署模型之間也不同。 下表列出的名稱可用來參照使用不同工具和[部署模型](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)的每種下一個躍點類型：

|下一個躍點類型                   |Azure CLI 和 PowerShell (Resource Manager) |Azure 傳統 CLI 和 PowerShell (傳統)|
|-------------                   |---------                                       |-----|
|虛擬網路閘道         |VirtualNetworkGateway                           |VPNGateway|
|虛擬網路                 |VNetLocal                                       |VNETLocal (不適用於 asm 模式下的傳統 CLI)|
|Internet                        |Internet                                        |Internet (不適用於 asm 模式下的傳統 CLI)|
|虛擬設備               |VirtualAppliance                                |VirtualAppliance|
|None                            |None                                            |Null (不適用於 asm 模式下的傳統 CLI)|
|虛擬網路對等互連         |VNet 對等互連                                    |不適用|
|虛擬網路服務端點|VirtualNetworkServiceEndpoint                   |不適用|

### <a name="border-gateway-protocol"></a>邊界閘道協定

內部部署網路閘道可以使用邊界閘道通訊協定 (BGP) 交換路由與 Azure 虛擬網路閘道。 是否要對 Azure 虛擬網路閘道使用 BGP，取決於您建立閘道時選取的類型。 如果您選取的類型為：

- **ExpressRoute**：您必須使用 BGP 將內部部署路由公佈至 Microsoft 邊緣路由器。 如果您部署的虛擬網路閘道是以 ExpressRoute 類型部署，就無法建立使用者定義路由來強制 ExpressRoute 虛擬網路閘道的流量。 您可以使用使用者定義的路由，強制執行從 Express Route 到例如網路虛擬裝置的流量。
- **VPN**：您可以選擇性地使用 BGP。 如需詳細資訊，請參閱[BGP 與站台對站 VPN 連線](../vpn-gateway/vpn-gateway-bgp-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

當您使用 BGP 交換 Azure 與路由時，系統會針對每個公佈的首碼，將個別路由新增至虛擬網路中的所有子網路路由表。 新增路由的來源和下一個躍點類型會列為*虛擬網路閘道*。 

在路由表上使用屬性，即可在子網路上停用 BGP 路由傳播。 當您使用 BGP 與 Azure 交換路由時，路由不會新增至已停用 BGP 傳播的所有子網路路由表。 使用下一個躍點類型為*虛擬網路閘道*的[自訂路由](#custom-routes)，即可進行 VPN 連線。 如需詳細資訊，請參閱[如何停用 BGP 路由傳播](manage-route-table.md#create-a-route-table)。

## <a name="how-azure-selects-a-route"></a>Azure 如何選取路由

當輸出流量從子網路送出時，Azure 會根據目的地 IP 位址選取路由 (使用最長的首碼比對演算法)。 例如，路由表有兩個路由：一個路由指定 10.0.0.0/24 位址首碼，而其他路由指定 10.0.0.0/16 位址首碼。 Azure 會將 10.0.0.5 指定流量路由至位址首碼為 10.0.0.0/24 之路由中指定的下一個躍點類型，因為 10.0.0.5 雖然都在兩個位址首碼中，但比起 10.0.0.0/16，10.0.0.0/24 是較長的首碼。 Azure 會將 10.0.1.5 指定流量路由至位址首碼為 10.0.0.0/16 之路由中指定的下一個躍點類型，因為 10.0.1.5 不包含在 10.0.0.0/24 位址首碼中，因此位址首碼為 10.0.0.0/16 的路由是符合的最長首碼。

如果多個路由包含相同的位址首碼，則 Azure 會根據下列優先順序選取路由類型：

1. 使用者定義路由
2. BGP 路由
3. 系統路由

> [!NOTE]
> 即使 BGP 路由是更為專用的路由，系統仍會慣用與虛擬網路、虛擬網路對等互連或虛擬網路服務端點相關之流量的系統路由。

例如，路由表包含下列路由：


|來源   |位址首碼  |下一個躍點類型           |
|---------|---------         |-------                 |
|預設值  | 0.0.0.0/0        |Internet                |
|使用者     | 0.0.0.0/0        |虛擬網路閘道 |

當流量的目的地 IP 位址不在路由表中任何其他路由的位址首碼內時，Azure 會選取具有**使用者**來源的路由，因為使用者定義路由的優先順序高於系統預設路由。

請參閱[路由範例](#routing-example)，以取得完整路由資料表，表中有路由的說明。

## <a name="default-route"></a>0.0.0.0/0 位址首碼

如果 IP 位址不在子網路路由表中任何其他路由的位址首碼內，則具有 0.0.0.0/0 位址首碼的路由會指示 Azure 如何路由該 IP 位址指定的流量。 建立子網路時，Azure 會建立 0.0.0.0/0 位址首碼的[預設](#default)路由，且下一個躍點類型為**網際網路**。 如果您不覆寫此路由，Azure 就會將不包含在任何其他路由位址首碼的 IP 位址所用流量，路由至網際網路。 但有個例外，Azure 服務公用 IP 位址的流量會保留在 Azure 骨幹網路中，而不會路由至網際網路。 如果您以[自訂](#custom-routes)路由覆寫此路由，其目的地位址不在路由表中任何其他路由位址首碼中的流量，就會將傳送至網路虛擬設備或虛擬網路閘道 (視您指定的自訂路由而定)。

當您覆寫 0.0.0.0/0 位址首碼後，除了輸出流量會從子網路流經虛擬網路閘道或虛擬設備外，使用 Azure 預設路由時也會發生下列變更： 

- Azure 會將所有流量傳送至路由中指定的下一個躍點類型，包括 Azure 服務公用 IP 位址指定的流量。 當 0.0.0.0/0 位址首碼路由的下個躍點類型是**網際網路**時，從子網路送往 Azure 服務公用 IP 位址的流量永遠不會離開 Azure 的骨幹網路，不論虛擬網路或 Azure 服務資源存在哪個 Azure 地區。 但當您以**虛擬網路閘道**或**虛擬設備**建立使用者定義或 BGP 路由時，所有流量 (包括傳送至您尚未對其啟用[服務端點](virtual-network-service-endpoints-overview.md)之 Azure 服務公用 IP 位址的流量) 會傳送至路由中指定的下一個躍點類型。 如果您已啟用服務的服務端點，服務的流量則不會路由至路由位址首碼為 0.0.0.0/0 的下一個躍點類型，因為當您啟用服務端點時，服務的位址首碼會在 Azure 建立的路由中指定，而服務的位址首碼會比 0.0.0.0/0 長。
- 您已無法從網際網路直接存取子網路中的資源。 如果輸入流量是通過路由位址首碼為 0.0.0.0/0 的下一個躍點所指定的裝置後，再抵達虛擬網路中的資源，則您可以直接從網際網路存取子網路中的資源。 如果路由包含下列的下一個躍點類型值：
    - **虛擬設備**：設備必須：
        - 能夠從網際網路存取
        - 已有指派的公用 IP 位址
        - 沒有相關聯的網路安全性群組規則會阻止裝置通訊
        - 不會拒絕通訊
        - 能夠進行網路位址轉譯和轉送，或對傳送至子網路中目的地資源的流量設定 Proxy，並將流量傳回網際網路。 
    - **虛擬網路閘道**：如果閘道是 ExpressRoute 虛擬網路閘道，則透過 ExpressRoute 的[私人對等互連](../expressroute/expressroute-circuit-peerings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-private-peering)，連線到網際網路的裝置在內部部署上可以進行網路位址轉譯和轉送，或對傳送至子網路中目的地資源的流量設定 Proxy。 

如果虛擬網路連線至 Azure VPN 閘道，請勿將路由表關聯至所含路由的目的地為 0.0.0.0/0 的[閘道子網路](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub)。 這麼做會讓閘道無法正常運作。 如需詳細資訊，請參閱 [VPN 閘道常見問題集](../vpn-gateway/vpn-gateway-vpn-faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#why-are-certain-ports-opened-on-my-vpn-gateway)中的*為什麼 VPN 閘道上的某些連接埠已開啟？* 問題。

請參閱 [Azure 與內部部署資料中心之間的 DMZ](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid?toc=%2fazure%2fvirtual-network%2ftoc.json)和 [Azure 與網際網路之間的 DMZ](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz?toc=%2fazure%2fvirtual-network%2ftoc.json)，以取得在網際網路和 Azure 之間使用虛擬網路閘道和虛擬設備的實作詳細資料。

## <a name="routing-example"></a>路由範例

為了說明本文中的概念，以下各節會說明：
- 案例與需求
- 必須符合需求的自訂路由
- 每個子網路都有的路由表，其中包含必須符合需求的預設與自訂路由

> [!NOTE]
> 此範例不適合作為建議的實作或最佳做法的實作。 相反地，此範例只適合用來說明本文中的概念。

### <a name="requirements"></a>需求

1. 在相同的 Azure 區域中實作兩個虛擬網路，並且讓資源可在虛擬網路之間通訊。
2. 啟用內部部署網路，以透過 VPN 通道在網際網路間安全地與兩個虛擬網路進行通訊。 *或者，也可以使用 ExpressRoute 連線，但此範例中會使用 VPN 連線。*
3. 針對一個虛擬網路中有一個子網路：
 
    - 強制子網路的所有輸出流量 (除了傳送至 Azure 儲存體和子網路內的流量) 通過網路的虛擬設備，以便進行檢查和記錄。
    - 不會檢查子網路內私人 IP 位址之間的流量；允許流量直接在所有資源之間的流動。 
    - 捨棄任何傳送至其他虛擬網路的輸出流量。
    - 允許傳送至 Azure 儲存體的輸出流量直接流向儲存體，而不強制透過網路虛擬設備。

4. 允許所有其他子網路和虛擬網路之間的所有流量。

### <a name="implementation"></a>實作

透過符合上述需求的 Azure Resource Manager 部署模型所執行的實作如下圖所示：

![網路圖表](./media/virtual-networks-udr-overview/routing-example.png)

箭號顯示流量的流動。 

### <a name="route-tables"></a>路由表

#### <a name="subnet1"></a>Subnet1

圖片中的 Subnet1 路由表包含下列路由：

|ID  |來源 |State  |位址首碼    |下一個躍點類型          |下一個躍點 IP 位址|使用者定義路由名稱| 
|----|-------|-------|------              |-------                |--------           |--------      |
|1   |預設值|無效|10.0.0.0/16         |虛擬網路        |                   |              |
|2   |使用者   |Active |10.0.0.0/16         |虛擬設備      |10.0.100.4         |Within-VNet1  |
|3   |使用者   |Active |10.0.0.0/24         |虛擬網路        |                   |Within-Subnet1|
|4   |預設值|無效|10.1.0.0/16         |VNet 對等互連           |                   |              |
|5   |預設值|無效|10.2.0.0/16         |VNet 對等互連           |                   |              |
|6   |使用者   |Active |10.1.0.0/16         |None                   |                   |ToVNet2-1-Drop|
|7   |使用者   |Active |10.2.0.0/16         |None                   |                   |ToVNet2-2-Drop|
|8   |預設值|無效|10.10.0.0/16        |虛擬網路閘道|[X.X.X.X]          |              |
|9   |使用者   |Active |10.10.0.0/16        |虛擬設備      |10.0.100.4         |To-On-Prem    |
|10  |預設值|Active |[X.X.X.X]           |VirtualNetworkServiceEndpoint    |         |              |
|11  |預設值|無效|0.0.0.0/0           |Internet|              |                   |              |
|12  |使用者   |Active |0.0.0.0/0           |虛擬設備      |10.0.100.4         |Default-NVA   |

每個路由 ID 的說明如下：

1. Azure 已自動為 Virtual-network-1 內的所有子網路新增此路由，因為 10.0.0.0/16 是虛擬網路位址空間中定義的唯一位址範圍。 如果未建立路由 ID2 中的使用者定義路由，傳送到 10.0.0.1 和 10.0.255.254 之間任何位址的流量就會在虛擬網路內進行路由，因為首碼長度大於 0.0.0.0/0，且不在任何其他路由的位址首碼內。 當 ID2 (使用者定義路由) 已新增時，Azure 會自動將狀態從「作用中」變更為「無效」，因為其首碼與預設路由一樣，而使用者定義路由會覆寫預設路由。 Subnet2 的此路由狀態仍然是「作用中」，因為其中有使用者定義路由 (ID2) 的路由表並未與 Subnet2 產生關聯。
2. 當 10.0.0.0/16 位址首碼的使用者定義路由已與 Virtual-network-1 中 Subnet1 子網路產生關聯時，Azure 就會新增此路由。 使用者定義路由會指定 10.0.100.4 作為虛擬設備的 IP 位址，因為該位址是指派給虛擬設備虛擬機器的私人 IP 位址。 此路由存在的路由表並未與 Subnet2 產生關聯，因此不會出現在 Subnet2 的路由表中。 此路由會覆寫 10.0.0.0/16 首碼 (ID1) 的預設路由，預設路由會透過虛擬網路的下一個躍點類型，自動在虛擬網路內路由位址 10.0.0.1 和 10.0.255.254 的流量。 此路由是為符合[需求](#requirements) 3 而存在，會強制所有輸出流量通過虛擬設備。
3. 當 10.0.0.0/24 位址首碼的使用者定義路由已與 Subnet1 子網路產生關聯時，Azure 就會新增此路由。 傳送至位址 10.0.0.1 和 10.0.0.254 之間的流量仍會在子網路內，而不是路由至上一個規則 (ID2) 中指定的虛擬設備，因為有比 ID2 路由更長的首碼。 此路由並未與Subnet2 產生關聯，因此路由不會出現在 Subnet2 的路由表中。 針對 Subnet1 內的流量，此路由有效地覆寫 ID2 路由。 此路由是為符合[需求](#requirements) 3 而存在。
4. 當虛擬網路與 Virtual-network-2 對等互連時，Azure 會針對 Virtual-network-1 內的所有子網路，自動在 ID 4 和 ID 5 中新增路由。 Virtual-network-2 在其位址空間中有兩個位址範圍：10.1.0.0/16 和 10.2.0.0/16，因此 Azure 會為每個範圍新增路由。 如果未建立路由 ID 6 和 ID 7 中的使用者定義路由，傳送到 10.1.0.1-10.1.255.254 和 10.2.0.1-10.2.255.254 之間任何位址的流量將會路由至對等虛擬網路，因為首碼長度大於 0.0.0.0/0，且不在任何其他路由的位址首碼內。 當 ID 6 和 ID 7 中的已新增時，Azure 會自動將狀態從「作用中」變更為「無效」，因為他們的首碼與路由 ID 4 和 ID 5 一樣，而使用者定義路由會覆寫預設路由。 Subnet2 中 ID 4 和 ID 5 的路由狀態仍然是「作用中」，因為包含 ID 4 和 ID 5 中使用者定義路由的路由表並未與 Subnet2 產生關聯。 虛擬網路對等互連是為符合[需求](#requirements) 1 而建立。
5. 與 ID4 的說明相同。
6. 當 10.1.0.0/16 和 10.2.0.0/16 位址首碼的使用者定義路由已與 Subnet1 子網路產生關聯時，Azure 就會新增此路由和 ID7 中的路由。 傳送至 10.1.0.1-10.1.255.254 和 10.2.0.1-10.2.255.254 之間位址的流量會遭到 Azure 捨棄，而不是路由至對等互連的虛擬網路，因為使用者定義路由會覆寫預設路由。 這些路由並未與 Subnet2 產生關聯，因此路由不會出現在 Subnet2 的路由表中。 針對離開 Subnet1 的流量，這些路由會覆寫 ID4 和 ID5 路由。 ID6 和 ID7 路由是為符合[需求](#requirements) 3 而存在，會捨棄其他虛擬網路指定的流量。
7. 與 ID6 的說明相同。
8. 當 VPN 類型虛擬網路閘道在虛擬網路內建立時，Azure 會針對 Virtual-network-1 內的所有子網路，自動新增此路由。 Azure 會將虛擬網路閘道的公用 IP 位址新增至路由表。 傳送到 10.10.0.1 和 10.10.255.254 之間任何位址的流量會路由至虛擬網路閘道。 首碼長度大於 0.0.0.0/0，且不在任何其他路由的位址首碼內。 虛擬網路閘道是為符合[需求](#requirements) 2 而建立。
9. 當 10.10.0.0/16 位址首碼的使用者定義路由已新增至與 Subnet1 相關聯的路由表時，Azure 就會新增此路由。 此路由會覆寫 ID8。 路由會將所有內部部署網路指定的流量傳送至 NVA 以進行檢查，而不是將流量直接路由至內部部署。 此路由是為符合[需求](#requirements) 3 而建立。
10. 當 Azure 服務的服務端點已為子網路啟用時，Azure 會自動將此路由新增至子網路。 Azure 會將流量從子網路路由至服務的公用 IP 位址 (透過 Azure 基礎結構網路)。 首碼長度大於 0.0.0.0/0，且不在任何其他路由的位址首碼內。 服務端點是為符合[需求](#requirements) 3 而建立，會允許 Azure 儲存體指定的流量可直接流向 Azure 儲存體。
11. Azure 會自動將此路由新增至 Virtual-network-1和 Virtual-network-2 內所有子網路的路由表。 0.0.0.0/0 位址首碼是最短的首碼。 若流量傳送至位址首碼較長的位址，則流量會以其他路由為基礎進行路由。 依預設，Azure 會將位址指定的所有流量路由至網際網路，除非位址已在其他某個路由中指定。 當 0.0.0.0/0 位址首碼 (ID12) 的使用者定義路由已與子網路產生關聯時，Azure 會自動將 Subnet1 子網路的狀態從「作用中」變更為「無效」。 對於兩個虛擬網路內的所有其他子網路而言，此路由的狀態仍是「作用中」，因為路由並未與任何其他虛擬網路內的任何其他子網路產生關聯。
12. 當 0.0.0.0/0 位址首碼的使用者定義路由已與 Subnet1 子網路產生關聯時，Azure 就會新增此路由。 使用者定義路由會指定 10.0.100.4 為虛擬設備的 IP 位址。 此路由並未與 Subnet2 產生關聯，因此路由不會出現在 Subnet2 的路由表中。 如果流量是由不在任何其他路由位址首碼內的位址所用，則所有流量都會傳送至虛擬設備。 此路由的增加會將 Subnet1 的 0.0.0.0/0 位址首碼 (ID11) 預設路由狀態從「作用中」變更為「無效」，因為使用者定義路由會覆寫預設路由。 此路由是為符合[需求](#requirements) 3 而存在。

#### <a name="subnet2"></a>Subnet2

圖片中的 Subnet2 路由表包含下列路由：

|來源  |State  |位址首碼    |下一個躍點類型             |下一個躍點 IP 位址|
|------- |-------|------              |-------                   |--------           
|預設值 |Active |10.0.0.0/16         |虛擬網路           |                   |
|預設值 |Active |10.1.0.0/16         |VNet 對等互連              |                   |
|預設值 |Active |10.2.0.0/16         |VNet 對等互連              |                   |
|預設值 |Active |10.10.0.0/16        |虛擬網路閘道   |[X.X.X.X]          |
|預設值 |Active |0.0.0.0/0           |Internet                  |                   |
|預設值 |Active |10.0.0.0/8          |None                      |                   |
|預設值 |Active |100.64.0.0/10       |None                      |                   |
|預設值 |Active |172.16.0.0/12       |None                      |                   |
|預設值 |Active |192.168.0.0/16      |None                      |                   |

Subnet2 的路由表包含所有 Azure 建立的預設路由和選擇性 VNet 對等互連與虛擬網路閘道選擇性路由。 當閘道和對等互連新增至虛擬網路時，Azure 就會將選擇性路由新增至虛擬網路中的所有子網路。 當 0.0.0.0/0 位址首碼的使用者定義路由新增至 Subnet1 時，Azure 就會從 Subnet1 路由表移除 10.0.0.0/8、172.16.0.0/12、192.168.0.0/16 和 100.64.0.0/10 位址首碼的路由。  

## <a name="next-steps"></a>後續步驟

- [使用路由和網路虛擬設備建立使用者定義路由表](tutorial-create-route-table-portal.md)
- [設定適用於 Azure VPN 閘道的 BGP](../vpn-gateway/vpn-gateway-bgp-resource-manager-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [搭配使用 ExpressRoute 與 BGP](../expressroute/expressroute-routing.md?toc=%2fazure%2fvirtual-network%2ftoc.json#route-aggregation-and-prefix-limits)
- [檢視子網路的所有路由](diagnose-network-routing-problem.md)。 使用者定義路由表只會顯示使用者定義路由，而不會顯示子網路的預設和 BGP 路由。 檢視所有路由會顯示子網路 (內含網路介面) 的預設、BGP 及使用者定義路由。
- 在虛擬機器和目的地 IP 位址間[判斷下一個躍點類型](../network-watcher/diagnose-vm-network-routing-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 Azure 網路監看員的下一個躍點功能，可讓您判斷流量是否離開子網路且正在路由到您覺得正確的地方。
