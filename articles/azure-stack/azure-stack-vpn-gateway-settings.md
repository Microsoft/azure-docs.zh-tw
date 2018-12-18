---
title: Azure Stack 的 VPN 閘道設定 | Microsoft Docs
description: 了解您搭配 Azure Stack 使用的 VPN 閘道設定。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: fa8d3adc-8f5a-4b4f-8227-4381cf952c56
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/14/2018
ms.author: sethm
ms.openlocfilehash: 2fa062621e551ce7182facc45ec84b39d4c2dad7
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078744"
---
# <a name="vpn-gateway-configuration-settings-for-azure-stack"></a>Azure Stack 的 VPN 閘道組態設定

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

VPN 閘道是一種虛擬網路閘道，可在 Azure Stack 中的虛擬網路與遠端 VPN 閘道之間傳送加密流量。 遠端 VPN 閘道可位於 Azure 中、您資料中心的裝置中或另一個網站的裝置中。  如果兩個端點之間有網路連線，您可以在這兩個網路之間建立安全的站對站 (S2S) VPN 連線。

VPN 閘道連線依賴多個資源的設定，每一個都包含可設定的設定值。 本文針對您在 Resource Manager 部署模型中建立的虛擬網路，討論其 VPN 閘道的相關資源和設定。 您可以在[關於 Azure Stack 的 VPN 閘道](azure-stack-vpn-gateway-about-vpn-gateways.md)中找到每個連線解決方案的描述和拓撲圖。

## <a name="vpn-gateway-settings"></a>VPN 閘道設定

### <a name="gateway-types"></a>閘道類型

每個 Azure Stack 虛擬網路都支援單一虛擬網路閘道，其類型必須是 **Vpn**。  這項支援不同於 Azure，後者可支援其他類型。  

在建立虛擬網路閘道時，您必須確定組態的閘道類型是正確的。 VPN 閘道需要 `-GatewayType Vpn`，例如：

```PowerShell
New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
-Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn
-VpnType RouteBased
```

### <a name="gateway-skus"></a>閘道 SKU

建立虛擬網路閘道時，您必須指定想要使用的閘道 SKU。 根據工作負載、輸送量、功能和 SLA 的類型，選取符合您需求的 SKU。

Azure Stack 會提供下表中顯示的 VPN 閘道 SKU。

|   | VPN 閘道輸送量 |VPN 閘道最大 IPsec 通道 |
|-------|-------|-------|
|**基本 SKU**  | 100 Mbps  | 10    |
|**標準 SKU**           | 100 Mbps  | 10    |
|**高效能 SKU** | 200 Mbps    | 5 |

### <a name="resizing-gateway-skus"></a>調整閘道 SKU 的大小

Azure Stack 不支援在支援的舊版 SKU 之間調整 SKU 大小。

同樣地，Azure Stack 不支援將大小從支援的舊版 SKU (基本、標準和高效能) 調整為 Azure 所支援的新版 SKU (VpnGw1、VpnGw2 和 VpnGw3)。

### <a name="configure-the-gateway-sku"></a>設定閘道 SKU

#### <a name="azure-stack-portal"></a>Azure Stack 入口網站

如果您使用 Azure Stack 入口網站來建立 Resource Manager 虛擬網路閘道，可以使用下拉式清單來選取閘道 SKU。 您看到的選項會對應於您所選取的閘道類型和 VPN 類型。

#### <a name="powershell"></a>PowerShell

下列 PowerShell 範例將 **-GatewaySku** 指定為 VpnGw1。

```PowerShell
New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
-Location 'West US' -IpConfigurations $gwipconfig -GatewaySku VpnGw1
-GatewayType Vpn -VpnType RouteBased
```

### <a name="connection-types"></a>連線類型

在 Resource Manager 部署模型中，每個組態皆需要特定的虛擬網路閘道連線類型。 **-ConnectionType** 的可用 Resource Manager PowerShell 值為：

