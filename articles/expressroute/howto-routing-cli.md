---
title: 如何設定 Azure ExpressRoute 線路的路由：CLI | Microsoft Docs
description: 本文將協助您為 ExpressRoute 線路建立和佈建私用、公用及 Microsoft 對等互連。 本文也示範如何檢查狀態、更新或刪除線路的對等。
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
ms.date: 10/11/2017
ms.author: cherylmc
ms.openlocfilehash: cd7a6c43cb791ed607a763f10e781794ef513500
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46950398"
---
# <a name="create-and-modify-routing-for-an-expressroute-circuit-using-cli"></a>使用 CLI 來建立和修改 ExpressRoute 路線的路由

本文將協助您使用 CLI，以 Resource Manager 部署模型建立和管理 ExpressRoute 線路的路由設定。 您還可以檢查狀態、更新或刪除和取消佈建 ExpressRoute 線路的對等互連。 如果您想要對線路使用不同的方法，可選取下列清單中的文章：

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

* 開始之前，請先安裝 CLI 命令的最新版本 (2.0 版或更新版本)。 如需關於安裝 CLI 命令的資訊，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。
* 開始設定之前，請確定您已經檢閱過[必要條件](expressroute-prerequisites.md)、[路由需求](expressroute-routing.md)和[工作流程](expressroute-workflows.md)分頁。
* 您必須擁有作用中的 ExpressRoute 線路。 繼續之前，請遵循指示來 [建立 ExpressRoute 線路](howto-circuit-cli.md) ，並由您的連線提供者來啟用該線路。 ExpressRoute 線路必須處於已佈建和已啟用狀態，您才能執行本文中的命令。

這些指示只適用於由提供第 2 層連線服務的服務提供者所建立的線路。 如果您使用的服務提供者提供受控第 3 層服務 (通常是 IPVPN，如 MPLS)，連線提供者會為您設定和管理路由。

您可以為 ExpressRoute 線路設定一個、兩個或全部三個對等 (Azure 私用、Azure 公用和 Microsoft)。 您可以依自己選擇的任何順序設定對等。 不過，您必須確定一次只完成一個對等的設定。 如需路由網域和對等互連的詳細資訊，請參閱 [ExpressRoute 路由網域](expressroute-circuit-peerings.md)。

## <a name="msft"></a>Microsoft 對等互連

本節將協助您為 ExpressRoute 線路建立、取得、更新和刪除 Microsoft 對等互連設定。

> [!IMPORTANT]
> 在 2017 年 8 月 1 日以前設定之 ExpressRoute 線路的 Microsoft 對等互連，會透過 Microsoft 對等互連公告所有服務首碼，即使未定義路由篩選也一樣。 在 2017 年 8 月 1 日當日或以後設定之 ExpressRoute 線路的 Microsoft 對等互連，不會公告任何首碼，直到路由篩選連結至線路為止。 如需詳細資訊，請參閱[設定 Microsoft 對等互連的路由篩選](how-to-routefilter-powershell.md)。
> 
> 

### <a name="to-create-microsoft-peering"></a>建立 Microsoft 對等

1. 安裝最新版的 Azure CLI。 使用最新版本的 Azure 命令列介面 (CLI)。* 在開始設定之前，檢閱[必要條件](expressroute-prerequisites.md)和[工作流程](expressroute-workflows.md)。

  ```azurecli
  az login
  ```

  選取您想要建立 ExpressRoute 線路的訂用帳戶。

  ```azurecli
  az account set --subscription "<subscription ID>"
  ```
2. 建立 ExpressRoute 線路。 請遵循指示建立 [ExpressRoute 線路](howto-circuit-cli.md) ，並由連線提供者佈建它。 若您的連線提供者提供受控第 3 層服務，您可以要求連線提供者為您啟用 Microsoft 對等互連。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不管理路由，請在建立線路之後繼續使用後續步驟進行設定。 

3. 檢查 ExpressRoute 線路，以確定已佈建且已啟用線路。 請使用下列範例：

  ```azurecli
  az network express-route list
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
  "serviceProviderProvisioningState": "Provisioned",
  "sku": {
    "family": "UnlimitedData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
  },
  "tags": null,
  "type": "Microsoft.Network/expressRouteCircuits]
  ```

