---
title: 最佳化 ExpressRoute 路由：Azure | Microsoft Docs
description: 此頁面提供當您有一個以上的 ExpressRoute 線路連接 Microsoft 與您的公司網路時，如何最佳化路由的詳細資訊。
documentationcenter: na
services: expressroute
author: charwen
manager: carmonm
editor: ''
ms.assetid: fca53249-d9c3-4cff-8916-f8749386a4dd
ms.service: expressroute
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/06/2017
ms.author: charwen
ms.openlocfilehash: c3a85b9445d69330c3f6c7d298169efddb6ecca0
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22709992"
---
# <a name="optimize-expressroute-routing"></a>最佳化 ExpressRoute 路由
當您有多個 ExpressRoute 線路時，會有一個以上的路徑來連線到 Microsoft。 因此，可能會產生次佳的路由 - 也就是，您的流量可能會經由較長的路徑連到 Microsoft，而 Microsoft 也可能會經由較長的路徑連到您的網路。 網路路徑愈常，延遲愈久。 延遲對於應用程式效能和使用者體驗有直接的影響。 本文將說明這個問題，並說明如何使用標準路由技術來最佳化路由。

## <a name="suboptimal-routing-from-customer-to-microsoft"></a>從客戶到 Microsoft 的次佳化路由
讓我們依照範例仔細觀察路由問題。 假設您在美國有兩個辦公室，一個在洛杉磯，一個在紐約。 您的辦公室是在廣域網路 (WAN) 上連線，該網路可以是您自己的骨幹網路或服務提供者的 IP VPN。 您有兩個也是在 WAN 上連線的 ExpressRoute 線路，一個在美國西部，一個在美國東部。 很明顯地，您有兩個路徑可連線到 Microsoft 網路。 現在假設您在美國西部和美國東部均有 Azure 部署 (例如 Azure App Service)。 您的用意是要將洛杉磯的使用者連接到 Azure 美國西部以及將紐約的使用者連接到 Azure 美國東部，因為您的服務系統管理員告知每個辦公室的使用者存取附近的 Azure 服務以獲得最佳的體驗。 不幸的是，此計畫比較適合用於東岸的使用者，但不適用於西岸的使用者。 此問題的原因如下所示。 在每個 ExpressRoute 線路上，我們會告知您 Azure 美國東部的前置詞 (23.100.0.0/16)和 Azure 美國西部的前置詞 (13.100.0.0/16)。 如果您不知道哪個前置詞來自哪個區域，您就無法將它視為不同。 您的 WAN 網路可能會認為這兩個前置詞比較接近美國東部 (相較於美國西部)，因此將兩個辦公室的使用者路由至美國東部的 ExpressRoute 線路。 最後，洛杉磯辦公室會有許多使用者不太滿意。

![ExpressRoute 案例 1 問題 - 從客戶到 Microsoft 的次佳化路由](./media/expressroute-optimize-routing/expressroute-case1-problem.png)

### <a name="solution-use-bgp-communities"></a>解決方法︰使用 BGP 社群
若要為這兩個辦公室的使用者最佳化路由，您需要知道哪個前置詞來自 Azure 美國西部以及哪個前置詞來自 Azure 美國東部。 我們使用 [BGP 社群值](expressroute-routing.md)來編碼這項資訊。 我們已將唯一的 BGP 社群值指派給每個 Azure 區域，例如"12076:51004" 適用於美國東部，"12076:51006" 適用於美國西部。 您現在知道哪個前置詞來自哪個 Azure 區域，即可設定應優先使用哪個 ExpressRoute 線路。 因為我們使用 BGP 來交換路由資訊，您可以使用 BGP 的本機喜好設定來影響路由。 在我們的範例中，您可以指派比美國東部還要高的本機喜好設定值給美國西部的 13.100.0.0/16，同樣地，指派比美國西部還要高的本機喜好設定值給美國東部的 23.100.0.0/16。 此組態可確保當Microsoft 的兩個路徑都可用時，洛杉磯的使用者將經由美國西部的 ExpressRoute 線路連接到 Azure 美國西部，而紐約的使用者會經由美國東部的 ExpressRoute 線路連接到 Azure 美國東部。 這兩端的路由均已最佳化。 

![ExpressRoute 案例 1 解決方案 - 使用 BGP 社群](./media/expressroute-optimize-routing/expressroute-case1-solution.png)

> [!NOTE]
> 使用本機喜好設定的相同技巧，也可以套用至從客戶到 Azure 虛擬網路的路由。 我們不會將 BGP 社群值標記到從 Azure 公告到網路的前置詞。 不過，由於您知道哪個虛擬網路部署較接近您的辦公室，因此您可以據以設定路由器，使其偏好某個 ExpressRoute 線路，而非另一個線路。
>
>

