---
title: 使用 VPN 閘道與 PowerShell 將虛擬網路連線到多個網站：傳統 | Microsoft Docs
description: 使用 VPN 閘道，將多個本機內部部署網站連線至傳統虛擬網路。
services: vpn-gateway
documentationcenter: na
author: yushwang
manager: rossort
editor: ''
tags: azure-service-management
ms.assetid: b043df6e-f1e8-4a4d-8467-c06079e2c093
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/14/2018
ms.author: yushwang
ms.openlocfilehash: c0af4271df0e88354edb717b8d6f4c99ab29e573
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29399367"
---
# <a name="add-a-site-to-site-connection-to-a-vnet-with-an-existing-vpn-gateway-connection-classic"></a>將站對站連線新增至具有現有 VPN 閘道連線的 VNet (傳統)

[!INCLUDE [deployment models](../../includes/vpn-gateway-classic-deployment-model-include.md)]

> [!div class="op_single_selector"]
> * [Azure 入口網站](vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)
> * [PowerShell (傳統)](vpn-gateway-multi-site.md)
>
>

這篇文章會引導您使用 PowerShell 將站台對站 (S2S) 連線新增至具有現有連線的 VPN 閘道。 這種連線通常稱為「多站台」組態。 本文中的步驟適用於使用傳統部署模型 (也稱為「服務管理」) 來建立的虛擬網路。 這些步驟不適用於 ExpressRoute/站台對站並存連線組態。

### <a name="deployment-models-and-methods"></a>部署模型和方法

