---
title: Azure 流量管理員 | Microsoft Docs
description: 本文提供 Azure 流量管理員的概觀。 了解它是否為平衡應用程式使用者流量負載的最佳選擇。
services: traffic-manager
documentationcenter: ''
author: kumudd
manager: jeconnoc
editor: ''
ms.service: traffic-manager
customer intent: As an IT admin, I want to learn about Traffic Manager and what I can use it for.
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/25/2018
ms.author: kumud
ms.openlocfilehash: 236137b87351e3c3a95c1103f7464256f41b9159
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39397210"
---
# <a name="what-is-traffic-manager"></a>什麼是流量管理員？
Azure 流量管理員是 DNS 型流量負載平衡器，可讓您跨全球的 Azure 區域將流量最佳分散至服務，同時提供高可用性和回應性。

流量管理員會使用 DNS，根據流量路由方法和端點的健康情況，將用戶端要求導向最適當的服務端點。 端點是裝載於 Azure 內部或外部的任何網際網路對向服務。 流量管理員提供[流量路由方法](traffic-manager-routing-methods.md)和[端點監視選項](traffic-manager-monitoring.md)的範圍，以符合不同的應用程式需求和自動容錯移轉模型。 流量管理員可針對失敗彈性應變，包括整個 Azure 區域失敗。

>[!NOTE]
> Azure 會為您的案例提供一套完整受控的負載平衡解決方案。 如果您要尋找傳輸層安全性 (TLS) 通訊協定終止 (「SSL 卸載」) 或每一 HTTP/HTTPS 要求的應用程式層處理，請檢閱[應用程式閘道](../application-gateway/application-gateway-introduction.md)。 如果您要尋求全域平衡，請檢閱[負載平衡器](../load-balancer/load-balancer-overview.md)。 視需要結合這些解決方案，可能有益於您的端對端案例。

流量管理員包含下列功能︰

## <a name="increase-application-availability"></a>提高應用程式可用性

流量管理員藉由監視端點，以及在端點停止運作時提供自動容錯移轉功能，為重要的應用程式提供高可用性。
    
## <a name="improve-application-performance"></a>改善應用程式效能

Azure 可讓您在世界各地的資料中心內執行雲端服務或網站。 流量管理員藉由將流量導向用戶端網路延遲最低的端點，改善應用程式的回應性。

## <a name="perform-service-maintenance-without-downtime"></a>在不需要停機的情況下執行服務維護

您可以在您的應用程式執行規劃的維護作業，而不需要停機。 當維護正在進行時，流量管理員會將流量導向替代的端點。

## <a name="combine-hybrid-applications"></a>合併混合式應用程式

流量管理員支援外部、非 Azure 端點，並可搭配混合式雲端和內部部署使用，包括「[高載至雲端](https://azure.microsoft.com/overview/what-is-cloud-bursting/)」、「移轉至雲端」和「容錯移轉至雲端」案例。

## <a name="distribute-traffic-for-complex-deployments"></a>複雜部署的流量分配

利用[巢狀的流量管理員設定檔](traffic-manager-nested-profiles.md)，可以組合流量路由方法，建立複雜且彈性的規則，以符合更大型且更複雜部署的需求。

## <a name="pricing"></a>價格

如需定價資訊，請參閱[流量管理員定價](https://azure.microsoft.com/pricing/details/traffic-manager/)。


## <a name="next-steps"></a>後續步驟

- 了解如何 [建立流量管理員設定檔](traffic-manager-create-profile.md)。
- 了解[流量管理員的運作方式](traffic-manager-how-it-works.md)。
- 檢視關於流量管理員的[常見問題集](traffic-manager-FAQs.md)。




