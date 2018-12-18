---
title: 關於 Azure ExpressRoute Direct | Microsoft Docs
description: 此頁面提供 ExpressRoute Direct (預覽) 的概觀
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/21/2018
ms.author: cherylmc
ms.openlocfilehash: c33bec76fe17336221c873778c2993d75fec81e8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46962215"
---
# <a name="about-expressroute-direct-preview"></a>關於 ExpressRoute Direct (預覽)

ExpressRoute Direct 為客戶提供能夠將策略性分散於世界各地的對等互連位置，都直接連線至 Microsoft 全球網路的能力。 ExpressRoute Direct 提供雙 100Gbps 連線，且支援大規模主動/主動連線。 

ExpressRoute Direct 提供的主要功能包括但不限於：

* 將資料大量擷取至儲存體與 Cosmos DB 等服務 
* 實體隔離，適用於銀行、政府與零售等受規範且需要專用、隔離之連線的產業 
* 根據業務單位決定的細微線路分散控制

> [!IMPORTANT]
> ExpressRoute Direct 目前為預覽狀態。
>
> 此公開預覽版是在沒有服務等級協定的情況下提供，不得用於生產工作負載。 可能不支援特定功能、可能已經限制功能，或者可能無法在所有 Azure 位置提供使用。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="enroll-in-the-preview"></a>在預覽版本中註冊

您必須先在預覽版中註冊訂用帳戶，才能使用 ExpressRoute Direct。 若要註冊，請將內含訂用帳戶識別碼的電子郵件傳送到 <ExpressRouteDirect@microsoft.com>。 ExpressRoute Direct 是企業級功能。 請提供其他詳細資料：

* 您想要使用 **ExpressRoute Direct** 完成的案例
* 位置喜好設定 - 請參閱[合作夥伴和對等互連位置](expressroute-locations-providers.md)以取得所有位置的完整清單
* 實作的時間軸
* 您對於服務的任何相關問題

## <a name="expressroute-using-a-service-provider-and-expressroute-direct"></a>使用服務提供者的 ExpressRoute 和 ExpressRoute Direct

| **使用服務提供者的 ExpressRoute** | **ExpressRoute Direct** | 
| --- | --- | 
| 使用服務提供者可快速登入並連線至現有的基礎結構 | 需要 100Gbps 的基礎結構，且必須完整管理每一層
| 與數百個提供者整合，包括乙太網路和 MPLS | 法定產業和大規模資料擷取具有直接/專用容量 | 
| 線路 SKU 介於 50Mbps 到 10Gbps | 線路 SKU 介於 1Gbps 到 100Gbps
| 適用於單一租用戶 | 適用於單一租用戶/雲端服務提供者/多個業務單位

## <a name="expressroute-direct-circuits"></a>ExpressRoute Direct 線路

Microsoft Azure ExpressRoute 可讓您透過連線提供者所提供的私人連線，將內部部署網路延伸至 Microsoft 雲端。 透過 ExpressRoute，您可以建立 Microsoft 雲端服務的連線，例如 Microsoft Azure、Office 365 和 Dynamics 365。  

每個對等互連位置皆可存取 Microsoft 的全球網路，且依預設可存取地理政治區域中的任何區域，並且可透過 Premium 線路存取全球各個區域。  

此功能在大部分情況下相當於使用 ExpressRoute 服務提供者運作的線路。 若要進一步支援使用 ExpressRoute Direct 所提供的精細功能和新功能，可利用 ExpressRoute Direct 線路所具備的特定重要功能。

## <a name="circuit-skus"></a>線路 SKU

ExpressRoute Direct 支援將大量資料擷取至 Azure 儲存體與其他巨量資料服務的案例。 ExpressRoute Direct 的 ExpressRoute 線路現在也支援 **40G** 和 **100G** 的線路 SKU。 

## <a name="vlan-tagging"></a>VLAN 標記

ExpressRoute Direct 支援 QinQ 與 Dot1Q VLAN 標記。

* **QinQ VLAN 標記**允許個別 ExpressRoute 線路上有隔離的路由網域。 Azure 會在線路建立時以動態方式配置 S-Tag，且無法變更。 線路上的每個對等互連 (私人和 Microsoft) 都會以唯一的 C-Tag 作為 VLAN。 C-Tag 在 ExpressRoute Direct 連接埠上的線路間不需要是唯一的。 

* **Dot1Q VLAN 標記**允許個別的 ExpressRoute Direct 連接埠配對上可以有單一標記的 VLAN。 對等互連使用的 C-Tag 在所有線路間和 ExpressRoute Direct 連接埠配對的對等互連間都必須是唯一的。

## <a name="workflow"></a>工作流程

![工作流程](./media/expressroute-erdirect-about/workflow1.png)

## <a name="sla"></a>SLA

ExpressRoute Direct 提供與 Microsoft 全球網路的主動/主動備援連線相同的企業級 SLA。 ExpressRoute 基礎結構可作為備援，而 Microsoft 全球網路的連線具有備援性和多樣性，並且可依據客戶的需求進行調整。 在預覽期間並沒有 SLA，且應將預覽版視為僅供非生產工作負載之用。

## <a name="next-steps"></a>後續步驟

[設定 ExpressRoute Direct](expressroute-howto-erdirect.md)