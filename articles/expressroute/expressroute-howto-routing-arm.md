---
title: 如何設定 ExpressRoute 線路的路由 (對等互連)：Resource Manager：PowerShell：Azure | Microsoft Docs
description: 本文將逐步引導您為 ExpressRoute 線路建立和佈建私用、公用及 Microsoft 對等。 本文也示範如何檢查狀態、更新或刪除線路的對等。
documentationcenter: na
services: expressroute
author: osamazia
manager: jonor
editor: ''
tags: azure-resource-manager
ms.assetid: 0a036d51-77ae-4fee-9ddb-35f040fbdcdf
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 1/3/2018
ms.author: osamaz, jaredr80
ms.openlocfilehash: d4f8f0e6c8fab5dfb4efcc4f1659ba90336c8bf0
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31593877"
---
# <a name="create-and-modify-peering-for-an-expressroute-circuit-using-powershell"></a>使用 PowerShell 建立和修改 ExpressRoute 線路的對等互連

本文將協助您使用 PowerShell，在 Resource Manager 部署模型中建立和管理 ExpressRoute 線路的路由設定。 您還可以檢查狀態、更新，或是刪除與取消佈建 ExpressRoute 線路的對等互連。 如果您想要對線路使用不同的方法，可選取下列清單中的文章：