* IPsec

下列 PowerShell 範例會建立需要 IPsec 連線類型的 S2S 連線。

```PowerShell
New-AzureRmVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg
-Location 'West US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local
-ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
```

### <a name="vpn-types"></a>VPN 類型

當您為 VPN 閘道組態建立虛擬網路閘道時，必須指定 VPN 類型。 您所選擇的 VPN 類型取決於您想要建立的連線拓撲。  VPN 類型也取決於您所使用的硬體。 S2S 組態需要 VPN 裝置。 有些 VPN 裝置僅支援特定 VPN 類型。

> [!IMPORTANT]  
> 目前，Azure Stack 僅支援路由式 VPN 類型。 如果您的裝置僅支援原則式 VPN，則不支援從 Azure Stack 連線至這些裝置。  
>
> 此外，Azure Stack 目前並不支援將原則式流量選取器用於路由式閘道，因為自訂 IPSec/IKE 原則組態不受支援。

* **原則式**：原則式 VPN 會根據使用內部部署網路與 Azure Stack VNet 之間的位址首碼組合所設定的 IPsec 原則，透過 IPsec 通道來加密和導向封包。 原則 (或流量選取器) 通常為 VPN 裝置組態中的存取清單。

  >[!NOTE]
  >原則式在 Azure 中受到支援，但在 Azure Stack 中則不受支援。

* **路由式**：路由式 VPN 會使用在 IP 轉送或路由表中設定的路由，將封包導向至其對應的通道介面。 然後，通道介面會加密或解密輸入和輸出通道的封包。 路由式 VPN 的原則 (或流量選取器) 會設定為任何對任何 (或使用萬用字元)。根據預設，這些設定無法變更。 路由式 VPN 類型的值是路由式。

下列 PowerShell 範例將 **-VpnType** 指定為 RouteBased。 在建立閘道時，您必須確定用於組態的 **-VpnType** 是正確的。

```PowerShell
New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
-Location 'West US' -IpConfigurations $gwipconfig
-GatewayType Vpn -VpnType RouteBased
```

### <a name="gateway-requirements"></a>閘道需求

下表列出 VPN 閘道的需求。

| |原則式基本 VPN 閘道 | 路由式基本 VPN 閘道 | 路由式標準 VPN 閘道 | 路由式高效能 VPN 閘道|
|--|--|--|--|--|
| **站對站連線能力 (S2S 連線能力)** | 不支援 | RouteBased VPN 組態 | RouteBased VPN 組態 | RouteBased VPN 組態 |
| **驗證方法**  | 不支援 | S2S 連線的預先共用金鑰  | S2S 連線的預先共用金鑰  | S2S 連線的預先共用金鑰  |   
| **S2S 連接的數目上限**  | 不支援 | 10 | 10| 5|
|**作用中路由支援 (BGP)** | 不支援 | 不支援 | 支援 | 支援 |

### <a name="gateway-subnet"></a>閘道器子網路

建立 VPN 閘道之前，您必須先建立閘道子網路。 閘道子網路含有虛擬網路閘道 VM 與服務所使用的 IP 位址。 建立虛擬網路閘道時，會將閘道 VM 部署到閘道子網路，並為 VM 設定必要的 VPN 閘道設定。 **請勿**將任何其他項目 (例如其他 VM) 部署到閘道子網路。

>[!IMPORTANT]
>此閘道子網路必須命名為 **GatewaySubnet** 才能正常運作。 Azure Stack 會使用此名稱來識別虛擬網路閘道 VM 和服務所要部署到的子網路。

當您建立閘道子網路時，您可指定子網路包含的 IP 位址數目。 閘道子網路中的 IP 位址會配置給閘道 VM 和閘道服務。 有些組態需要的 IP 位址比其他組態多。 請查看您想要建立之組態的指示﹐並確認您想要建立的閘道子網路將符合這些需求。

