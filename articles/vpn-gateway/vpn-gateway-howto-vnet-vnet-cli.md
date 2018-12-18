---
title: 使用 VNet 對 VNet 連線將虛擬網路連線至另一個 VNet︰Azure CLI | Microsoft Docs
description: 使用 VNet 對 VNet 連線和 Azure CLI，將虛擬網路連在一起。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jpconnock
editor: ''
tags: azure-resource-manager
ms.assetid: 0683c664-9c03-40a4-b198-a6529bf1ce8b
ms.service: vpn-gateway
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/14/2018
ms.author: cherylmc
ms.openlocfilehash: 2fc25235325db8a403c2b258dd5e4b3effc46ace
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971955"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-using-azure-cli"></a>使用 Azure CLI 設定 VNet 對 VNet 的 VPN 閘道連線

本文協助您使用 VNet 對 VNet 連線類型來連線虛擬網路。 虛擬網路可位於相同或不同的區域，以及來自相同或不同的訂用帳戶。 連線來自不同訂用帳戶的 VNet 時，訂用帳戶不需與相同的 Active Directory 租用戶相關聯。

本文中的步驟適用於 Resource Manager 部署模型並使用 Azure CLI。 您也可從下列清單中選取不同的選項，以使用不同的部署工具或部署模型來建立此組態：