## <a name="suboptimal-routing-from-microsoft-to-customer"></a>從 Microsoft 到客戶的次佳化路由
在下面的另一個範例中，來自 Microsoft 的連線經由較長的路徑連到您的網路。 在此情況下，您可在 [混合式環境](https://technet.microsoft.com/library/jj200581%28v=exchg.150%29.aspx)中使用內部部署 Exchange 伺服器和 Exchange Online。 您的辦公室已連線到 WAN。 您告知兩個辦公室中內部部署伺服器的前置詞，以透過兩個 ExpressRoute 線路連到 Microsoft。 在信箱移轉的情況下，Exchange Online 會起始對內部部署伺服器的連線。 不幸的是，在橫越整個大陸回到西岸之前，洛杉磯辦公室的連線會路由至美國東部的 ExpressRoute 線路。 此問題的原因類似第一個問題。 沒有任何提示，Microsoft 網路無法分辨哪個客戶前置詞接近美國東部，哪個接近美國西部。 有可能會對洛杉磯辦公室挑選錯誤的路徑。

![ExpressRoute 案例 2 - 從 Microsoft 到客戶的次佳化路由](./media/expressroute-optimize-routing/expressroute-case2-problem.png)

### <a name="solution-use-as-path-prepending"></a>解決方案︰使用 AS PATH 前置
這個問題有兩個解決方案。 第一個解決方案是您只需告知在美國西部 ExpressRoute 線路上洛杉磯辦公室的內部部署前置詞 (177.2.0.0/31)，以及美國東部 ExpressRoute 線路上紐約辦公室的內部部署前置詞 (177.2.0.2/31)。 如此一來，只有一個路徑可供 Microsoft 連接到每個辦公室。 沒有模稜兩可的狀況，而且路由已最佳化。 利用這個設計，您需要考慮您的容錯移轉策略。 在經由 ExpressRoute 的 Microsoft 路徑已損毀的事件中，您需要確定 Exchange Online 仍可連線至內部部署伺服器。 

第二個解決方案是您繼續告知兩個 ExpressRoute 線路上的兩個前置詞，此外請提供哪個前置詞接近哪個辦公室的提示。 因為我們支援 BGP AS PATH 前置，所以您可以設定前置詞的 AS PATH 來影響路由。 在此範例中，您可以延長美國東部 172.2.0.0/31 的 AS PATH，以致我們偏好將美國西部的 ExpressRoute 線路用於以此前置詞為目的地的流量 (因為我們的網路會認為此前置詞的路徑在西部比較短)。 同樣地，您可以延長美國西部 172.2.0.2/31 的 AS PATH，以致我們偏好美國東部的 ExpressRoute 線路。 這兩個辦公室的路由均已最佳化。 採用這個設計，如果一個 ExpressRoute 路線已中斷，Exchange Online 仍可透過另一個 ExpressRoute 線路和您的 WAN 來觸達您。 

> [!IMPORTANT]
> 我們會針對在 Microsoft 對等上收到的前置詞，移除 AS PATH 中的私用 AS 編號。 您必須在 AS PATH 中附加公用 AS 編號，才能影響 Microsoft 對等的路由。
> 
> 

![ExpressRoute 案例 2 解決方案 - 使用 AS PATH 前置](./media/expressroute-optimize-routing/expressroute-case2-solution.png)

> [!NOTE]
> 雖然此處提供的範例適用於 Microsoft 和公用對等互連，但我們仍支援私人對等互連的相同功能。 此外，AS 路徑前置可在單一 ExpressRoute 路線內運作，以影響主要和次要路徑的選取範圍。
> 
> 

## <a name="suboptimal-routing-between-virtual-networks"></a>虛擬網路之間的次佳化路由
透過 ExpressRoute，您可以啟用虛擬網路對虛擬網路 (也稱為「VNet」) 的通訊，方法是將它們連結至 ExpressRoute 線路。 當您將它們連結至多個 ExpressRoute 線路時，VNet 之間就會發生次佳化路由。 我們來看一個範例。 您有兩個 ExpressRoute 線路，一個在美國西部，一個在美國東部。 而在這兩個區域中，您各有兩個 VNet。 您的 Web 伺服器部署在其中一個 VNet 中，應用程式伺服器則部署在另一個 VNet 中。 為了備援，您將每個區域中的兩個 VNet 同時連結至本機 ExpressRoute 線路和遠端 ExpressRoute 線路。 如下所示，每個 VNet 有兩條路徑可通往另一個 VNet。 VNet 不知道哪一個是本機 ExpressRoute 線路，哪一個是遠端 ExpressRoute 線路。 因此，當它們進行等價多路徑 (Equal-Cost-Multi-Path, ECMP) 路由作業以將 VNet 間的流量負載平衡時，某些流量會經由較長的路徑，在遠端 ExpressRoute 線路進行路由。

![ExpressRoute 案例 3 - 虛擬網路之間的次佳化路由](./media/expressroute-optimize-routing/expressroute-case3-problem.png)

### <a name="solution-assign-a-high-weight-to-local-connection"></a>解決方案︰對本機連線指派高權數
解決方法很簡單。 既然您知道 VNet 與線路在哪，您可以告訴我們每個 VNet 應該偏好使用的路徑。 具體來說，在此範例中，您可以對本機連線指派比遠端連線更高的權數 (請參閱[這裡](expressroute-howto-linkvnet-arm.md#modify-a-virtual-network-connection)的組態範例)。 當 VNet 在多個連線上收到另一個 VNet 的前置詞時，它會偏好使用權數最高的連線，來傳送以該前置詞為目的地的流量。

![ExpressRoute 案例 3 解決方案 - 對本機連線指派高權數](./media/expressroute-optimize-routing/expressroute-case3-solution.png)

> [!NOTE]
> 如果您有多個 ExpressRoute 線路，您也可以影響從 VNet 到內部部署網路的路由，方法是設定連線的權數而非套用 AS PATH 前置，後面這個技巧已在上面的第二個案例中做過說明。 對於每個前置詞，當我們在決定如何傳送流量時，一律會先查看連線權數再看 AS 路徑長度。
>
>
