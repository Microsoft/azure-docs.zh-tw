---
title: 如何設定 ExpressRoute 線路的路由 (對等互連)：Azure：傳統 | Microsoft Docs
description: 本文將逐步引導您為 ExpressRoute 線路建立和佈建私用、公用及 Microsoft 對等。 本文也示範如何檢查狀態、更新或刪除線路的對等。
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 07/27/2018
ms.author: cherylmc;ganesr
ms.openlocfilehash: 14e96a36eed99d9967ac6f8188a161c922d6f334
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39343656"
---
# <a name="create-and-modify-peering-for-an-expressroute-circuit-classic"></a>建立和修改 ExpressRoute 線路的對等互連 (傳統)
> [!div class="op_single_selector"]
> * [Azure 入口網站](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [Azure CLI](howto-routing-cli.md)
> * [視訊 - 私用對等互連](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [視訊 - 公用對等互連](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-public-peering-for-your-expressroute-circuit)
> * [視訊 - Microsoft 對等互連](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (傳統)](expressroute-howto-routing-classic.md)
> 

本文將逐步引導您使用 PowerShell 和傳統部署模型，以建立和管理 ExpressRoute 電路的路由組態。 下列步驟也會示範如何檢查狀態、更新或刪除和取消佈建 ExpressRoute 線路的對等。 您可以為 ExpressRoute 線路設定一個、兩個或全部三個對等 (Azure 私用、Azure 公用和 Microsoft)。 您可以依自己選擇的任何順序設定對等。 不過，您必須確定一次只完成一個對等的設定。 

這些指示只適用於由提供第 2 層連線服務的服務提供者所建立的線路。 如果您使用的服務提供者提供受控第 3 層服務 (通常是 IPVPN，如 MPLS)，連線提供者會為您設定和管理路由。

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**關於 Azure 部署模型**

[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## <a name="configuration-prerequisites"></a>組態必要條件

* 開始設定之前，請確定您已經檢閱過[必要條件](expressroute-prerequisites.md)頁面、[路由需求](expressroute-routing.md)頁面和[工作流程](expressroute-workflows.md)頁面。
* 您必須擁有作用中的 ExpressRoute 線路。 繼續之前，請遵循指示來 [建立 ExpressRoute 線路](expressroute-howto-circuit-classic.md) ，並由您的連線提供者來啟用該線路。 ExpressRoute 線路必須處於已佈建和已啟用狀態，您才能執行如下所述的 Cmdlet。

### <a name="download-the-latest-powershell-cmdlets"></a>下載最新的 PowerShell Cmdlet

安裝最新版的 Azure 服務管理 (SM) PowerShell 模組和 ExpressRoute 模組。 當您使用下列範例時，請注意版本號碼 (此範例中為 5.1.1) 會因為發行較新的 Cmdlet 版本而變更。

```powershell
Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\Azure\Azure.psd1'
Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\ExpressRoute\ExpressRoute.psd1'
```

如需詳細資訊，請參閱[開始使用 Azure PowerShell Cmdlet](/powershell/azure/overview) 中的指示，來取得如何設定您的電腦以使用 Azure PowerShell 模組的逐步指導方針。

### <a name="sign-in"></a>登入

若要登入您的 Azure 帳戶，請使用下列範例：

1. 以提高的權限開啟 PowerShell 主控台並連接到您的帳戶。

  ```powershell
  Connect-AzureRmAccount
  ```
2. 檢查帳戶的訂用帳戶。

  ```powershell
  Get-AzureRmSubscription
  ```
3. 如果您有多個訂用帳戶，請選取您要使用的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"
  ```

4. 接下來，使用下列 Cmdlet，將您的 Azure 訂用帳戶新增到 PowerShell，以供傳統部署模型使用。

  ```powershell
  Add-AzureAccount
  ```

## <a name="azure-private-peering"></a>Azure 私用對等

本節提供如何為 ExpressRoute 線路建立、取得、更新和刪除 Azure 私用對等組態的指示。 

### <a name="to-create-azure-private-peering"></a>建立 Azure 私用對等

1. **建立 ExpressRoute 線路。**

  請遵循指示建立 [ExpressRoute 線路](expressroute-howto-circuit-classic.md) ，並由連線提供者佈建它。 如果您的連線提供者是提供受控第 3 層服務，您可以要求連線提供者為您啟用 Azure 私用對等。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不會為您管理路由，請在建立線路之後遵循下列指示。
2. **檢查 ExpressRoute 線路以確定已佈建。**
   
  檢查 ExpressRoute 線路是否為 Provisioned 和 Enabled。

  ```powershell
  Get-AzureDedicatedCircuit -ServiceKey "*********************************"
  ```

  傳回：

  ```powershell
  Bandwidth                        : 200
  CircuitName                      : MyTestCircuit
  Location                         : Silicon Valley
  ServiceKey                       : *********************************
  ServiceProviderName              : equinix
  ServiceProviderProvisioningState : Provisioned
  Sku                              : Standard
  Status                           : Enabled
  ```
   
  請確定線路顯示為 Provisioned 和 Enabled。 如果不是，請與連線提供者合作，讓線路變成所需的狀態。

  ```powershell
  ServiceProviderProvisioningState : Provisioned
  Status                           : Enabled
  ```
3. **設定線路的 Azure 私用對等。**

  繼續執行接下來的步驟之前，請確定您有下列項目：
   
  * 主要連結的 /30 子網路。 這不能在保留給虛擬網路的任何位址空間中。
  * 次要連結的 /30 子網路。 這不能在保留給虛擬網路的任何位址空間中。
  * 供建立此對等的有效 VLAN ID。 確定線路中沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。 您可以將私用 AS 編號用於此對等。 請確定您不是使用 65515。
  * MD5 雜湊 (如果選擇使用)。 **選擇性**。
     
  您可以使用下列範例來為線路設定 Azure 私用對等互連：

  ```powershell
  New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100
  ```    

  如果您想要使用 MD5 雜湊，可以使用下列範例來為線路設定私用對等互連：

  ```powershell
  New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100 -SharedKey "A1B2C3D4"
  ```
     
  > [!IMPORTANT]
  > 請確定您將 AS 編號指定為對等 ASN，而不是客戶 ASN。
  > 

### <a name="to-view-azure-private-peering-details"></a>檢視 Azure 私用對等詳細資訊

您可以使用下列 Cmdlet 來檢視設定詳細資料：

```powershell
Get-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"
```

傳回：

```
AdvertisedPublicPrefixes       : 
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 10.0.0.0/30
RoutingRegistryName            : 
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 10.0.0.4/30
State                          : Enabled
VlanId                         : 100
```

### <a name="to-update-azure-private-peering-configuration"></a>更新 Azure 私用對等組態

您可以使用下列 Cmdlet 來更新組態的任何部分。 在下列範例中，線路的 VLAN ID 從 100 更新為 500。

```powershell
Set-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 500 -SharedKey "A1B2C3D4"
```

### <a name="to-delete-azure-private-peering"></a>刪除 Azure 私用對等

您可以執行下列 Cmdlet 來移除對等組態。 執行此 Cmdlet 之前，您必須確定所有虛擬網路都已經與 ExpressRoute 取消連結。

```powershell
Remove-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"
```

## <a name="azure-public-peering"></a>Azure 公用對等

本節提供如何為 ExpressRoute 線路建立、取得、更新和刪除 Azure 公用對等組態的指示。

### <a name="to-create-azure-public-peering"></a>建立 Azure 公用對等

1. **建立 ExpressRoute 線路**

  請遵循指示建立 [ExpressRoute 線路](expressroute-howto-circuit-classic.md) ，並由連線提供者佈建它。 如果您的連線提供者提供受控第 3 層服務，您可以要求連線提供者為您啟用 Azure 公用對等。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不會為您管理路由，請在建立線路之後遵循下列指示。
2. **檢查 ExpressRoute 線路以確定已佈建**

  您必須先檢查 ExpressRoute 線路是否為 Provisioned 和 Enabled。

  ```powershell
  Get-AzureDedicatedCircuit -ServiceKey "*********************************"
  ```

  傳回：

  ```powershell
  Bandwidth                        : 200
  CircuitName                      : MyTestCircuit
  Location                         : Silicon Valley
  ServiceKey                       : *********************************
  ServiceProviderName              : equinix
  ServiceProviderProvisioningState : Provisioned
  Sku                              : Standard
  Status                           : Enabled
  ```
   
  請確定線路顯示為 Provisioned 和 Enabled。 如果不是，請與連線提供者合作，讓線路變成所需的狀態。

  ```powershell
  ServiceProviderProvisioningState : Provisioned
  Status                           : Enabled
  ```
4. **設定線路的 Azure 公用對等**
   
  繼續之前，請確定您具有下列資訊：
   
  * 主要連結的 /30 子網路。 這必須是有效的公用 IPv4 首碼。
  * 次要連結的 /30 子網路。 這必須是有效的公用 IPv4 首碼。
  * 供建立此對等的有效 VLAN ID。 確定線路中沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。
  * MD5 雜湊 (如果選擇使用)。 **選擇性**。

  > [!IMPORTANT]
  > 請確定您將 AS 編號指定為對等 ASN，而不是客戶 ASN。
  >  
     
  您可以執行下列範例來為線路設定 Azure 公用對等互連：

  ```powershell
  New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200
  ```
     
  如果您想要使用 MD5 雜湊，可以使用下列範例來設定線路：
     
  ```powershell
  New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200 -SharedKey "A1B2C3D4"
  ```
     
### <a name="to-view-azure-public-peering-details"></a>檢視 Azure 公用對等詳細資訊

若要檢視設定詳細資料，請使用下列 Cmdlet：

```powershell
Get-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"
```

傳回：

```powershell
AdvertisedPublicPrefixes       : 
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 131.107.0.0/30
RoutingRegistryName            : 
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 131.107.0.4/30
State                          : Enabled
VlanId                         : 200
```

### <a name="to-update-azure-public-peering-configuration"></a>更新 Azure 公用對等組態

您可以使用下列 Cmdlet 來更新組態的任何部分。 在此範例中，線路的 VLAN ID 從 200 更新為 600。

```powershell
Set-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 600 -SharedKey "A1B2C3D4"
```

請確定線路顯示為 Provisioned 和 Enabled。 
### <a name="to-delete-azure-public-peering"></a>刪除 Azure 公用對等

您可以執行下列 Cmdlet 來移除對等組態：

```powershell
Remove-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"
```

## <a name="microsoft-peering"></a>Microsoft 對等互連

本節提供如何為 ExpressRoute 線路建立、取得、更新和刪除 Microsoft 對等組態的指示。 

### <a name="to-create-microsoft-peering"></a>建立 Microsoft 對等

1. **建立 ExpressRoute 線路**
  
  請遵循指示建立 [ExpressRoute 線路](expressroute-howto-circuit-classic.md) ，並由連線提供者佈建它。 如果您的連線提供者是提供受控第 3 層服務，您可以要求連線提供者為您啟用 Azure 私用對等。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不會為您管理路由，請在建立線路之後遵循下列指示。
2. **檢查 ExpressRoute 線路以確定已佈建**

  請確定線路顯示為 Provisioned 和 Enabled。 
   
  ```powershell
  Get-AzureDedicatedCircuit -ServiceKey "*********************************"
  ```

  傳回：
   
  ```powershell
  Bandwidth                        : 200
  CircuitName                      : MyTestCircuit
  Location                         : Silicon Valley
  ServiceKey                       : *********************************
  ServiceProviderName              : equinix
  ServiceProviderProvisioningState : Provisioned
  Sku                              : Standard
  Status                           : Enabled
  ```
   
  請確定線路顯示為 Provisioned 和 Enabled。 如果不是，請與連線提供者合作，讓線路變成所需的狀態。

  ```powershell
  ServiceProviderProvisioningState : Provisioned
  Status                           : Enabled
  ```
3. **設定線路的 Microsoft 對等**
   
    繼續之前，請確定您擁有下列資訊：
   
   * 主要連結的 /30 子網路。 這必須是您所擁有且註冊在 RIR / IRR 中的有效公用 IPv4 首碼。
   * 次要連結的 /30 子網路。 這必須是您所擁有且註冊在 RIR / IRR 中的有效公用 IPv4 首碼。
   * 供建立此對等的有效 VLAN ID。 確定線路中沒有其他對等使用相同的 VLAN ID。
   * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。
   * 公告的首碼：您必須提供一份您打算在 BGP 工作階段上公告的所有首碼的清單。 只接受公用 IP 位址首碼。 如果您打算傳送一組首碼，您可以傳送逗號分隔清單。 這些首碼必須在 RIR / IRR 中註冊給您。
   * 客戶 ASN：如果您要公告的首碼未註冊給對等 AS 編號，您可以指定它們所註冊的 AS 編號。 **選擇性**。
   * 路由登錄名稱：您可以指定可供註冊 AS 編號和首碼的 RIR / IRR。
   * MD5 雜湊 (如果選擇使用)。 **選用。**
     
  執行下列 Cmdlet 來為線路設定 Microsoft 對等：
 
  ```powershell
  New-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -VlanId 300 -PeerAsn 1234 -CustomerAsn 2245 -AdvertisedPublicPrefixes "123.0.0.0/30" -RoutingRegistryName "ARIN" -SharedKey "A1B2C3D4"
  ```

### <a name="to-view-microsoft-peering-details"></a>檢視 Microsoft 對等詳細資訊

您可以使用下列 Cmdlet 來檢視設定詳細資料：

```powershell
Get-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"
```
傳回：

```powershell
AdvertisedPublicPrefixes       : 123.0.0.0/30
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 2245
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 10.0.0.0/30
RoutingRegistryName            : ARIN
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 10.0.0.4/30
State                          : Enabled
VlanId                         : 300
```

### <a name="to-update-microsoft-peering-configuration"></a>更新 Microsoft 對等組態

您可以使用下列 Cmdlet 來更新組態的任何部分：

```powershell
Set-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -VlanId 300 -PeerAsn 1234 -CustomerAsn 2245 -AdvertisedPublicPrefixes "123.0.0.0/30" -RoutingRegistryName "ARIN" -SharedKey "A1B2C3D4"
```

### <a name="to-delete-microsoft-peering"></a>刪除 Microsoft 對等

您可以執行下列 Cmdlet 來移除對等組態：

```powershell
Remove-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"
```

## <a name="next-steps"></a>後續步驟

接著， [將 VNet 連結到 ExpressRoute 線路](expressroute-howto-linkvnet-classic.md)。

* 如需有關工作流程的詳細資訊，請參閱 [ExpressRoute 工作流程](expressroute-workflows.md)。
* 如需線路對等的詳細資訊，請參閱 [ExpressRoute 線路和路由網域](expressroute-circuit-peerings.md)。