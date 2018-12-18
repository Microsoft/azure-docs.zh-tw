---
title: 在 Azure VPN 閘道上設定 BGP：Resource Manager 和 CLI | Microsoft Docs
description: 本文將逐步引導您使用 Azure Resource Manager 和 CLI 來設定 Azure VPN 閘道上的 BGP。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: cherylmc
ms.openlocfilehash: 657cad69fff66aaaeba9819569442981d229f644
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971122"
---
# <a name="how-to-configure-bgp-on-an-azure-vpn-gateway-by-using-cli"></a>如何使用 CLI 在 Azure VPN 閘道上設定 BGP

本文將協助您使用 Azure Resource Manager 部署模型和 Azure CLI，在跨單位網站間 (S2S) VPN 連線和 VNet 對 VNet 連線 (也就是虛擬網路間的連線) 上啟用 BGP。

## <a name="about-bgp"></a>關於 BGP

BGP 是常用於網際網路的標準路由通訊協定，可交換兩個或多個網路之間的路由和連線性資訊。 BGP 能使 VPN 閘道和您的內部部署 VPN 裝置 (稱為 BGP 對等體或芳鄰) 得以交換路由。 路由會向這兩個閘道通知有關前置詞通過所涉及之閘道或路由的可用性和連線性資訊。 BGP 也可以將 BGP 閘道從某個 BGP 對等體所取得的路由傳播到所有其他 BGP 對等體，以啟用多個網路之間的傳輸路由。

如需 BGP 優點的詳細資訊，以及了解使用 BGP 的技術需求和考量，請參閱 [BGP 與 Azure VPN 閘道概觀](vpn-gateway-bgp-overview.md)。

本文可協助您完成下列工作：

