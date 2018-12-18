---
title: 適用於 ExpressRoute 的 QoS 需求 |Microsoft Docs
description: 此頁面提供用來設定和管理適用於 ExpressRoute 循環之 QoS 的詳細需求。
documentationcenter: na
services: expressroute
author: cherylmc
manager: carmonm
editor: ''
ms.assetid: db1c1447-0283-4a09-907b-ae481adc40c7
ms.service: expressroute
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/25/2017
ms.author: cherylmc
ms.openlocfilehash: c097a9ccba91f59b323215d42d37e6d85e0981ce
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22709907"
---
# <a name="expressroute-qos-requirements"></a>ExpressRoute QoS 需求
商務用 Skype 具有各種工作負載，其所要求的 QoS 處理方式各有差異。 如果您打算透過 ExpressRoute 取用語音服務，應遵守以下所述的需求。

![](./media/expressroute-qos/expressroute-qos.png)

> [!NOTE]
> QoS 需求僅適用於 Microsoft 對等項目。 在 Azure 公用對等和 Azure 私用對等上收到的網路流量的 DSCP 值將會重設為 0。 
> 
> 

下表提供一份商務用 Skype 所使用的 DSCP 標示清單。 如需詳細資訊，請參閱 [管理適用於商務用 Skype 的 QoS](https://technet.microsoft.com/library/gg405409.aspx) 。

| **傳輸類別** | **處理方式 (DSCP 標示)** | **商務用 Skype 的工作負載** |
| --- | --- | --- |
| **語音** |EF (46) |Skype / Lync 語音 |
| **互動式** |AF41 (34) |影片、VBSS |
| AF21 (18) |APP 共用 | |
| **預設值** |AF11 (10) |檔案傳輸 |
| CS0 (0) |任何其他項目 | |

* 您應將工作負載分類，並標記正確的 DSCP 值。 遵循 [這裡](https://technet.microsoft.com/library/gg405409.aspx) 所提供的指引，以了解如何在您的網路中設定 DSCP 標示。
* 您應在網路中設定並支援多個 QoS 佇列。 語音必須是獨立的類別，並可接收 RFC 3246 中指定的 EF 處理方式。 
* 您可以決定適用於每個流量類別的佇列機制、壅塞偵測原則和頻寬配置。 但是必須保留適用於商務用 Skype 工作負載的 DSCP 標示。 如果您使用以上未列出的 DSCP 標示 (例如 AF31 (26))，就必須先將此 DSCP 值重寫為 0，才能將封包傳送給 Microsoft。 Microsoft 只會傳送使用上表所列之 DSCP 值標記的封包。 

## <a name="next-steps"></a>後續步驟
* 請參閱[路由](expressroute-routing.md)和 [NAT](expressroute-nat.md) 的需求。
* 請參閱下列連結，以設定 ExpressRoute 連線。
  
  * [建立 ExpressRoute 線路](expressroute-howto-circuit-classic.md)
  * [設定路由](expressroute-howto-routing-classic.md)
  * [將 VNet 連結到 ExpressRoute 循環](expressroute-howto-linkvnet-classic.md)

