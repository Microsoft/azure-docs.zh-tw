---
title: Azure 網路效能監控的定價常見問題集 | Microsoft Docs
description: 常見問題集 - Azure 網路效能監控
ms.subservice: logs
ms.topic: conceptual
author: agummadi
ms.author: agummadi
ms.date: 04/02/2018
ms.openlocfilehash: f44afd84c58c94c6a8d3e6145e8a4f66e0e2e782
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "86539647"
---
# <a name="pricing-changes-for-azure-network-performance-monitor"></a>Azure 網路效能監控的定價變更

我們持續聆聽您的意見，並針對整個 Azure 的各種監控服務，於近期推出[新的定價體驗](https://azure.microsoft.com/blog/introducing-a-new-way-to-purchase-azure-monitoring-services/)。 本文章以容易閱讀的問題與解答形式，收錄有關 Azure [網路效能監控](../../networking/network-monitoring-overview.md) (NPM) 的定價變更。

網路效能監控是由三個元件所組成：
* [效能監視器](../../networking/network-monitoring-overview.md#performance-monitor)
* [服務端點監視](../../networking/network-monitoring-overview.md)
* [ExpressRoute 監視](../../networking/network-monitoring-overview.md#expressroute-monitor)

下列各節說明 NPM 元件的定價變更。

## <a name="performance-monitor"></a>效能監視器

**效能監控使用量在舊的模型中如何計費？**

NPM 的計費基礎是兩個元件的使用量和耗用量：
* ****：所有綜合交易都是來自於節點且結束於節點。 節點也稱為代理程式或 Microsoft Monitoring Agent。
* **資料**：各種網路測試的結果會儲存在 Log Analytics 工作區中。

在舊的模型中，帳單的計算方式是根據節點的數目和所產生的資料量。 

**效能監控使用量在新的模型中如何計費？**

NPM 中的效能監控功能現在是根據下列組合計費： 

* 所監控的子網路連結
* 資料量

**什麼是子網路連結？**

效能監控會監視網路上兩個或多個位置之間的連線。 一個子網路上的節點/代理程式群組與另一個子網路上的節點群組之間的連線，就稱為子網路連結。

**我有兩個子網 (A 和 B) ，而且在每個子網上有數個代理程式。效能監視器會監視子網 A 上所有代理程式到子網 B 上所有代理程式的連線能力。我是否會根據子網間連線的數目向我收取費用？**

否。 為了計費目的，從子網路 A 到子網路 B 的所有連線會群組在一起成為一個子網路連結， 而會向您收取單一連線的費用。 效能監控會持續監控每個子網路上各種代理程式之間的連線。

**監視子網路連結的成本為何？**

如需監控單一子網路連結一個月的成本，請參閱 [Ping 網格](https://azure.microsoft.com/pricing/details/network-watcher/)區段。

**效能監控所產生的資料如何收費？**

您可以在 Log Analytics 的 [ [定價] 頁面](https://azure.microsoft.com/pricing/details/log-analytics/) 的 [資料內嵌] 區段中，取得內嵌 (資料上傳至 log analytics 工作區 Azure 監視器、處理和編制索引) 的費用。 [定價頁面](https://azure.microsoft.com/pricing/details/log-analytics/)的「資料保留」區段，另提供資料保留 (也就是超過第一個月後，依客戶的選項所保留的資料) 的計費方式。


## <a name="expressroute-monitor"></a>ExpressRoute 監視

**ExpressRoute 監視的使用量如何收費？**

ExpressRoute 監視費用是根據監視期間所產生的資料量計費。 如需詳細資訊，請參閱「效能監控所產生的資料如何收費？」

**我使用 ExpressRoute 監視器來監視多個 ExpressRoute 線路。我會根據受監視的線路數目來收費嗎？**

並不會根據線路數目或對等互連類型 (例如私人對等互連、Microsoft 對等互連) 向您收費。 如上所述，會根據資料量向您收費。

**ExpressRoute 監控單一線路時，所產生的資料量為何？**

ExpressRoute 監控私人對等互連連線時，每個月產生的資料量如下所示：

|百分位數      |資料/月 (MB)|
| :---:          |           ---:|
|50<sup>th</sup> |            192|
|60<sup>th</sup> |            256|
|70<sup>th</sup> |            360|
|80<sup>th</sup> |            498|
|90<sup>th</sup> |            870|
|95<sup>th</sup> |           1560|


根據上述資料表，在第 50 個百分位數的客戶需支付 192 MB 資料的費用。 第一個月為 2.30 美元/GB，監控線路所產生的成本是 0.43 美元 (= 192 * 2.30 / 1024)。

**資料量發生變化有哪些原因？**

所產生的監控資料量取決於許多因素，例如：
* 代理程式數目。 錯誤隔離的精確度會隨代理程式的數目增加而增加。
* 網路上的躍點數目。
* 來源與目的地之間的路徑數目。

在較高百分位數的客戶 (在上表中)，通常會在其內部部署網路上從數個有利點監控其線路。 同時也會將多個代理程式放在網路更深處，遠離服務提供者邊緣路由器。 代理程式通常會放在資料中心的數個使用者站台、分支和機架。

## <a name="service-endpoint-monitor"></a>服務端點監視

**服務端點監視的使用量如何收費？**

服務端點監視的使用量收費計算方式是根據：
* 連線數目
* 資料量

**什麼是連線？**

連線是測試整個月從單一代理程式對一個端點 (URL 或網路服務) 觸達能力的作業。 例如，監視三個代理程式對 bing.com 的連線會構成三個連線。

**服務端點監視的成本為何？**

請參閱連線 [監視](https://azure.microsoft.com/pricing/details/network-watcher/) 一節，以瞭解整個月監視端點的成本。 Log Analytics [定價頁面](https://azure.microsoft.com/pricing/details/log-analytics/)的「資料擷取」區段提供資料的費用。

## <a name="references"></a>參考

[Log Analytics 定價常見問題集](https://azure.microsoft.com/pricing/details/log-analytics/)：常見問題集區段包含關於免費層、每個節點定價和其他定價詳細資料。