4. 設定線路的 Microsoft 對等。 繼續之前，請確定您擁有下列資訊。

  * 主要連結的 /30 子網路。 這必須是您所擁有且註冊在 RIR / IRR 中的有效公用 IPv4 首碼。
  * 次要連結的 /30 子網路。 這必須是您所擁有且註冊在 RIR / IRR 中的有效公用 IPv4 首碼。
  * 供建立此對等的有效 VLAN ID。 請確定線路有沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。
  * 公告的首碼：您必須提供一份您打算在 BGP 工作階段上公告的所有首碼的清單。 只接受公用 IP 位址首碼。 如果計劃傳送一組首碼，可以傳送以逗號分隔的清單。 這些首碼必須在 RIR / IRR 中註冊給您。
  * **選用：** 客戶 ASN：如果您要公告的首碼未註冊給對等互連 AS 編號，則可以指定它們所註冊的 AS 編號。
  * 路由登錄名稱：您可以指定可供註冊 AS 編號和首碼的 RIR / IRR。
  * **選用 -** MD5 雜湊 (如果選擇使用)。

   執行下列範例來為線路設定 Microsoft 對等互連：

  ```azurecli
  az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 123.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 123.0.0.4/30 --vlan-id 300 --peering-type MicrosoftPeering --advertised-public-prefixes 123.1.0.0/24
  ```

### <a name="getmsft"></a>檢視 Microsoft 對等詳細資訊

您可以使用下列範例來取得設定詳細資料：

```azurecli
az network express-route peering show -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzureMicrosoftPeering
```

輸出類似於下列範例：

```azurecli
{
  "azureAsn": 12076,
  "etag": "W/\"2e97be83-a684-4f29-bf3c-96191e270666\"",
  "gatewayManagerEtag": "18",
  "id": "/subscriptions/9a0c2943-e0c2-4608-876c-e0ddffd1211b/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/peerings/AzureMicrosoftPeering",
  "lastModifiedBy": "Customer",
  "microsoftPeeringConfig": {
    "advertisedPublicPrefixes": [
        ""
      ],
     "advertisedPublicPrefixesState": "",
     "customerASN": ,
     "routingRegistryName": ""
  }
  "name": "AzureMicrosoftPeering",
  "peerAsn": ,
  "peeringType": "AzureMicrosoftPeering",
  "primaryAzurePort": "",
  "primaryPeerAddressPrefix": "",
  "provisioningState": "Succeeded",
  "resourceGroup": "ExpressRouteResourceGroup",
  "routeFilter": null,
  "secondaryAzurePort": "",
  "secondaryPeerAddressPrefix": "",
  "sharedKey": null,
  "state": "Enabled",
  "stats": null,
  "vlanId": 100
}
```

### <a name="updatemsft"></a>更新 Microsoft 對等組態

您可以更新設定的任何部分。 在下列範例中，已公告之線路的前置詞從 123.1.0.0/24 更新為 124.1.0.0/24：

```azurecli
az network express-route peering update --circuit-name MyCircuit -g ExpressRouteResourceGroup --peering-type MicrosoftPeering --advertised-public-prefixes 124.1.0.0/24
```

### <a name="addIPv6msft"></a>在現有的 IPv4 設定中，新增 IPv6 Microsoft 對等互連設定

```azurecli
az network express-route peering update -g ExpressRouteResourceGroup --circuit-name MyCircuit --peering-type MicrosoftPeering --ip-version ipv6 --primary-peer-subnet 2002:db00::/126 --secondary-peer-subnet 2003:db00::/126 --advertised-public-prefixes 2002:db00::/126
```

### <a name="deletemsft"></a>刪除 Microsoft 對等

您可以執行下列範例來移除對等互連設定：

```azurecli
az network express-route peering delete -g ExpressRouteResourceGroup --circuit-name MyCircuit --name MicrosoftPeering
```

## <a name="private"></a>Azure 私人對等互連

本節將協助您為 ExpressRoute 線路建立、取得、更新和刪除 Azure 私用對等互連設定。

### <a name="to-create-azure-private-peering"></a>建立 Azure 私用對等

