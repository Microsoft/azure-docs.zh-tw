---
title: 建立路由型 VPN 閘道：Azure 入口網站 | Microsoft Docs
description: 使用 Azure 入口網站快速建立路由型 VPN 閘道
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/04/2018
ms.author: cherylmc
ms.openlocfilehash: 550f655f6eac5a114636978255578eb3753e0d4b
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30918127"
---
# <a name="create-a-route-based-vpn-gateway-using-the-azure-portal"></a>使用 Azure 入口網站建立路由型 VPN 閘道

本文將協助您使用 Azure 入口網站，快速建立路由型 Azure VPN 閘道。  建立與內部部署網路的 VPN 連線時，會使用 VPN 閘道。 您也可以使用 VPN 閘道來連線至 VNet。 

本文中的步驟將會建立 VNet、子網路、閘道子網路，以及路由型 VPN 閘道 (虛擬網路閘道)。 完成閘道建立之後，您接著就可建立連線。 這些步驟需要 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="vnet"></a>建立虛擬網路

1. 透過瀏覽器瀏覽至 [Azure 入口網站](http://portal.azure.com) ，並使用您的 Azure 帳戶登入。
2. 按一下 [建立資源]。 在 [搜尋 Marketplace] 欄位中，輸入「虛擬網路」。 在傳回的清單中找到 [虛擬網路]，並按一下以開啟 [虛擬網路] 頁面。
3. 從接近 [虛擬網路] 頁面底部的 [選取部署模型] 清單中，確認已選取下拉式清單中的 [Resource Manager]，然後按一下 [建立]。 這會開啟 [建立虛擬網路] 頁面。
4. 在 [建立虛擬網路] 頁面上進行 VNet 設定。 當您填寫欄位時，若欄位中輸入的字元有效，紅色驚嘆號就會變成綠色核取記號。 輸入下列值：

  - **名稱**︰TestVNet1
  - **位址空間**：10.1.0.0/16
  - **訂用帳戶**：確認列出的訂用帳戶是您想要使用的訂用帳戶。 您可以使用下拉式清單變更訂用帳戶。
  - **資源群組**︰TestRG1
  - **位置**：美國東部
  - **子網路**：Frontend
  - **位址範圍**：10.1.0.0/24

  ![建立虛擬網路頁面](./media/create-routebased-vpn-gateway-portal/create-virtual-network.png "建立虛擬網路頁面")
5. 輸入值之後，選取 [釘選到儀表板]，讓您能夠輕易地在儀表板上找到您的 VNet，然後按一下 [建立]。 按一下 [建立] 之後，您會看到儀表板上有一個圖格會反映 VNet 的進度。 建立 VNet 時，此圖格會變更。

## <a name="gwsubnet"></a>新增閘道子網路

閘道子網路包含虛擬網路閘道服務所使用的保留 IP 位址。 建立閘道子網路。

1. 在入口網站中，瀏覽至要建立虛擬網路閘道的虛擬網路。
2. 在您的虛擬網路頁面上，按一下 [子網路] 以展開 [VNet1 - 子網路] 頁面。
3. 按一下頂端的 [+閘道子網路] 以開啟 [新增子網路] 頁面。

  ![新增閘道子網路](./media/create-routebased-vpn-gateway-portal/add-gateway-subnet.png "新增閘道子網路")
4. 子網路的**名稱**會自動填入 'GatewaySubnet' 這個必要值。 調整自動填入的**位址範圍**值，以符合下列值：

  **位址範圍 (CIDR 區塊)**：10.1.255.0/27

  ![新增閘道子網路](./media/create-routebased-vpn-gateway-portal/gateway-subnet.png "新增閘道子網路")
5. 若要建立閘道子網路，按一下頁面底部的 [確定]。

## <a name="gwvalues"></a>設定閘道設定

1. 在入口網站頁面的左側，按一下 [+ 建立資源]，並在搜尋方塊中輸入「虛擬網路閘道」，然後按 **Enter**。 在 [結果] 中，找出並按一下 [虛擬網路閘道]。
2. 在 [虛擬網路閘道] 頁面底部，按一下 [建立]，以開啟 [建立虛擬網路閘道] 頁面。
3. 在 [建立虛擬網路閘道] 頁面上，指定您虛擬網路閘道的值。

  - **名稱**：Vnet1GW
  - **閘道類型**：VPN 
  - **VPN 類型**：路由型
  - **SKU**：VpnGw1
  - **位置**：美國東部
  - **虛擬網路**：按一下 [虛擬網路/選擇虛擬網路] 以開啟 [選擇虛擬網路] 頁面。 選取 [VNet1]。

  ![設定閘道設定](./media/create-routebased-vpn-gateway-portal/configure-gateway.png "設定閘道設定")

## <a name="pip"></a>建立公用 IP 位址

VPN 閘道必須具有動態配置的公用 IP 位址。 當您建立與 VPN 閘道的連線時，這就是您內部部署裝置連線的 IP 位址。

1. 選取 [第一個 IP 設定] > [建立閘道 IP 設定]，以要求公用 IP 位址。

  ![第一個 IP 設定](./media/create-routebased-vpn-gateway-portal/add-public-ip-address.png "第一個 IP 設定")
2. 在 [選擇公用 IP 位址] 頁面上，按一下 [+新建] 以開啟 [建立公用 IP 位址] 頁面。
3. 使用下列值進行設定：

  - **名稱**：**VNet1GWIP**
  - **SKU**：**基本**

  ![建立公用 IP](./media/create-routebased-vpn-gateway-portal/public-ip-address-name.png "建立 PIP")
4. 按一下此頁面底部的 [確定] 來儲存變更。

## <a name="creategw"></a>建立 VPN 閘道

1. 確認 [建立虛擬網路閘道] 頁面上的設定。 如有必要，請調整值。

  ![建立 VPN 閘道](./media/create-routebased-vpn-gateway-portal/create-vpn-gateway.png "建立 VPN 閘道")
2. 按一下頁面底部的 [建立]。

按一下 [建立] 之後，即會驗證設定，而 [部署虛擬網路閘道] 圖格會出現在儀表板上。 可能需要 45 分鐘的時間才能建立 VPN 閘道。 您可能需要重新整理入口網站頁面，才能看到完成的狀態。

## <a name="viewgw"></a>檢視 VPN 閘道

1. 建立閘道之後，瀏覽至入口網站中的 VNet1。 VPN 閘道會在 [概觀] 頁面上顯示為連線的裝置。

  ![連線的裝置](./media/create-routebased-vpn-gateway-portal/view-connected-devices.png "連線的裝置")

2. 在裝置清單中，按一下 [VNet1GW] 以檢視詳細資訊。

  ![檢視 VPN 閘道](./media/create-routebased-vpn-gateway-portal/view-gateway.png "檢視 VPN 閘道")

## <a name="next-steps"></a>後續步驟

閘道建立完成之後，您便可在您的虛擬網路與另一個 VNet 之間建立連線。 或是在您的虛擬網路與內部部署位置之間建立連線。

> [!div class="nextstepaction"]
> [建立站對站連線](vpn-gateway-howto-site-to-site-resource-manager-portal.md)<br><br>
> [建立點對站連線](vpn-gateway-howto-point-to-site-resource-manager-portal.md)<br><br>
> [建立與另一個 VNet 的連線](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)