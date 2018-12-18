---
title: Azure Front Door Service | Microsoft Docs
description: 本文提供 Azure Front Door 的概觀。 了解它是否為平衡應用程式使用者流量負載的最佳選擇。
services: frontdoor
documentationcenter: ''
author: sharad4u
editor: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/29/2018
ms.author: sharadag
ms.openlocfilehash: 346fa5721df6c9ce0cf355adea21f59c93a394a9
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47040477"
---
# <a name="what-is-azure-front-door-service"></a>Azure 前端服務是什麼？
Azure Front Door Service 可讓您針對最佳效能和立即全域容錯移轉以獲得高可用性最佳化，定義、管理及監視網路流量的全域路由。 透過 Front Door，您可以將全球 (多區域) 消費者和企業應用程式轉換成健全、高效能的個人化新式應用程式、API，以及觸及 Azure 全球對象的內容。

Front Door 會在第 7 層或 HTTP/HTTPS 層上運作，並使用任一傳播通訊協定搭配分割 TCP 和 Microsoft 的全球網路來改善全球連線能力。 因此，根據您在組態中選取的路由方法，您可以確保 Front Door 將用戶端要求路由傳送至最快速且最可用的應用程式後端。 應用程式後端是裝載於 Azure 內部或外部的任何網際網路對向服務。 Front Door 提供[流量路由方法](front-door-routing-methods.md)和[後端健康情況監視選項](front-door-health-probes.md)的範圍，以符合不同的應用程式需求和自動容錯移轉模型。 Front Door 類似於[流量管理員](../traffic-manager/traffic-manager-overview.md)，可針對失敗彈性應變，包括整個 Azure 區域失敗。

>[!NOTE]
> Azure 會為您的案例提供一套完整受控的負載平衡解決方案。 如果您要尋找 DNS 型全域路由，而且**沒有**傳輸層安全性 (TLS) 通訊協定終止 (「SSL 卸載」) 或每一 HTTP/HTTPS 要求的需求，請檢閱[流量管理員](../traffic-manager/traffic-manager-overview.md)。 如果您要尋求區域中伺服器之間的負載平衡，請針對應用程式層檢閱[應用程式閘道](../application-gateway/application-gateway-introduction.md)，並針對網路層負載平衡檢閱[負載平衡器](../load-balancer/load-balancer-overview.md)。 視需要結合這些解決方案，可能有益於您的端對端案例。

Front Door 包含下列功能︰

