---
title: 建立與修改 Azure ExpressRoute 線路：CLI | Microsoft Docs
description: 本文說明如何使用 CLI 建立、佈建、驗證、更新、刪除和取消佈建 ExpressRoute 線路。
documentationcenter: na
services: expressroute
author: cherylmc
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/19/2017
ms.author: anzaman;cherylmc
ms.openlocfilehash: 21cd06692fe1c26cfa276d1f8ec7ace77b11d3c2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46979976"
---
# <a name="create-and-modify-an-expressroute-circuit-using-cli"></a>使用 CLI 建立和修改 ExpressRoute 線路


本文說明如何使用命令列介面 (CLI) 來建立 Azure ExpressRoute 線路。 本文也會示範如何檢查狀態、更新或刪除和取消佈建線路。 如果您想要對 ExpressRoute 線路使用不同的方法，您可以從下列清單選取文章：

> [!div class="op_single_selector"]
> * [Azure 入口網站](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [Azure CLI](howto-circuit-cli.md)
> * [影片 - Azure 入口網站](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (傳統)](expressroute-howto-circuit-classic.md)
> 

## <a name="before-you-begin"></a>開始之前

* 開始之前，請先安裝 CLI 命令的最新版本 (2.0 版或更新版本)。 如需關於安裝 CLI 命令的資訊，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli) 和[開始使用 Azure CLI](/cli/azure/get-started-with-azure-cli)。
* 開始設定之前，請檢閱[必要條件](expressroute-prerequisites.md)和[工作流程](expressroute-workflows.md)。

## <a name="create"></a>建立和佈建 ExpressRoute 線路

### <a name="1-sign-in-to-your-azure-account-and-select-your-subscription"></a>1.登入您的 Azure 帳戶並且選取您的訂用帳戶

若要開始您的組態，請登入您的 Azure 帳戶。 使用下列範例來協助您連接：

```azurecli
az login
```

檢查帳戶的訂用帳戶。

```azurecli
az account list
```

選取您想要建立 ExpressRoute 線路的訂用帳戶。

```azurecli
az account set --subscription "<subscription ID>"
```

### <a name="2-get-the-list-of-supported-providers-locations-and-bandwidths"></a>2.取得支援的提供者、位置和頻寬清單

建立 ExpressRoute 線路之前，您需要有支援的連線提供者、位置和頻寬選項的清單。 CLI 命令 'az network express-route list-service-providers' 會傳回此資訊，您將在稍後的步驟中使用：

```azurecli
az network express-route list-service-providers
```

回應如下列範例所示：

```azurecli
[
  {
    "bandwidthsOffered": [
      {
        "offerName": "50Mbps",
        "valueInMbps": 50
      },
      {
        "offerName": "100Mbps",
        "valueInMbps": 100
      },
      {
        "offerName": "200Mbps",
        "valueInMbps": 200
      },
      {
        "offerName": "500Mbps",
        "valueInMbps": 500
      },
      {
        "offerName": "1Gbps",
        "valueInMbps": 1000
      },
      {
        "offerName": "2Gbps",
        "valueInMbps": 2000
      },
      {
        "offerName": "5Gbps",
        "valueInMbps": 5000
      },
      {
        "offerName": "10Gbps",
        "valueInMbps": 10000
      }
    ],
    "id": "/subscriptions//resourceGroups//providers/Microsoft.Network/expressRouteServiceProviders/",
    "location": null,
    "name": "AARNet",
    "peeringLocations": [
      "Melbourne",
      "Sydney"
    ],
    "provisioningState": "Succeeded",
    "resourceGroup": "",
    "tags": null,
    "type": "Microsoft.Network/expressRouteServiceProviders"
  },
```

請檢查回應以查看是否列出您的連線提供者。 記下下列資訊，當您建立線路時將會用到：

* 名稱
* PeeringLocations
* BandwidthsOffered

您現在已經準備好建立 ExpressRoute 線路。

### <a name="3-create-an-expressroute-circuit"></a>3.建立 ExpressRoute 線路

> [!IMPORTANT]
> ExpressRoute 線路會從發出服務金鑰時開始收費。 在連線提供者準備好佈建線路之後，再執行這項作業。
> 
> 

若您還沒有資源群組，您必須在建立 ExpressRoute 線路之前建立一個。 您可以藉由使用下列命令來建立資源群組：

```azurecli
az group create -n ExpressRouteResourceGroup -l "West US"
```

以下示範如何透過矽谷的 Equinix 建立 200-Mbps ExpressRoute 線路。 如果您使用不同的提供者和不同的設定，請在您提出要求時替換成該資訊。 

請確定您指定正確的 SKU 層和 SKU 系列：

* SKU 層會判斷 ExpressRoute 標準或 ExpressRoute 高階附加元件是否啟用。 您可以指定 [標準] 來取得標準 SKU，或指定 [進階] 來取得進階附加元件。
* SKU 系列決定計費類型。 您可以指定 [Metereddata] 以採用計量付費數據傳輸方案，選取 [Unlimiteddata] 以採用無限行動數據方案。 您可以將計費類型從 [Metereddata] 變更為 [Unlimiteddata]，但無法將類型從 [Unlimiteddata] 變更為 [Metereddata]。


