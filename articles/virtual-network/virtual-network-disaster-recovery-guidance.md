---
title: 虛擬網路商務持續性 | Microsoft Docs
description: 了解發生影響 Azure 虛擬網路的 Azure 服務中斷事件時該怎麼辦
services: virtual-network
documentationcenter: ''
author: NarayanAnnamalai
manager: jefco
editor: ''
ms.assetid: ad260ab9-d873-43b3-8896-f9a1db9858a5
ms.service: virtual-network
ms.workload: virtual-network
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/16/2016
ms.author: narayan;aglick
ms.openlocfilehash: d993144006d1fb17d79ffee4f2da538264a309a4
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/09/2018
ms.locfileid: "29854150"
---
# <a name="virtual-network--business-continuity"></a>虛擬網路 – 商務持續性

## <a name="overview"></a>概觀
「虛擬網路」(VNet) 是您網路在雲端的邏輯呈現方式。 它可讓您定義自己的私人 IP 位址空間，並將網路分割成子網路。 VNet 可做為信任界限來裝載您的運算資源，例如「Azure 虛擬機器」和「雲端服務」(Web/背景工作角色)。 VNet 可允許其裝載的資源之間進行直接私人 IP 通訊。 您可以透過「VPN 閘道」或 ExpressRoute 將虛擬網路連結至內部部署網路。

VNet 是在區域的範圍內建立。 您可以在兩個不同的區域 (例如美國東部和美國西部) 中建立具有相同位址空間的 VNet，但無法將它們連線在一起。 

## <a name="business-continuity"></a>商務持續性

可能有數種不同的方式會讓您的應用程式中斷。 區域可能因天然災害或因多個裝置或服務失敗而導致的部分災害，而被完全切斷。 這每一種情況對 VNet 服務的影響各有不同。

**問：如果整個區域發生服務中斷，該怎麼辦？例如，如果區域因為天然災害而完全中斷時會怎樣？裝載於該區域中的虛擬網路會發生什麼情況？**

答︰在服務中斷期間，受影響區域中的虛擬網路和資源都會持續無法被存取。

![簡單的虛擬網路圖表](./media/virtual-network-disaster-recovery-guidance/vnet.png)

**問︰若要在不同的區域中重新建立相同的虛擬網路，該怎麼做？**

答︰虛擬網路是非常輕量型的資源。 您可以叫用 Azure API，在不同的區域中建立具有相同位址空間的 VNet。 若要重新建立受影響區域中所具有的相同環境，您需進行 API 呼叫來重新部署「雲端服務」Web 和背景工作角色，以及您所擁有的虛擬機器。 如果您有內部部署連線 (例如在混合式部署中)，則必須部署新的「VPN 閘道」並連接到內部部署網路。

若要建立虛擬網路，請參閱[建立虛擬網路](manage-virtual-network.md#create-a-virtual-network)。

**問︰是否可以事先在另一個區域中重新建立指定區域中的 VNet 複本？**

答︰是，您可以事先在兩個不同的區域中，使用相同的私人 IP 位址空間和資源來建立兩個 VNet。 如果您在 VNet 中裝載網際網路對應服務，則您可能已設定「流量管理員」將流量異地路由傳送至作用中的區域。 不過，您無法將兩個具有相同位址空間的 VNet 連線到內部部署網路，因為這會造成路由問題。 當發生災害而失去一個區域中的 VNet 時，您可以將位於可用區域中具有相符位址空間的另一個 VNet 連線到您的內部部署網路。

若要建立虛擬網路，請參閱[建立虛擬網路](manage-virtual-network.md#create-a-virtual-network)。

