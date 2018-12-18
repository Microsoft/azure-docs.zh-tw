---
title: 使用 Azure Active Directory 應用程式 Proxy 時的網路拓撲考量 | Microsoft Docs
description: 涵蓋使用 Azure AD 應用程式 Proxy 時的網路拓撲考量。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/28/2017
ms.author: barbkess
ms.reviewer: harshja
ms.custom: it-pro
ms.openlocfilehash: 2321ccf115e3b517bdc593c0c428c61d5dd90968
ms.sourcegitcommit: f86e5d5b6cb5157f7bde6f4308a332bfff73ca0f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39367084"
---
# <a name="network-topology-considerations-when-using-azure-active-directory-application-proxy"></a>使用 Azure Active Directory 應用程式 Proxy 時的網路拓撲考量

本文說明使用 Azure Active Directory 應用程式 Proxy 遠端發佈及存取您的應用程式時的網路拓撲考量。

## <a name="traffic-flow"></a>流量

當應用程式透過 Azure AD App Proxy 發佈時，來自使用者往應用程式的流量會流過三個連線︰

1. 使用者連線至 Azure 上的 Azure AD 應用程式 Proxy 服務公用端點
2. 應用程式 Proxy 服務連線至應用程式 Proxy 連接器
3. 應用程式 Proxy 連接器會連線到目標應用程式

![顯示從使用者至目標應用程式之流量的圖表](./media/application-proxy-network-topology/application-proxy-three-hops.png)

## <a name="tenant-location-and-application-proxy-service"></a>租用戶位置與應用程式 Proxy 服務

當您註冊 Azure AD 租用戶時，您租用戶的區域取決於您所指定的國家/地區。 當您啟用應用程式 Proxy 時，租用戶的應用程式 Proxy 服務執行個體會選取或建立在與 Azure AD 租用戶位於相同區域或最近的區域。

例如，如果 Azure AD 租用戶的區域是歐盟 (EU)，則所有的應用程式 Proxy 連接器會使用歐盟 Azure 資料中心的服務執行個體。 當使用者存取發佈的應用程式時，其流量會通過這個位置的應用程式 Proxy 服務執行個體。

## <a name="considerations-for-reducing-latency"></a>降低延遲的考量

所有 Proxy 解決方案都會在您的網路連線中造成延遲。 不論您選擇哪一個 Proxy 或 VPN 解決方案做為遠端存取解決方案，它一定包含一組伺服器，啟用您公司網路內部的連線。

組織通常包含其周邊網路中的伺服器端點。 不過，使用 Azure AD 應用程式 Proxy，流量都會流過雲端中的 Proxy 服務，而連接器會位於公司的網路上。 不需要周邊網路。

下一節包含的其他建議可協助您更進一步降低延遲。 

### <a name="connector-placement"></a>連接器放置

應用程式 Proxy 會根據您的租用戶位置，為您選擇執行個體的位置。 不過，您可以決定在何處安裝連接器，讓您能夠定義網路流量的延遲特性。

設定應用程式 Proxy 服務時，請詢問下列問題︰

* 應用程式位於何處？
* 大多數存取應用程式的使用者位於何處？
* 應用程式 Proxy 執行個體所在的位置？
* 您是否已設定 Azure 資料中心的專用網路，例如 Azure ExpressRoute 或類似的 VPN？

連接器必須與 Azure 和您的應用程式 (流量圖表中的步驟 2 和 3) 進行通訊，因此連接器的位置會影響這些兩個連線的延遲。 評估連接器的位置時，請記住下列幾點：

* 如果您想要使用 Kerberos 限制委派 (KCD) 進行單一登入，連接器需要資料中心的視野。 此外，連接器伺服器必須加入網域。  
* 有疑問時，請安裝較接近應用程式的連接器。

### <a name="general-approach-to-minimize-latency"></a>將延遲降至最低的常見方式

最佳化每個網路連線，即可將端對端流量的延遲降至最低。 可以將每個連線最佳化，方法為︰

* 減少躍點兩端之間的距離。
* 選擇正確的網路來周遊。 例如，周遊私人網路而非公用網際網路可能會因為專用的連結而更快。

