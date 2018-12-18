---
title: 將多個 VPN 閘道站對站連線新增到 VNet：Azure 入口網站：Resource Manager| Microsoft Docs
description: 將多個 S2S 連線新增至具有現有連線的 VPN 閘道
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: jpconnock
editor: ''
tags: azure-resource-manager
ms.assetid: f3e8b165-f20a-42ab-afbb-bf60974bb4b1
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/14/2018
ms.author: cherylmc
ms.openlocfilehash: 5830b3a4bdcd12c01626d9ff3f814d2e7612eaaa
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29398597"
---
# <a name="add-a-site-to-site-connection-to-a-vnet-with-an-existing-vpn-gateway-connection"></a>將站台對站台連線新增至使用現有 VPN 閘道連線的 VNet

> [!div class="op_single_selector"]
> * [Azure 入口網站](vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)
> * [PowerShell (傳統)](vpn-gateway-multi-site.md)
>
> 

本文可協助您使用 Azure 入口網站，將站對站 (S2S) 連線新增至具有現有連線的 VPN 閘道。 這種連線通常稱為「多站台」組態。 您可以將 S2S 連線新增到已經有 S2S 連線、點對站連線或 VNet 對 VNet 連線的 VNet。 新增連線有一些限制。 開始設定之前，請先閱讀本文的[開始之前](#before)一節。 

本文適用於具有路由型 VPN 閘道的 Resource Manager VNet。 這些步驟不適用於 ExpressRoute/站台對站並存連線組態。 如需有關並存連線的資訊，請參閱 [ExpressRoute/S2S 並存連線](../expressroute/expressroute-howto-coexist-resource-manager.md)。

### <a name="deployment-models-and-methods"></a>部署模型和方法
[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

當此組態有新文章和額外工具可以使用時，我們就會更新此資料表。 當文章可用時，我們會直接從此資料表連結至該文章。

[!INCLUDE [vpn-gateway-table-multi-site](../../includes/vpn-gateway-table-multisite-include.md)]

## <a name="before"></a>開始之前
請確認下列事項：

* 您尚未建立 ExpressRoute/S2S 並存連線。
* 您有使用具有現有連線之 Resource Manager 部署模型建立的虛擬網路。
* 您的 VNet 的虛擬網路閘道是路由型。 如果您的 VPN 閘道為原則型，您就必須刪除虛擬網路閘道，並建立新的路由型 VPN 閘道。
* 此 VNet 連接的所有 VNet 的位址範圍沒有重疊。
* 您有相容的 VPN 裝置以及能夠對其進行設定的人員。 請參閱 [關於 VPN 裝置](vpn-gateway-about-vpn-devices.md)。 如果不熟悉設定 VPN 裝置，或不熟悉位於內部部署網路組態的 IP 位址範圍，則您需要與能夠提供那些詳細資料的人協調。
* 您的 VPN 裝置有對外開放的公用 IP 位址。 此 IP 位址不能位於 NAT 後方。

## <a name="part1"></a>第 1 部分 - 設定連線
1. 透過瀏覽器瀏覽至 [Azure 入口網站](http://portal.azure.com) ，並視需要使用您的 Azure 帳戶登入。
2. 按一下 [所有資源]，在資源清單中找到您的 [虛擬網路閘道]，並按一下它。
3. 在 [虛擬網路閘道] 頁面上，按一下 [連線]。
   
    ![連線頁面](./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/connectionsblade.png "連線頁面")<br>
4. 在 [連線] 頁面上，按一下 [+新增]。
   
    ![新增連線按鈕](./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/addbutton.png "新增連線按鈕")<br>
5. 在 [新增連線] 頁面上，填寫下列欄位︰
   
   * **名稱︰** 為您要建立連線的網站提供名稱。
   * **連線類型：** 選取 [站對站 (IPSec)]。
     
     ![新增連線頁面](./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/addconnectionblade.png "新增連線頁面")<br>

## <a name="part2"></a>第 2 部分 - 新增區域網路閘道
1. 按一下 [區域網路閘道] > [選擇區域網路閘道]。 這會開啟 [選擇區域網路閘道] 頁面。
   
    ![選擇區域網路閘道](./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/chooselng.png "選擇區域網路閘道")<br>
2. 按一下 [建立新的] 以開啟 [建立區域網路閘道] 頁面。
   
    ![建立區域網路閘道頁面](./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/createlngblade.png "建立區域網路閘道")<br>
3. 在 [建立區域網路閘道] 頁面上，填寫下列欄位︰
   
   * **名稱：** 您要給區域網路閘道資源的名稱。
   * **IP 位址：** 您要連線之網站上 VPN 裝置的公用 IP 位址。
   * **位址空間︰** 您要路由傳送至新的區域網路網站的位址空間。
4. 按一下 [建立區域網路閘道] 頁面上的 [確定]，即可儲存變更。

## <a name="part3"></a>第 3 部 - 新增共用金鑰並建立連線
1. 在 [新增連線] 頁面上，新增您想要用來建立連線的共用金鑰。 您可以從您的 VPN 裝置取得共用金鑰，或將此處製作一個，然後設定您的 VPN 裝置使用同一共用金鑰。 重點是金鑰必須完全相同。
   
    ![共用金鑰](./media/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/sharedkey.png "共用金鑰")<br>
2. 按一下頁面底部的 [確定] 以建立連線。

## <a name="part4"></a>第 4 部份︰驗證 VPN 連線


[!INCLUDE [vpn-gateway-verify-connection-ps-rm](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="next-steps"></a>後續步驟

一旦完成您的連接，就可以將虛擬機器加入您的虛擬網路。 如需詳細資訊，請參閱虛擬機器 [學習路徑](https://azure.microsoft.com/documentation/learning-paths/virtual-machines) 。
