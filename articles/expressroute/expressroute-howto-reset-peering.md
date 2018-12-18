---
title: 重設 Azure ExpressRoute 對等互連 | Microsoft Docs
description: 如何停用和啟用 ExpressRoute 線路的對等互連。
services: expressroute
author: charwen
ms.service: expressroute
ms.topic: conceptual
ms.date: 08/15/2018
ms.author: charwen
ms.openlocfilehash: b8c9bc1944e9ed0281616062a84958c953d08694
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "40180466"
---
# <a name="reset-expressroute-peerings"></a>重設 ExpressRoute 對等互連

本文說明如何使用 PowerShell 來停用和啟用 ExpressRoute 線路的對等互連。 當您停用對等互連時，位於 ExpressRoute 線路之主要連線與次要連線上的 BGP 工作階段將會關閉。 您將遺失透過此對等互連來連至 Microsoft 的連線。 當您啟用對等互連時，位於 ExpressRoute 線路之主要連線與次要連線上的 BGP 工作階段將會上線。 您將重新獲得透過此對等互連來連至 Microsoft 的連線。 您可以在 ExpressRoute 線路上單獨啟用和停用 Microsoft 對等互連和 Azure 私用對等互連。 當您第一次在 ExpressRoute 線路上設定對等互連時，預設會啟用對等互連。 

有數個案例可能會讓您覺得重設 ExpressRoute 對等互連很實用。
* 測試您的災害復原設計和實作。 例如，您有兩個 ExpressRoute 線路。 您可以停用一個線路的對等互連，並強制將網路流量容錯移轉至另一個線路。
* 在 ExpressRoute 線路的 Azure 私用對等互連上啟用雙向轉送偵測 (BFD)。 如果您的 ExpressRoute 線路是在 2018 年 8 月 1 日之後建立的，預設會啟用 BFD。 如果您的線路是在那之前建立的，則不會啟用 BFD。 您可以藉由停用對等互連並重新啟用它來啟用 BFD。 請務必注意，只有在 Azure 私用對等互連上才支援 BFD。


## <a name="reset-a-peering"></a>重設對等互連

1. 安裝最新版的 Azure Resource Manager PowerShell Cmdlet。 如需詳細資訊，請參閱[安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)。

2. 以提高的權限開啟 PowerShell 主控台並連線至您的帳戶。 使用下列範例來協助您連接：

  ```powershell
  Connect-AzureRmAccount
  ```
3. 如果您有多個 Azure 訂用帳戶，請檢查帳戶的訂用帳戶。

  ```powershell
  Get-AzureRmSubscription
  ```
4. 指定您要使用的訂用帳戶。

  ```powershell
  Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"
  ```
5. 執行下列命令來擷取您的 ExpressRoute 線路。

  ```powershell
  $ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
  ```
6. 識別您想要停用或啟用的對等互連。 「對等互連」是一個陣列。 在下列範例中，Peerings[0] 是 Azure 私用對等互連，而 Peerings[1] 是 Microsoft 對等互連。

  ```powershell
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/########-####-####-####-############/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"cd011bef-dc79-49eb-b4c6-81fb6ea5d178"
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
                                     "ServiceProviderName": "Coresite",
                                     "PeeringLocation": "Los Angeles",
                                     "BandwidthInMbps": 50
                                   }
ServiceKey                       : ########-####-####-####-############
Peerings                         : [
                                     {
                                       "Name": "AzurePrivatePeering",
                                       "Etag": "W/\"cd011bef-dc79-49eb-b4c6-81fb6ea5d178\"",
                                       "Id": "/subscriptions/########-####-####-####-############/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit/peerings/AzurePrivatePeering",
                                       "PeeringType": "AzurePrivatePeering",
                                       "State": "Enabled",
                                       "AzureASN": 12076,
                                       "PeerASN": 123,
                                       "PrimaryPeerAddressPrefix": "10.0.0.0/30",
                                       "SecondaryPeerAddressPrefix": "10.0.0.4/30",
                                       "PrimaryAzurePort": "",
                                       "SecondaryAzurePort": "",
                                       "VlanId": 789,
                                       "MicrosoftPeeringConfig": {
                                         "AdvertisedPublicPrefixes": [],
                                         "AdvertisedCommunities": [],
                                         "AdvertisedPublicPrefixesState": "NotConfigured",
                                         "CustomerASN": 0,
                                         "LegacyMode": 0,
                                         "RoutingRegistryName": "NONE"
                                       },
                                       "ProvisioningState": "Succeeded",
                                       "GatewayManagerEtag": "",
                                       "LastModifiedBy": "Customer",
                                       "Connections": []
                                     },
                                     {
                                       "Name": "MicrosoftPeering",
                                       "Etag": "W/\"cd011bef-dc79-49eb-b4c6-81fb6ea5d178\"",
                                       "Id": "/subscriptions/########-####-####-####-############/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit/peerings/MicrosoftPeering",
                                       "PeeringType": "MicrosoftPeering",
                                       "State": "Enabled",
                                       "AzureASN": 12076,
                                       "PeerASN": 123,
                                       "PrimaryPeerAddressPrefix": "3.0.0.0/30",
                                       "SecondaryPeerAddressPrefix": "3.0.0.4/30",
                                       "PrimaryAzurePort": "",
                                       "SecondaryAzurePort": "",
                                       "VlanId": 345,
                                       "MicrosoftPeeringConfig": {
                                         "AdvertisedPublicPrefixes": [
                                           "3.0.0.3/32"
                                         ],
                                         "AdvertisedCommunities": [],
                                         "AdvertisedPublicPrefixesState": "ValidationNeeded",
                                         "CustomerASN": 0,
                                         "LegacyMode": 0,
                                         "RoutingRegistryName": "NONE"
                                       },
                                       "ProvisioningState": "Succeeded",
                                       "GatewayManagerEtag": "",
                                       "LastModifiedBy": "Customer",
                                       "Connections": []
                                     }
                                   ]
Authorizations                   : []
AllowClassicOperations           : False
GatewayManagerEtag               :
  ```
7. 執行下列命令以變更對等互連的狀態。

  ```powershell
  $ckt.Peerings[0].State = "Disabled"
  Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
  ```
對等互連應該處於您所設定的狀態。 

## <a name="next-steps"></a>後續步驟
如果您需要協助以進行 ExpressRoute 問題的疑難排解，請參閱下列文章：
* [驗證 ExpressRoute 連線能力](expressroute-troubleshooting-expressroute-overview.md)
* [網路效能疑難排解](expressroute-troubleshooting-network-performance.md)