此外，您應確定閘道子網路有足夠的 IP 位址可處理未來的額外組態。 雖然您可以建立像 /29 這麼小的閘道子網路，但仍建議您建立 /28 或更大 (/28、/27、/26 等) 的閘道子網路。如此，如果您在未來新增功能，就不須卸除閘道然後刪除並重新建立閘道子網路，以提供更多的 IP 位址。

下列 Resource Manager PowerShell 範例顯示名為 GatewaySubnet 的閘道子網路。 您可以看到 CIDR 標記法指定 /27，這可提供足以供大多數現有組態使用的 IP 位址。

```PowerShell
Add-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/27
```

> [!IMPORTANT]
> 使用閘道子網路時，避免將網路安全性群組 (NSG) 與閘道子網路產生關聯。 將網路安全性群組與此子網路產生關聯，可能會導致您的 VPN 閘道如預期般停止運作。 如需有關網路安全性群組的詳細資訊，請參閱[什麼是網路安全性群組？](/azure/virtual-network/virtual-networks-nsg)。

### <a name="local-network-gateways"></a>區域網路閘道

在 Azure 中建立 VPN 閘道組態時，區域網路閘道通常代表您的內部部署位置。 在 Azure Stack 中，它代表位於 Azure Stack 外部的任何遠端 VPN 裝置。 這可能是您的資料中心 (或遠端資料中心) 中的 VPN 裝置，或 Azure 中的 VPN 閘道。

您需指定區域網路閘道的名稱 (即 VPN 裝置的公用 IP 位址)，並指定內部部署位置上的位址首碼。 Azure 會查看網路流量的目的地位址首碼、查閱您為區域網路閘道指定的組態，然後根據這些來路由傳送封包。

下一個 PowerShell 範例會建立新的區域網路閘道︰

```PowerShell
New-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg
-Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.5.51.0/24'
```

有時，您會需要修改區域網路閘道設定。 例如，當您新增或修改位址範圍時，或 VPN 裝置的 IP 位址變更時。 請參閱[使用 PowerShell 修改區域網路閘道設定](/azure/vpn-gateway/vpn-gateway-modify-local-network-gateway)。

## <a name="ipsecike-parameters"></a>IPsec/IKE 參數

當您在 Azure Stack 中設定 VPN 連線時，您需要在兩端設定連線。  如果您要設定 Azure Stack 與硬體裝置 (例如作為 VPN 閘道的交換器或路由器) 之間的 VPN 連線，則該裝置可能會要求您進行其他設定。

不同於可支援多個供應項目同時作為啟動程式和回應程式的 Azure，Azure Stack 只支援一個供應項目。

### <a name="ike-phase-1-main-mode-parameters"></a>IKE 階段 1 (主要模式) 參數

| 屬性              | 值|
|-|-|
| IKE 版本           | IKEv2 |
|Diffie-Hellman 群組   | 群組 2 (1024 位元) |
| 驗證方法 | 預先共用金鑰 |
|加密與雜湊演算法 | AES256、SHA256 |
|SA 存留期 (時間)     | 28,800 秒|

### <a name="ike-phase-2-quick-mode-parameters"></a>IKE 階段 2 (快速模式) 參數

| 屬性| 值|
|-|-|
|IKE 版本 |IKEv2 |
|加密與雜湊演算法 (加密)     | GCMAES256|
|加密與雜湊演算法 (驗證) | GCMAES256|
|SA 存留期 (時間)  | 27,000 秒  |
|SA 存留期 (位元組) | 33,553,408     |
|完整轉寄密碼 (PFS) |無<sup>請參閱附註 1</sup> |
|停用的對等偵測 | 支援|  

* *附註 1：* 在 1807 之前的版本，Azure Stack 會使用 PFS2048 作為完整轉寄密碼 (PFS) 的值。

## <a name="next-steps"></a>後續步驟

[使用 ExpressRoute 連線](azure-stack-connect-expressroute.md)