如果在 Azure 與您的公司網路之間有專用的 VPN 或 ExpressRoute 連結，您可加以使用。

## <a name="focus-your-optimization-strategy"></a>專注於最佳化策略

您幾乎沒有辦法可以控制您的使用者與應用程式 Proxy 服務之間的連線。 使用者可以從家用網路、咖啡廳或不同的國家/地區存取您的應用程式。 反之，您可以將應用程式 Proxy 服務與應用程式的應用程式 Proxy 連接器之連線最佳化。 請考慮將下列模式整合在您的環境中。

### <a name="pattern-1-put-the-connector-close-to-the-application"></a>模式 1︰將連接器放在應用程式附近

將連接器放在接近客戶網路中的目標應用程式。 因為連接器和應用程式已關閉，這項設定會將拓撲圖表中的步驟 3 降到最低。 

如果您的連接器需要能夠觀察網域控制站，此模式是有利的。 大部分的客戶會使用此模式，因為它非常適合大部分情節。 還可以將這個模式與模式 2 結合，以將服務與連接器之間的流量最佳化。

### <a name="pattern-2-take-advantage-of-expressroute-with-public-peering"></a>模式 2︰利用 ExpressRoute 與公用對等互連

如果 ExpressRoute 已設定公用對等互連，您可以對應用程式 Proxy 與連接器之間的流量利用更快速的 ExpressRoute 連線。 連接器仍在您的網路上，接近應用程式。

### <a name="pattern-3-take-advantage-of-expressroute-with-private-peering"></a>模式 3︰利用具有私人對等互連的 ExpressRoute

如果有專用的 VPN 或 ExpressRoute 在 Azure 和公司網路之間設定私人對等互連，您會有另一個選項。 在此組態中，Azure 中的虛擬網路通常會被視為公司網路的擴充功能。 因此，您可以在 Azure 資料中心內安裝連接器，並仍能滿足連接器對應用程式連線之低延遲需求。

延遲不會受到危害，因為流量會流經專用的連接。 您的應用程式 Proxy 服務對連接器延遲也會有所改進，因為連接器會安裝在接近 Azure AD 租用戶位置的 Azure 資料中心。

![顯示安裝在 Azure 資料中心內之連接器的圖表](./media/application-proxy-network-topology/application-proxy-expressroute-private.png)

### <a name="other-approaches"></a>其他方法

雖然本文的重點是連接器放置，但您也可以變更應用程式的位置，以取得較佳的延遲特性。

有愈來愈多的組織將其網路移至託管環境。 這可讓它們將自己的應用程式放在託管環境中，這也是其公司網路的一部分，並仍在網域內。 在此情況下，前幾節中所討論的模式可以套用至新的應用程式位置。 如果您正在考慮此選項，請參閱 [AD Domain Services](../../active-directory-domain-services/active-directory-ds-overview.md)。

此外，請考慮使用[連接器群組](application-proxy-connector-groups.md)來組織連接器，以鎖定位於不同位置和網路的應用程式。 

## <a name="common-use-cases"></a>一般使用案例

在本節中，我們將逐步解說幾個常見案例。 假設 Azure AD 租用戶 (以及 Proxy 服務端點) 位於美國 (US)。 這些使用案例中討論的考量也適用於全球各地的其他區域。

這些情況下，我們將每個連線稱為「躍點」，並將它們編號以便於討論︰

- **躍點 1**：使用者至應用程式 Proxy 服務
- **躍點 2**：應用程式 Proxy 服務至應用程式 Proxy 連接器
- **躍點 3**：應用程式 Proxy 連接器至目標應用程式 

### <a name="use-case-1"></a>使用案例 1

**案例︰** 應用程式在美國組織的網路中，其使用者位於相同的區域中。 Azure 資料中心與公司網路之間沒有 ExpressRoute 或 VPN 存在。

**建議︰** 遵循模式 1，如上一節所述。 如需改良的延遲，請視需要考慮使用 ExpressRoute。