> [!div class="op_single_selector"]
> * [Azure 入口網站](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [Azure CLI](howto-routing-cli.md)
> * [視訊 - 私用對等互連](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [視訊 - 公用對等互連](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-public-peering-for-your-expressroute-circuit)
> * [視訊 - Microsoft 對等互連](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (傳統)](expressroute-howto-routing-classic.md)
> 

## <a name="configuration-prerequisites"></a>組態必要條件

* 您需要最新版的 Azure Resource Manager PowerShell Cmdlet。 如需詳細資訊，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/overview)。 
* 開始設定之前，請確定您已經檢閱過[必要條件](expressroute-prerequisites.md)頁面、[路由需求](expressroute-routing.md)頁面和[工作流程](expressroute-workflows.md)頁面。
* 您必須擁有作用中的 ExpressRoute 線路。 繼續之前，請遵循指示來 [建立 ExpressRoute 線路](expressroute-howto-circuit-arm.md) ，並由您的連線提供者來啟用該線路。 ExpressRoute 線路必須處於已佈建和已啟用狀態，您才能執行本文中的 Cmdlet。

這些指示只適用於由提供第 2 層連線服務的服務提供者所建立的線路。 如果您使用的服務提供者提供受控第 3 層服務 (通常是 IPVPN，如 MPLS)，連線提供者會為您設定和管理路由。

> [!IMPORTANT]
> 我們目前不會透過服務管理入口網站來公告服務提供者所設定的對等。 我們正努力在近期推出這項功能。 設定 BGP 對等互連之前，請洽詢您的服務提供者。
> 
> 

您可以為 ExpressRoute 線路設定一個、兩個或全部三個對等 (Azure 私用、Azure 公用和 Microsoft)。 您可以依自己選擇的任何順序設定對等。 不過，您必須確定一次只完成一個對等的設定。 如需路由網域和對等互連的詳細資訊，請參閱 [ExpressRoute 路由網域](expressroute-circuit-peerings.md)。

## <a name="msft"></a>Microsoft 對等互連

本節將協助您為 ExpressRoute 線路建立、取得、更新和刪除 Microsoft 對等互連設定。

> [!IMPORTANT]
> 在 2017 年 8 月 1 日以前設定之 ExpressRoute 線路的 Microsoft 對等互連，會透過 Microsoft 對等互連公告所有服務首碼，即使未定義路由篩選也一樣。 在 2017 年 8 月 1 日當日或以後設定之 ExpressRoute 線路的 Microsoft 對等互連，不會公告任何首碼，直到路由篩選連結至線路為止。 如需詳細資訊，請參閱[設定 Microsoft 對等互連的路由篩選](how-to-routefilter-powershell.md)。
> 
> 

### <a name="to-create-microsoft-peering"></a>建立 Microsoft 對等

1. 匯入 ExpressRoute 的 PowerShell 模組。

  您必須從 [PowerShell 資源庫](http://www.powershellgallery.com/) 安裝最新的 PowerShell 安裝程式並將 Azure Resource Manager 模組匯入至 PowerShell 工作階段，以開始使用 ExpressRoute Cmdlet。 您必須以系統管理員身分執行 PowerShell。

  ```powershell
  Install-Module AzureRM

  Install-AzureRM
  ```

  匯入已知語意版本範圍內所有的 AzureRM.* 模組。

  ```powershell
  Import-AzureRM
  ```

  您也可以只匯入已知語意版本範圍內選取的模組。

  ```powershell
  Import-Module AzureRM.Network
  ```

  登入您的帳戶。

  ```powershell
  Connect-AzureRmAccount
  ```

  選取您想要建立 ExpressRoute 線路的訂用帳戶。

  ```powershell
Select-AzureRmSubscription -SubscriptionId "<subscription ID>"
  ```
2. 建立 ExpressRoute 線路。

  請遵循指示建立 [ExpressRoute 線路](expressroute-howto-circuit-arm.md) ，並由連線提供者佈建它。 如果您的連線提供者提供受控第 3 層服務，您可以要求連線提供者為您啟用 Microsoft 對等互連。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不管理路由，請在建立線路之後繼續使用後續步驟進行設定。 

3. 檢查 ExpressRoute 線路，以確定已佈建且已啟用線路。 請使用下列範例：

  ```powershell
  Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
  ```

  回應如下列範例所示：

  ```
  Name                             : ExpressRouteARMCircuit
  ResourceGroupName                : ExpressRouteResourceGroup
  Location                         : westus
  Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
  Etag                             : W/"################################"
  ProvisioningState                : Succeeded
  Sku                              : {
                                       "Name": "Standard_MeteredData",
                                       "Tier": "Standard",
                                       "Family": "MeteredData"
                                     }
  CircuitProvisioningState         : Enabled
  ServiceProviderProvisioningState : Provisioned
  ServiceProviderNotes             : 
  ServiceProviderProperties        : {
                                       "ServiceProviderName": "Equinix",
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
  ServiceKey                       : **************************************
  Peerings                         : []
  ```
4. 設定線路的 Microsoft 對等。 繼續之前，請確定您擁有下列資訊。

  * 主要連結的 /30 或 /126 子網路。 這必須是您所擁有且註冊在 RIR / IRR 中的有效公用 IPv4或 IPv6 首碼。
  * 次要連結的 /30 或/126 子網路。 這必須是您所擁有且註冊在 RIR / IRR 中的有效公用 IPv4或 IPv6 首碼。
  * 供建立此對等的有效 VLAN ID。 請確定線路有沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。
  * 公告的首碼：您必須提供一份您打算在 BGP 工作階段上公告的所有首碼的清單。 只接受公用 IP 位址首碼。 如果計劃傳送一組首碼，可以傳送以逗號分隔的清單。 這些首碼必須在 RIR / IRR 中註冊給您。 IPv4 BGP 工作階段需要 IPv4 已公告首碼，而 IPv6 BGP 工作階段需要 IPv6 已公告首碼。 
  * 路由登錄名稱：您可以指定可供註冊 AS 編號和首碼的 RIR / IRR。
  * 選用：
    * 客戶 ASN：如果您要公告的首碼未註冊給對等 AS 編號，您可以指定它們所註冊的 AS 編號。
    * MD5 雜湊 (如果選擇使用)。

  使用下列範例來為線路設定 Microsoft 對等互連：

  ```powershell
  Add-AzureRmExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv4 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "123.1.0.0/24" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

  Add-AzureRmExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv6 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "3FFE:FFFF:0:CD31::/120" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

  Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
  ```

### <a name="getmsft"></a>取得 Microsoft 對等互連詳細資料

您可以使用下列範例來取得設定詳細資料：

```powershell
$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

Get-AzureRmExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt
```

### <a name="updatemsft"></a>更新 Microsoft 對等組態

您可以使用下列範例來更新設定的任何部分：

```powershell
Set-AzureRmExpressRouteCircuitPeeringConfig  -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv4 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "124.1.0.0/24" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

Set-AzureRmExpressRouteCircuitPeeringConfig  -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv6 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "3FFE:FFFF:0:CD31::/120" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="deletemsft"></a>刪除 Microsoft 對等

您可以執行下列 Cmdlet 來移除對等組態：

```powershell
Remove-AzureRmExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="private"></a>Azure 私人對等互連

本節將協助您為 ExpressRoute 線路建立、取得、更新和刪除 Azure 私用對等互連設定。

### <a name="to-create-azure-private-peering"></a>建立 Azure 私用對等

1. 匯入 ExpressRoute 的 PowerShell 模組。

  您必須從 [PowerShell 資源庫](http://www.powershellgallery.com/) 安裝最新的 PowerShell 安裝程式並將 Azure Resource Manager 模組匯入至 PowerShell 工作階段，以開始使用 ExpressRoute Cmdlet。 您必須以系統管理員身分執行 PowerShell。

  ```powershell
  Install-Module AzureRM
  Install-AzureRM
  ```

  匯入已知語意版本範圍內所有的 AzureRM.* 模組。

  ```powershell
  Import-AzureRM
  ```

  您也可以只匯入已知語意版本範圍內選取的模組。

  ```powershell
  Import-Module AzureRM.Network 
  ```

  登入您的帳戶。

  ```powershell
  Connect-AzureRmAccount
  ```

  選取您想要建立 ExpressRoute 線路的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionId "<subscription ID>"
  ```
2. 建立 ExpressRoute 線路。

  請遵循指示建立 [ExpressRoute 線路](expressroute-howto-circuit-arm.md) ，並由連線提供者佈建它。 若您的連線提供者是提供受控第 3 層服務，您可以要求連線提供者為您啟用 Azure 私人對等互連。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不管理路由，請在建立線路之後繼續使用後續步驟進行設定。

3. 檢查 ExpressRoute 線路，以確定已佈建且已啟用線路。 請使用下列範例：

  ```powershell
  Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
  ```

  回應如下列範例所示：

  ```
  Name                             : ExpressRouteARMCircuit
  ResourceGroupName                : ExpressRouteResourceGroup
  Location                         : westus
  Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
  Etag                             : W/"################################"
  ProvisioningState                : Succeeded
  Sku                              : {
                                       "Name": "Standard_MeteredData",
                                       "Tier": "Standard",
                                       "Family": "MeteredData"
                                     }
  CircuitProvisioningState         : Enabled
  ServiceProviderProvisioningState : Provisioned
  ServiceProviderNotes             : 
  ServiceProviderProperties        : {
                                       "ServiceProviderName": "Equinix",
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
  ServiceKey                       : **************************************
  Peerings                         : []
  ```
4. 設定線路的 Azure 私用對等。 繼續執行接下來的步驟之前，請確定您有下列項目：

  * 主要連結的 /30 子網路。 子網路不能在保留給虛擬網路的任何位址空間中。
  * 次要連結的 /30 子網路。 子網路不能在保留給虛擬網路的任何位址空間中。
  * 供建立此對等的有效 VLAN ID。 請確定線路有沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。 您可以將私用 AS 編號用於此對等。 請確定您不是使用 65515。
  * 選用：
    * MD5 雜湊 (如果選擇使用)。

  使用下列範例來為線路設定 Azure 私用對等互連：

  ```powershell
  Add-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200

  Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
  ```

  如果您選擇使用 MD5 雜湊，請使用下列範例：

  ```powershell
  Add-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200  -SharedKey "A1B2C3D4"
  ```

  > [!IMPORTANT]
  > 請確定您將 AS 編號指定為對等 ASN，而不是客戶 ASN。
  > 
  >

### <a name="getprivate"></a>取得 Azure 私用對等互連詳細資訊

您可以使用下列範例來取得設定詳細資料：

```powershell
$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

Get-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt
```

### <a name="updateprivate"></a>更新 Azure 私用對等組態

您可以使用下列範例來更新設定的任何部分。 在此範例中，線路的 VLAN ID 從 100 更新為 500。

```powershell
Set-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="deleteprivate"></a>刪除 Azure 私用對等

您可以執行下列範例來移除對等互連設定：

> [!WARNING]
> 執行此範例之前，您必須確定所有虛擬網路都已經與 ExpressRoute 線路取消連結。 
> 
> 

```powershell
Remove-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="public"></a>Azure 公用對等互連

本節將協助您為 ExpressRoute 線路建立、取得、更新和刪除 Azure 公用對等互連設定。

### <a name="to-create-azure-public-peering"></a>建立 Azure 公用對等

1. 匯入 ExpressRoute 的 PowerShell 模組。

  您必須從 [PowerShell 資源庫](http://www.powershellgallery.com/) 安裝最新的 PowerShell 安裝程式並將 Azure Resource Manager 模組匯入至 PowerShell 工作階段，以開始使用 ExpressRoute Cmdlet。 您必須以系統管理員身分執行 PowerShell。

  ```powershell
  Install-Module AzureRM

  Install-AzureRM
```

  匯入已知語意版本範圍內所有的 AzureRM.* 模組。

  ```powershell
  Import-AzureRM
  ```

  您也可以只匯入已知語意版本範圍內選取的模組。

  ```powershell
  Import-Module AzureRM.Network
```

  登入您的帳戶。

  ```powershell
  Connect-AzureRmAccount
  ```

  選取您想要建立 ExpressRoute 線路的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionId "<subscription ID>"
  ```
2. 建立 ExpressRoute 線路。

  請遵循指示建立 [ExpressRoute 線路](expressroute-howto-circuit-arm.md) ，並由連線提供者佈建它。 若您的連線提供者提供受控第 3 層服務，您可以要求連線提供者為您啟用 Azure 公用對等互連。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不管理路由，請在建立線路之後繼續使用後續步驟進行設定。

3. 檢查 ExpressRoute 線路，以確定已佈建且已啟用線路。 請使用下列範例：

  ```powershell
  Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
  ```

  回應如下列範例所示：

  ```
  Name                             : ExpressRouteARMCircuit
  ResourceGroupName                : ExpressRouteResourceGroup
  Location                         : westus
  Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
  Etag                             : W/"################################"
  ProvisioningState                : Succeeded
  Sku                              : {
                                      "Name": "Standard_MeteredData",
                                       "Tier": "Standard",
                                       "Family": "MeteredData"
                                     }
  CircuitProvisioningState         : Enabled
  ServiceProviderProvisioningState : Provisioned
  ServiceProviderNotes             : 
  ServiceProviderProperties        : {
                                       "ServiceProviderName": "Equinix",
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
  ServiceKey                       : **************************************
  Peerings                         : []
  ```
4. 設定線路的 Azure 公用對等。 進一步執行之前，請確定您具有下列資訊。

  * 主要連結的 /30 子網路。 這必須是有效的公用 IPv4 首碼。
  * 次要連結的 /30 子網路。 這必須是有效的公用 IPv4 首碼。
  * 供建立此對等的有效 VLAN ID。 請確定線路有沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。
  * 選用：
    * MD5 雜湊 (如果選擇使用)。

  執行下列範例來為線路設定 Azure 公用對等互連

  ```powershell
  Add-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "12.0.0.0/30" -SecondaryPeerAddressPrefix "12.0.0.4/30" -VlanId 100

  Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
  ```

  如果您選擇使用 MD5 雜湊，請使用下列範例：

  ```powershell
  Add-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "12.0.0.0/30" -SecondaryPeerAddressPrefix "12.0.0.4/30" -VlanId 100  -SharedKey "A1B2C3D4"

  Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
  ```

  > [!IMPORTANT]
  > 請確定您將 AS 編號指定為對等 ASN，而不是客戶 ASN。
  > 
  >

### <a name="getpublic"></a>取得 Azure 公用對等互連詳細資訊

您可以使用下列 Cmdlet 來取得組態詳細資料：

```powershell
  $ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

  Get-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -Circuit $ckt
  ```

### <a name="updatepublic"></a>更新 Azure 公用對等組態

您可以使用下列範例來更新設定的任何部分。 在此範例中，線路的 VLAN ID 從 200 更新為 600。

```powershell
Set-AzureRmExpressRouteCircuitPeeringConfig  -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 600

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="deletepublic"></a>刪除 Azure 公用對等

您可以執行下列範例來移除對等互連設定：

```powershell
Remove-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt
Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="next-steps"></a>後續步驟

下一步， [將 VNet 連結到 ExpressRoute 線路](expressroute-howto-linkvnet-arm.md)。

* 如需 ExpressRoute 工作流程的詳細資訊，請參閱 [ExpressRoute 工作流程](expressroute-workflows.md)。
* 如需線路對等的詳細資訊，請參閱 [ExpressRoute 線路和路由網域](expressroute-circuit-peerings.md)。
* 如需使用虛擬網路的詳細資訊，請參閱 [虛擬網路概觀](../virtual-network/virtual-networks-overview.md)。
