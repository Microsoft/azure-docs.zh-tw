---
title: Azure ExpressRoute 觸角擴及全球 | Microsoft Docs
description: 本文可協助您將 ExpressRoute 線路連結在一起，在內部部署網路之間產生私人網路，並使觸角擴及全球。
documentationcenter: na
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/21/2018
ms.author: cherylmc
ms.openlocfilehash: 90f34da10e0193e31a0f70187463799013f52be8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46966810"
---
# <a name="link-expressroute-circuits-to-enable-expressroute-global-reach-preview"></a>連結 ExpressRoute 線路，使 ExpressRoute 觸角擴及全球 (預覽)

ExpressRoute 是私人且有彈性的方式，可讓您將內部部署網路連線到 Microsoft Cloud。 您可以從私人資料中心或公司網路存取許多 Microsoft 雲端服務，例如 Azure、Office 365 和 Dynamics 365。 例如，您在舊金山分公司的 ExpressRoute 線路可能在美國矽谷，而另一個巴爾的摩分公司的 ExpressRoute 線路則在華盛頓特區。 

這兩個分公司可能都有在美國東部和美國西部的 Azure 資源高速連線。不過，分公司之間無法直接交換資料。 換句話說，10.0.1.0/24 可以與 10.0.3.0/24 和 10.0.4.0/24 交換資料，但「不」可與 10.0.2.0/24 交換資料。

![不含][1]

透過 **ExpressRoute 觸角擴及全球**，您可以將 ExpressRoute 線路連結在一起，以在內部部署網路之間產生私人資料網路。 在上述範例中，透過新增 ExpressRoute 觸角擴及全球，舊金山辦公室 (10.0.1.0/24) 可以透過現有的 ExpressRoute 線路及透過 Microsoft 的全球網路，直接與巴爾的摩辦公室 (10.0.2.0/24) 交換資料。 

![含有][2]

下列國家/地區目前支援 ExpressRoute 觸角擴及全球：

* 美國
* 英國 
* 日本

您必須在上述國家/地區的 [ExpressRoute 對等互連位置](expressroute-locations.md)建立 ExpressRoute 線路。 若要在[不同的地緣政治區域](expressroute-locations.md)之間啟用 ExpressRoute 觸角擴及全球，您的線路必須是進階 SKU。


## <a name="before-you-begin"></a>開始之前

> [!IMPORTANT]
> 此公開預覽版是在沒有服務等級協定的情況下提供，不得用於生產工作負載。 可能不支援特定功能、可能已經限制功能，或者可能無法在所有 Azure 位置提供使用。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。
>

您必須先檢查下列需求，才能開始設定。

* 安裝最新版的 Azure PowerShell。 請參閱[安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)。
* 了解 ExpressRoute 線路佈建[工作流程](expressroute-workflows.md)。
* 請確定 ExpressRoute 線路處於已佈建狀態。
* 請確定 ExpressRoute 線路上已設定 Azure 私用對等互連。  

### <a name="log-into-your-azure-account"></a>登入 Azure 帳戶
若要開始這項設定，您必須登入 Azure 帳戶。 

以提高的權限開啟 PowerShell 主控台並連線至您的帳戶。 此命令會提示您輸入 Azure 帳戶的登入認證。  

```powershell
Connect-AzureRmAccount
```

如果您有多個 Azure 訂用帳戶，請檢查帳戶的訂用帳戶。

```powershell
Get-AzureRmSubscription
```

指定您要使用的訂用帳戶。

```powershell
Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"
```

### <a name="identify-your-expressroute-circuits-for-configuration"></a>識別要設定的 ExpressRoute 線路
您可以在任兩個 ExpressRoute 線路之間啟用 ExpressRoute 觸角擴及全球，前提是它們位於支援的國家/地區，且建立在不同的對等互連位置。 如果兩個線路皆為訂用帳戶所擁有，則可以選擇使用任一線路來執行下列各節中的設定。 如果兩個線路位於不同的 Azure 訂用帳戶中，則需要其中一個 Azure 訂用帳戶的授權，並在您於另一個 Azure 訂用帳戶中執行設定命令時傳入授權金鑰。

## <a name="to-enable-connectivity-between-your-on-premises-networks"></a>在內部部署網路之間啟用連線

使用下列命令來取得線路 1 和線路 2。 這兩個線路位於相同的訂用帳戶中。

```powershell
$ckt_1 = Get-AzureRmExpressRouteCircuit -Name "Your_circuit_1_name" -ResourceGroupName "Your_resource_group"
$ckt_2 = Get-AzureRmExpressRouteCircuit -Name "Your_circuit_2_name" -ResourceGroupName "Your_resource_group"
```

