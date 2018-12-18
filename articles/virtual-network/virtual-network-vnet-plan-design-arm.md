---
title: 規劃 Azure 虛擬網路 | Microsoft Docs
description: 了解如何根據您的隔離、連線和位置需求規劃虛擬網路。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
ms.assetid: 3a4a9aea-7608-4d2e-bb3c-40de2e537200
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/16/2018
ms.author: jdial
ms.openlocfilehash: 1ec60300d179edf69e32dc07653b6c69c7cc8e52
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42140872"
---
# <a name="plan-virtual-networks"></a>規劃虛擬網路

建立虛擬網路並加以實驗是一件很簡單的事，但您很可能必須隨時間部署多個虛擬網路，以支援組織的生產需求。 藉由一些規劃，您將能夠更有效地部署虛擬網路並連線所需的資源。 本文章在您已經很熟悉虛擬網路，並具備相關使用經驗的前提之下，將能提供最多的幫助。 如果您不熟悉虛擬網路，建議先閱讀[虛擬網路概觀](virtual-networks-overview.md)。

## <a name="naming"></a>命名

所有 Azure 資源都有名稱。 名稱在範圍內必須是唯一的，而範圍可能會依每個資源類型而有所不同。 例如，虛擬網路的名稱在[資源群組](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group)內必須是唯一的，但在[訂用帳戶](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription)或 Azure [區域](https://azure.microsoft.com/regions/#services)內則可以重複。 在隨時間管理數個網路資源的情況下，定義可讓您在命名資源時一致地使用的命名慣例將會很有幫助。 如需建議，請參閱[命名慣例](/azure/architecture/best-practices/naming-conventions?toc=%2fazure%2fvirtual-network%2ftoc.json#networking)。

## <a name="regions"></a>區域

所有的 Azure 資源都是建立於 Azure 區域和訂用帳戶中。 只有當虛擬網路位於與資源相同的區域和訂用帳戶時，才能在虛擬網路中建立資源。 不過，您可以連線存在於不同訂用帳戶和區域中的虛擬網路。 如需詳細資訊，請參閱[連線能力](#connectivity)。 決定要將資源部署在哪些區域時，請考慮資源的取用者實際位於何處：

- 資源的取用者通常會希望將取用資源的網路延遲降至最低。 若要判斷指定的位置與 Azure 區域之間的相對延遲，請參閱[檢視相對延遲](../network-watcher/view-relative-latencies.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。
- 您是否有資料存留、主權、合規性或復原等需求？ 如果是，選擇能符合這些需求的區域很重要。 如需詳細資訊，請參閱 [Azure 的地理區](https://azure.microsoft.com/global-infrastructure/geographies/)。
- 針對您部署資源的 Azure 區域，相同區域內的 Azure 可用性區域之間是否有復原需求？ 您可以將虛擬機器 (VM) 等資源部署到相同虛擬網路內的不同可用性區域。 不過，並非所有的 Azure 區域都支援可用性區域。 若要深入了解可用性區域和支援此功能的區域，請參閱[可用性區域](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

## <a name="subscriptions"></a>訂用帳戶

您可以在每個訂用帳戶內，部署[限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits)之內的多個虛擬網路。 例如，有些組織針對不同部門會有不同的訂用帳戶。 如需訂用帳戶的詳細資訊和相關考量，請參閱[訂用帳戶治理](/azure/architecture/cloud-adoption-guide/subscription-governance#define-your-hierarchy)。

## <a name="segmentation"></a>分割

您可以針對每個訂用帳戶和每個區域建立多個虛擬網路。 您可以在每個虛擬網路內建立多個子網路。 下列考量可協助您判斷自己需要多少虛擬網路和子網路：

### <a name="virtual-networks"></a>虛擬網路

虛擬網路為 Azure 公用網路虛擬、隔離的部分。 每個虛擬網路都專屬於您的訂用帳戶。 決定要在訂用帳戶中建立一或多個虛擬網路時，要考量的事項包括：

- 組織針對將流量隔離至不同的虛擬網路上，是否有任何安全性需求？ 您可以選擇是否要連線虛擬網路。 如果您連線虛擬網路，就可以實作網路虛擬設備 (例如防火牆) 以控制虛擬網路之間流量的流動。 如需詳細資訊，請參閱[安全性](#security)和[連線能力](#connectivity)。
- 組織是否有任何將虛擬網路隔離到不同[訂用帳戶](#subscriptions)或[區域](#regions)的需求？
- [網路介面](virtual-network-network-interface.md)可讓 VM 與其他資源進行通訊。 每個網路介面都會被指派一或多個私人 IP 位址。 您在虛擬網路中需要多少個網路介面和[私人 IP 位址](virtual-network-ip-addresses-overview-arm.md#private-ip-addresses)？ 您在虛擬網路內可以擁有的網路介面和私人 IP 位址數目是有[限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits)的。
- 您是否要將虛擬網路連線至另一個虛擬網路或內部部署網路？ 您可以選擇讓虛擬網路互相連線，或是連線到內部部署網路，但不能連線到其他網路。 如需詳細資訊，請參閱[連線能力](#connectivity)。 您連線到其他虛擬網路或內部部署網路的每個虛擬網路，都必須有唯一的位址空間。 每個虛擬網路都要有一或多個公用或私人位址範圍指派至其位址空間。 位址範圍是以無類別網域間路由選擇 (CIDR) 格式指定，例如 10.0.0.0/16。 深入了解虛擬網路的[位址範圍](manage-virtual-network.md#add-or-remove-an-address-range)。
- 您針對不同虛擬網路中的資源，是否有任何組織管理需求？ 如果是，您可以將資源分散至不同的虛擬網路，以簡化對組織中個人的[權限指派](#permissions)，或對不同的虛擬網路指派不同的[原則](#policies)。
- 當您將某些 Azure 服務資源部署到虛擬網路時，它們會建立自己的虛擬網路。 若要判斷 Azure 服務是否會建立自己的虛擬網路，請參閱針對每個[可部署到虛擬網路的 Azure 服務](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network)的資訊。

### <a name="subnets"></a>子網路

虛擬網路可根據[限制](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits)分割成一或多個子網路。 決定要在訂用帳戶中建立一或多個子網路時，要考量的事項包括：

- 每個子網路於虛擬網路的位址空間內都必須具有唯一的位址範圍 (以 CIDR 格式指定)。 此位址範圍不能與虛擬網路內的其他子網路重疊。
- 如果您打算將一些 Azure 服務資源部署到虛擬網路中，它們可能需要 (或會建立) 自己的子網路，因此必須要有足夠的未配置空間以供它們進行。 若要判斷 Azure 服務是否會建立自己的子網路，請參閱針對每個[可部署到虛擬網路的 Azure 服務](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network)的資訊。 例如，如果您使用 Azure VPN 閘道將虛擬網路連線到內部部署網路，該虛擬網路針對閘道必須有專用的子網路。 深入了解[閘道子網路](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub)。
- 根據預設，Azure 會在虛擬網路中的所有子網路之間路由傳送網路流量。 您可以覆寫 Azure 的預設路由，以避免 Azure 在子網路之間進行路由傳送，或透過網路虛擬設備路由傳送子網路之間的流量。 如果您需要相同虛擬網路中資源間的流量流經網路虛擬設備 (NVA)，請將資源部署到不同的子網路。 深入了解[安全性](#security)。
- 您可以將針對 Azure 資源 (例如 Azure 儲存體帳戶或 Azure SQL 資料庫) 的存取，限制為具備虛擬網路服務端點的特定子網路。 此外，您可以拒絕來自網際網路的資源存取。 您可以建立多個子網路，並只針對某些子網路啟用某個服務端點。 深入了解[服務端點](virtual-network-service-endpoints-overview.md)，以及您可以針對它們啟用的 Azure 資源。
- 您可以將零或一個網路安全性群組關聯至虛擬網路中的每個子網路。 您可以將相同或不同的網路安全性群組關聯至每個子網路。 每個網路安全性群組都包含規則，能允許或拒絕進出來源與目的地的流量。 深入了解[網路安全性群組](#traffic-filtering)。

## <a name="security"></a>安全性

您可以使用網路安全性群組和網路虛擬設備來篩選虛擬網路中進出資源的網路流量。 您可以控制 Azure 路由傳送來自子網路流量的方法。 您也可以限制組織中能使用虛擬網路中資源的人員。

### <a name="traffic-filtering"></a>流量篩選

- 您可以使用網路安全性群組、能篩選網路流量的 NVA，或是上述兩者來篩選虛擬網路中資源之間的網路流量。 若要部署 NVA (例如防火牆) 來篩選網路流量，請參閱 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?subcategories=appliances&page=1)。 使用 NVA 時，您也可以建立自訂路由，以將來自子網路的流量路由傳送到 NVA。 深入了解[流量路由](#traffic-routing)。
- 網路安全性群組包含數個預設的安全性規則，能允許或拒絕進出資源的流量。 網路安全性群組可以與網路介面、網路介面所在的子網路，或是上述兩者建立關聯。 為了簡化安全性規則的管理，建議您盡量將網路安全性群組關聯至個別的子網路，而不是子網路內的個別網路介面。
- 如果子網路內不同的 VM 需要套用不同的安全性規則，您可以將 VM 中的網路介面與一或多個應用程式安全性群組建立關聯。 安全性規則可以在其來源、目的地，或是上述兩者中指定應用程式安全性群組。 該規則接著只會套用到身為該應用程式安全性群組成員的網路介面。 深入了解[網路安全性群組](security-overview.md)和[應用程式安全性群組](security-overview.md#application-security-groups)。
- Azure 會在每個網路安全性群組內建立數個預設的安全性規則。 其中一個預設規則會允許流量流經虛擬網路中所有資源。 若要覆寫這個行為，請使用網路安全性群組、透過自訂路由將流量路由傳送到 NVA，或是上述兩者。 建議您熟悉 Azure 的所有[預設安全性規則](security-overview.md#default-security-rules)，並了解網路安全性群組規則套用到資源的方法。

您可以檢視使用 [NVA](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz?toc=%2Fazure%2Fvirtual-network%2Ftoc.json) 或[網路安全性群組](virtual-networks-dmz-nsg.md)來在 Azure 和網際網路之間實作 DMZ 的範例設計。

### <a name="traffic-routing"></a>流量路由

Azure 會針對來自子網路的輸出流量建立數個預設路由。 您可以透過建立路由表並將它關聯至子網路，來覆寫 Azure 的預設路由。 覆寫 Azure 預設路由的常見原因包括：
- 您想讓子網路之間的流量流經 NVA。 深入了解如何[設定路由表以強制流量流經 NVA](tutorial-create-route-table-portal.md)。
- 您想要強制所有流經 NVA (或內部部署) 的網際網路繫結流量，流經 Azure VPN 閘道。 強制網際網路流量流經內部以進行檢查及記錄，通常稱為「強制通道」。 深入了解如何設定[強制通道](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md?toc=%2Fazure%2Fvirtual-network%2Ftoc.json)。

如果您需要實作自訂路由，建議先熟悉 [Azure 中的路由](virtual-networks-udr-overview.md)。

## <a name="connectivity"></a>連線能力

您可以使用虛擬網路對等互連將虛擬網路連線到其他虛擬網路，或是使用 Azure VPN 閘道將虛擬網路連線到您的內部部署網路。

### <a name="peering"></a>對等互連

使用[虛擬網路對等互連](virtual-network-peering-overview.md)時，虛擬網路可位於相同或不同的 Azure 支援區域。 虛擬網路可以位於相同或不同的 Azure 訂用帳戶；針對不同 Azure 訂用帳戶的情況，前提必須將這兩個訂用帳戶都指派到相同的 Azure Active Directory 租用戶。 在建立對等互連之前，建議您先熟悉對等互連的所有[需求和限制條件](virtual-network-manage-peering.md#requirements-and-constraints)。 相同區域中對等互連的虛擬網路中資源間的頻寬會相同，就像這些資源都位於相同的虛擬網路一樣。

### <a name="vpn-gateway"></a>VPN 閘道

您可以使用 Azure [VPN 閘道](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，利用[站對站 VPN](../vpn-gateway/vpn-gateway-tutorial-vpnconnection-powershell.md?toc=%2fazure%2fvirtual-network%2ftoc.json)或搭配 Azure [ExpressRoute](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 的專用連線，將虛擬網路連線到您的內部部署網路。

您可以結合對等互連和 VPN 閘道來建立[中樞和輪輻網路](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json)，其中輪輻虛擬網路會連線到中樞虛擬網路，而中樞則會連線到內部部署網路。

### <a name="name-resolution"></a>名稱解析

單一虛擬網路中的資源無法使用 Azure 的[內建 DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md) 來解析對等互連虛擬網路中的資源名稱。 若要解析對等互連虛擬網路中的名稱，請[部署您自己的 DNS 伺服器](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)，或使用 Azure DNS [私人網域](../dns/private-dns-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 若要解析虛擬網路和內部部署網路中資源間的名稱，也需要您部署自己的 DNS 伺服器。

## <a name="permissions"></a>權限

Azure 會針對資源使用[角色型存取控制](../role-based-access-control/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (RBAC)。 系統會將權限指派到下列階層的[範圍](../role-based-access-control/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#scope)：訂用帳戶、管理群組、資源群組，以及個別資源。 若要深入了解階層，請參閱[組織您的資源](../azure-resource-manager/management-groups-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 若要使用 Azure 虛擬網路和其所有相關功能，例如對等互連、網路安全性群組、服務端點和路由表，您可以將組織的成員指派為內建的[擁有者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#owner)、[參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#contributor)或[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)角色，再將該角色指派到適當的範圍。 如果您想要針對一組虛擬網路功能指派特定權限，請建立[自訂角色](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，並針對該角色指派[虛擬網路](manage-virtual-network.md#permissions)、[子網路和服務端點](virtual-network-manage-subnet.md#permissions)、[網路介面](virtual-network-network-interface.md#permissions)、[對等互連](virtual-network-manage-peering.md#permissions)、[網路和應用程式安全性群組](manage-network-security-group.md#permissions)或[路由表](manage-route-table.md#permissions)所需的特定權限。

## <a name="policy"></a>原則

Azure 原則可讓您建立、指派和管理原則定義。 原則定義會對您的資源強制執行不同的規則，讓資源能持續符合組織標準和服務等級協定的規範。 Azure 原則會針對資源執行評估，掃描出不符合您所擁有原則定義規範的資源。 例如，您可以定義並套用一個原則，以允許只在特定資源群組或區域中建立虛擬網路。 另一個原則可能會要求每個子網路都要有相關聯的網路安全性群組。 接著，在建立和更新資源時，系統就會評估這些原則。

原則會套用到下列階層：訂用帳戶、管理群組和資源群組。 深入了解 [Azure 原則](../azure-policy/azure-policy-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，或是部署某些虛擬網路[原則範本](policy-samples.md)範例。

## <a name="next-steps"></a>後續步驟

深入了解[虛擬網路](manage-virtual-network.md)、[子網路與服務端點](virtual-network-manage-subnet.md)、[網路介面](virtual-network-network-interface.md)、[對等互連](virtual-network-manage-peering.md)、[網路和應用程式安全性群組](manage-network-security-group.md)或[路由表](manage-route-table.md)的所有工作、設定和選項。