ExpressRoute 線路會從發出服務金鑰時開始收費。 下列是新服務金鑰的要求範例：

```azurecli
az network express-route create --bandwidth 200 -n MyCircuit --peering-location "Silicon Valley" -g ExpressRouteResourceGroup --provider "Equinix" -l "West US" --sku-family MeteredData --sku-tier Standard
```

回應包含服務金鑰。

### <a name="4-list-all-expressroute-circuits"></a>4.列出所有 ExpressRoute 循環

若要取得您已建立之所有 ExpressRoute 線路的清單，請執行 'az network express-route list' 命令。 您隨時可以使用此命令來擷取此資訊。 若要列出所有線路，在不使用任何參數的情況下進行呼叫。

```azurecli
az network express-route list
```

回應的 [服務金鑰] 欄位會列出您的服務金鑰。

```azurecli
"allowClassicOperations": false,
"authorizations": [],
"circuitProvisioningState": "Enabled",
"etag": "W/\"1262c492-ffef-4a63-95a8-a6002736b8c4\"",
"gatewayManagerEtag": null,
"id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit",
"location": "westus",
"name": "MyCircuit",
"peerings": [],
"provisioningState": "Succeeded",
"resourceGroup": "ExpressRouteResourceGroup",
"serviceKey": "1d05cf70-1db5-419f-ad86-1ca62c3c125b",
"serviceProviderNotes": null,
"serviceProviderProperties": {
  "bandwidthInMbps": 200,
  "peeringLocation": "Silicon Valley",
  "serviceProviderName": "Equinix"
},
"serviceProviderProvisioningState": "NotProvisioned",
"sku": {
  "family": "UnlimitedData",
  "name": "Standard_MeteredData",
  "tier": "Standard"
},
"tags": null,
"type": "Microsoft.Network/expressRouteCircuits]
```

您可以藉由使用 '-h' 來執行命令，以取得所有參數的詳細描述。

```azurecli
az network express-route list -h
```

### <a name="5-send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>5.將服務金鑰傳送給連線提供者以進行佈建