1. 安裝最新版的 Azure CLI。 您必須使用最新版本的 Azure 命令列介面 (CLI)。* 在開始設定之前，檢閱[必要條件](expressroute-prerequisites.md)和[工作流程](expressroute-workflows.md)。

  ```azurecli
  az login
  ```

  選取您想要建立 ExpressRoute 線路的訂用帳戶

  ```azurecli
  az account set --subscription "<subscription ID>"
  ```
2. 建立 ExpressRoute 線路。 請遵循指示建立 [ExpressRoute 線路](howto-circuit-cli.md) ，並由連線提供者佈建它。 若您的連線提供者是提供受控第 3 層服務，您可以要求連線提供者為您啟用 Azure 私人對等互連。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不管理路由，請在建立線路之後繼續使用後續步驟進行設定。

3. 檢查 ExpressRoute 線路，以確定已佈建且已啟用線路。 請使用下列範例：

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
  "serviceProviderProvisioningState": "Provisioned",
  "sku": {
    "family": "UnlimitedData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
  },
  "tags": null,
  "type": "Microsoft.Network/expressRouteCircuits]
  ```

4. 設定線路的 Azure 私用對等。 繼續執行接下來的步驟之前，請確定您有下列項目：

  * 主要連結的 /30 子網路。 子網路不能在保留給虛擬網路的任何位址空間中。
  * 次要連結的 /30 子網路。 子網路不能在保留給虛擬網路的任何位址空間中。
  * 供建立此對等的有效 VLAN ID。 請確定線路有沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。 您可以將私用 AS 編號用於此對等。 請確定您不是使用 65515。
  * **選用：** MD5 雜湊 (如果選擇使用)。

  使用下列範例來為線路設定 Azure 私用對等互連：

  ```azurecli
  az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 10.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 10.0.0.4/30 --vlan-id 200 --peering-type AzurePrivatePeering
  ```

  如果您選擇使用 MD5 雜湊，請使用下列範例：

  ```azurecli
  az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 10.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 10.0.0.4/30 --vlan-id 200 --peering-type AzurePrivatePeering --SharedKey "A1B2C3D4"
  ```

  > [!IMPORTANT]
  > 請確定您將 AS 編號指定為對等 ASN，而不是客戶 ASN。
  > 
  > 

### <a name="getprivate"></a>檢視 Azure 私用對等詳細資訊

您可以使用下列範例來取得設定詳細資料：

```azurecli
az network express-route peering show -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePrivatePeering
```

輸出類似於下列範例：

```azurecli
{
  "azureAsn": 12076,
  "etag": "W/\"2e97be83-a684-4f29-bf3c-96191e270666\"",
  "gatewayManagerEtag": "18",
  "id": "/subscriptions/9a0c2943-e0c2-4608-876c-e0ddffd1211b/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/peerings/AzurePrivatePeering",
  "ipv6PeeringConfig": null,
  "lastModifiedBy": "Customer",
  "microsoftPeeringConfig": null,
  "name": "AzurePrivatePeering",
  "peerAsn": 7671,
  "peeringType": "AzurePrivatePeering",
  "primaryAzurePort": "",
  "primaryPeerAddressPrefix": "",
  "provisioningState": "Succeeded",
  "resourceGroup": "ExpressRouteResourceGroup",
  "routeFilter": null,
  "secondaryAzurePort": "",
  "secondaryPeerAddressPrefix": "",
  "sharedKey": null,
  "state": "Enabled",
  "stats": null,
  "vlanId": 100
}
```

### <a name="updateprivate"></a>更新 Azure 私用對等組態

您可以使用下列範例來更新設定的任何部分。 在此範例中，線路的 VLAN ID 從 100 更新為 500。

```azurecli
az network express-route peering update --vlan-id 500 -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePrivatePeering
```

### <a name="deleteprivate"></a>刪除 Azure 私用對等

您可以執行下列範例來移除對等互連設定：

> [!WARNING]
> 執行此範例之前，您必須確定所有虛擬網路都已經與 ExpressRoute 線路取消連結。 
> 
> 

```azurecli
az network express-route peering delete -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePrivatePeering
```

## <a name="public"></a>Azure 公用對等互連

本節將協助您為 ExpressRoute 線路建立、取得、更新和刪除 Azure 公用對等互連設定。

### <a name="to-create-azure-public-peering"></a>建立 Azure 公用對等

1. 安裝最新版的 Azure CLI。 您必須使用最新版本的 Azure 命令列介面 (CLI)。* 在開始設定之前，檢閱[必要條件](expressroute-prerequisites.md)和[工作流程](expressroute-workflows.md)。

  ```azurecli
  az login
  ```

  選取您想要建立 ExpressRoute 線路的訂用帳戶。

  ```azurecli
  az account set --subscription "<subscription ID>"
  ```
2. 建立 ExpressRoute 線路。  請遵循指示建立 [ExpressRoute 線路](howto-circuit-cli.md) ，並由連線提供者佈建它。 若您的連線提供者提供受控第 3 層服務，您可以要求連線提供者為您啟用 Azure 公用對等互連。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不管理路由，請在建立線路之後繼續使用後續步驟進行設定。

3. 檢查 ExpressRoute 線路，以確定已佈建且已啟用線路。 請使用下列範例：

  ```azurecli
  az network express-route list
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
  "serviceProviderProvisioningState": "Provisioned",
  "sku": {
    "family": "UnlimitedData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
  },
  "tags": null,
  "type": "Microsoft.Network/expressRouteCircuits]
  ```

4. 設定線路的 Azure 公用對等。 進一步執行之前，請確定您具有下列資訊。

  * 主要連結的 /30 子網路。 這必須是有效的公用 IPv4 首碼。
  * 次要連結的 /30 子網路。 這必須是有效的公用 IPv4 首碼。
  * 供建立此對等的有效 VLAN ID。 請確定線路有沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。
  * **選用：** MD5 雜湊 (如果選擇使用)。

  執行下列範例來為線路設定 Azure 公用對等互連：

  ```azurecli
  az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 12.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 12.0.0.4/30 --vlan-id 200 --peering-type AzurePublicPeering
  ```

  如果您選擇使用 MD5 雜湊，請使用下列範例：

  ```azurecli
  az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 12.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 12.0.0.4/30 --vlan-id 200 --peering-type AzurePublicPeering --SharedKey "A1B2C3D4"
  ```

  > [!IMPORTANT]
  > 請確定您將 AS 編號指定為對等 ASN，而不是客戶 ASN。

### <a name="getpublic"></a>檢視 Azure 公用對等詳細資訊

您可以使用下列範例來取得設定詳細資料：

```azurecli
az network express-route peering show -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