這是簡單的模式。 將連接器放在應用程式附近，以最佳化躍點 3。 這是也很自然的選擇，因為連接器通常安裝在直視應用程式及資料中心以執行 KCD 作業。

![圖表顯示使用者、Proxy、連接器及應用程式全都在美國](./media/application-proxy-network-topology/application-proxy-pattern1.png)

### <a name="use-case-2"></a>使用案例 2

**案例︰** 應用程式在美國組織的網路中，其使用者分散於全球各地。 Azure 資料中心與公司網路之間沒有 ExpressRoute 或 VPN 存在。

**建議︰** 遵循模式 1，如上一節所述。 

同樣地，最常見的模式是最佳化躍點 3，您將連接器放在應用程式附近的位置。 躍點 3 通常不是高成本，如果全部都在相同區域內。 不過，躍點 1 可能會根據使用者所在的位置而更高成本，因為世界各地的使用者必須存取美國的應用程式 Proxy 執行個體。 值得注意的是，就散佈於全球的使用者而言，任何 Proxy 解決方案會有相似特性。

![圖表顯示使用者散佈於全球，但 Proxy、連接器及應用程式位於美國](./media/application-proxy-network-topology/application-proxy-pattern2.png)

### <a name="use-case-3"></a>使用案例 3

**案例︰** 應用程式在美國組織的網路中。 Azure 與公司網路之間存在 ExpressRoute 與公用對等互連。

**建議︰** 遵循模式 1 和 2，如上一節所述。

首先，盡可能將連接器放置接近應用程式。 然後，系統會自動使用躍點 2 的 ExpressRoute。 

如果 ExpressRoute 連結使用公用互連，則 Proxy 與連接器之間的流量會流過該連結。 躍點 2 具有最佳化延遲。

![圖表顯示 Proxy 與連接器之間的 ExpressRoute](./media/application-proxy-network-topology/application-proxy-pattern3.png)

### <a name="use-case-4"></a>使用案例 4

**案例︰** 應用程式在美國組織的網路中。 Azure 與公司網路之間存在 ExpressRoute 與私人對等互連。

**建議︰** 遵循模式 3，如上一節所述。

將連接器放置於透過 ExpressRoute 私人對等互連連線到公司網路的 Azure 資料中心。 

連接器可放置於 Azure 資料中心。 因為連接器仍可透過私人網路直視應用程式和資料中心，躍點 3 會維持最佳化。 此外，躍點 2 已進一步最佳化。

![圖表顯示 Azure 資料中心內的連接器，和連接器與應用程式之間的 ExpressRoute](./media/application-proxy-network-topology/application-proxy-pattern4.png)

### <a name="use-case-5"></a>使用案例 5

**案例︰** 用程式在歐盟組織的網路中，而應用程式 Proxy 執行個體與大部分使用者位於美國。

**建議︰** 將連接器放置在應用程式附近。 因為美國使用者會存取剛好在相同區域中的應用程式 Proxy 執行個體，躍點 1 並不過於昂貴。 躍點 3 已最佳化。 請考慮使用 ExpressRoute 將躍點 2 最佳化。 

![圖表顯示使用者和 Proxy 在美國，而連接器與應用程式在歐盟](./media/application-proxy-network-topology/application-proxy-pattern5b.png)

您也可以考慮在此情況下使用另一個變體。 如果組織中的大部分使用者不在美國，則可能您的網路也會延伸至美國。 將連接器放在美國，並使用歐盟國家應用程式的專用內部企業網路。 如此一來，躍點 2 和躍點 3 便已最佳化。

![圖表顯示使用者、Proxy 和連接器在美國，而應用程式在歐盟](./media/application-proxy-network-topology/application-proxy-pattern5c.png)

## <a name="next-steps"></a>後續步驟

- [啟用應用程式 Proxy](application-proxy-enable.md)
- [啟用單一登入](application-proxy-configure-single-sign-on-with-kcd.md)
- [啟用條件式存取](application-proxy-integrate-with-sharepoint-server.md)
- [使用應用程式 Proxy 疑難排解您遇到的問題](application-proxy-troubleshoot.md)
