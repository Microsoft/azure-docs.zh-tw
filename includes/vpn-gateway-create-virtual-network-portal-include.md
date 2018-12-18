---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 04/04/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 1d648b6f7727743c6d2f2c6e050778777e44ddf4
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30921375"
---
若要使用 Azure 入口網站在 Resource Manager 部署模型中建立 VNet，請遵循下列步驟。 如果您使用下列步驟作為教學課程，請使用[範例值](#values)。 如果您不執行這些步驟作為教學課程，請務必以自己的值取代這些值。 如需使用虛擬網路的詳細資訊，請參閱 [虛擬網路概觀](../articles/virtual-network/virtual-networks-overview.md)。

>[!NOTE]
>為了讓此 VNet 連線到內部部署位置，您需要與內部部署網路系統管理員協調，以切割出此虛擬網路專用的 IP 位址範圍。 如果 VPN 連線的兩端存在重複的位址範圍，流量就不會如預期的方式進行路由。 此外，如果您想要將此 VNet 連線到另一個 VNet，則位址空間不能與其他 VNet 重疊。 因此，請謹慎規劃您的網路組態。
>
>

1. 透過瀏覽器瀏覽至 [Azure 入口網站](http://portal.azure.com) ，並使用您的 Azure 帳戶登入。
2. 按一下 [建立資源]。 在 [搜尋 Marketplace] 欄位中，輸入「虛擬網路」。 在傳回的清單中找到 [虛擬網路]，並按一下以開啟 [虛擬網路] 頁面。
3. 從接近 [虛擬網路]頁面底部的 [選取部署模型] 清單中，選取 [Resource Manager]，然後按一下 [建立]。 這會開啟 [建立虛擬網路] 頁面。

  ![建立虛擬網路頁面](./media/vpn-gateway-create-virtual-network-portal-include/create-virtual-network.png "建立虛擬網路頁面")
1. 在 [建立虛擬網路] 頁面上進行 VNet 設定。 當您填寫欄位時，若欄位中輸入的字元有效，紅色驚嘆號就會變成綠色核取記號。

  - **名稱**：輸入虛擬網路的名稱。 在此範例中，我們使用 VNet1。
  - **位址空間**：輸入位址空間。 如果您有多個要新增的位址空間，請新增您的第一個位址空間。 稍後，您可以在建立 VNet 之後新增其他位址空間。 確定您指定的位址空間並未與您內部部署位置的位址空間重疊。
  - **訂用帳戶**：確認列出的訂用帳戶是否正確。 您可以使用下拉式清單變更訂用帳戶。
  - **資源群組**：選取現有資源群組，或輸入新資源群組的名稱以建立新的資源群組。 如果您要建立新的群組，請根據您計劃的組態值來命名資源群組。 如需資源群組的詳細資訊，請造訪 [Azure 資源管理員概觀](../articles/azure-resource-manager/resource-group-overview.md#resource-groups)。
  - **位置**：選取 VNet 的位置。 此位置會決定您部署到此 VNet 之資源所在的位置。
  - **子網路**：新增第一個子網路名稱和子網路位址範圍。 稍後，您可以在建立此 VNet 之後，新增其他子網路和網路閘道。 

5. 如果想要能夠在儀表板上輕鬆地尋找您的 VNet，請選取 [釘選到儀表板]，然後按一下 [建立]。 按一下 [建立] 之後，您會看到儀表板上有一個圖格會反映 VNet 的進度。 建立 VNet 時，此圖格會變更。