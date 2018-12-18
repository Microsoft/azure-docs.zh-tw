---
title: 將虛擬網路閘道新增到 ExpressRoute 的 VNet：入口網站：Azure | Microsoft Docs
description: 本文將逐步引導您完成將虛擬網路閘道新增到已經建立之 ExpressRoute 的 Resource Manager VNet。
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
ms.date: 04/17/2017
ms.author: cherylmc
ms.openlocfilehash: 2bd0cf8be87937044ad515a2c6f253b1711bb2bf
ms.sourcegitcommit: b5c6197f997aa6858f420302d375896360dd7ceb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/21/2017
ms.locfileid: "23013263"
---
# <a name="configure-a-virtual-network-gateway-for-expressroute-using-the-azure-portal"></a>使用 Azure 入口網站為 ExpressRoute 設定虛擬網路閘道
> [!div class="op_single_selector"]
> * [Resource Manager - Azure 入口網站](expressroute-howto-add-gateway-portal-resource-manager.md)
> * [Resource Manager - PowerShell](expressroute-howto-add-gateway-resource-manager.md)
> * [傳統 - PowerShell](expressroute-howto-add-gateway-classic.md)
> * [影片 - Azure 入口網站](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-vpn-gateway-for-your-virtual-network)
> 
> 

本文將逐步引導您完成為既存的 VNet 新增虛擬網路閘道的步驟。 本文將逐步引導您完成為既存的 VNet 新增虛擬網路 (VNet) 閘道、調整該閘道大小及移除該閘道的步驟。 此組態的步驟是使用 Resource Manager 部署模型來建立的 VNet 所專用，而在 ExpressRoute 組態中也將使用該部署模型。 如需有關 ExpressRoute 之虛擬網路閘道和閘道組態設定的詳細資訊，請參閱[關於 ExpressRoute 的虛擬網路閘道](expressroute-about-virtual-network-gateways.md)。 


## <a name="before-beginning"></a>開始之前

此工作的步驟會根據下列組態參考清單中的值來使用 VNet。 我們會在範例步驟中使用此清單。 您可以複製清單以供參考，並使用您自己的值來取代其中的值。

**組態參考清單**

* 虛擬網路名稱 = "TestVNet"
* 虛擬網路位址空間 = 192.168.0.0/16
* 子網路名稱 = "FrontEnd" 
    * 子網路位址空間 = "192.168.1.0/24"
* 資源群組 = "TestRG"
* 位置 = "美國東部"
* 閘道器子網路名稱："GatewaySubnet"，您必須一律將閘道器子網路命名為 *GatewaySubnet*。
    * 閘道子網路位址空間 = "192.168.200.0/26"
* 閘道名稱 = "ERGW"
* 閘道 IP 名稱 = "MyERGWVIP"
* 閘道類型 = "ExpressRoute"，ExpressRoute 組態需要這個類型。
* 閘道公用 IP 名稱 = "MyERGWVIP"

您可以先檢視這些步驟的[影片](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-vpn-gateway-for-your-virtual-network)，再開始進行設定。

## <a name="create-the-gateway-subnet"></a>建立閘道子網路

1. 在[入口網站](http://portal.azure.com)中，瀏覽至要建立虛擬網路閘道的 Resource Manager 虛擬網路。
2. 在 VNet 刀鋒視窗的 [設定] 中，按一下 [子網路] 以展開 [子網路] 刀鋒視窗。
3. 在 [子網路] 刀鋒視窗中，按一下 [+閘道子網路] 以開啟 [新增子網路] 刀鋒視窗。 
   
    ![新增閘道子網路](./media/expressroute-howto-add-gateway-portal-resource-manager/addgwsubnet.png "新增閘道子網路")


4. 子網路的 [名稱] 會自動填入 'GatewaySubnet' 這個值。 為了讓 Azure 將此子網路視為閘道子網路，需要有這個值。 調整自動填入的 [位址範圍] 值，以符合您的組態需求。 建議您以 /27 或更大的值 (/26、/25 等) 建立閘道子網路。 然後，按一下 [確定] 來儲存值並建立閘道子網路。

    ![新增子網路](./media/expressroute-howto-add-gateway-portal-resource-manager/addsubnetgw.png "新增子網路")

## <a name="create-the-virtual-network-gateway"></a>建立虛擬網路閘道

1. 在入口網站中，按一下左側的 **+** 並在搜尋中輸入「虛擬網路閘道」。 在搜尋傳回的結果中找出**虛擬網路閘道**，然後按一下該項目。 在 [虛擬網路閘道] 刀鋒視窗上，按一下刀鋒視窗底部的 [建立]。 這會開啟 [建立虛擬網路閘道] 刀鋒視窗。
2. 在 [建立虛擬網路閘道] 刀鋒視窗上，填入您虛擬網路閘道的值。

    ![建立虛擬網路閘道刀鋒視窗欄位](./media/expressroute-howto-add-gateway-portal-resource-manager/gw.png "建立虛擬網路閘道刀鋒視窗欄位")
3. **名稱**：為您的閘道命名。 這與為閘道子網路命名不同。 這是您要建立之閘道物件的名稱。
4. **閘道類型**︰選取 [ExpressRoute]。
5. **SKU**︰從下拉式清單中選取閘道 SKU。
6. **位置**：調整 [位置] 欄位以指向您的虛擬網路所在的位置。 如果位置並未指向您的虛擬網路所在的區域，則此虛擬網路不會出現在 [選擇虛擬網路] 下拉式清單中。
7. 選擇您要新增此閘道的虛擬網路。 按一下 [虛擬網路] 以開啟 [選擇擇虛擬網路] 刀鋒視窗。 選取 VNet。 如果您沒看到您的 VNet，請確定 [位置] 欄位是指向您的虛擬網路所在的區域。
9. 選擇公用 IP 位址。 按一下 [公用 IP 位址] 以開啟 [選擇公用 IP 位址] 刀鋒視窗。 按一下 [+新建] 以開啟 [建立公用 IP 位址] 刀鋒視窗。 輸入公用 IP 位址的名稱。 此刀鋒視窗會建立將動態獲派公用 IP 位址的公用 IP 位址物件。 按一下 [確定] 將變更儲存至此刀鋒視窗。
10. **訂用帳戶**：請確認已選取正確的訂用帳戶。
11. **資源群組**：此設定取決於您選取的虛擬網路。
12. 指定上述設定之後，請勿調整 [位置]。
13. 確認設定。 如果您希望閘道顯示在儀表板上，可以選取刀鋒視窗底部的 [釘選到儀表板]。
14. 按一下 [建立]  即可開始建立閘道。 系統會驗證設定並部署閘道。 建立虛擬網路閘道最多可能需要花費 45 分鐘的時間來完成。

## <a name="next-steps"></a>後續步驟
建立 VNet 閘道之後，您可以將 VNet 連結至 ExpressRoute 循環。 請參閱 [將虛擬網路連結到 ExpressRoute 循環](expressroute-howto-linkvnet-portal-resource-manager.md)。