> [!div class="op_single_selector"]
> * [Azure 入口網站](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Azure 入口網站 (傳統)](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [連線不同的部署模型 - Azure 入口網站](vpn-gateway-connect-different-deployment-models-portal.md)
> * [連線不同的部署模型 - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

## <a name="about"></a>關於連線 VNet

VNet 的連線方法有很多種。 下列各節說明不同的虛擬網路連線方式。

### <a name="vnet-to-vnet"></a>VNet 對 VNet

設定 VNet 對 VNet 連線是輕鬆連線 VNet 的好方法。 使用 VNet 對 VNet 連線類型將虛擬網路連線到另一個虛擬網路，類似於建立內部部署位置的站對站 IPsec 連線。 這兩種連線類型都使用 VPN 閘道提供使用 IPsec/IKE 的安全通道，且兩者在通訊時的運作方式相同。 連線類型之間的差異在於區域網路閘道的設定方式。 當您建立 VNet 對 VNet 連線時，會看不到區域網路閘道的位址空間。 系統會自動建立並填入該位址空間。 如果您更新一個 VNet 的位址空間，另一個 VNet 就會自動得知要路由到已更新的位址空間。 相較於在 VNet 之間建立站對站連線，建立 VNet 對 VNet 連線通常比較快速而容易。

### <a name="connecting-vnets-using-site-to-site-ipsec-steps"></a>使用站對站 (IPsec) 步驟來連線 VNet

如果您使用複雜的網路組態，您可能偏好使用[站對站](vpn-gateway-howto-site-to-site-resource-manager-cli.md)步驟 (而不是 VNet 對 VNet 步驟) 來連線 VNet。 當您使用站對站步驟時，您會以手動方式建立及設定區域網路閘道。 每個 VNet 的區域網路閘道都會將其他 VNet 視為本機網站。 這可讓您指定區域網路閘道的其他位址空間，以便路由傳送流量。 如果 VNet 的位址空間變更，您需要手動更新對應的區域網路閘道，才會反映變更。 它並不會自動更新。

### <a name="vnet-peering"></a>VNet 對等互連

建議您使用 VNet 對等互連來進行 VNet 連線。 VNet 對等互連不會使用 VPN 閘道，且具有不同的條件約束。 此外，[VNet 對等互連價格](https://azure.microsoft.com/pricing/details/virtual-network)與 [VNet 對 VNet VPN 閘道價格](https://azure.microsoft.com/pricing/details/vpn-gateway)的計算方式不同。 如需詳細資訊，請參閱 [VNet 對等互連](../virtual-network/virtual-network-peering-overview.md)。

## <a name="why"></a>為何要建立 VNet 對 VNet 連線？

基於下列原因，建議您使用 VNet 對 VNet 連線來進行虛擬網路連線：

* **跨區域的異地備援和異地目前狀態**

  * 您可以使用安全連線設定自己的異地複寫或同步處理，而不用查看網際網路對向端點。
  * 您可以使用 Azure 流量管理員和負載平衡器，利用異地備援跨多個 Azure 區域設定高度可用的工作負載。 其中一個重要的範例就是使用分散在多個 Azure 區域的可用性群組來設定 SQL Always On。
* **具有隔離或管理界限的區域性多層式應用程式**

  * 在相同區域中，您可以因為隔離或管理需求，設定將多層式應用程式與多個虛擬網路連線在一起。

您可以將 VNet 對 VNet 通訊與多站台組態結合。 這可讓您建立結合了跨單位連線與內部虛擬網路連線的網路拓撲。

## <a name="steps"></a>我應該使用哪些 VNet 對 VNet 步驟？

在本文中，您會看到兩組不同的 VNet 對 VNet 連線步驟。 一組步驟適用於[位於相同訂用帳戶中的 VNet](#samesub)，一組步驟適用於[位於不同訂用帳戶中的 VNet](#difsub)。 

在此練習中，您可以合併組態，或只選擇您需要使用的一個組態。 所有組態都會使用 VNet 對 VNet 連線類型。 網路流量會在彼此直接連線的 VNet 之間流動。 在此練習中，來自 TestVNet4 的流量不會路由傳送至 TestVNet5。

* [位於相同訂用帳戶中的 VNet：](#samesub)此組態的步驟會使用 TestVNet1 和 TestVNet4。

  ![v2v 圖表](./media/vpn-gateway-howto-vnet-vnet-cli/v2vrmps.png)

* [位於不同訂用帳戶中的 VNet：](#difsub)此組態的步驟會使用 TestVNet1 和 TestVNet5。

  ![v2v 圖表](./media/vpn-gateway-howto-vnet-vnet-cli/v2vdiffsub.png)


## <a name="samesub"></a>與相同訂用帳戶中的 VNet 連線

### <a name="before-you-begin"></a>開始之前

開始之前，請先安裝 CLI 命令的最新版本 (2.0 版或更新版本)。 如需關於安裝 CLI 命令的資訊，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

### <a name="Plan"></a>規劃 IP 位址範圍

在下列步驟中，您會建立兩個虛擬網路，以及它們各自的閘道子網路和組態。 接著建立這兩個 VNet 之間的 VPN 連線。 請務必規劃您的網路組態的 IP 位址範圍。 請記住，您必須先確定您的 VNet 範圍或區域網路範圍沒有以任何方式重疊。 在這些範例中，我們不會包含 DNS 伺服器。 如果您想要了解虛擬網路的名稱解析，請參閱[名稱解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)。

我們會在範例中使用下列值：

**TestVNet1 的值︰**

* VNet 名稱︰TestVNet1
* 資源群組︰TestRG1
* 位置：美國東部
* TestVNet1：10.11.0.0/16 和 10.12.0.0/16
* FrontEnd：10.11.0.0/24
* BackEnd：10.12.0.0/24
* GatewaySubnet：10.12.255.0/27
* GatewayName：VNet1GW
* 公用 IP: VNet1GWIP
* VPNType：RouteBased
* Connection(1to4)：VNet1toVNet4
* Connection(1to5)：VNet1toVNet5 (適用於不同訂用帳戶中的 Vnet)

**TestVNet4 的值︰**

* VNet 名稱︰TestVNet4
* TestVNet2：10.41.0.0/16 和 10.42.0.0/16
* FrontEnd：10.41.0.0/24
* BackEnd：10.42.0.0/24
* GatewaySubnet：10.42.255.0/27
* 資源群組：TestRG4
* 位置：美國西部
* GatewayName：VNet4GW
* 公用 IP：VNet4GWIP
* VPNType：RouteBased
* 連線︰VNet4toVNet1

### <a name="Connect"></a>步驟 1 - 連線至您的訂用帳戶

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-numbers-include.md)]

### <a name="TestVNet1"></a>步驟 2 - 建立及設定 TestVNet1

1. 建立資源群組。

  ```azurecli
  az group create -n TestRG1  -l eastus
  ```
2. 建立 TestVNet1 及其子網路。 此範例會建立名為 TestVNet1 的虛擬網路和名為 FrontEnd 的子網路。

  ```azurecli
  az network vnet create -n TestVNet1 -g TestRG1 --address-prefix 10.11.0.0/16 -l eastus --subnet-name FrontEnd --subnet-prefix 10.11.0.0/24
  ```
3. 為後端子網路建立其他位址空間。 請注意，在此步驟中，我們會指定稍早建立的位址空間，以及我們想要新增的額外位址空間。 這是因為 [az network vnet update](https://docs.microsoft.com/cli/azure/network/vnet#az_network_vnet_update) 命令會覆寫先前的設定。 使用此命令時，務必指定所有的位址前置詞。

  ```azurecli
  az network vnet update -n TestVNet1 --address-prefixes 10.11.0.0/16 10.12.0.0/16 -g TestRG1
  ```
4. 建立後端子網路。
  
  ```azurecli
  az network vnet subnet create --vnet-name TestVNet1 -n BackEnd -g TestRG1 --address-prefix 10.12.0.0/24 
  ```
5. 建立閘道子網路。 請注意，閘道子網路會命名為 'GatewaySubnet'。 此名稱是必要的。 在此範例中，閘道子網路使用 /27。 雖然您可以建立小至 /29 的閘道子網路，我們建議您選取至少 /28 或 /27，建立包含更多位址的較大子網路。 這將允許足夠的位址，以容納您未來可能需要的其他組態。

  ```azurecli 
  az network vnet subnet create --vnet-name TestVNet1 -n GatewaySubnet -g TestRG1 --address-prefix 10.12.255.0/27
  ```
6. 要求一個公用 IP 位址，以配置給您將建立給 VNet 使用的閘道。 請注意，AllocationMethod 是動態的。 您無法指定想要使用的 IP 位址。 該 IP 位址會以動態方式配置給您的閘道。

  ```azurecli
  az network public-ip create -n VNet1GWIP -g TestRG1 --allocation-method Dynamic
  ```
7. 建立 TestVNet1 的虛擬網路閘道。 VNet 對 VNet 組態需要 RouteBased VpnType。 如果您使用 '--no-wait' 參數執行此命令，您不會看到任何意見反應或輸出。 '--no-wait' 參數允許在背景中建立閘道。 它並不表示 VPN 閘道立即完成建立。 視您使用的閘道 SKU 而定，建立閘道通常可能需要 45 分鐘或更久的時間。

  ```azurecli
  az network vnet-gateway create -n VNet1GW -l eastus --public-ip-address VNet1GWIP -g TestRG1 --vnet TestVNet1 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
  ```

### <a name="TestVNet4"></a>步驟 3 - 建立及設定 TestVNet4

1. 建立資源群組。

  ```azurecli
  az group create -n TestRG4  -l westus
  ```
2. 建立 TestVNet4。

  ```azurecli
  az network vnet create -n TestVNet4 -g TestRG4 --address-prefix 10.41.0.0/16 -l westus --subnet-name FrontEnd --subnet-prefix 10.41.0.0/24
  ```

3. 為 TestVNet4 建立其他子網路。

  ```azurecli
  az network vnet update -n TestVNet4 --address-prefixes 10.41.0.0/16 10.42.0.0/16 -g TestRG4 
  az network vnet subnet create --vnet-name TestVNet4 -n BackEnd -g TestRG4 --address-prefix 10.42.0.0/24 
  ```
4. 建立閘道子網路。

  ```azurecli
   az network vnet subnet create --vnet-name TestVNet4 -n GatewaySubnet -g TestRG4 --address-prefix 10.42.255.0/27
  ```
5. 要求公用 IP 位址。

  ```azurecli
  az network public-ip create -n VNet4GWIP -g TestRG4 --allocation-method Dynamic
  ```
6. 建立 TestVNet4 虛擬網路閘道。

  ```azurecli
  az network vnet-gateway create -n VNet4GW -l westus --public-ip-address VNet4GWIP -g TestRG4 --vnet TestVNet4 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
  ```

### <a name="createconnect"></a>步驟 4 - 建立連線

您現在有兩個具有 VPN 閘道的 VNet。 下一個步驟是建立虛擬網路閘道之間的 VPN 閘道連線。 如果您使用上述範例，則 VNet 閘道位於不同的資源群組。 當閘道位於不同的資源群組時，您必須在建立連線時，識別並指定每個閘道的資源識別碼。 如果 VNet 位於相同的資源群組，您可以使用[第二組指示](#samerg)，因為您不需要指定資源識別碼。

### <a name="diffrg"></a>連線位於不同資源群組的 VNet

1. 從下列命令的輸出取得 VNet1GW 的資源識別碼︰

  ```azurecli
  az network vnet-gateway show -n VNet1GW -g TestRG1
  ```

  在輸出中，尋找 "id:" 行。 需要引號中的值，才能在下一節中建立連線。 將這些值複製到文字編輯器 (例如 [記事本])，以便您在建立連線時輕鬆地貼上。

  範例輸出︰

  ```
  "activeActive": false, 
  "bgpSettings": { 
    "asn": 65515, 
    "bgpPeeringAddress": "10.12.255.30", 
    "peerWeight": 0 
   }, 
  "enableBgp": false, 
  "etag": "W/\"ecb42bc5-c176-44e1-802f-b0ce2962ac04\"", 
  "gatewayDefaultSite": null, 
  "gatewayType": "Vpn", 
  "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW", 
  "ipConfigurations":
  ```

  複製引號中 **"id":** 之後的值。

  ```
  "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
 ```

2. 取得 VNet4GW 的資源識別碼，並將值複製到文字編輯器。

  ```azurecli
  az network vnet-gateway show -n VNet4GW -g TestRG4
  ```

3. 建立 TestVNet1 至 TestVNet4 的連線。 在此步驟中，您會從 TestVNet1 建立連線至 TestVNet4。 範例會一個共用金鑰。 您可以使用自己的值，作為共用金鑰。 但請務必確認該共用金鑰必須適用於這兩個連線。 建立連線可能需要一段時間才能完成。

  ```azurecli
  az network vpn-connection create -n VNet1ToVNet4 -g TestRG1 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW -l eastus --shared-key "aabbcc" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG4/providers/Microsoft.Network/virtualNetworkGateways/VNet4GW 
  ```
4. 建立 TestVNet4 至 TestVNet1 的連線。 此步驟類似上面的步驟，只不過您是建立 TestVNet4 至 TestVNet1 的連線。 請確認共用的金鑰相符。 建立連線需花費數分鐘的時間。

  ```azurecli
  az network vpn-connection create -n VNet4ToVNet1 -g TestRG4 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG4/providers/Microsoft.Network/virtualNetworkGateways/VNet4GW -l westus --shared-key "aabbcc" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1G
  ```
5. 確認您的連線。 請參閱[驗證您的連線](#verify)。

### <a name="samerg"></a>與位於相同資源群組的 VNet 連線

1. 建立 TestVNet1 至 TestVNet4 的連線。 在此步驟中，您會從 TestVNet1 建立連線至 TestVNet4。 請注意，在範例中的資源群組相同。 您也會看到範例使用一個共用金鑰。 您可以將自己的值使用於共用金鑰，不過，共用金鑰必須符合這兩個連線。 建立連線可能需要一段時間才能完成。

  ```azurecli
  az network vpn-connection create -n VNet1ToVNet4 -g TestRG1 --vnet-gateway1 VNet1GW -l eastus --shared-key "eeffgg" --vnet-gateway2 VNet4GW
  ```
2. 建立 TestVNet4 至 TestVNet1 的連線。 此步驟類似上面的步驟，只不過您是建立 TestVNet4 至 TestVNet1 的連線。 請確認共用的金鑰相符。 建立連線需花費數分鐘的時間。

  ```azurecli
  az network vpn-connection create -n VNet4ToVNet1 -g TestRG1 --vnet-gateway1 VNet4GW -l eastus --shared-key "eeffgg" --vnet-gateway2 VNet1GW
  ```
3. 確認您的連線。 請參閱[驗證您的連線](#verify)。

## <a name="difsub"></a>與位於不同訂用帳戶的 VNet 連線

在此案例中，您會連接 TestVNet1 和 TestVNet5。 VNet 位於不同的訂用帳戶。 訂用帳戶不需與相同的 Active Directory 租用戶相關聯。 此設定的步驟會加入其他的 VNet 對 VNet 連線，以便將 TestVNet1 連接到 TestVNet5。

### <a name="TestVNet1diff"></a>步驟 5 - 建立及設定 TestVNet1

這些指示延續自前幾節中的步驟。 您必須完成[步驟 1](#Connect) 和[步驟 2](#TestVNet1)，以建立並設定 TestVNet1 和 TestVNet1 的 VPN 閘道。 在此設定中，您不需要建立前一節的 TestVNet4 ，雖然您若建立它，它就不與這些步驟發生衝突。 完成步驟 1 和步驟 2 後，繼續進行下面的步驟 6。

### <a name="verifyranges"></a>步驟 6 - 驗證 IP 位址範圍

建立其他連線時，請務必確認新虛擬網路的 IP 位址空間不會與任何其他 VNet 範圍或區域網路閘道範圍重疊。 在這個練習中，您可以對 TestVNet5 使用下列的值：

**TestVNet5 的值︰**

* VNet 名稱︰TestVNet5
* 資源群組：TestRG5
* 位置：日本東部
* TestVNet5：10.51.0.0/16 和 10.52.0.0/16
* FrontEnd：10.51.0.0/24
* BackEnd：10.52.0.0/24
* GatewaySubnet：10.52.255.0.0/27
* GatewayName：VNet5GW
* 公用 IP：VNet5GWIP
* VPNType：RouteBased
* 連線︰VNet5toVNet1
* ConnectionType：VNet2VNet

### <a name="TestVNet5"></a>步驟 7 - 建立及設定 TestVNet5

在新訂用帳戶 (訂用帳戶 5) 的內容中，必須完成這個步驟。 此部分可能會由不同組織中擁有訂用帳戶的系統管理員執行。 若要使用 'az account list --all' 來切換訂用帳戶，以列出您的帳戶可用的訂用帳戶，則使用 'az account set --subscription <subscriptionID>' 來切換至您想要使用的訂用帳戶。

1. 確定您已連線到訂用帳戶 5，然後建立資源群組。

  ```azurecli
  az group create -n TestRG5  -l japaneast
  ```
2. 建立 TestVNet5。

  ```azurecli
  az network vnet create -n TestVNet5 -g TestRG5 --address-prefix 10.51.0.0/16 -l japaneast --subnet-name FrontEnd --subnet-prefix 10.51.0.0/24
  ```

3. 新增子網路。

  ```azurecli
  az network vnet update -n TestVNet5 --address-prefixes 10.51.0.0/16 10.52.0.0/16 -g TestRG5
  az network vnet subnet create --vnet-name TestVNet5 -n BackEnd -g TestRG5 --address-prefix 10.52.0.0/24
  ```

4. 新增閘道子網路。

  ```azurecli
  az network vnet subnet create --vnet-name TestVNet5 -n GatewaySubnet -g TestRG5 --address-prefix 10.52.255.0/27
  ```

5. 要求公用 IP 位址。

  ```azurecli
  az network public-ip create -n VNet5GWIP -g TestRG5 --allocation-method Dynamic
  ```
6. 建立 TestVNet5 閘道

  ```azurecli
  az network vnet-gateway create -n VNet5GW -l japaneast --public-ip-address VNet5GWIP -g TestRG5 --vnet TestVNet5 --gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait
  ```

### <a name="connections5"></a>步驟 8 - 建立連線

此步驟會分成標示為 **[訂用帳戶 1]** 和 **[訂用帳戶 5]** 的兩個 CLI 工作階段，因為閘道位於不同的訂用帳戶。 若要使用 'az account list --all' 來切換訂用帳戶，以列出您的帳戶可用的訂用帳戶，則使用 'az account set --subscription <subscriptionID>' 來切換至您想要使用的訂用帳戶。

1. **[訂用帳戶 1]** 登入並連線到訂用帳戶 1。 執行下列命令，從輸出中取得閘道的名稱和識別碼︰

  ```azurecli
  az network vnet-gateway show -n VNet1GW -g TestRG1
  ```

  複製 "id:" 的輸出。 透過電子郵件或其他方法，將 VNet 閘道 (VNet1GW) 的識別碼和名稱傳送給訂用帳戶 5 的系統管理員。

  範例輸出︰

  ```
  "id": "/subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
  ```

2. **[訂用帳戶 5]** 登入並連線到訂用帳戶 5。 執行下列命令，從輸出中取得閘道的名稱和識別碼︰

  ```azurecli
  az network vnet-gateway show -n VNet5GW -g TestRG5
  ```

  複製 "id:" 的輸出。 透過電子郵件或其他方法，將 VNet 閘道 (VNet5GW) 的識別碼和名稱傳送給訂用帳戶 1 的系統管理員。

3. **[訂用帳戶 1]** 在此步驟中，您會從 TestVNet1 建立連線至 TestVNet5。 您可以將自己的值使用於共用金鑰，不過，共用金鑰必須符合這兩個連線。 建立連線可能需要一段時間才能完成。 確定您已連接到訂用帳戶 1。

  ```azurecli
  az network vpn-connection create -n VNet1ToVNet5 -g TestRG1 --vnet-gateway1 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW -l eastus --shared-key "eeffgg" --vnet-gateway2 /subscriptions/e7e33b39-fe28-4822-b65c-a4db8bbff7cb/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW
  ```

4. **[訂用帳戶 5]** 此步驟類似上面的步驟，只不過您是建立 TestVNet5 至 TestVNet1 的連線。 確定共用金鑰相符，而且您會連線至訂用帳戶 5。

  ```azurecli
  az network vpn-connection create -n VNet5ToVNet1 -g TestRG5 --vnet-gateway1 /subscriptions/e7e33b39-fe28-4822-b65c-a4db8bbff7cb/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW -l japaneast --shared-key "eeffgg" --vnet-gateway2 /subscriptions/d6ff83d6-713d-41f6-a025-5eb76334fda9/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
  ```

## <a name="verify"></a>驗證連線
[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

[!INCLUDE [verify connections](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]

## <a name="faq"></a>VNet 對 VNet 常見問題集
[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>後續步驟

* 一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 如需詳細資訊，請參閱[虛擬機器文件](https://docs.microsoft.com/azure/#pivot=services&panel=Compute)。
* 如需 BGP 的相關資訊，請參閱 [BGP 概觀](vpn-gateway-bgp-overview.md)和[如何設定 BGP](vpn-gateway-bgp-resource-manager-ps.md)。