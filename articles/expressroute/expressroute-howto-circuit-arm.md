---
title: 建立和修改 ExpressRoute 線路：PowerShell：Azure Resource Manager | Microsoft Docs
description: 本文說明如何建立、佈建、驗證、更新、刪除和取消佈建 ExpressRoute 線路。
documentationcenter: na
services: expressroute
author: ganesr
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: f997182e-9b25-4a7a-b079-b004221dadcc
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/18/2017
ms.author: ganesr;cherylmc
ms.openlocfilehash: 0d45e97cc42826375a99df16a73c9a7b0c359224
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31594032"
---
# <a name="create-and-modify-an-expressroute-circuit-using-powershell"></a>使用 PowerShell 建立和修改 ExpressRoute 線路
> [!div class="op_single_selector"]
> * [Azure 入口網站](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [Azure CLI](howto-circuit-cli.md)
> * [影片 - Azure 入口網站](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (傳統)](expressroute-howto-circuit-classic.md)
>

本文說明如何使用 PowerShell Cmdlet 和 Azure Resource Manager 部署模型來建立 Azure ExpressRoute 線路。 本文也會示範如何檢查線路的狀態、加以更新，或是加以刪除及取消佈建。

## <a name="before-you-begin"></a>開始之前
* 安裝最新版的 Azure Resource Manager PowerShell Cmdlet。 如需詳細資訊，請參閱 [Azure PowerShell 概觀](/powershell/azure/overview)。
* 開始設定之前，請檢閱[必要條件](expressroute-prerequisites.md)和[工作流程](expressroute-workflows.md)。


## <a name="create"></a>建立和佈建 ExpressRoute 線路
### <a name="1-sign-in-to-your-azure-account-and-select-your-subscription"></a>1.登入您的 Azure 帳戶並且選取您的訂用帳戶
若要開始您的組態，請登入您的 Azure 帳戶。 使用下列範例來協助您連接：

```powershell
Connect-AzureRmAccount
```

檢查帳戶的訂用帳戶：

```powershell
Get-AzureRmSubscription
```

選取您想要建立 ExpressRoute 循環的訂用帳戶：

```powershell
Select-AzureRmSubscription -SubscriptionId "<subscription ID>"
```

### <a name="2-get-the-list-of-supported-providers-locations-and-bandwidths"></a>2.取得支援的提供者、位置和頻寬清單
建立 ExpressRoute 線路之前，您需要有支援的連線提供者、位置和頻寬選項的清單。

PowerShell Cmdlet **Get-AzureRmExpressRouteServiceProvider** 會傳回此資訊，在稍後的步驟中將會用到：

```powershell
Get-AzureRmExpressRouteServiceProvider
```

請檢查是否列出您的連線服務提供者。 記下下列資訊，稍後當您建立線路時將會用到：

* Name
* PeeringLocations
* BandwidthsOffered

您現在已經準備好建立 ExpressRoute 線路。   

### <a name="3-create-an-expressroute-circuit"></a>3.建立 ExpressRoute 線路
若您還沒有資源群組，您必須在建立 ExpressRoute 線路之前建立一個。 您可以執行下列命令來這麼做：

```powershell
New-AzureRmResourceGroup -Name "ExpressRouteResourceGroup" -Location "West US"
```


以下示範如何透過矽谷的 Equinix 建立 200-Mbps ExpressRoute 線路。 如果您使用不同的提供者和不同的設定，請在您提出要求時替換成該資訊。 使用下列範例來要求新服務金鑰：

```powershell
New-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup" -Location "West US" -SkuTier Standard -SkuFamily MeteredData -ServiceProviderName "Equinix" -PeeringLocation "Silicon Valley" -BandwidthInMbps 200
```

請確定您指定正確的 SKU 層和 SKU 系列：

* SKU 層會判斷 ExpressRoute 標準或 ExpressRoute 高階附加元件是否啟用。 您可以指定 [標準] 來取得標準 SKU，或指定 [進階] 來取得進階附加元件。
* SKU 系列決定計費類型。 您可以指定 [Metereddata] 以採用計量付費數據傳輸方案，選取 [Unlimiteddata] 以採用無限行動數據方案。 您可以將計費類型從 *Metereddata* 變更為 *Unlimiteddata*，但無法將類型從 *Unlimiteddata* 變更為 *Metereddata*。

> [!IMPORTANT]
> ExpressRoute 線路會從發出服務金鑰時開始收費。 請確定在連線提供者準備好佈建線路之後，再執行這項作業。
> 
> 

回應包含服務金鑰。 您可以執行下列命令來取得所有參數的詳細描述：

```powershell
get-help New-AzureRmExpressRouteCircuit -detailed
```


### <a name="4-list-all-expressroute-circuits"></a>4.列出所有 ExpressRoute 循環
若要取得您已建立之所有 ExpressRoute 線路的清單，請執行 **Get-AzureRmExpressRouteCircuit** 命令：

```powershell
Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```

回應大致如下列範例所示：

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
    CircuitProvisioningState          : Enabled
    ServiceProviderProvisioningState  : NotProvisioned
    ServiceProviderNotes              :
    ServiceProviderProperties         : {
                                          "ServiceProviderName": "Equinix",
                                          "PeeringLocation": "Silicon Valley",
                                          "BandwidthInMbps": 200
                                        }
    ServiceKey                        : **************************************
    Peerings                          : []

您隨時可以使用 `Get-AzureRmExpressRouteCircuit` Cmdlet 擷取此資訊。 執行呼叫時，若未指定任何參數，將會列出所有線路。 [服務金鑰] 欄位會列出您的服務金鑰：

```powershell
Get-AzureRmExpressRouteCircuit
```


回應大致如下列範例所示：

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
    ServiceProviderProvisioningState : NotProvisioned
    ServiceProviderNotes             :
    ServiceProviderProperties        : {
                                         "ServiceProviderName": "Equinix",
                                         "PeeringLocation": "Silicon Valley",
                                         "BandwidthInMbps": 200
                                          }
    ServiceKey                       : **************************************
    Peerings                         : []


您可以執行下列命令來取得所有參數的詳細描述：

```powershell
get-help Get-AzureRmExpressRouteCircuit -detailed
```

### <a name="5-send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>5.將服務金鑰傳送給連線提供者以進行佈建
 提供服務提供者端目前的佈建狀態相關資訊。 [狀態] 提供 Microsoft 端的狀態。 如需線路佈建狀態的詳細資訊，請參閱[工作流程](expressroute-workflows.md#expressroute-circuit-provisioning-states)。

當您建立新的 ExpressRoute 線路時，線路會是下列狀態：

    ServiceProviderProvisioningState : NotProvisioned
    CircuitProvisioningState         : Enabled



當連線提供者正在為您啟用線路時，線路會變更為下列狀態：

    ServiceProviderProvisioningState : Provisioning
    Status                           : Enabled

若要能夠使用 ExpressRoute 線路，它必須處於下列狀態：

    ServiceProviderProvisioningState : Provisioned
    CircuitProvisioningState         : Enabled

### <a name="6-periodically-check-the-status-and-the-state-of-the-circuit-key"></a>6.定期檢查線路金鑰的情況和狀態
檢查線路金鑰的情況和狀態將能讓您得知提供者是否已啟用您的線路。 設定線路之後，[ServiceProviderProvisioningState] 會顯示為 [Provisioned]，如下列範例所示：

```powershell
Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```


回應大致如下列範例所示：

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

### <a name="7-create-your-routing-configuration"></a>7.建立路由組態
如需逐步指示，請參閱 [ExpressRoute 線路路由組態](expressroute-howto-routing-arm.md) 一文以建立和修改線路對等。

> [!IMPORTANT]
> 這些指示只適用於由提供第 2 層連線服務的服務提供者所建立的線路。 如果您使用的服務提供者是提供受控第 3 層服務 (通常是 IP VPN，如 MPLS)，您的連線提供者會為您設定和管理路由。
> 
> 

### <a name="8-link-a-virtual-network-to-an-expressroute-circuit"></a>8.將虛擬網路連結到 ExpressRoute 線路
接下來，將虛擬網路連結到 ExpressRoute 線路。 當使用 Resource Manager 部署模型時，使用[將虛擬網路連結到 ExpressRoute 線路](expressroute-howto-linkvnet-arm.md)文章。

## <a name="getting-the-status-of-an-expressroute-circuit"></a>取得 ExpressRoute 線路的狀態
您可以隨時使用 **Get-AzureRmExpressRouteCircuit** Cmdlet 來擷取此資訊。 執行呼叫時，若未指定任何參數，將會列出所有線路。

```powershell
Get-AzureRmExpressRouteCircuit
```


回應如下列範例所示：

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


您可以透過將資源群組名稱和線路名稱做為參數傳遞至呼叫，取得特定 ExpressRoute 線路的資訊：

```powershell
Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```


回應大致如下列範例所示：

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


您可以執行下列命令來取得所有參數的詳細描述：

```powershell
get-help get-azurededicatedcircuit -detailed
```

## <a name="modify"></a>修改 ExpressRoute 線路
您可以修改 ExpressRoute 線路的某些屬性，而不會影響連線。

您可以執行下列工作，而無需中途停機：

* 啟用或停用 ExpressRoute 線路的 ExpressRoute 進階附加元件。
* 只要連接埠有可用的容量，就增加 ExpressRoute 線路的頻寬。 不支援將線路的頻寬降級。 
* 將計量方案從 [計量付費] 變更為 [無限制]。 不支援將計量方案從 [無限制資料] 變更為 [已計量資料]。
* 您可以啟用和停用 [允許傳統作業] 。

如需限制的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。

### <a name="to-enable-the-expressroute-premium-add-on"></a>啟用 ExpressRoute 進階附加元件
您可以使用下列 PowerShell 程式碼片段，為現有的線路啟用 ExpressRoute 進階附加元件：

```powershell
$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Tier = "Premium"
$ckt.sku.Name = "Premium_MeteredData"

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

線路現在會啟用 ExpressRoute 進階附加功能。 順利執行命令之後，我們就會立即開始針對進階附加元件功能計費。

### <a name="to-disable-the-expressroute-premium-add-on"></a>停用 ExpressRoute 進階附加元件
> [!IMPORTANT]
> 如果您使用的資源超出標準線路所允許的數量，這項作業可能會失敗。
> 
> 

請注意下列資訊︰

* 從進階降級為標準之前，您必須確定連結至線路的虛擬網路數目小於 10。 如果您不這樣做，更新要求就會失敗，且我們會以進階費率計費。
* 您必須取消連結其他地理政治區域中的所有虛擬網路。 如果您不這樣做，更新要求就會失敗，且我們會以進階費率計費。
* 就私用對等設定而言，路由表必須少於 4000 個路由。 如果路由表大小超過 4000 個路由，BGP 工作階段將會中斷，而且在通告的首碼數目降到 4000 以下之前不會重新啟用。

您可以使用下列 PowerShell Cmdlet，為現有的線路停用 ExpressRoute 進階附加元件：

```powershell
$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Tier = "Standard"
$ckt.sku.Name = "Standard_MeteredData"

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-update-the-expressroute-circuit-bandwidth"></a>更新 ExpressRoute 線路頻寬
請查閱 [ExpressRoute 常見問題集](expressroute-faqs.md)，以取得提供者支援的頻寬選項。 您可以挑選任何比現有線路規模還大的大小。

> [!IMPORTANT]
> 如果現有的連接埠上沒有足夠的容量，您可能必須重新建立 ExpressRoute 線路。 如果該位置已無額外的容量，您無法升級線路。
>
> 降低 ExpressRoute 線路的頻寬時必須中斷運作。 頻寬降級需要取消佈建 ExpressRoute 線路，然後重新佈建新的 ExpressRoute 線路。
> 

一旦決定需要的大小後，可以使用下列命令來重新調整線路大小。

```powershell
$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.ServiceProviderProperties.BandwidthInMbps = 1000

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```


我們會在 Microsoft 端調整線路的大小。 接下來，您必須連絡連線提供者，將他們的設定更新為符合這項變更。 通知之後，我們會開始以更新的頻寬選項計費。

### <a name="to-move-the-sku-from-metered-to-unlimited"></a>將 SKU 從計量移動為無限
您可以使用下列 PowerShell 程式碼片段變更 ExpressRoute 循環的 SKU：

```powershell
$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Family = "UnlimitedData"
$ckt.sku.Name = "Premium_UnlimitedData"

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-control-access-to-the-classic-and-resource-manager-environments"></a>控制對傳統和 Resource Manager 環境的存取
請檢閱 [將 ExpressRoute 線路從傳統部署模型移至 Resource Manager 部署模型](expressroute-howto-move-arm.md)中的指示。  

## <a name="delete"></a>取消佈建和刪除 ExpressRoute 線路
請注意下列資訊︰

* 您必須取消連結 ExpressRoute 線路的所有虛擬網路。 如果此作業失敗，請確認是否有任何虛擬網路連結至循環。
* 如果 ExpressRoute 線路服務提供者佈建狀態為 **Provisioning** 或 **Provisioned**，您就必須與服務提供者一起合作，取消佈建他們那邊的線路。 我們會繼續保留資源並向您收取費用，直到線路服務提供者完成取消佈建並通知我們。
* 若服務提供者已取消佈建線路 (服務提供者佈建狀態設定為 [未佈建])，則可以刪除線路。 這樣會停止針對線路計費。

您可以執行下列命令來刪除 ExpressRoute 線路：

```powershell
Remove-AzureRmExpressRouteCircuit -ResourceGroupName "ExpressRouteResourceGroup" -Name "ExpressRouteARMCircuit"
```

## <a name="next-steps"></a>後續步驟

建立好線路後，請務必執行下列步驟：

* [建立和修改 ExpressRoute 線路的路由](expressroute-howto-routing-arm.md)
* [將虛擬網路連結至 ExpressRoute 線路](expressroute-howto-linkvnet-arm.md)
