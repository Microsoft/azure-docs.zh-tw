---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 932aab3f16a571d4c83c77c1cc2274ae60ea3d35
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30196880"
---
1. 在入口網站中，從 [所有資源]，按一下 [+新增]。
2. 在 [一切] 頁面搜尋方塊中，輸入**區域網路閘道**，然後按一下以傳回資源清單。 按一下 [區域網路閘道] 以開啟頁面，然後按一下 [建立] 開啟 [建立區域網路閘道] 頁面。

  ![建立區域網路閘道](./media/vpn-gateway-add-lng-rm-portal-include/lng.png)
3. 在 [建立區域網路閘道] 頁面中，指定區域網路閘道的值。

  - **名稱：** 指定區域網路閘道物件的名稱。 可能的話，請使用直覺式名稱，例如 **ClassicVNetLocal** 或 **TestVNet1Local**。 這可讓您更輕鬆地識別入口網站中的區域網路閘道。
  - **IP 位址：** 為您要連線的 VPN 裝置或虛擬網路閘道指定有效的公用 **IP 位址**。

    * **如果此區域網路代表內部部署位置：** 指定您想要連線之 VPN 裝置的公用 IP 位址。 它不能在 NAT 後方且必須可讓 Azure 連線。
    * **如果此區域網路代表另一個 VNet：** 指定已指派給該 Vnet 之虛擬網路閘道的公用 IP 位址。
    * **如果您還沒有 IP 位址︰** 您可以構成有效的預留位置 IP 位址，而後返回並在連線前修改此設定。
  -  是指此區域網路所代表之網路的位址範圍。 您可以新增多個位址空間範圍。 確定您在此指定的範圍，不會與您要連線的其他網路範圍重疊。
  - **設定 BGP 設定：** 僅在設定 BGP 時才使用。 否則，請勿選取此項目。
  - **訂用帳戶：** 確認顯示的是正確的訂用帳戶。
  - **資源群組：** 選取要使用的資源群組。 您可以建立新的資源群組或選取已建立的資源群組。
  - **位置：** 選取將要建立此物件的位置。 建議您選取 VNet 所在的相同位置，但您可以不用這麼做。
4. 按一下 [建立]  來建立區域網路閘道。