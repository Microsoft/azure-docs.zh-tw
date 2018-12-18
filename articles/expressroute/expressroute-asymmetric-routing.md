---
title: 非對稱式路由 | Microsoft Docs
description: 本文逐步解說在有多個連結連至一個目的地的網路中，客戶可能會面對的非對稱式路由問題。
documentationcenter: na
services: expressroute
author: osamazia
manager: carmonm
editor: ''
ms.assetid: a754bff9-95c9-44b5-9796-377fc21e8322
ms.service: expressroute
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/10/2016
ms.author: osamam
ms.openlocfilehash: 8568c13d2834a0643e15ab1814a35c92123837d1
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22710009"
---
# <a name="asymmetric-routing-with-multiple-network-paths"></a>非對稱式路由與多個網路路徑
本文說明如何在網路來源與目的地之間有多個路徑時，向前和回傳網路流量如何採取不同的路由。

一定要先了解兩個概念，才能了解非對稱式路由。 一個是多個網路路徑的效果。 另一個概念是裝置 (如防火牆) 如何保持狀態。 這些裝置類型稱為具狀態裝置。 這兩個因素的組合可產生網路流量會隨著具狀態裝置下降的案例，因為具狀態裝置未偵測到透過裝置本身所產生的流量。

## <a name="multiple-network-paths"></a>多個網路路徑
當企業網路只有一個透過其網際網路服務提供者連至網際網路的連結時，則往返網際網路的所有流量都會經過相同的路徑。 公司通常會購買多個線路，做為備援路徑，以改善網路運作時間。 發生這種情況時，離開網路而朝向網際網路的流量可能會通過一個連結，而回傳流量則是通過不同的連結。 這通常稱為非對稱式路由。 在非對稱式路由中，反向網路流量會採取與原始流程不同的路徑。

![具有多個路徑的網路](./media/expressroute-asymmetric-routing/AsymmetricRouting3.png)

雖然這主要發生在網際網路上，非對稱式路由也適用於其他多個路徑的組合。 舉例來說，它適用於前往相同目的地的網際網路路徑和私用路徑，以及前往相同目的地的多個私用路徑。

從來源一路到目的地的每個路由器都會計算到達目的地的最佳路徑。 路由器會根據兩個主要因素決定可能最佳的路徑：

* 外部網路之間的路由是以「邊界閘道通訊協定」(BGP) 路由通訊協定為基礎。 BGP 會接受芳鄰的通告並經由一連串的步驟來執行這些通告，以決定到達預定目的地的最佳路徑。 它會在其路由表中儲存最佳路徑。
* 與路由相關聯的子網路遮罩的長度會影響路由路徑。 如果路由器收到相同 IP 位址，但不同子網路遮罩的多個通告，則路由器偏好採用具有較長子網路遮罩的通告，因為它被視為比較明確的路由。

## <a name="stateful-devices"></a>具狀態裝置
路由器會為了進行路由傳送而查看封包的 IP 標頭。 有些裝置看起來在封包的更深處。 這些裝置通常會查看 Layer4 (傳輸控制通訊協定 (TCP) 或使用者資料標通訊協定 (UDP))，或甚至是 Layer7 (應用程式層) 標頭。 這幾種裝置不是安全性裝置，就是頻寬最佳化裝置。 

防火牆是具狀態裝置的常見範例。 防火牆會根據各種欄位 (例如通訊協定、TCP/UDP 連接埠、URL 標頭)，允許或拒絕封包通過其介面。 這個層級的封包審查會對裝置造成沈重的處理負載。 為了改善效能，防火牆會檢查流程的第一個封包。 如果允許封包繼續進行，它會將流程資訊保留在其狀態資料表中。 根據最初的判斷會允許與此流程相關的所有後續封包。 屬於現有流程一部分的封包可到達防火牆。 如果防火牆沒有其先前的狀態資訊，則防火牆會捨棄封包。

## <a name="asymmetric-routing-with-expressroute"></a>非對稱式路由與 ExpressRoute
當您透過 Azure ExpressRoute 連接到 Microsoft 時，您的網路會發生下列變更：

* 您有 Microsoft 的多個連結。 一個連結是您現有的網際網路連線，而另一個則是透過 ExpressRoute。 Microsoft 一些流量可能會通過網際網路，但透過 ExpressRoute 返回，反之亦然。
* 您會透過 ExpressRoute 收到更多明確的 IP 位址。 因此，對於透過 ExpressRoute 提供的服務而言，路由器一律偏好對從您的網路至 Microsoft 的流量使用 ExpressRoute。

