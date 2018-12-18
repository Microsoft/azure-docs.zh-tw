---
title: 如何設定 Azure ExpressRoute Direct | Microsoft Docs
description: 此頁面可協助您設定 ExpressRoute Direct (預覽)
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/21/2018
ms.author: cherylmc
ms.openlocfilehash: e0009791263c45e0172abcb4836aaadde26f3ace
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48887184"
---
# <a name="how-to-configure-expressroute-direct-preview"></a>如何設定 ExpressRoute Direct (預覽)

ExpressRoute Direct 可讓您在策略性分散於世界各地的對等互連位置，直接連線至 Microsoft 的全球網路。 如需詳細資訊，請參閱[關於 ExpressRoute Direct Connect](expressroute-erdirect-about.md)。

> [!IMPORTANT]
> ExpressRoute Direct 目前為預覽狀態。
>
> 此公開預覽版是在沒有服務等級協定的情況下提供，不得用於生產工作負載。 可能不支援特定功能、可能已經限制功能，或者可能無法在所有 Azure 位置提供使用。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="resources"></a>1.建立資源

1. 登入 Azure 並選取訂用帳戶。 ExpressRoute Direct 資源和 ExpressRoute 線路必須位於相同的訂用帳戶中。

  ```powershell
  Connect-AzureRMAccount 

  Select-AzureRMSubscription -Subscription “<SubscriptionID or SubscriptionName>”
  ```
2. 列出支援 ExpressRoute Direct 的所有位置。
  
  ```powershell
  Get-AzureRmExpressRoutePortsLocation
  ```

  **範例輸出**
  
  ```powershell
  Name                : Equinix-Ashburn-DC2
  Id                  : /subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Ashburn-D
                        C2
  ProvisioningState   : Succeeded
  Address             : 21715 Filigree Court, DC2, Building F, Ashburn, VA 20147
  Contact             : support@equinix.com
  AvailableBandwidths : []

  Name                : Equinix-Dallas-DA3
  Id                  : /subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Dallas-DA
                        3
  ProvisioningState   : Succeeded
  Address             : 1950 N. Stemmons Freeway, Suite 1039A, DA3, Dallas, TX 75207
  Contact             : support@equinix.com
  AvailableBandwidths : []

  Name                : Equinix-San-Jose-SV1
  Id                  : /subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-San-Jose-
                        SV1
  ProvisioningState   : Succeeded
  Address             : 11 Great Oaks Blvd, SV1, San Jose, CA 95119
  Contact             : support@equinix.com
  AvailableBandwidths : []
  ```
3. 判斷以上所列的位置是否有可用的頻寬

  ```powershell
  Get-AzureRMExpressRoutePortsLocations -Name "Equinix-San-Jose-SV1"
  ```

  **範例輸出**

  ```powershell
  Name                : Equinix-San-Jose-SV1
  Id                  : /subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-San-Jose-
                        SV1
  ProvisioningState   : Succeeded
  Address             : 11 Great Oaks Blvd, SV1, San Jose, CA 95119
  Contact             : support@equinix.com
  AvailableBandwidths : [
                          {
                            "OfferName": "100 Gbps",
                            "ValueInGbps": 100
                          }
                        ]
  ```
4. 根據以上所選的位置建立 ExpressRoute Direct 資源

  ExpressRoute Direct 支援 QinQ 與 Dot1Q 封裝。 如果選取 QinQ，則每個 ExpressRoute 線路都會動態獲得指派的 S-Tag，並將成為整個 ExpressRoute Direct 資源中唯一的。 線路上的每個 C-Tag 必須是線路上唯一的，但不是 ExpressRoute Direct 上唯一的。  

  如果選取 Dot1Q 封裝，您必須管理 C-Tag (VLAN) 在整個 ExpressRoute Direct 資源中的唯一性。  

  > [!IMPORTANT]
  > ExpressRoute Direct 只能是一種封裝類型。 建立 ExpressRoute Direct 之後，就無法變更封裝。
  > 
 
  ```powershell 
  $ERDirect = New-AzureRMExpressRoutePort -Name $Name -ResourceGroupName -$RGName -PeeringLocation $PeeringLocationName -BandwidthInGbps 100.0 -Encapsulation QinQ | Dot1Q -Location $AzureRegion
  ```

  > [!NOTE]
  > 封裝屬性也可以設定為 Dot1Q。 
  >

  **範例輸出︰**

  ```powershell
  Name                       : Contoso-Direct
  ResourceGroupName          : Contoso-Direct-rg
  Location                   : westcentralus
  Id                         : /subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/exp
                               ressRoutePorts/Contoso-Direct
  Etag                       : W/"<etagnumber> "
  ResourceGuid               : <number>
  ProvisioningState          : Succeeded
  PeeringLocation            : Equinix-Seattle-SE2
  BandwidthInGbps            : 100
  ProvisionedBandwidthInGbps : 0
  Encapsulation              : QinQ
  Mtu                        : 1500
  EtherType                  : 0x8100
  AllocationDate             : Saturday, September 1, 2018
  Links                      : [
                                 {
                                   "Name": "link1",
                                   "Etag": "W/\"<etagnumber>\"",
                                   "Id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.
                               Network/expressRoutePorts/Contoso-Direct/links/link1",
                                   "RouterName": "tst-09xgmr-cis-1",
                                   "InterfaceName": "HundredGigE2/2/2",
                                   "PatchPanelId": "PPID",
                                   "RackId": "RackID",
                                   "ConnectorType": "SC",
                                   "AdminState": "Disabled",
                                   "ProvisioningState": "Succeeded"
                                 },
                                 {
                                   "Name": "link2",
                                   "Etag": "W/\"<etagnumber>\"",
                                   "Id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.
                               Network/expressRoutePorts/Contoso-Direct/links/link2",
                                   "RouterName": "tst-09xgmr-cis-2",
                                   "InterfaceName": "HundredGigE2/2/2",
                                   "PatchPanelId": "PPID",
                                   "RackId": "RackID",
                                   "ConnectorType": "SC",
                                   "AdminState": "Disabled",
                                   "ProvisioningState": "Succeeded"
                                 }
                               ]
  Circuits                   : []
  ```

