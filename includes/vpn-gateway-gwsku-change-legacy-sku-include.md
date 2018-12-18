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
ms.openlocfilehash: 2c1a4a1931bc2e38b0bee5f90518b01fdf4767a1
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "30196762"
---
如果您使用的是資源管理員部署模型，則可以變更為新的閘道 SKU。 從舊版閘道 SKU 變更到新的 SKU 時，您會刪除現有的 VPN 閘道，並建立新的 VPN 閘道。

工作流程：

1. 移除虛擬網路閘道的任何連線。
2. 刪除舊的 VPN 閘道。
3. 建立新的 VPN 閘道。
4. 以新的 VPN 閘道 IP 位址更新內部部署 VPN 裝置 (適用於站對站連線)。
5. 針對將連線到此閘道的任何 VNet 對 VNet 區域網路閘道，更新其閘道 IP 位址值。
6. 下載新的用戶端 VPN 組態套件，以便 P2S 用戶端透過此 VPN 閘道連線至虛擬網路。
7. 重新建立虛擬網路閘道的連線。

考量：

* 若要移至新的 SKU，您的 VPN 閘道必須在資源管理員部署模型中。
* 如果您有傳統的 VPN 閘道，您必須繼續使用該閘道較舊的舊版 SKU；不過，您可以調整舊版 SKU 間的大小。 您無法變更為新的 SKU。
* 從舊版的 SKU 變更到新的 SKU 時，會有一段時間連線中斷。
* 變更為新的閘道 SKU 時，您 VPN 閘道的公用 IP 位址也會變更。 即使您指定了之前用過的相同公用 IP 位址物件，也會發生這種情況。