輸出類似於下列範例：

```azurecli
{
  "azureAsn": 12076,
  "etag": "W/\"2e97be83-a684-4f29-bf3c-96191e270666\"",
  "gatewayManagerEtag": "18",
  "id": "/subscriptions/9a0c2943-e0c2-4608-876c-e0ddffd1211b/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/peerings/AzurePublicPeering",
  "lastModifiedBy": "Customer",
  "microsoftPeeringConfig": null,
  "name": "AzurePublicPeering",
  "peerAsn": 7671,
  "peeringType": "AzurePublicPeering",
  "primaryAzurePort": "",
  "primaryPeerAddressPrefix": "",
  "provisioningState": "Succeeded",
  "resourceGroup": "ExpressRouteResourceGroup",
  "routeFilter": null,
  "secondaryAzurePort": "",
  "secondaryPeerAddressPrefix": "",
  "sharedKey": null,
  "state": "Enabled",
  "stats": null,
  "vlanId": 100
}
```

### <a name="updatepublic"></a>更新 Azure 公用對等組態

您可以使用下列範例來更新設定的任何部分。 在此範例中，線路的 VLAN ID 從 200 更新為 600。

```azurecli
az network express-route peering update --vlan-id 600 -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

### <a name="deletepublic"></a>刪除 Azure 公用對等

您可以執行下列範例來移除對等互連設定：

```azurecli
az network express-route peering delete -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

## <a name="next-steps"></a>後續步驟

下一步， [將 VNet 連結到 ExpressRoute 線路](howto-linkvnet-cli.md)。

* 如需 ExpressRoute 工作流程的詳細資訊，請參閱 [ExpressRoute 工作流程](expressroute-workflows.md)。
* 如需線路對等的詳細資訊，請參閱 [ExpressRoute 線路和路由網域](expressroute-circuit-peerings.md)。
* 如需使用虛擬網路的詳細資訊，請參閱 [虛擬網路概觀](../virtual-network/virtual-networks-overview.md)。