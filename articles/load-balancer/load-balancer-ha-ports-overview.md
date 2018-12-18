---
title: Azure 中的高可用性連接埠概觀 | Microsoft Docs
description: 深入了解內部負載平衡器上的高可用性連接埠負載平衡。
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 46b152c5-6a27-4bfc-bea3-05de9ce06a57
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/07/2018
ms.author: kumud
ms.openlocfilehash: 716a3dafe08e896924bd28e44d69141e4c229687
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39005498"
---
# <a name="high-availability-ports-overview"></a>高可用性連接埠概觀

Azure Standard Load Balancer 可協助您在使用內部負載平衡器時，同時對所有連接埠上的 TCP 和 UDP 流量進行負載平衡。 

高可用性 (HA) 連接埠規則是內部 Standard Load Balancer 上所設定負載平衡規則的變化。 您可以透過提供單一規則來對抵達內部 Standard Load Balancer 所有連接埠的所有 TCP 和 UDP 流量進行負載平衡，以簡化對於負載平衡器的使用。 每次都會針對流量進行負載平衡決策。 此動作是以下列五個 Tuple 連線為基礎：來源 IP 位址、來源連接埠、目的地 IP 位址、目的地連接埠和通訊協定。

HA 連接埠功能可協助您處理重要案例，例如虛擬網路內網路虛擬設備 (NVA) 的高可用性和規模調整。 此功能也可以在必須對大量連接埠進行負載平衡時提供協助。 

當您將前端和後端連接埠設定為 **0**，並將通訊協定設定為 [全部] 時，會設定 HA 連接埠功能。 接著，內部負載平衡器資源會對所有 TCP 和 UDP 流量進行平衡，不論連接埠號碼為何。

## <a name="why-use-ha-ports"></a>為何要使用 HA 連接埠？

### <a name="nva"></a>網路虛擬設備

您可以使用 NVA 保護 Azure 工作負載，使其不會遭受多個類型的安全性威脅。 當您在這些案例中使用 NVA 時，它們必須具有可靠性、高可用性，並可依需要相應放大。

若要達成這些目標，您只需要將 NVA 執行個體新增至內部負載平衡器的後端集區，並設定 HA 連接埠負載平衡器規則即可。

針對 NVA HA 案例，HA 連接埠可提供以下優點：
- 透過執行個體健康情況探查，快速容錯移轉至狀況良好的執行個體
- 透過相應放大為 *n*-active 執行個體，確保較高效能
- 提供 *n*-active 和主動-被動案例
- 不需要使用複雜的解決方案 (例如，Apache ZooKeeper 節點) 來監視設備

下圖顯示中樞和支點虛擬網路部署。 支點會強制將其通道導向中樞虛擬網路並且透過 NVA，然後再退出信任的空間。 在 HA 連接埠設定中，NVA 位於內部 Standard Load Balancer 後面。 所有流量皆可據以處理並轉送。

![中樞和支點虛擬網路 (具有以 HA 模式部署的 NVA)](./media/load-balancer-ha-ports-overview/nvaha.png)

>[!NOTE]
> 如果您要使用 NVA，請與其提供者確認如何以最佳方式使用 HA 連接埠，並了解所支援的案例有哪些。

### <a name="load-balancing-large-numbers-of-ports"></a>對大量連接埠進行負載平衡

您也可以對需要負載平衡大量連接埠的應用程式使用 HA 連接埠。 您可以藉由使用具有 HA 連接埠的內部 [Standard Load Balancer](load-balancer-standard-overview.md)，來簡化這些案例。 單一負載平衡規則會取代多個個別的負載平衡規則，每個連接埠一個規則。

## <a name="region-availability"></a>區域可用性

HA 連接埠功能適用於所有全域 Azure 區域。

## <a name="supported-configurations"></a>支援的設定

### <a name="a-single-non-floating-ip-non-direct-server-return-ha-ports-configuration-on-an-internal-standard-load-balancer"></a>內部 Standard Load Balancer 上的單一非浮動 IP (非伺服器直接回傳) HA 連接埠設定

此設定是基本的 HA 連接埠設定。 若要針對單一前端 IP 位址設定 HA 連接埠負載平衡規則，請執行下列步驟：
1. 設定 Standard Load Balancer 時，在 Load Balancer 規則設定中選取 [HA 連接埠] 核取方塊。
2. 針對 [浮動 IP]，請選取 [已停用]。

此設定在目前的負載平衡器資源上，不允許其他任何負載平衡規則設定。 對於一組指定的後端執行個體，它也不允許其他任何內部負載平衡器資源設定。

不過，除了這個 HA 連接埠規則之外，您還能為後端執行個體設定公用 Standard Load Balancer。

### <a name="a-single-floating-ip-direct-server-return-ha-ports-configuration-on-an-internal-standard-load-balancer"></a>內部 Standard Load Balancer 上的單一浮動 IP (伺服器直接回傳) HA 連接埠設定

您同樣可以將負載平衡器設定為搭配含單一前端的 **HA 連接埠**使用負載平衡規則，方法是將 [浮動 IP] 設定為 [已啟用]。 

您可以使用此設定新增更多浮動 IP 負載平衡規則和/或一個公用負載平衡器。 不過，您無法在此設定上使用非浮動 IP、HA 連接埠負載平衡設定。

### <a name="multiple-ha-ports-configurations-on-an-internal-standard-load-balancer"></a>內部 Standard Load Balancer 上的多個 HA 連接埠設定

如果您的案例要求您針對相同的後端集區設定多個 HA 連接埠前端，則可以執行下列動作： 
- 針對單一內部 Standard Load Balancer 資源，設定多個前端私人 IP 位址。
- 設定多個負載平衡規則，其中每個規則都已選取單一且唯一的前端 IP 位址。
- 選取 [HA 連接埠] 選項，然後針對所有負載平衡規則，將 [浮動 IP] 設定為 [已啟用]。

### <a name="an-internal-load-balancer-with-ha-ports-and-a-public-load-balancer-on-the-same-back-end-instance"></a>在相同後端執行個體上具有 HA 連接埠與公用負載平衡器的內部負載平衡器

您可以針對後端資源設定「一個」公用 Standard Load Balancer 資源，以及含 HA 連接埠的單一內部 Standard Load Balancer。

>[!NOTE]
>此功能目前可透過 Azure Resource Manager 範本取得，但無法透過 Azure 入口網站取得。

## <a name="limitations"></a>限制

- HA 連接埠設定僅適用於內部負載平衡器。 它不適用於公用負載平衡器。

- 不支援 HA 連接埠負載平衡規則與非 HA 連接埠負載平衡規則的組合。

- HA 連接埠功能不適用於 IPv6。

- 只有單一 NIC 支援 NVA 案例的流程對稱。 請參閱[網路虛擬設備](#nva)的描述和圖表。 不過，如果目的地 NAT 適用於您的案例，您就可以使用它來確定內部負載平衡器會將傳回流量傳送到相同的 NVA。


## <a name="next-steps"></a>後續步驟

- [在內部 Standard Load Balancer 上設定 HA 連接埠](load-balancer-configure-ha-ports.md)
- [了解 Standard Load Balancer](load-balancer-standard-overview.md)
