---
title: Azure DDoS Protection Standard 概觀 | Microsoft Docs
description: 深入了解 Azure DDoS Protection 服務。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/29/2018
ms.author: jdial
ms.openlocfilehash: f25da8c1eedc31209a67ae05aef9dded45b706e0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46962397"
---
# <a name="azure-ddos-protection-standard-overview"></a>Azure DDoS Protection Standard 概觀

分散式阻斷服務 (DDoS) 攻擊是將應用程式移至雲端的客戶所面臨的最大可用性和安全性顧慮之一。 DDoS 攻擊會嘗試耗盡應用程式的資源，讓合法使用者無法使用該應用程式。 DDoS 攻擊可以鎖定可透過網際網路公開觸達的任何端點。

結合應用程式設計最佳做法的 Azure DDoS 保護可提供 DDoS 攻擊的防禦。 Azure DDoS 保護提供下列服務層：

- **基本**：已在 Azure 平台中自動啟用。 常見網路層級攻擊的永遠可用流量監視和即時安全防護功能，可提供 Microsoft 的線上服務所使用的相同防禦。 Azure 全域網路的整體規模可用於分散和減少跨區域攻擊流量。 針對 IPv4 和 IPv6 Azure [公用 IP 位址](virtual-network-public-ip-address.md)提供保護。
- **標準**：提供 [基本] 服務層以外特別針對 Azure 虛擬網路資源調整的額外安全防護功能。 DDoS 保護「標準」層級很容易啟用，而且不需要變更應用程式。 保護原則是透過專用的流量監視和機器學習演算法進行調整。 原則會套用至與虛擬網路中所部署資源相關聯的公用 IP 位址，例如 Azure Load Balancer、Azure 應用程式閘道和 Azure Service Fabric 執行個體。 在攻擊期間，可透過 Azure 監視器檢視取得即時遙測與歷史記錄。 透過診斷設定可以獲得豐富的攻擊風險降低分析。 透過 [Azure 應用程式閘道 Web 應用程式防火牆](../application-gateway//application-gateway-web-application-firewall-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)，或者從 Azure Marketplace 安裝第 3 方防火牆，可以加上應用程式層保護。 針對 IPv4 Azure [公用 IP 位址](virtual-network-public-ip-address.md)提供保護。

![Azure DDoS 保護部落格與標準](./media/ddos-protection-overview/ddoscomparison.png)

## <a name="types-of-ddos-attacks-that-ddos-protection-standard-mitigates"></a>DDoS Protection Standard 可降低風險的 DDoS 攻擊類性

DDoS 保護標準層可降低下列攻擊類型的風險：

- **巨流量攻擊**：此攻擊的目的在於以大量看似合法的流量填滿網路層。 它包括 UDP 洪水攻擊、放大洪水攻擊和其他詐騙封包洪水攻擊。 DDoS 保護標準層可降低這些潛在多 GB 攻擊的風險，方法是自動使用 Azure 的全球網路規模來快吸並清除這些攻擊。
- **通訊協定攻擊**：這些攻擊透過利用第 3 層和第 4 層通訊協定堆疊中的弱點，讓目標無法供存取。 它包括 SYN 洪水攻擊、反映攻擊和其他通訊協定攻擊。 DDoS 保護標準層可透過與用戶端互動來區別惡意與合法流量並封鎖惡意流量，以降低這些攻擊的風險。 
- **資源 (應用程式) 層攻擊**：這些攻擊會鎖定 Web 應用程式封包，以中斷主機之間的資料傳輸。 攻擊包括 HTTP 通訊協定違規、SQL 插入、跨網站指令碼和其他第 7 層攻擊。 使用 Azure [應用程式閘道 Web 應用程式防火牆](../application-gateway/application-gateway-web-application-firewall-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)搭配 DDoS Protection Standard，以提供這些攻擊的防禦。 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?page=1&search=web%20application%20firewall) 還提供了協力廠商 Web 應用程式防火牆供應項目。

DDoS Protection Standard 可保護虛擬網路中的資源，包括與虛擬機器相關聯的公用 IP 位址、內部負載平衡器，以及應用程式閘道。 與應用程式閘道 Web 應用程式防火牆搭配的 DDoS Protection Standard 可以提供完整的第 3 層至第 7 層安全防護功能。

