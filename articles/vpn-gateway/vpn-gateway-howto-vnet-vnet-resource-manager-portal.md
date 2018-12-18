---
title: 將 Azure 虛擬網路連接至另一個 VNet︰入口網站 | Microsoft Docs
description: 使用 Resource Manager 和 Azure 入口網站建立 VNet 間的 VPN 閘道連接。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jpconnock
editor: ''
tags: azure-resource-manager
ms.assetid: a7015cfc-764b-46a1-bfac-043d30a275df
ms.service: vpn-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/21/2018
ms.author: cherylmc
ms.openlocfilehash: 7e72e9ff9be0b092bc9425828bab46ffd60f0822
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38630262"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-using-the-azure-portal"></a>使用 Azure 入口網站設定 VNet 對 VNet 的 VPN 閘道連線

本文協助您使用 VNet 對 VNet 連線類型來連線虛擬網路。 虛擬網路可位於相同或不同的區域，以及來自相同或不同的訂用帳戶。 連線來自不同訂用帳戶的 VNet 時，訂用帳戶不需與相同的 Active Directory 租用戶相關聯。 

本文中的步驟適用於 Resource Manager 部署模型並使用 Azure 入口網站。 您也可從下列清單中選取不同的選項，以使用不同的部署工具或部署模型來建立此組態：

