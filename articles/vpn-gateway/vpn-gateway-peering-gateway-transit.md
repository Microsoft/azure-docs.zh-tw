---
title: 為虛擬網路對等互連設定 VPN 閘道傳輸：Azure Resource Manager | Microsoft Docs
description: 為虛擬網路對等互連設定 VPN 閘道傳輸。
services: vpn-gateway
documentationcenter: na
author: yushwang
manager: rossort
editor: ''
tags: azure-resource-manager
ms.assetid: 0683c664-9c03-40a4-b198-a6529bf1ce8b
ms.service: vpn-gateway
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/25/2018
ms.author: yushwang
ms.openlocfilehash: cced9a313664eaeaed4dd1298488670960b76413
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32195765"
---
# <a name="configure-vpn-gateway-transit-for-virtual-network-peering"></a>為虛擬網路對等互連設定 VPN 閘道傳輸

本文將協助您為虛擬網路對等互連設定閘道傳輸。 [虛擬網路對等互連](../virtual-network/virtual-network-peering-overview.md)可緊密連接兩個 Azure 虛擬網路，將兩個虛擬網路合併為一個以用於連線。 [閘道傳輸](../virtual-network/virtual-network-peering-overview.md#gateways-and-on-premises-connectivity)是一個對等互連的屬性，可讓一個虛擬網路利用對等虛擬網路中的 VPN 閘道，來進行跨單位或 VNet 對 VNet 連線。 下圖顯示閘道傳輸搭配虛擬網路對等互連的運作方式。

![閘道傳輸](./media/vpn-gateway-peering-gateway-transit/gatewaytransit.png)

在圖表中，閘道傳輸可讓對等的虛擬網路在 Hub-RM 中使用 Azure VPN 閘道。 VPN 閘道上可用的連線 (包括 S2S、P2S 和 VNet 對 VNet 連線) 皆適用於這三個虛擬網路。 傳輸選項可用於相同或不同部署模型之間的對等互連。 條件是 VPN 閘道只能位在使用資源管理員部署模型的虛擬網路中，如圖中所示。

在中樞與輪輻式 (hub-and-spoke) 網路架構中，閘道傳輸會讓輪輻虛擬網路共用中樞內的 VPN 閘道，而不是將 VPN 閘道部署在每個輪輻虛擬網路中。 通往閘道連線虛擬網路或內部部署網路的路由，將會傳播到使用閘道傳輸的對等虛擬網路路由表。 您可以從 VPN 閘道中停用自動路由傳播 建立具有「**停用 BGP 路由傳播**」選項的路由表，然後讓路由表與子網路產生關聯，以防止路由發佈至這些子網路。 如需詳細資訊，請參閱[虛擬網路由表](../virtual-network/manage-route-table.md)。

本文件中描述兩種案例：

1. 兩個虛擬網路都使用資源管理員部署模型
2. 輪輻虛擬網路使用傳統部署，而具有閘道的中樞虛擬網路位於資源管理員中

> [!IMPORTANT]
> 閘道傳輸目前不支援與全域虛擬網路對等互連。

## <a name="requirements"></a>需求

本文件中的範例需要建立下列資源：

1. 具有 VPN 閘道的 Hub-RM 虛擬網路
2. Spoke-RM 虛擬網路
3. 具有傳統部署模型的 Spoke-Classic 虛擬網路
4. 您使用的帳戶需要必要角色和權限。 如需詳細資訊，請參閱本文的[權限](#permissions)一節。

請參閱下列文件的指示：

1. [在虛擬網路中建立 VPN 閘道](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
2. [以相同部署模型建立虛擬網路對等互連](../virtual-network/tutorial-connect-virtual-networks-portal.md)
3. [以不同部署模型建立虛擬網路對等互連](../virtual-network/create-peering-different-deployment-models.md)

## <a name="permissions"></a>權限

您用於建立虛擬網路對等互連的帳戶必須具有必要角色或權限。 在以下範例中，如果您要將兩個虛擬網路 (分別名為 Hub-RM 和 Spoke-Classic) 對等互連，您的帳戶必須具有每個虛擬網路的下列角色或權限：
    
|虛擬網路|部署模型|角色|權限|
|---|---|---|---|
|Hub-RM|Resource Manager|[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write|
| |傳統|[傳統網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|N/A|
|Spoke-Classic|Resource Manager|[網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/peer|
||傳統|[傳統網路參與者](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|Microsoft.ClassicNetwork/virtualNetworks/peer|

深入了解[內建角色](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)以及如何對[自訂角色](../active-directory/role-based-access-control-custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json)指派特定權限 (僅限 Resource Manager)。

## <a name="resource-manager-to-resource-manager-peering-with-gateway-transit"></a>使用閘道傳輸的資源管理員對資源管理員對等互連

請遵循指示來建立或更新虛擬網路對等互連，以啟用閘道傳輸。

1. 從 Azure 入口網站建立或更新從 Spoke-RM 到 Hub-RM 的虛擬網路對等互連。 瀏覽至 Spoke-RM 虛擬網路資源，並依序按一下 [對等互連] 和 [新增]：
    - 設定 [資源管理員] 選項
    - 在對應的訂用帳戶中選取 Hub-RM 虛擬網路
    - 請確定 [允許虛擬網路存取] 是「啟用」狀態
    - 設定 [使用遠端閘道] 選項
    - 按一下 [確定]

    ![spokerm-to-hubrm](./media/vpn-gateway-peering-gateway-transit/spokerm-hubrm-peering.png)

2. 如果已建立對等互連，請瀏覽至對等互連的資源，然後啟用 [使用遠端閘道] 選項，如同步驟 (1) 所示的螢幕擷取畫面

3. 從 Azure 入口網站建立或更新從 Hub-RM 到 Spoke-RM 的虛擬網路對等互連。 瀏覽至 Hub-RM 虛擬網路資源，並依序按一下 [對等互連] 和 [新增]：
    - 設定 [資源管理員] 選項
    - 請確定 [允許虛擬網路存取] 是「啟用」狀態
    - 在對應的訂用帳戶中選取 "Spoke-RM" 虛擬網路
    - 設定 [允許閘道傳輸] 選項
    - 按一下 [確定]

    ![hubrm-to-spokerm](./media/vpn-gateway-peering-gateway-transit/hubrm-spokerm-peering.png)

4. 如果已建立對等互連，請瀏覽至對等互連的資源，然後啟用 [允許閘道傳輸] 選項，如同步驟 (3) 所示的螢幕擷取畫面

5. 確認兩個虛擬網路上的對等互連狀態為「**已連線**」

### <a name="powershell-sample"></a>PowerShell 範例

您也可以使用 PowerShell 來建立或更新上述範例的對等互連。 使用您虛擬網路和資源群組的名稱取代變數。

```azurepowershell-interactive
$SpokeRG = "SpokeRG1"
$SpokeRM = "Spoke-RM"
$HubRG   = "HubRG1"
$HubRM   = "Hub-RM"

$spokermvnet = Get-AzureRmVirtualNetwork -Name $SpokeRM -ResourceGroup $SpokeRG
$hubrmvnet   = Get-AzureRmVirtualNetwork -Name $HubRM -ResourceGroup $HubRG

Add-AzureRmVirtualNetworkPeering `
  -Name SpokeRMtoHubRM `
  -VirtualNetwork $spokermvnet `
  -RemoteVirtualNetworkId $hubrmvnet.Id `
  -UseRemoteGateways

Add-AzureRmVirtualNetworkPeering `
  -Name HubRMToSpokeRM `
  -VirtualNetwork $hubrmvnet `
  -RemoteVirtualNetworkId $spokermvnet.Id `
  -AllowGatewayTransit
```

## <a name="classic-to-resource-manager-peering-with-gateway-transit"></a>使用閘道傳輸的傳統部署對資源管理員對等互連

這些步驟與資源管理員範例類似，但不包括只適用於 Hub-RM 虛擬網路的作業。

1. 從 Azure 入口網站建立或更新從 Hub-RM 到 Spoke-RM 的虛擬網路對等互連。 瀏覽至 Hub-RM 虛擬網路資源，並依序按一下 [對等互連] 和 [新增]：
    - 設定虛擬網路部署模型的「傳統」選項
    - 在對應的訂用帳戶中選取 "Spoke-Classic" 虛擬網路
    - 請確定 [允許虛擬網路存取] 是「啟用」狀態
    - 設定 [允許閘道傳輸] 選項
    - 按一下 [確定]

    ![hubrm-to-spokeclassic](./media/vpn-gateway-peering-gateway-transit/hubrm-spokeclassic-peering.png)

2. 如果已建立對等互連，請瀏覽至對等互連的資源，然後啟用 [允許閘道傳輸] 選項，如同步驟 (1) 所示的螢幕擷取畫面

3. 無須在 Spoke-Classic 虛擬網路上執行任何操作

4. 確認 Hub-RM 虛擬網路上的對等互連狀態為「**已連線**」

一旦狀態顯示「已連線」，輪輻虛擬網路就可以透過中樞虛擬網路中的 VPN 閘道，開始使用 VNet 對 VNet 或跨單位連線。

### <a name="powershell-sample"></a>PowerShell 範例

您也可以使用 PowerShell 來建立或更新上述範例的對等互連。 使用您虛擬網路、資源群組和訂用帳戶的值，來取代變數和訂用帳戶識別碼。 您只需要在中樞虛擬網路上建立虛擬網路對等互連。

```azurepowershell-interactive
$HubRG   = "HubRG1"
$HubRM   = "Hub-RM"

$hubrmvnet   = Get-AzureRmVirtualNetwork -Name $HubRM -ResourceGroup $HubRG

Add-AzureRmVirtualNetworkPeering `
  -Name HubRMToSpokeRM `
  -VirtualNetwork $hubrmvnet `
  -RemoteVirtualNetworkId "/subscriptions/<subscription Id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/Spoke-Classic" `
  -AllowGatewayTransit
```

## <a name="next-steps"></a>後續步驟

* 請先深入了解[虛擬網路對等互連的條件約束和行為](../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints)及[虛擬網路對等互連設定](../virtual-network/virtual-network-manage-peering.md#create-a-peering)，再建立虛擬網路對等互連以供生產環境使用。
* 了解如何透過虛擬網路對等互連和閘道傳輸來[建立中樞和輪輻網路拓撲](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json#vnet-peering)。
