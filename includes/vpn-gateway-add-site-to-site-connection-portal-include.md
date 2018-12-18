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
ms.openlocfilehash: 0dcb591db5f487a103feccf92ffa577a1cbf8b23
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30921371"
---
1. 瀏覽至虛擬網路閘道的頁面並加以開啟。 瀏覽的方法有好幾種。 您可以藉由移至 [TestVNet1] -> [概觀]- > [已連線的裝置] -> [VNet1GW]，瀏覽至 'VNet1GW' 閘道。
2. 在 VNet1GW 的頁面上，按一下 [連線]。 在 [連線] 頁面的頂部，按一下 [+新增] 以開啟 [新增連線] 頁面。

  ![建立站對站連線](./media/vpn-gateway-add-site-to-site-connection-portal-include/configure-site-to-site-connection.png)
3. 在 [新增連線] 頁面上，設定您連線的值。

  - **名稱：** 為您的連線命名。
  - **連線類型：** 選取 [站對站 (IPSec)]。
  - **虛擬網路閘道：** 因為您正從此閘道連線，所以值是固定的。
  - **區域網路閘道：** 按一下 [選擇區域網路閘道] 並選取要使用的區域網路閘道。
  - **共用金鑰：** 這裡的值必須與您用於本機內部部署 VPN 裝置的值相符。 範例中使用的是 'abc123'，但是您可以(且應該) 使用更為複雜的值。 重要的是，您在此指定的值必須與您在設定 VPN 裝置時指定的值相同。
  - **訂用帳戶**、**資源群組**和**位置**的其餘值是固定的。

4. 按一下 [確定]  來建立連線。 您會看到畫面上閃爍「正在建立連線」  。
5. 您可以在虛擬網路閘道的 [連線] 頁面中檢視連線。 狀態將會從 [未知] 變成 [連線中]，然後變成 [成功]。