## <a name="accelerate-application-performance"></a>加快應用程式效能
使用分割 TCP 型任一傳播通訊協定，Front Door 可確保使用者可立即連線到最接近的 Front Door POP (網路連接點)。 使用 Microsoft 的全球網路，從 Front Door POP 連線到您的應用程式後端，可確保較高的可用性和可靠性，同時維持效能。 對您後端的這個連線也是以最低網路延遲為基礎。 深入了解 Front Door 路由技術，像是[分割 TCP](front-door-routing-architecture.md#splittcp) 和[任一傳播通訊協定](front-door-routing-architecture.md#anycast)。

## <a name="increase-application-availability-with-smart-health-probes"></a>利用智慧型健康情況探查提高應用程式可用性

Front Door 使用其智慧型健康情況探查、監視您後端的延遲和可用性，以及在後端關閉時提供即時自動容錯移轉，為您的關鍵應用程式提供高可用性。 因此，您可以在您的應用程式執行規劃的維護作業，而不需要停機。 當維護正在進行時，Front Door 會將流量導向替代後端。

## <a name="url-based-routing"></a>URL 型路由
URL 路徑型路由可讓您根據要求的 URL 路徑，將流量路由傳送至後端集區。 有一個案例是將對於不同內容類型的要求路由傳送至不同的後端集區。

例如，對 `http://www.contoso.com/users/*` 的要求會路由傳送至 UserProfilePool，而對 `http://www.contoso.com/products/*` 的要求則會路由傳送至 ProductInventoryPool。  Front Door 使用最佳比對演算法允許更加複雜的路由比對案例，因此如果沒有任何路徑模式相符，則會針對 `http://www.contoso.com/*` 選取您的預設路由規則，而且流量會導向預設 catch-all 路由規則。 深入了解[路由比對](front-door-route-matching.md)。

## <a name="multiple-site-hosting"></a>多網站裝載
多網站裝載可讓您在相同的 Front Door 組態上設定多個網站。 此功能可讓您將不同的網站新增到單一 Front Door 組態，為您的部署設定更有效率的拓撲。 根據您的應用程式架構，您可以將 Azure Front Door Service 設定為將每個網站導向其自己的後端集區，或者讓各種網站導向相同的後端集區。 例如，Front Door 可以從稱為 ImagePool 和 VideoPool 的兩個後端集區，提供 `images.contoso.com` 和 `videos.contoso.com` 的流量。 或者，您可以設定兩個前端主機，將流量導向稱為 MediaPool 的單一後端集區。

同樣地，您可以在同一個 Front Door 上設定兩個不同的網域 `www.contoso.com` 和 `www.fabrikam.com`。

## <a name="session-affinity"></a>工作階段親和性
當您想要在同一個應用程式後端保留使用者工作階段時，以 Cookie 為基礎的工作階段親和性非常有用。 使用受 Front Door 管理的 Cookie，即可將來自使用者工作階段的後續流量導向至相同應用程式後端進行處理。 當使用者工作階段的工作階段狀態是儲存在後端本機上時，這項功能很重要。

## <a name="secure-sockets-layer-ssl-termination"></a>安全通訊端層 (SSL) 終止
Front Door 在邊緣支援 SSL 終止，也就是個別使用者可以設定與 Front Door 環境的 SSL 連線，而不是透過與應用程式後端的持久連線來建立連線。 此外，Front Door 支援 Front Door 環境與您後端之間的 HTTP 和 HTTPS 連線。 因此，您也可以設定端對端 SSL 加密。 比方說，如果您應用程式工作負載的 Front Door 在一分鐘內收到超過 5000 個要求，但由於重複使用暖連線，所以對作用中服務，它只會與應用程式後端建立大約 500 次連線，因而降低來自後端的大量負載。

## <a name="custom-domains-and-certificate-management"></a>自訂網域和憑證管理
使用 Front Door 來傳遞內容時，如果您想要在 Front Door URL 中顯示您自己的網域名稱，則需要自訂網域。 有可見的網域名稱對您的客戶而言較為方便，並且也有助於宣傳商標。
Front Door 也支援自訂網域名稱使用 HTTPS。 針對您的流量選擇受 Front Door 管理的憑證，或上傳您自己的自訂 SSL 憑證，即可使用此功能。

## <a name="application-layer-security"></a>應用程式層安全性
Azure Front Door 可讓您撰寫自訂 Web 應用程式防火牆 (WAF) 規則進行存取控制，以保護您的 HTTP/HTTPS 工作負載，免於遭受以用戶端 IP 位址、國家/地區代碼及 http 參數為基礎的攻擊。 此外，Front Door 也可讓您建立速率限制規則來對抗惡意 Bot 流量。 

Front Door 本身受到基本 [Azure DDoS 保護](../virtual-network/ddos-protection-overview.md)所保護。 如需進一步保護，可在您的 VNET 啟用標準 Azure DDoS 保護，以透過自動調整和風險降低，保護資源免於遭受網路層 (TCP/UDP) 攻擊。 Front Door 是第 7 層反向 Proxy，只允許 Web 流量傳遞到後端，且預設會封鎖其他類型的流量。

## <a name="url-rewrite"></a>URL 重寫
Front Door 能透過允許您設定選擇性的[自訂轉送路徑](front-door-url-rewrite.md)，以用於建構要轉送至後端之要求的情況，來支援 URL 重寫功能。 Front Door 可進一步讓您將 Host 標頭設定為在將要求轉送到後端時傳送。

## <a name="protocol-support---ipv6-and-http2-traffic"></a>通訊協定支援 - IPv6 和 HTTP/2 流量
Azure Front Door 原本就支援端對端 IPv6 連線能力以及 HTTP/2 通訊協定。 

HTTP/2 通訊協定都可透過長時間執行的 TCP 連線，讓應用程式後端與用戶端之間能進行全雙工通訊。 HTTP/2 可讓後端和用戶端之間進行互動性更高的通訊，此通訊可以是雙向的，而不需要像 HTTP 型實作所要求的進行輪詢。 不同於 HTTP，HTTP/2 通訊協定的負荷很低，而且可以對多個要求或回應重複使用相同的 TCP 連線，進而提升資源使用效率。 深入了解 [Azure Front Door Service 中的 HTTP/2 支援](front-door-http2.md)。

## <a name="pricing"></a>價格

如需定價資訊，請參閱 [Front Door 定價](https://azure.microsoft.com/pricing/details/frontdoor/)。

## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
- 了解 [Front Door 的運作方式](front-door-routing-architecture.md)。