## <a name="ddos-protection-standard-features"></a>DDoS Protection Standard 功能

![DDoS 功能](./media/ddos-protection-overview/ddosfeatures.png)

DDoS Protection Standard 功能包括：

- **原生平台整合**：原生整合到 Azure。 包括透過 Azure 入口網站進行設定。 DDoS Protection Standard 了解您的資源和資源組態。
- **現成的保護：** 經過簡化的組態會在啟用 DDoS Protection Standard 後，立即保護虛擬網路上的所有資源。 不需要任何介入或使用者定義。 一旦偵測到攻擊，DDoS Protection Standard 就會立即自動減輕攻擊。
- **永遠可用流量監視：** 您的應用程式流量模式受到全年無休的全天候監視，以尋找 DDoS 攻擊的指標。 超出保護原則時，就會執行安全防護功能。
- **自適性調整：** 智慧型流量分析功能可了解不同時間的應用程式流量，並選取及更新最適合您服務的設定檔。 設定檔會隨著時間調整流量變更。
- **多層保護：** 搭配 Web 應用程式防火牆使用時，可提供完整堆疊 DDoS 保護。
- **廣泛的安全防護範圍：** 可利用全域功能降低超過 60 種不同攻擊類型的風險，以抵禦最大的已知 DDoS 攻擊。
- **攻擊分析：** 在攻擊期間取得以五分鐘為增量的詳細報告，並在攻擊結束後取得完整摘要。 將風險降低流量記錄串流至離線安全性資訊與事件管理 (SIEM) 系統，以便在攻擊期間進行近乎即時的監視。
- **攻擊計量：** 透過 Azure 監視器可以存取每個攻擊的摘要計量。
- **攻擊警示：** 警示可設定為在開始和停止攻擊時，並且在攻擊的持續時間內使用內建攻擊計量。 警示會整合到您的作業軟體，例如 Microsoft Azure Log Analytics、Splunk、Azure 儲存體、電子郵件和 Azure 入口網站。
- **成本保證：** 資料傳輸和應用程式相應放大服務會針對記載的 DDoS 攻擊計算點數。

## <a name="ddos-protection-standard-mitigation"></a>DDoS Protection Standard 安全防護功能

DDoS 保護標準層會監視實際流量使用率，且時常將它與 DDoS 原則中定義的閾值比較。 當超過該流量閾值時，就會自動起始 DDoS 安全防護功能。 當傳回流量低於閾值時，就會移除安全防護功能。

![緩和](./media/ddos-protection-overview/mitigation.png)

在安全防護期間，DDoS 保護服務會重新導向傳送至受保護資源的流量，而且會執行數個檢查，例如下列檢查：

- 確定封包符合網際網路規格，且格式正確無誤。
- 與用戶端互動，判斷流量是否可能是詐騙封包 (例如： SYN Auth 或 SYN Cookie 或置放封包以供來源將它重新傳輸)。
- 速率限制封包 (如果沒有其他強制方法可以執行)。

DDoS 保護會封鎖攻擊流量並將剩餘流量轉送至其目的地。 在偵測到攻擊的幾分鐘內，系統會使用 Azure 監視器計量通知您。 藉由設定登入 DDoS Protection Standard 遙測，您可以將記錄寫入至可用的選項，以供日後分析。 適用於 DDoS 保護標準層的 Azure 監視器中的計量資料會保留 30 天。

Microsoft 已與 [BreakingPoint Cloud](https://www.ixiacom.com/products/breakingpoint-cloud) 合作建立一個介面，您可以針對啟用 DDoS 保護的公用 IP 位址產生流量，以進行模擬。 BreakPoint Cloud 模擬可讓您：

- 驗證 Microsoft Azure DDoS 保護標準層如何保護您的 Azure資源免受 DDoS 攻擊
- 將遭受 DDoS 攻擊時的事件回應程序最佳化
- 記載 DDoS 合規性
- 訓練您的網路安全性小組

## <a name="next-steps"></a>後續步驟

- [設定 DDoS 保護標準層](manage-ddos-protection.md)