若要了解這兩項變更對網路的影像，讓我們來考量一些案例。 舉例來說，您只有一個連到網際網路的線路，而且您透過網際網路使用所有的 Microsoft 服務。 從您的網路至 Microsoft 的來回流量會周遊相同的網際網路連結並通過防火牆。 防火牆會在看到第一個封包時記錄流程，並因為此流程存在於狀態資料表中而允許傳回封包。

![非對稱式路由與 ExpressRoute](./media/expressroute-asymmetric-routing/AsymmetricRouting1.png)

然後，您會開啟 ExpressRoute，並透過 ExpressRoute 使用 Microsoft 所提供的服務。 Microsoft 提供的所有其他服務都是透過網際網路使用。 您會在連接到 ExpressRoute 的邊緣部署不同的防火牆。 Microsoft 會透過 ExpressRoute，針對特定服務，向您的網路通告更多特定的前置詞。 您的路由基礎結構會選擇 ExpressRoute 做為這些前置詞的慣用路徑。 如果您不是透過 ExpressRoute 向 Microsoft 通告您的公用 IP 位址，Microsoft 會透過網際網路與您的公用 IP 位址通訊。 從您的網路至 Microsoft 的向前流量會使用 ExpressRoute，而來自 Microsoft 的反向流量則會使用網際網路。 當邊緣防火牆看到對於在狀態資料表中找不到的流程的回應封包時，它會捨棄回傳流量。

如果您選擇將相同的網路位址轉譯 (NAT) 集區使用於 ExpressRoute 和網際網路，您會發現您的網路中私人 IP 位址上的用戶端有類似的問題。 Windows Update 等服務的要求會透過網際網路輸送，因為這些服務的 IP 位址不會透過 ExpressRoute 通告。 不過，回傳流量會透過 ExpressRoute 返回。 如果 Microsoft 從網際網路和 ExpressRoute 收到具有相同子網路遮罩的 IP 位址，則會偏好透過網際網路的 ExpressRoute。 如果在您的網路邊緣且面對 ExpressRoute 的防火牆或其他具狀態裝置，沒有任何關於流程的先前資訊，它會捨棄屬於該流程的封包。

## <a name="asymmetric-routing-solutions"></a>非對稱式路由解決方案
主要有兩個選項可解決非對稱式路由問題。 一個是透過路由，而另一個則是使用以來源為基礎 NAT (SNAT)。

### <a name="routing"></a>路由
確定已向適當的廣域網路 (WAN) 連結通告您的公用 IP 位址。 例如，如果您想要將網際網路使用於驗證流量，而將 ExpressRoute 使用於郵件流量，就不應該透過 ExpressRoute 通告 Active Directory 同盟服務 (AD FS) 公用 IP 位址。 同樣地，確保不會對路由器透過 ExpressRoute 接收的 IP 位址公開內部部署 AD FS 伺服器。 透過 ExpressRoute 收到的路由更加明確，所以會讓 ExpressRoute 成為 Microsoft 驗證流量的慣用路徑。 這會導致非對稱式路由。

如果您想要使用 ExpressRoute 進行驗證，請確定您是透過 ExpressRoute (未經 NAT) 通告 ADFS 公用 IP 位址。 如此一來，源自 Microsoft 並前往內部部署 AD FS 伺服器的流量會經過 ExpressRoute 。 從客戶到 Microsoft 的回傳流量會使用 ExpressRoute，因為這是透過網際網路慣用的路由。

### <a name="source-based-nat"></a>以來源為基礎的 NAT
解決非對稱式路由問題的另一種方式是使用 NAT。 例如，您尚未透過 ExpressRoute 通告內部部署簡易郵件傳送通訊協定 (SMTP) 伺服器的公用 IP 位址，因為您傾向於使用網際網路進行這類型的通訊。 由 Microsoft 向您的內部部署 SMTP 伺服器提出的要求會周遊網際網路。 您會對內部 IP 位址的連入要求進行 NAT 處理。 來自 SMTP 伺服器的反向流量會前往邊緣防火牆 (您用於進行 NAT)，而不是透過 ExpressRoute。 回傳流量會透過網際網路返回。

![以來源為基礎的 NAT 網路組態](./media/expressroute-asymmetric-routing/AsymmetricRouting2.png)

## <a name="asymmetric-routing-detection"></a>非對稱式路由偵測
路徑追蹤是確保網路流量會周遊預期路徑的最佳方式。 如果您預期從內部部署 SMTP 伺服器至 Microsoft 的流量會採用網際網路路徑，則預期的路徑追蹤是從 SMTP 伺服器至 Office 365。 結果會驗證流量確實離開您的網路朝向網際網路，而不是朝向 ExpressRoute。

