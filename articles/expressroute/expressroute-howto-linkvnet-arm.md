---
title: 將虛擬網路連結到 ExpressRoute 線路：PowerShell：Azure | Microsoft Docs
description: 本文提供以下內容的概觀：如何使用 Resource Manager 部署模型和 PowerShell 將虛擬網路 (VNet) 連結到 ExpressRoute 線路。
services: expressroute
documentationcenter: na
author: ganesr
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: daacb6e5-705a-456f-9a03-c4fc3f8c1f7e
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/08/2018
ms.author: ganesr
ms.openlocfilehash: 354f7c455e1a2846bbdd63fa12b1cc01e2a1b9c5
ms.sourcegitcommit: a0be2dc237d30b7f79914e8adfb85299571374ec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/12/2018
ms.locfileid: "29877546"
---
# <a name="connect-a-virtual-network-to-an-expressroute-circuit"></a>將虛擬網路連線到 ExpressRoute 電路
> [!div class="op_single_selector"]
> * [Azure 入口網站](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [Azure CLI](howto-linkvnet-cli.md)
> * [影片 - Azure 入口網站](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (傳統)](expressroute-howto-linkvnet-classic.md)
>

本文會協助您使用 Resource Manager 部署模型和 PowerShell，將虛擬網路 (VNet) 連結到 Azure ExpressRoute 線路。 虛擬網路可以位於相同的訂用帳戶中，或屬於另一個訂用帳戶。 本文也會示範如何更新虛擬網路連結。 

## <a name="before-you-begin"></a>開始之前
* 安裝最新版的 Azure PowerShell 模組。 如需詳細資訊，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/overview)。

* 開始設定之前，請先檢閱[必要條件](expressroute-prerequisites.md)、[路由需求](expressroute-routing.md)及[工作流程](expressroute-workflows.md)。

* 您必須擁有作用中的 ExpressRoute 線路。 
  * 遵循指示來 [建立 ExpressRoute 線路](expressroute-howto-circuit-arm.md) ，並由您的連線提供者來啟用該線路。 
  * 確定您已針對循環設定了 Azure 私用對等。 請參閱 [設定路由](expressroute-howto-routing-arm.md) 一文，以取得路由指示。 
  * 請確定已設定 Azure 私用對等，且已開啟您的網路與 Microsoft 之間的 BGP 對等，讓您可以啟用端對端連線。
  * 請確定您有已建立且完整佈建的虛擬網路和虛擬網路閘道。 請依照指示[為 ExpressRoute 建立虛擬網路閘道](expressroute-howto-add-gateway-resource-manager.md)。 ExpressRoute 的虛擬網路閘道會使用 GatewayType 'ExpressRoute'，而不是 VPN。

* 您最多可以將 10 個虛擬網路連結至標準 ExpressRoute 電路。 在使用標準 ExpressRoute 電路時，所有虛擬網路都必須位於相同的地理政治區域內。 

* 單一 VNet 最多可連結到四個 ExpressRoute 線路。 使用下列程序來建立您要連線之每個 ExpressRoute 線路的新連線物件。 ExpressRoute 線路可以位於相同的訂用帳戶、不同的訂用帳戶或兩者的混合。

* 如果您已啟用 ExpressRoute 高階附加元件，則可連結 ExpressRoute 電路的地理政治區域以外的虛擬網路，或是將大量的虛擬網路連接到 ExpressRoute 電路。 如需高階附加元件的詳細資訊，請參閱 [常見問題集](expressroute-faqs.md) 。


## <a name="connect-a-virtual-network-in-the-same-subscription-to-a-circuit"></a>將相同訂用帳戶中的虛擬網路連接到線路
您可以使用下列 Cmdlet，將虛擬網路閘道連結到 ExpressRoute 線路。 執行 Cmdlet 之前，請確定您已建立虛擬網路閘道，並準備好進行連結。

```powershell
$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$gw = Get-AzureRmVirtualNetworkGateway -Name "ExpressRouteGw" -ResourceGroupName "MyRG"
$connection = New-AzureRmVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName "MyRG" -Location "East US" -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
```

## <a name="connect-a-virtual-network-in-a-different-subscription-to-a-circuit"></a>將不同訂用帳戶中的虛擬網路連接到線路
您可以讓多個訂用帳戶共用 ExpressRoute 線路。 下圖顯示簡單的圖解，示範多個訂用帳戶共用 ExpressRoute 線路的方式。