[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

當此組態有新文章和額外工具可以使用時，我們就會更新此資料表。 當文章可用時，我們會直接從此資料表連結至該文章。

[!INCLUDE [vpn-gateway-table-multi-site](../../includes/vpn-gateway-table-multisite-include.md)]

## <a name="about-connecting"></a>關於連線

您可以將多個內部部署網站連線到單一虛擬網路。 這對建立混合式雲端解決方案特別具有吸引力。 建立對 Azure 虛擬網路閘道的多站台連線，與建立其他站對站連線類似。 事實上，您可以使用現有的 Azure VPN 閘道，只要該閘道是動態的 (路由式)。

如果已經有連線至虛擬網路的靜態閘道，您可以將閘道類型變更為動態，而不需要為了容納多個網站重建虛擬網路。 在變更路由類型之前，請確定您的內部部署 VPN 閘道支援路由式 VPN 組態。

![多重網站圖表](./media/vpn-gateway-multi-site/multisite.png "多重網站")

## <a name="points-to-consider"></a>考慮事項

**您將無法使用入口網站來對此虛擬網路進行變更。** 您必須對網路組態檔進行變更，而不是使用入口網站。 如果您在入口網站中進行變更，這些變更將會覆寫此虛擬網路的多站台參考設定。

在您完成多站台程序之後，應該就能夠輕鬆使用網路組態檔。 不過，如果有多人在處理您的網路組態，您必須確定每個人都知道這項限制。 這並不表示您完全無法使用入口網站。 除了無法對這個特定虛擬網路進行組態變更，您可以用它來執行所有其他事情。

## <a name="before-you-begin"></a>開始之前

開始設定之前，請確認您具有下列必要條件：

* 每個內部部署位置都有相容的 VPN 硬體。 查看 [關於虛擬網路連線的 VPN 裝置](vpn-gateway-about-vpn-devices.md) ，確認您想要使用的裝置是否為已知相容的項目。
* 每個 VPN 裝置都有對外的公用 IPv4 IP 位址。 此 IP 位址不能位於 NAT 後方。 這是必要條件。
* 您必須安裝最新版的 Azure PowerShell Cmdlet。 請確定除了 Resource Manager 版本之外，也安裝「服務管理」(SM) 版本。 如需詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 。
* 熟悉如何設定 VPN 硬體的人員。 您將必須非常了解如何設定 VPN 裝置，或是與了解的人員一起進行。
* 您想要用於虛擬網路的 IP 位址範圍 (如果尚未建立的話)。
* 您要連線之每個區域網路網站的 IP 位址範圍。 您必須確定您要連線之每個區域網路網站的 IP 位址範圍沒有重疊。 否則，入口網站或 REST API 將會拒絕所要上傳的組態。<br>例如，如果您有兩個區域網路網站都包含 IP 位址範圍 10.2.3.0/24，而您有一個目的地位址為 10.2.3.3 的封裝，Azure 就會不知道您想要將封裝傳送到哪個網站，因為位址範圍重疊了。 為了防止路由問題，Azure 不允許您上傳具有重疊範圍的組態檔。

## <a name="1-create-a-site-to-site-vpn"></a>1.建立站對站 VPN
如已有動態路由閘道的站對站 VPN，太棒了！ 您可以繼續 [匯出虛擬網路組態設定](#export)。 如果沒有，請執行下列動作：

### <a name="if-you-already-have-a-site-to-site-virtual-network-but-it-has-a-static-policy-based-routing-gateway"></a>如果您已經有站對站虛擬網路，但其有靜態 (原則式) 路由閘道：
1. 將您的閘道類型變更為動態路由。 多網站 VPN 需要動態 (亦稱作路由式) 路由閘道。 若要變更閘道類型，您必須先刪除現有的閘道，然後建立新的閘道。
2. 設定新的閘道，並建立 VPN 通道。 如需指示，請參閱[指定 SKU 和 VPN 類型](vpn-gateway-howto-site-to-site-classic-portal.md#sku)。 請務必將路由類型指定為「動態」。

### <a name="if-you-dont-have-a-site-to-site-virtual-network"></a>如果您沒有站對站虛擬網路：
1. 使用下列指示建立站對站虛擬網路：[建立具有站對站 VPN 連線的虛擬網路](vpn-gateway-site-to-site-create.md)。  
2. 使用下列指示設定動態路由閘道： [設定 VPN 閘道](vpn-gateway-configure-vpn-gateway-mp.md)。 閘道類型務必選取 [動態路由]  。

## <a name="export"></a>2.匯出網路組態檔
執行下列命令以匯出 Azure 網路組態檔。 您可以視需要變更此檔案的位置，以匯出至不同的位置。

```powershell
Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
```

## <a name="3-open-the-network-configuration-file"></a>3.開啟網路組態檔
開啟您在最後一個步驟下載的網路組態檔。 使用您喜歡的任何 xml 編輯器。 檔案看起來應該像下面這樣：

        <NetworkConfiguration xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
          <VirtualNetworkConfiguration>
            <LocalNetworkSites>
              <LocalNetworkSite name="Site1">
                <AddressSpace>
                  <AddressPrefix>10.0.0.0/16</AddressPrefix>
                  <AddressPrefix>10.1.0.0/16</AddressPrefix>
                </AddressSpace>
                <VPNGatewayAddress>131.2.3.4</VPNGatewayAddress>
              </LocalNetworkSite>
              <LocalNetworkSite name="Site2">
                <AddressSpace>
                  <AddressPrefix>10.2.0.0/16</AddressPrefix>
                  <AddressPrefix>10.3.0.0/16</AddressPrefix>
                </AddressSpace>
                <VPNGatewayAddress>131.4.5.6</VPNGatewayAddress>
              </LocalNetworkSite>
            </LocalNetworkSites>
            <VirtualNetworkSites>
              <VirtualNetworkSite name="VNet1" AffinityGroup="USWest">
                <AddressSpace>
                  <AddressPrefix>10.20.0.0/16</AddressPrefix>
                  <AddressPrefix>10.21.0.0/16</AddressPrefix>
                </AddressSpace>
                <Subnets>
                  <Subnet name="FE">
                    <AddressPrefix>10.20.0.0/24</AddressPrefix>
                  </Subnet>
                  <Subnet name="BE">
                    <AddressPrefix>10.20.1.0/24</AddressPrefix>
                  </Subnet>
                  <Subnet name="GatewaySubnet">
                    <AddressPrefix>10.20.2.0/29</AddressPrefix>
                  </Subnet>
                </Subnets>
                <Gateway>
                  <ConnectionsToLocalNetwork>
                    <LocalNetworkSiteRef name="Site1">
                      <Connection type="IPsec" />
                    </LocalNetworkSiteRef>
                  </ConnectionsToLocalNetwork>
                </Gateway>
              </VirtualNetworkSite>
            </VirtualNetworkSites>
          </VirtualNetworkConfiguration>
        </NetworkConfiguration>

## <a name="4-add-multiple-site-references"></a>4.新增多個網站參考
當您新增或移除網站參考資訊時，將會對 ConnectionsToLocalNetwork/LocalNetworkSiteRef 進行組態變更。 加入新的本機網站參考資訊會促使 Azure 建立新的通道。 下列範例中的網路組態適用於單一網站連線。 完成變更後請儲存檔案。

```
  <Gateway>
    <ConnectionsToLocalNetwork>
      <LocalNetworkSiteRef name="Site1"><Connection type="IPsec" /></LocalNetworkSiteRef>
    </ConnectionsToLocalNetwork>
  </Gateway>
```

若要新增額外的站台參考 (建立多站台組態)，只需新增額外的 "LocalNetworkSiteRef" 行即可，如以下範例所示：

```
  <Gateway>
    <ConnectionsToLocalNetwork>
      <LocalNetworkSiteRef name="Site1"><Connection type="IPsec" /></LocalNetworkSiteRef>
      <LocalNetworkSiteRef name="Site2"><Connection type="IPsec" /></LocalNetworkSiteRef>
    </ConnectionsToLocalNetwork>
  </Gateway>
```

## <a name="5-import-the-network-configuration-file"></a>5.匯入網路組態檔
匯入網路組態檔。 當您匯入含有變更的這個檔案時，就會加入新的通道。 這些通道會使用您稍早建立的動態閘道。 您可以使用 PowerShell 匯入檔案。

## <a name="6-download-keys"></a>6.下載金鑰
新增您的新通道之後，請使用 PowerShell Cmdlet 'Get-AzureVNetGatewayKey'，以取得每個通道的 IPsec/IKE 預先共用金鑰。

例如︰

```powershell
Get-AzureVNetGatewayKey –VNetName "VNet1" –LocalNetworkSiteName "Site1"
Get-AzureVNetGatewayKey –VNetName "VNet1" –LocalNetworkSiteName "Site2"
```

如果您想要，也可以使用 *取得虛擬網路閘道共用金鑰* REST API 來取得預先共用金鑰。

## <a name="7-verify-your-connections"></a>7.確認您的連線
檢查多網站通道狀態。 下載每個通道的金鑰之後，您會想要確認連線。 使用 'Get-AzureVnetConnection' 來取得虛擬網路通道清單，如以下範例所示。 VNet1 是 VNet 的名稱。

```powershell
Get-AzureVnetConnection -VNetName VNET1
```

範例傳回：

```
    ConnectivityState         : Connected
    EgressBytesTransferred    : 661530
    IngressBytesTransferred   : 519207
    LastConnectionEstablished : 5/2/2014 2:51:40 PM
    LastEventID               : 23401
    LastEventMessage          : The connectivity state for the local network site 'Site1' changed from Not Connected to Connected.
    LastEventTimeStamp        : 5/2/2014 2:51:40 PM
    LocalNetworkSiteName      : Site1
    OperationDescription      : Get-AzureVNetConnection
    OperationId               : 7f68a8e6-51e9-9db4-88c2-16b8067fed7f
    OperationStatus           : Succeeded

    ConnectivityState         : Connected
    EgressBytesTransferred    : 789398
    IngressBytesTransferred   : 143908
    LastConnectionEstablished : 5/2/2014 3:20:40 PM
    LastEventID               : 23401
    LastEventMessage          : The connectivity state for the local network site 'Site2' changed from Not Connected to Connected.
    LastEventTimeStamp        : 5/2/2014 2:51:40 PM
    LocalNetworkSiteName      : Site2
    OperationDescription      : Get-AzureVNetConnection
    OperationId               : 7893b329-51e9-9db4-88c2-16b8067fed7f
    OperationStatus           : Succeeded
```

## <a name="next-steps"></a>後續步驟

若要深入了解 VPN 閘道，請參閱 [關於 VPN 閘道](vpn-gateway-about-vpngateways.md)。