> [!div class="op_single_selector"]
> * [Azure 入口網站](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Azure 入口網站 (傳統)](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [連線不同的部署模型 - Azure 入口網站](vpn-gateway-connect-different-deployment-models-portal.md)
> * [連線不同的部署模型 - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

![v2v 圖表](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/v2vrmps.png)

## <a name="about"></a>關於連線 VNet

VNet 的連線方法有很多種。 下列各節說明不同的虛擬網路連線方式。

### <a name="vnet-to-vnet"></a>VNet 對 VNet

設定 VNet 對 VNet 連線是輕鬆連線 VNet 的好方法。 使用 VNet 對 VNet 連線類型 (VNet2VNet) 將虛擬網路連線到另一個虛擬網路，類似於建立內部部署位置的站對站 IPsec 連線。 這兩種連線類型都使用 VPN 閘道提供使用 IPsec/IKE 的安全通道，且兩者在通訊時的運作方式相同。 連線類型之間的差異在於區域網路閘道的設定方式。 當您建立 VNet 對 VNet 連線時，會看不到區域網路閘道的位址空間。 系統會自動建立並填入該位址空間。 如果您更新一個 VNet 的位址空間，另一個 VNet 就會自動得知要路由到已更新的位址空間。 相較於在 VNet 之間建立站對站連線，建立 VNet 對 VNet 連線通常比較快速而容易。

### <a name="site-to-site-ipsec"></a>站對站 (IPsec)

如果您使用複雜的網路組態，您可能偏好使用[站對站](vpn-gateway-howto-site-to-site-resource-manager-portal.md)步驟來連線 VNet。 當您使用站對站 IPsec 步驟時，您會以手動方式建立及設定區域網路閘道。 每個 VNet 的區域網路閘道都會將其他 VNet 視為本機網站。 這可讓您指定區域網路閘道的其他位址空間，以便路由傳送流量。 如果 VNet 的位址空間變更，您需要更新對應的區域網路閘道，才會反映變更。 它並不會自動更新。

### <a name="vnet-peering"></a>VNet 對等互連

建議您使用 VNet 對等互連來進行 VNet 連線。 VNet 對等互連不會使用 VPN 閘道，且具有不同的條件約束。 此外，[VNet 對等互連價格](https://azure.microsoft.com/pricing/details/virtual-network)與 [VNet 對 VNet VPN 閘道價格](https://azure.microsoft.com/pricing/details/vpn-gateway)的計算方式不同。 如需詳細資訊，請參閱 [VNet 對等互連](../virtual-network/virtual-network-peering-overview.md)。

## <a name="why"></a>為何要建立 VNet 對 VNet 連線？

基於下列原因，建議您使用 VNet 對 VNet 連線來進行虛擬網路連線：

* **跨區域的異地備援和異地目前狀態**

  * 您可以使用安全連線設定自己的異地複寫或同步處理，而不用查看網際網路對向端點。
  * 您可以使用 Azure 流量管理員和負載平衡器，利用異地備援跨多個 Azure 區域設定高度可用的工作負載。 其中一個重要的範例就是使用分散在多個 Azure 區域的可用性群組來設定 SQL Always On。
* **具有隔離或管理界限的區域性多層式應用程式**

  * 在相同區域中，您可以因為隔離或管理需求，設定將多層式應用程式與多個虛擬網路連線在一起。

您可以將 VNet 對 VNet 通訊與多站台組態結合。 這可讓您建立使用內部虛擬網路連線結合跨單位連線的網路拓撲，如下圖所示：

![關於連接](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/aboutconnections.png "關於連接")

本文協助您使用 VNet 對 VNet 連線類型來連線 VNet。 練習這些步驟時，您可以使用設定值範例。 在範例中，虛擬網路位於相同的訂用帳戶，但在不同的資源群組。 如果您的 Vnet 位於不同的訂用帳戶中，就無法在入口網站中建立連線。 您可以使用 [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md) 或 [CLI](vpn-gateway-howto-vnet-vnet-cli.md)。 如需 VNet 對 VNet 連線的詳細資訊，請參閱本文結尾處的 [VNet 對 VNet 常見問題集](#faq) 。

### <a name="values"></a>設定範例

**TestVNet1 的值︰**

* VNet 名稱︰TestVNet1
* 位址空間︰10.11.0.0/16
* 訂用帳戶：選取您需要使用的訂用帳戶
* 資源群組︰TestRG1
* 位置：美國東部
* 子網路名稱：FrontEnd
* 子網路位址範圍︰10.11.0.0/24
* 閘道子網路名稱︰GatewaySubnet (這會在入口網站中自動填入)
* 閘道子網路位址範圍︰10.11.255.0/27
* DNS 伺服器︰使用您的 DNS 伺服器的 IP 位址
* 虛擬網路閘道名稱︰TestVNet1GW
* 閘道類型：VPN
* VPN 類型：路由式
* SKU︰選取您想要使用的閘道 SKU
* 公用 IP 位址名稱︰TestVNet1GWIP
* 連線名稱︰TestVNet1toTestVNet4
* 共用金鑰︰您可以自行建立共用金鑰。 此範例中，我們將使用 abc123。 重點是當您建立 VNet 之間的連接時，此值必須符合。

**TestVNet4 的值︰**

* VNet 名稱︰TestVNet4
* 位址空間︰10.41.0.0/16
* 訂用帳戶：選取您需要使用的訂用帳戶
* 資源群組：TestRG4
* 位置：美國西部
* 子網路名稱：FrontEnd
* 子網路位址範圍︰10.41.0.0/24
* 閘道子網路名稱︰GatewaySubnet (這會在入口網站中自動填入)
* 閘道子網路位址範圍︰10.41.255.0/27
* DNS 伺服器︰使用您的 DNS 伺服器的 IP 位址
* 虛擬網路閘道名稱︰TestVNet4GW
* 閘道類型：VPN
* VPN 類型：路由式
* SKU︰選取您想要使用的閘道 SKU
* 公用 IP 位址名稱︰TestVNet4GWIP
* 連線名稱︰TestVNet4toTestVNet1
* 共用金鑰︰您可以自行建立共用金鑰。 此範例中，我們將使用 abc123。 重點是當您建立 VNet 之間的連接時，此值必須符合。

## <a name="CreatVNet"></a>1.建立及設定 TestVNet1
如果您已經有 VNet，請驗證設定是否與您的 VPN 閘道設計相容。 請特別注意任何可能與其他網路重疊的子網路。 如果有重疊的子網路，您的連線便無法正常運作。 如果您的 VNet 已設定為正確的設定，即可開始執行 [指定 DNS 伺服器](#dns) 一節中的步驟。

### <a name="to-create-a-virtual-network"></a>建立虛擬網路
[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="subnets"></a>2.新增其他位址空間和建立子網路
建立 VNet 之後，您可以新增其他位址空間和建立子網路。

[!INCLUDE [vpn-gateway-additional-address-space](../../includes/vpn-gateway-additional-address-space-include.md)]

## <a name="gatewaysubnet"></a>3.建立閘道子網路
建立虛擬網路的虛擬網路閘道之前，您必須先建立閘道子網路。 閘道子網路包含虛擬網路閘道所使用的 IP 位址。 可能的話，最好使用 /28 或 /27 的 CIDR 區塊建立閘道子網路，以便提供足以容納未來其他組態需求的 IP 位址。

如果您要練習建立此組態，請在建立閘道子網路時參考這些[範例值](#values) 。

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

### <a name="to-create-a-gateway-subnet"></a>建立閘道子網路
[!INCLUDE [vpn-gateway-add-gwsubnet-rm-portal](../../includes/vpn-gateway-add-gwsubnet-rm-portal-include.md)]

## <a name="dns"></a>4.指定 DNS 伺服器 (選擇性)
VNet 對 VNet 連線不需要 DNS。 不過，如果您想要對部署至虛擬網路的資源進行名稱解析，則應指定 DNS 伺服器。 此設定可讓您指定要用於此虛擬網路之名稱解析的 DNS 伺服器服務。 它不會建立 DNS 伺服器。

[!INCLUDE [vpn-gateway-add-dns-rm-portal](../../includes/vpn-gateway-add-dns-rm-portal-include.md)]

## <a name="VNetGateway"></a>5.建立虛擬網路閘道
此步驟將帶您建立 VNet 的虛擬網路閘道。 建立閘道通常可能需要 45 分鐘或更久，視選取的閘道 SKU 而定。 如果您要練習建立此組態，您可以參考[範例設定](#values)。

### <a name="to-create-a-virtual-network-gateway"></a>建立虛擬網路閘道
[!INCLUDE [vpn-gateway-add-gw-rm-portal](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

## <a name="CreateTestVNet4"></a>6.建立及設定 TestVNet4
設定 TestVNet1 之後，請重複先前步驟來建立 TestVNet4，並換成 TestVNet4 的值。 您不需要等到 TestVNet1 的虛擬網路閘道建立完成後才設定 TestVNet4。 如果您使用自己的值，請確定位址空間沒有與任何您想要連接的 VNet 重疊。

## <a name="TestVNet1Connection"></a>7.設定 TestVNet1 閘道連線
當 TestVNet1 和 TestVNet4 的虛擬網路閘道完成後，您可以建立虛擬網路閘道連接。 在本節中，您要建立從 VNet1 到 VNet4 的連線。 這些步驟只適用於相同的訂用帳戶中的 VNet。 如果您的 VNet 位於不同的訂用帳戶中，則必須使用 PowerShell 來進行連線。 請參閱 [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md) 文件。 不過，如果 Vnet 位於相同訂用帳戶中的不同資源群組，您可以使用入口網站將它們連線。

1. 在 [所有資源] 中，瀏覽至 VNet 的虛擬網路閘道。 例如，**TestVNet1GW**。 按一下 [TestVNet1GW] 以開啟 [虛擬網路閘道] 頁面。

  ![連線頁面](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/1to4connect2.png "連線頁面")
2. 按一下 [+新增] 以開啟 [新增連線] 頁面。

  ![新增連接](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/add.png "新增連接")
3. 在 [新增連線] 頁面的 [名稱] 欄位中，輸入連線名稱。 例如，**TestVNet1toTestVNet4**。
4. 針對 [連線類型]，從下拉式清單選取 [VNet 對 VNet]。
5. 因為您是從指定的虛擬網路閘道建立此連接，將會自動填入 [第一個虛擬網路閘道] 欄位值。
6. [第二個虛擬網路閘道] 欄位是您想要建立連接的 VNet 的虛擬網路閘道。 按一下 [選擇另一個虛擬網路閘道]，以開啟 [選擇虛擬網路閘道] 頁面。
7. 檢視此頁面上列出的虛擬網路閘道。 請注意，只會列出您的訂用帳戶中的虛擬網路閘道。 如果想要連接的虛擬網路閘道不在您的訂用帳戶中，請參閱 [PowerShell 文章](vpn-gateway-vnet-vnet-rm-ps.md)。
8. 按一下您想要連接虛擬網路閘道。
9. 在 [共用金鑰] 欄位中，輸入連接的共用金鑰。 您可以產生此金鑰，或自行建立此金鑰。 在網站間連接中，您用於內部部署裝置與虛擬網路閘道連接的金鑰完全相同。 此處的概念類似，差別在於是連接到另一個虛擬網路閘道，而不是連接到 VPN 裝置。
10. 按一下頁面底部的 [確定] 來儲存變更。

## <a name="TestVNet4Connection"></a>8.設定 TestVNet4 閘道連線
接下來，建立從 TestVNet4 至 TestVNet1 的連接。 在入口網站中，找出與 TestVNet4 相關聯的虛擬網路閘道。 遵循上一節中的步驟，取代建立 TestVNet4 到 TestVNet1 連線的值。 請確定您使用相同的共用金鑰。

## <a name="VerifyConnection"></a>9.確認您的連線

在入口網站中找出虛擬網路閘道。 在虛擬網路閘道頁面中，按一下 [連線]，以檢視虛擬網路閘道的連線頁面。 一旦建立連線後，您會看到 [狀態] 值變更為 [成功] 和 [已連線]。 您可以按兩下連線來開啟 [基本資訊] 頁面，並檢視的詳細資訊。

![已成功](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/connected.png "已成功")

當資料開始流動時，您會看到資料輸入和資料輸出的值。

![程式集](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/essentials.png "程式集")

## <a name="to-add-additional-connections"></a>若要新增其他連線

如果您需要新增其他連線，請瀏覽至您需要建立連線的虛擬網路閘道，然後按一下 [連線]。 您可以建立另一個 VNet 對 VNet 連線，或建立 IPsec 站對站連線到內部部署位置。 請務必調整 [連線類型] 來符合您需要建立的連線類型。 建立其他連線之前，請確認您虛擬網路的位址空間與任何您需要連線的位址空間不重疊。 如需建立站對站連線的步驟，請參閱[建立站對站連線](vpn-gateway-howto-site-to-site-resource-manager-portal.md)。

## <a name="faq"></a>VNet 對 VNet 常見問題集
檢視常見問題集詳細資料以取得 VNet 對 VNet 連線的其他資訊。

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>後續步驟

如需如何在虛擬網路中限制資源之網路流量的資訊，請參閱[網路安全性](../virtual-network/security-overview.md)。

如需 Azure 如何在 Azure、內部部署和網際網路資源間路由流量的資訊，請參閱[虛擬網路流量路由](../virtual-network/virtual-networks-udr-overview.md)。