'ServiceProviderProvisioningState' 提供服務提供者端目前的佈建狀態相關資訊。 狀態提供 Microsoft 端的狀態。 如需詳細資訊，請參閱[工作流程文章](expressroute-workflows.md#expressroute-circuit-provisioning-states)。

當您建立新的 ExpressRoute 線路時，線路會是下列狀態：

```azurecli
"serviceProviderProvisioningState": "NotProvisioned"
"circuitProvisioningState": "Enabled"
```

當連線提供者正在為您啟用線路時，線路會變更為下列狀態：

```azurecli
"serviceProviderProvisioningState": "Provisioning"
"circuitProvisioningState": "Enabled"
```

若要能夠使用 ExpressRoute 線路，它必須處於下列狀態：

```azurecli
"serviceProviderProvisioningState": "Provisioned"
"circuitProvisioningState": "Enabled
```

### <a name="6-periodically-check-the-status-and-the-state-of-the-circuit-key"></a>6.定期檢查線路金鑰的情況和狀態

檢查線路金鑰的情況和狀態將能讓您得知提供者是否已啟用您的線路。 設定線路之後，[ServiceProviderProvisioningState] 會顯示為 [Provisioned]，如下列範例所示：

```azurecli
az network express-route show --resource-group ExpressRouteResourceGroup --name MyCircuit
```

回應如下列範例所示：

```azurecli
"allowClassicOperations": false,
"authorizations": [],
"circuitProvisioningState": "Enabled",
"etag": "W/\"1262c492-ffef-4a63-95a8-a6002736b8c4\"",
"gatewayManagerEtag": null,
"id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit",
"location": "westus",
"name": "MyCircuit",
"peerings": [],
"provisioningState": "Succeeded",
"resourceGroup": "ExpressRouteResourceGroup",
"serviceKey": "1d05cf70-1db5-419f-ad86-1ca62c3c125b",
"serviceProviderNotes": null,
"serviceProviderProperties": {
  "bandwidthInMbps": 200,
  "peeringLocation": "Silicon Valley",
  "serviceProviderName": "Equinix"
},
"serviceProviderProvisioningState": "NotProvisioned",
"sku": {
  "family": "UnlimitedData",
  "name": "Standard_MeteredData",
  "tier": "Standard"
},
"tags": null,
"type": "Microsoft.Network/expressRouteCircuits]
```

### <a name="7-create-your-routing-configuration"></a>7.建立路由組態

如需逐步指示，請參閱 [ExpressRoute 線路路由組態](howto-routing-cli.md) 一文以建立和修改線路對等。

> [!IMPORTANT]
> 這些指示只適用於由提供第 2 層連線服務的服務提供者所建立的線路。 如果您使用的服務提供者是提供受控第 3 層服務 (通常是 IP VPN，如 MPLS)，您的連線提供者會為您設定和管理路由。
> 
> 

### <a name="8-link-a-virtual-network-to-an-expressroute-circuit"></a>8.將虛擬網路連結到 ExpressRoute 線路

接下來，將虛擬網路連結到 ExpressRoute 線路。 使用[將虛擬網路連結至 ExpressRoute 線路](howto-linkvnet-cli.md)一文。

## <a name="modify"></a>修改 ExpressRoute 線路

您可以修改 ExpressRoute 線路的某些屬性，而不會影響連線。 您可以進行下列變更，而不會有停機時間：

* 您可以啟用或停用 ExpressRoute 線路的 ExpressRoute 進階附加元件。
* 只要連接埠有可用的容量，您就可以增加 ExpressRoute 線路的頻寬。 但是，不支援將線路的頻寬降級。 
* 您可以將計量方案從 [已計量資料] 變更為 [無限制資料]。 但是，不支援將計量方案從 [無限制資料] 變更為 [已計量資料]。
* 您可以啟用和停用 [允許傳統作業] 。

如需限制的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。

### <a name="to-enable-the-expressroute-premium-add-on"></a>啟用 ExpressRoute 進階附加元件

您可以使用下列命令，為現有的線路啟用 ExpressRoute 進階附加元件：

```azurecli
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-tier Premium
```

線路現在會啟用 ExpressRoute 進階附加功能。 順利執行命令之後，我們就會立即開始針對進階附加元件功能計費。

### <a name="to-disable-the-expressroute-premium-add-on"></a>停用 ExpressRoute 進階附加元件

> [!IMPORTANT]
> 如果您使用的資源超出標準線路所允許的數量，這項作業可能會失敗。
> 
> 

停用 ExpressRoute 進階附加元件之前，請了解下列準則：

* 從進階降級為標準之前，您必須確定連結至線路的虛擬網路數目小於 10。 如果數目大於 10，更新要求就會失敗，且我們會以進階費率計費。
* 您必須取消連結其他地理政治區域中的所有虛擬網路。 如果您未將所有虛擬網路取消連結，更新要求就會失敗，且我們會以進階費率計費。
* 就私用對等設定而言，路由表必須少於 4000 個路由。 如果您的路由資料表大小大於 4000 個路由，BGP 工作階段會卸除。 直到已公告的前置詞數目低於 4000 之前，不會重新啟用工作階段。

您可以使用下列範例，為現有的線路停用 ExpressRoute 進階附加元件：

```azurecli
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-tier Standard
```

### <a name="to-update-the-expressroute-circuit-bandwidth"></a>更新 ExpressRoute 線路頻寬

請查閱 [ExpressRoute 常見問題集](expressroute-faqs.md)，以取得提供者支援的頻寬選項。 您可以挑選任何比現有線路規模還大的大小。

> [!IMPORTANT]
> 如果現有的連接埠上沒有足夠的容量，您可能必須重新建立 ExpressRoute 線路。 如果該位置已無額外的容量，您無法升級線路。
>
> 降低 ExpressRoute 線路的頻寬時必須中斷運作。 頻寬降級需要取消佈建 ExpressRoute 線路，然後重新佈建新的 ExpressRoute 線路。
>

一旦決定需要的大小後，可以使用下列命令來重新調整線路大小：

```azurecli
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --bandwidth 1000
```

我們會在 Microsoft 端調整線路的大小。 接下來，您必須連絡連線提供者，將他們的設定更新為符合這項變更。 通知之後，我們會開始以更新的頻寬選項計費。

### <a name="to-move-the-sku-from-metered-to-unlimited"></a>將 SKU 從計量移動為無限

您可以使用下列範例來變更 ExpressRoute 線路的 SKU：

```azurecli
az network express-route update -n MyCircuit -g ExpressRouteResourceGroup --sku-family UnlimitedData
```

### <a name="to-control-access-to-the-classic-and-resource-manager-environments"></a>控制對傳統和 Resource Manager 環境的存取

請檢閱 [將 ExpressRoute 線路從傳統部署模型移至 Resource Manager 部署模型](expressroute-howto-move-arm.md)中的指示。

## <a name="delete"></a>取消佈建和刪除 ExpressRoute 線路

若要取消佈建並刪除 ExpressRoute 線路，請確定您了解下列準則：

* 您必須取消連結 ExpressRoute 線路的所有虛擬網路。 如果此作業失敗，請確認是否有任何虛擬網路連結至循環。
* 如果 ExpressRoute 線路服務提供者佈建狀態為 **Provisioning** 或 **Provisioned**，您就必須與服務提供者一起合作，取消佈建他們那邊的線路。 我們會繼續保留資源並向您收取費用，直到線路服務提供者完成取消佈建並通知我們。
* 您只能在服務提供者已取消佈建線路時刪除線路。 當線路取消佈建時，服務提供者佈建狀態會設定為「未佈建」。 這樣會停止針對線路計費。

您可以執行下列命令來刪除 ExpressRoute 線路：

```azurecli
az network express-route delete  -n MyCircuit -g ExpressRouteResourceGroup
```

## <a name="next-steps"></a>後續步驟

建立好線路後，請務必執行下列工作：

* [建立和修改 ExpressRoute 線路的路由](howto-routing-cli.md)
* [將虛擬網路連結至 ExpressRoute 線路](howto-linkvnet-cli.md)