## <a name="state"></a>2.變更連結的管理狀態

此程序應用於進行第 1 層測試，確保每個交叉連線都已在每個主要和次要路由器中正確修補。

1. 取得 ExpressRoute Direct 詳細資料。

  ```powershell
  $ERDirect = Get-AzureRmExpressRoutePort -Name $Name -ResourceGroupName $ResourceGroupName
  ```
2. 將 [連結] 設定為 [已啟用]。 重複此步驟，將每個連結設定為 [已啟用]。

  連結 [0] 是主要連接埠，而連結 [1] 是次要連接埠。

  ```powershell
  $ERDirect.Links[0].AdminState = “Enabled”
  Set-AzureRmExpressRoutePort -ExpressRoutePort $ERDirect
  $ERDirect = Get-AzureRmExpressRoutePort -Name $Name -ResourceGroupName $ResourceGroupName
  $ERDirect.Links[1].AdminState = “Enabled”
  Set-AzureRmExpressRoutePort -ExpressRoutePort $ERDirect
  ```
  **範例輸出︰**

  ```powershell
  Name                       : Contoso-Direct
  ResourceGroupName          : Contoso-Direct-rg
  Location                   : westcentralus
  Id                         : /subscriptions/<number>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/exp
                             ressRoutePorts/Contoso-Direct
  Etag                       : W/"<etagnumber> "
  ResourceGuid               : <number>
  ProvisioningState          : Succeeded
  PeeringLocation            : Equinix-Seattle-SE2
  BandwidthInGbps            : 100
  ProvisionedBandwidthInGbps : 0
  Encapsulation              : QinQ
  Mtu                        : 1500
  EtherType                  : 0x8100
  AllocationDate             : Saturday, September 1, 2018
  Links                      : [
                               {
                                 "Name": "link1",
                                 "Etag": "W/\"<etagnumber>\"",
                                 "Id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.
                             Network/expressRoutePorts/Contoso-Direct/links/link1",
                                 "RouterName": "tst-09xgmr-cis-1",
                                 "InterfaceName": "HundredGigE2/2/2",
                                 "PatchPanelId": "PPID",
                                 "RackId": "RackID",
                                 "ConnectorType": "SC",
                                 "AdminState": "Enabled",
                                 "ProvisioningState": "Succeeded"
                               },
                               {
                                 "Name": "link2",
                                 "Etag": "W/\"<etagnumber>\"",
                                 "Id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.
                             Network/expressRoutePorts/Contoso-Direct/links/link2",
                                 "RouterName": "tst-09xgmr-cis-2",
                                 "InterfaceName": "HundredGigE2/2/2",
                                 "PatchPanelId": "PPID",
                                 "RackId": "RackID",
                                 "ConnectorType": "SC",
                                 "AdminState": "Enabled",
                                 "ProvisioningState": "Succeeded"
                               }
                             ]
  Circuits                   : []
  ```

使用與 `AdminState = “Disabled”` 相同的程序來關閉連接埠。

## <a name="circuit"></a>3.建立線路

根據預設，您可以在 ExpressRoute Direct 資源所在的訂用帳戶中建立 10 個線路。 支援人員可以增加此數目。 您則負責追蹤已佈建和已使用的頻寬。 已佈建的頻寬是 ExpressRoute Direct 資源上所有線路的頻寬總和，而已使用的頻寬則是基礎實體介面的實際使用量。

ExpressRoute Direct 上有只可用於支援以上所述案例的額外線路頻寬。 這些是：40Gbps 和 100Gbps。

您可以建立標準或進階線路。 標準線路會納入成本中，而進階線路則是根據所選的頻寬計算成本。 只能依照計量建立線路，因為 ExpressRoute Direct 不支援無限量。

在 ExpressRoute Direct 資源上建立線路。

```powershell
New-AzureRmExpressRouteCircuit -Name $Name -ResourceGroupName $ResourceGroupName -ExpressRoutePort $ERDirect -BandwidthinGbps 1.0 | 2.0 | 5.0 | 10.0 | 40.0 | 100.0  -Location $AzureRegion -SkuTier Premium -SkuFamily MeteredData 
```

其他頻寬包括：1.0、2.0、5.0、10.0 和 40.0

**範例輸出︰**

```powershell
Name                             : ExpressRoute-Direct-ckt
ResourceGroupName                : Contoso-Direct-rg
Location                         : westcentralus
Id                               : /subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Netwo
                                   rk/expressRouteCircuits/ExpressRoute-Direct-ckt
Etag                             : W/"<etagnumber>"
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Premium_MeteredData",
                                     "Tier": "Premium",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             : 
ServiceProviderProperties        : null
ExpressRoutePort                 : {
                                     "Id": "/subscriptions/<subscriptionID>n/resourceGroups/Contoso-Direct-rg/providers/Micros
                                   oft.Network/expressRoutePorts/Contoso-Direct"
                                   }
BandwidthInGbps                  : 10
Stag                             : 2
ServiceKey                       : <number>
Peerings                         : []
Authorizations                   : []
AllowClassicOperations           : False
GatewayManagerEtag     
```

## <a name="next-steps"></a>後續步驟

如需有關 ExpressRoute Direct 的詳細資訊，請參閱[概觀](expressroute-erdirect-about.md)。