* [為您的 VPN 閘道啟用 BGP](#enablebgp) (必要)

  您可以接著完成下列其中一個小節 (或兩者) ：

* [建立與 BGP 的跨單位連線](#crossprembgp)
* [建立與 BGP 的 VNet 對 VNet 連線](#v2vbgp)

這三個小節的內容所提及的工作，皆為在網路連線中啟用 BGP 的基本建構元素。 如果您完成這三個小節的步驟，您將會建置出如下圖所示的拓撲︰

![BGP 拓樸](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

您可以結合這些小節的內容，以建置出更加複雜且能符合您需求的多重躍點傳輸網路。

## <a name ="enablebgp"></a>為您的 VPN 閘道啟用 BGP

在您可以執行其他兩個設定小節中的任何步驟之前，必須先完成本節的內容。 下列設定步驟會設定 Azure VPN 閘道的 BGP 參數，如下圖所示︰

![BGP 閘道](./media/vpn-gateway-bgp-resource-manager-ps/bgp-gateway.png)

### <a name="before-you-begin"></a>開始之前

安裝 CLI 命令的最新版本 (2.0 版或更新版本)。 如需關於安裝 CLI 命令的資訊，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli) 和[開始使用 Azure CLI](/cli/azure/get-started-with-azure-cli)。

### <a name="step-1-create-and-configure-testvnet1"></a>步驟 1：建立及設定 TestVNet1

#### <a name="Login"></a>1.連線至您的訂用帳戶

[!INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-include.md)]

#### <a name="2-create-a-resource-group"></a>2.建立資源群組

下列範例會在 "eastus" 位置建立名為 TestRG1 的資源群組。 如果您在該區域中已有想要用來建立虛擬網路的資源群組，則可以改為使用該資源群組。

```azurecli
az group create --name TestBGPRG1 --location eastus
```

#### <a name="3-create-testvnet1"></a>3.建立 TestVNet1

下列範例會建立一個名為 TestVNet1 的虛擬網路，以及三個子網路：GatewaySubnet、FrontEnd 和 Backend。 當您在替代值時，請務必將閘道子網路命名為 GatewaySubnet。 如果您將其命名為其他名稱，閘道建立會失敗。

第一個命令會建立前端位址空間及 FrontEnd 子網路。 第二個命令會為後端子網路建立額外的位址空間。 第三個和第四個命令會建立 BackEnd 子網路和 GatewaySubnet。

```azurecli
az network vnet create -n TestVNet1 -g TestBGPRG1 --address-prefix 10.11.0.0/16 -l eastus --subnet-name FrontEnd --subnet-prefix 10.11.0.0/24 
 
az network vnet update -n TestVNet1 --address-prefixes 10.11.0.0/16 10.12.0.0/16 -g TestBGPRG1 
 
az network vnet subnet create --vnet-name TestVNet1 -n BackEnd -g TestBGPRG1 --address-prefix 10.12.0.0/24 
 
az network vnet subnet create --vnet-name TestVNet1 -n GatewaySubnet -g TestBGPRG1 --address-prefix 10.12.255.0/27 
```

### <a name="step-2-create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>步驟 2：使用 BGP 參數建立 TestVNet1 的 VPN 閘道

#### <a name="1-create-the-public-ip-address"></a>1.建立公用 IP 位址

要求公用 IP 位址。 公用 IP 位址將會配置給您為虛擬網路建立的 VPN 閘道。

```azurecli
az network public-ip create -n GWPubIP -g TestBGPRG1 --allocation-method Dynamic 
```

#### <a name="2-create-the-vpn-gateway-with-the-as-number"></a>2.透過 AS 號碼建立 VPN 閘道

建立 TestVNet1 的虛擬網路閘道。 BGP 需要路由式 VPN 閘道。 您也需要額外的參數 `-Asn` 來為 TestVNet1 設定自發系統編號 (ASN)。 建立閘道可能需要花費一段時間 (45 分鐘或更久) 才能完成。 

如果您使用 `--no-wait` 參數來執行此命令，就不會看到任何意見反應或輸出。 `--no-wait` 參數允許在背景中建立閘道。 但是，並不表示 VPN 閘道會立即建立。

```azurecli
az network vnet-gateway create -n VNet1GW -l eastus --public-ip-address GWPubIP -g TestBGPRG1 --vnet TestVNet1 --gateway-type Vpn --sku HighPerformance --vpn-type RouteBased --asn 65010 --no-wait
```

#### <a name="3-obtain-the-azure-bgp-peer-ip-address"></a>3.取得 Azure BGP 對等體 IP 位址

建立閘道後，您必須取得 Azure VPN 閘道上的 BGP 對等體 IP 位址。 需要有此位址，才能將 VPN 閘道設定為內部部署 VPN 裝置的 BGP 對等體。

執行下列命令並檢查輸出頂端的 `bgpSettings` 區段：

```azurecli
az network vnet-gateway list -g TestBGPRG1 
 
  
"bgpSettings": { 
      "asn": 65010, 
      "bgpPeeringAddress": "10.12.255.30", 
      "peerWeight": 0 
    }
```

建立閘道後，您可以使用此閘道來建立與 BGP 的跨單位連線或 VNet 對 VNet 連線。

## <a name ="crossprembgp"></a>建立與 BGP 的跨單位連線

若要建立跨單位連線，您需要建立區域網路閘道來代表您的內部部署 VPN 裝置。 然後，就可以透過區域網路閘道來與 Azure VPN 閘道連線。 雖然這些步驟和建立其他連線類似，但是其中包含指定 BGP 設定參數所需的其他屬性。

![跨單位的 BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crossprem.png)


### <a name="step-1-create-and-configure-the-local-network-gateway"></a>步驟 1：建立及設定區域網路閘道

本練習將繼續建置圖中所示的組態。 請務必使用您想用於設定的值來取代該值。 處理區域網路閘道時，請留意下列事項：

* 區域網路閘道可以位於與 VPN 閘道相同的位置和資源群組中，或位於不同的位置和資源群組中。 此範例會顯示閘道位於不同位置的不同資源群組中。
* 您需要針對區域網路閘道宣告的最小前置詞是 VPN 裝置上 BGP 對等體 IP 位址的主機位址。 在此情況下是 10.52.255.254/32 的前置詞 /32。
* 請注意，您必須在內部部署網路與 Azure 虛擬網路之間使用不同的 BGP ASN。 在兩者相同的情況下，如果內部部署 VPN 裝置已經使用您的 VNet ASN 來與其他 BGP 芳鄰進行對等，您就需要變更該 ASN。

繼續進行之前，請確定您已完成本練習的[為您的 VPN 閘道啟用 BGP](#enablebgp)一節，且仍然與訂用帳戶 1 保持連線。 請注意，在此範例中，您會建立新的資源群組。 同時，也請注意區域網路閘道的兩個額外參數︰`Asn` 與 `BgpPeerAddress`。

```azurecli
az group create -n TestBGPRG5 -l eastus2 
 
az network local-gateway create --gateway-ip-address 23.99.221.164 -n Site5 -g TestBGPRG5 --local-address-prefixes 10.51.255.254/32 --asn 65050 --bgp-peering-address 10.51.255.254
```

### <a name="step-2-connect-the-vnet-gateway-and-local-network-gateway"></a>步驟 2：連線 VNet 閘道與區域網路閘道

在此步驟中，您將建立從 TestVNet1 至 Site5 的連線。 您必須指定 `--enable-bgp` 參數才能為此連線啟用 BGP。 

在此範例中，虛擬網路閘道和區域網路閘道是位於不同的資源群組中。 當閘道位於不同的資源群組中時，您必須指定這兩個閘道的整個資源識別碼，才能在虛擬網路之間設定連線。

#### <a name="1-get-the-resource-id-of-vnet1gw"></a>1.取得 VNet1GW 的資源識別碼

使用下列命令的輸出來取得 VNet1GW 的資源識別碼︰

```azurecli
az network vnet-gateway show -n VNet1GW -g TestBGPRG1
```

在輸出中，尋找 `"id":` 行。 您需要引號內的值，以在下一節中建立連線。

範例輸出︰

```
{ 
  "activeActive": false, 
  "bgpSettings": { 
    "asn": 65010, 
    "bgpPeeringAddress": "10.12.255.30", 
    "peerWeight": 0 
  }, 
  "enableBgp": true, 
  "etag": "W/\"<your etag number>\"", 
  "gatewayDefaultSite": null, 
  "gatewayType": "Vpn", 
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW",
```

將 `"id":` 之後的值複製到文字編輯器 (例如 [記事本])，以便在建立連線時能夠輕鬆地貼上。 

```
"id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW"
```

#### <a name="2-get-the-resource-id-of-site5"></a>2.取得 Site5 的資源識別碼

使用下列命令來從輸出取得 Site5 的資源識別碼︰

```azurecli
az network local-gateway show -n Site5 -g TestBGPRG5
```

#### <a name="3-create-the-testvnet1-to-site5-connection"></a>3.建立 TestVNet1 對 Site5 連線

在此步驟中，您將建立從 TestVNet1 至 Site5 的連線。 如先前所討論，相同的 Azure VPN 閘道可以同時有 BGP 和非 BGP 連線。 除非已在連線屬性中啟用 BGP，否則即使已在兩個閘道上設定 BGP 參數，Azure 也不會為此連線啟用 BGP。 請將訂用帳戶識別碼以您自己的訂用帳戶識別碼來取代。

```azurecli
az network vpn-connection create -n VNet1ToSite5 -g TestBGPRG1 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW --enable-bgp -l eastus --shared-key "abc123" --local-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG5/providers/Microsoft.Network/localNetworkGateways/Site5 --no-wait
```

針對本練習，下列範例會列出要輸入至內部部署 VPN 裝置 BGP 設定區段中的參數︰

```
Site5 ASN            : 65050
Site5 BGP IP         : 10.52.255.254
Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
Azure VNet ASN       : 65010
Azure VNet BGP IP    : 10.12.255.30
Static route         : Add a route for 10.12.255.30/32, with nexthop being the VPN tunnel interface on your device
eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

請稍候幾分鐘，應該就會建立連線。 BGP 對等處理工作階段會在建立 IPsec 連線之後開始。

## <a name ="v2vbgp"></a>建立與 BGP 的 VNet 對 VNet 連線

本節新增與 BGP 的 VNet 對 VNet 連線，如下圖所示： 

![VNet 對 VNet 的 BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-vnet2vnet.png)

以下指示延續自前幾節中所述的步驟。 若要利用 BGP 建立和設定 TestVNet1 和 VPN 閘道，您必須完成[為您的 VPN 閘道啟用 BGP](#enablebgp)一節。

### <a name="step-1-create-testvnet2-and-the-vpn-gateway"></a>步驟 1：建立 TestVNet2 和 VPN 閘道

請務必確定新虛擬網路 TestVNet2 的 IP 位址空間不會與任何 VNet 範圍重疊。

在此範例中，虛擬網路屬於相同的訂用帳戶。 您可以設定不同訂用帳戶之間的 VNet 對 VNet 連線。 若要深入了解，請參閱[設定 VNet 對 VNet 連線](vpn-gateway-howto-vnet-vnet-cli.md)。 建立連線時，請務必新增 `-EnableBgp $True`，才能啟用 BGP。

#### <a name="1-create-a-new-resource-group"></a>1.建立新的資源群組

```azurecli
az group create -n TestBGPRG2 -l westus
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2.在新的資源群組中建立 TestVNet2

第一個命令會建立前端位址空間及 FrontEnd 子網路。 第二個命令會為後端子網路建立額外的位址空間。 第三個和第四個命令會建立 BackEnd 子網路和 GatewaySubnet。

```azurecli
az network vnet create -n TestVNet2 -g TestBGPRG2 --address-prefix 10.21.0.0/16 -l westus --subnet-name FrontEnd --subnet-prefix 10.21.0.0/24 
 
az network vnet update -n TestVNet2 --address-prefixes 10.21.0.0/16 10.22.0.0/16 -g TestBGPRG2 
 
az network vnet subnet create --vnet-name TestVNet2 -n BackEnd -g TestBGPRG2 --address-prefix 10.22.0.0/24 
 
az network vnet subnet create --vnet-name TestVNet2 -n GatewaySubnet -g TestBGPRG2 --address-prefix 10.22.255.0/27
```

#### <a name="3-create-the-public-ip-address"></a>3.建立公用 IP 位址

要求公用 IP 位址。 公用 IP 位址將會配置給您為虛擬網路建立的 VPN 閘道。

```azurecli
az network public-ip create -n GWPubIP2 -g TestBGPRG2 --allocation-method Dynamic
```

#### <a name="4-create-the-vpn-gateway-with-the-as-number"></a>4.透過 AS 號碼建立 VPN 閘道

建立 TestVNet2 的虛擬網路閘道。 您必須覆寫您 Azure VPN 閘道上的預設 ASN。 已連線虛擬網路的 ASN 必須不同，才能啟用 BGP 與傳輸路由。
 
```azurecli
az network vnet-gateway create -n VNet2GW -l westus --public-ip-address GWPubIP2 -g TestBGPRG2 --vnet TestVNet2 --gateway-type Vpn --sku Standard --vpn-type RouteBased --asn 65020 --no-wait
```

### <a name="step-2-connect-the-testvnet1-and-testvnet2-gateways"></a>步驟 2：連線 TestVNet1 與 TestVNet2 閘道

在此步驟中，您將建立從 TestVNet1 至 Site5 的連線。 若要為此連線啟用 BGP，您必須指定 `--enable-bgp` 參數。

在下列範例中，虛擬網路閘道和區域網路閘道是位於不同的資源群組中。 當閘道位於不同的資源群組中時，您必須指定這兩個閘道的整個資源識別碼，才能在虛擬網路之間設定連線。 

#### <a name="1-get-the-resource-id-of-vnet1gw"></a>1.取得 VNet1GW 的資源識別碼 

從下列命令的輸出取得 VNet1GW 的資源識別碼︰

```azurecli
az network vnet-gateway show -n VNet1GW -g TestBGPRG1
```

#### <a name="2-get-the-resource-id-of-vnet2gw"></a>2.取得 VNet2GW 的資源識別碼

從下列命令的輸出取得 VNet2GW 的資源識別碼︰

```azurecli
az network vnet-gateway show -n VNet2GW -g TestBGPRG2
```

#### <a name="3-create-the-connections"></a>3.建立連線

建立從 TestVNet1 到 TestVNet2 的連線，以及從 TestVNet2 到 TestVNet1 的連線。 請將訂用帳戶識別碼以您自己的訂用帳戶識別碼來取代。

```azurecli
az network vpn-connection create -n VNet1ToVNet2 -g TestBGPRG1 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW --enable-bgp -l eastus --shared-key "efg456" --vnet-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG2/providers/Microsoft.Network/virtualNetworkGateways/VNet2GW
```

```azurecli
az network vpn-connection create -n VNet2ToVNet1 -g TestBGPRG2 --vnet-gateway1 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG2/providers/Microsoft.Network/virtualNetworkGateways/VNet2GW --enable-bgp -l westus --shared-key "efg456" --vnet-gateway2 /subscriptions/<subscription ID>/resourceGroups/TestBGPRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
```

> [!IMPORTANT]
> 對這兩個連線啟用 BGP。
> 
> 

完成這些步驟之後，連線將會在幾分鐘之後建立。 當 VNet 對 VNet 連線完成後，便會啟動 BGP 對等處理工作階段。

## <a name="next-steps"></a>後續步驟

一旦完成您的連線，就可以將虛擬機器加入您的虛擬網路。 如需相關步驟，請參閱[建立虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
