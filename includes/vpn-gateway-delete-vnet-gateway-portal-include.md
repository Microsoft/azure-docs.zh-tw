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
ms.openlocfilehash: 5b2aa7fedbc203c50796a0e0c8d9cdb3895ae6c1
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30198140"
---
### <a name="step-1-navigate-to-the-virtual-network-gateway"></a>步驟 1：瀏覽至虛擬網路閘道

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 [所有資源]。 
2. 若要開啟虛擬網路閘道頁面，請瀏覽至您想要刪除的虛擬網路閘道，並按一下。

### <a name="step-2-delete-connections"></a>步驟 2︰刪除連線

1. 在虛擬網路閘道的頁面中，按一下 [連線] 以檢視與閘道的所有連線。
2. 按一下連線名稱資料列上的 [...]，然後從下拉式清單選取 [刪除]。
3. 按一下 [是] 確認要刪除連線。 如果您有多個連線，請刪除每個連線。

### <a name="step-3-delete-the-virtual-network-gateway"></a>步驟 3：刪除虛擬網路閘道

請注意，如果除了 S2S 組態，您還擁有此 VNet 適用的 P2S 組態，刪除虛擬網路閘道將自動中斷所有的 P2S 用戶端連線，而不會發出任何警告。

1. 在虛擬網路閘道頁面上，按一下 [概觀]。
2. 在 [概觀] 頁面中，按一下 [刪除] 以刪除閘道。