大型雲端內的每個較小型雲端，會用來代表屬於組織內不同部門的訂用帳戶。 組織內的每個部門都可以使用自己的訂用帳戶來部署它們的服務，但可共用單一 ExpressRoute 線路，以連接回內部部署網路。 單一部門 (在此範例中：IT) 可以擁有 ExpressRoute 循環。 組織內的其他訂用帳戶可以使用 ExpressRoute 電路。

> [!NOTE]
> 訂用帳戶擁有者需支付 ExpressRoute 循環的連線和頻寬費用。 所有虛擬網路都會共用相同的頻寬。
> 
> 

![跨訂用帳戶的連線能力](./media/expressroute-howto-linkvnet-classic/cross-subscription.png)


### <a name="administration---circuit-owners-and-circuit-users"></a>系統管理 - 線路擁有者和線路使用者

「線路擁有者」是 ExpressRoute 線路資源的已授權「進階使用者」。 電路擁有者能夠建立可由「電路使用者」兌換的授權。 線路使用者是虛擬網路閘道的擁有者，與 ExpressRoute 線路位於不同的訂用帳戶內。 電路使用者可以兌換授權 (每個虛擬網路一個授權)。

電路擁有者能夠隨時修改及撤銷授權。 如果撤銷授權，則在存取權遭撤銷的訂用帳戶中，所有連結連線均會被刪除。

### <a name="circuit-owner-operations"></a>循環擁有者作業

**建立授權**

電路擁有者會建立授權。 這樣即會建立授權金鑰，讓電路使用者可用來將其虛擬網路閘道連接到 ExpressRoute 電路。 一個授權僅適用於一個連線。

下列 Cmdlet 程式碼片段示範如何建立授權：

```powershell
$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
Add-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization1"
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit

$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$auth1 = Get-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization1"
```


對此動作的回應將包含授權金鑰和狀態：

    Name                   : MyAuthorization1
    Id                     : /subscriptions/&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&/resourceGroups/ERCrossSubTestRG/providers/Microsoft.Network/expressRouteCircuits/CrossSubTest/authorizations/MyAuthorization1
    Etag                   : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&& 
    AuthorizationKey       : ####################################
    AuthorizationUseStatus : Available
    ProvisioningState      : Succeeded



**檢閱授權**

線路擁有者可以藉由執行下列 Cmdlet，來檢閱特定線路上發出的所有授權：

```powershell
$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$authorizations = Get-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit
```

**新增授權**

線路擁有者可以使用下列 Cmdlet 來新增授權：

```powershell
$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
Add-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization2"
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit

$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$authorizations = Get-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit
```

**刪除授權**

線路擁有者可以使用下列 Cmdlet 來撤銷/刪除使用者的授權：

```powershell
Remove-AzureRmExpressRouteCircuitAuthorization -Name "MyAuthorization2" -ExpressRouteCircuit $circuit
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit
```    

### <a name="circuit-user-operations"></a>循環使用者作業

電路使用者需要具備對等識別碼以及電路擁有者所提供的授權金鑰。 授權金鑰是 GUID。

您可以從下列命令檢查「對等識別碼」：

```powershell
Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
```

**兌換連線授權**

線路使用者可以執行下列 Cmdlet 來兌換連結授權：

```powershell
$id = "/subscriptions/********************************/resourceGroups/ERCrossSubTestRG/providers/Microsoft.Network/expressRouteCircuits/MyCircuit"    
$gw = Get-AzureRmVirtualNetworkGateway -Name "ExpressRouteGw" -ResourceGroupName "MyRG"
$connection = New-AzureRmVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName "RemoteResourceGroup" -Location "East US" -VirtualNetworkGateway1 $gw -PeerId $id -ConnectionType ExpressRoute -AuthorizationKey "^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^"
```

**釋出連線授權**

您可以藉由刪除將 ExpressRoute 線路連結到虛擬網路的連線來釋出授權。

## <a name="modify-a-virtual-network-connection"></a>修改虛擬網路連線
您可以更新虛擬網路連線的特定屬性。 

**更新連線權數**

您的虛擬網路可以連接到多個 ExpressRoute 線路。 您可能會收到來自多個 ExpressRoute 線路的相同前置詞。 若要選擇要使用哪一個連線來傳送目的為此前置詞的流量，您可以變更連線的 *RoutingWeight*。 流量將透過具有最高 *RoutingWeight* 的連線來傳送。

```powershell
$connection = Get-AzureRmVirtualNetworkGatewayConnection -Name "MyVirtualNetworkConnection" -ResourceGroupName "MyRG"
$connection.RoutingWeight = 100
Set-AzureRmVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection
```

*RoutingWeight* 的範圍是從 0 到 32000。 預設值為 0。

## <a name="next-steps"></a>後續步驟
如需有關 ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。