針對線路 1 執行下列命令，並傳入線路 2 私用對等互連的識別碼。

請注意，私用對等互連的識別碼看起來像這樣：

`/subscriptions/{your_subscription_id}/resourceGroups/{your_resource_group}/providers/Microsoft.Network/expressRouteCircuits/{your_circuit_name}/peerings/AzurePrivatePeering`


```powershell
Add-AzureRmExpressRouteCircuitConnectionConfig -Name 'Your_connection_name' -ExpressRouteCircuit $ckt_1 -PeerExpressRouteCircuitPeering $ckt_2.Peerings[0].Id -AddressPrefix '__.__.__.__/29'
```

-AddressPrefix 必須是 /29 IPv4 子網路，例如 "10.0.0.0/29"。 我們將使用此子網路中的 IP 位址在兩個 ExpressRoute 線路之間建立連線。 您不應該在 Azure VNet 或內部部署網路中使用此子網路中的位址。 


在線路 1 上儲存設定。

```powershell
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt_1
```

先前的作業完成時，您應該就可在內部部署網路的兩端，透過兩個 ExpressRoute 線路連線到另一端。

### <a name="expressroute-circuits-in-different-azure-subscriptions"></a>在不同 Azure 訂用帳戶中的 ExpressRoute 線路

如果兩個線路不在相同的 Azure 訂用帳戶中，您將需要授權。 在下列設定中，線路 2 的訂用帳戶中會產生授權，且授權金鑰會傳遞至線路 1。

產生授權金鑰。 
```powershell
$ckt_2 = Get-AzureRmExpressRouteCircuit -Name "Your_circuit_2_name" -ResourceGroupName "Your_resource_group"
Add-AzureRmExpressRouteCircuitAuthorization -ExpressRouteCircuit $ckt_2 -Name "Name_for_auth_key"
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt_2
```
請記下線路 2 的私用對等互連識別碼，以及授權金鑰。

針對線路 1 執行下列命令。 傳入線路 2 的私用對等互連識別碼以及授權金鑰

```powershell
Add-AzureRmExpressRouteCircuitConnectionConfig -Name 'Your_connection_name' -ExpressRouteCircuit $ckt_1 -PeerExpressRouteCircuitPeering "circuit_2_private_peering_id" -AddressPrefix '__.__.__.__/29' -AuthorizationKey '########-####-####-####-############'
```

在線路 1 上儲存設定。

```powershell
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt_1
```

先前的作業完成時，您應該就可在內部部署網路的兩端，透過兩個 ExpressRoute 線路連線到另一端。

## <a name="to-get-and-verify-the-configuration"></a>取得及驗證設定

使用下列命令來驗證所設定線路上的設定。 此範例會使用上一個範例中的線路 1。

```powershell
$ckt1 = Get-AzureRmExpressRouteCircuit -Name "Your_circuit_1_name" -ResourceGroupName "Your_resource_group"
```

如果您在 PowerShell 中僅執行了 $ckt1，則會在輸出中看到 CircuitConnectionStatus。 它會告訴您連線狀態是已建立 (「已連線」) 還是未建立 (「已中斷連線」)。 

## <a name="to-disable-connectivity-between-your-on-premises-networks"></a>在內部部署網路之間停用連線

若要停用，請針對所設定的線路執行命令。 此範例會使用前面範例中的線路 1。

```powershell
$ckt1 = Get-AzureRmExpressRouteCircuit -Name "Your_circuit_1_name" -ResourceGroupName "Your_resource_group"
Remove-AzureRmExpressRouteCircuitConnectionConfig -Name "Your_connection_name" -ExpressRouteCircuit $ckt_1
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt_1
```

您可以執行 Get 作業以驗證狀態。 

在上述作業完成後，您就無法再透過 ExpressRoute 線路，於內部部署網路之間進行連線。 

## <a name="next-steps"></a>後續步驟
1. [深入了解 ExpressRoute 觸角擴及全球](expressroute-faqs.md#globalreach)
2. [驗證 ExpressRoute 連線](expressroute-troubleshooting-expressroute-overview.md)
3. [將 ExpressRoute 線路連結到 Azure 虛擬網路](expressroute-howto-linkvnet-arm.md)


<!--Image References-->
[1]: ./media/expressroute-global-reach/1.png "不含觸角擴及全球的圖表"
[2]: ./media/expressroute-global-reach/2.png "包含觸角擴及全球的圖表"