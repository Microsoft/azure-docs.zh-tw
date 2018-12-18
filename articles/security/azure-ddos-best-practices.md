---
title: Azure DDoS 保護最佳做法與參考架構 | Microsoft Docs
description: 了解如何使用記錄資料來取得應用程式的深入解析。
services: security
author: barclayn
manager: mbaldwin
editor: TomSh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/06/2018
ms.author: barclayn
ms.openlocfilehash: 29aea0171c767b2188e3eac23b1d61ffbda8e284
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986843"
---
# <a name="azure-ddos-protection-best-practices-and-reference-architectures"></a>Azure DDoS 保護：最佳做法與參考架構

本文適用於 IT 決策者和安全性人員。 本文預設您對 Azure、網路、安全性相當熟悉。

設計分散式阻斷服務 (DDoS) 復原需要規劃與設計各種不同的失敗模式。 本文提供於 Azure 設計應用程式的最佳做法，使其在遭遇 DDoS 攻擊時具備恢復功能。

## <a name="types-of-attacks"></a>攻擊類型

DDoS 是一種嘗試耗盡應用程式資源的攻擊類型。 目標是影響應用程式的可用性，及其處理合法要求的能力。 攻擊會變得越來越複雜，而且大小和影響越來越大。 DDoS 攻擊可以鎖定可透過網際網路公開觸達的任何端點。

Azure 提供持續保護來抵禦 DDoS 攻擊。 此項保護功能會預設整合到 Azure 平台內，不需任何額外成本。 

除了平台的核心 DDoS 保護外，[Azure DDoS 保護標準](https://azure.microsoft.com/services/ddos-protection/)提供了進階的 DDoS 風險降低功能，以對抗網路攻擊。 其會自動調整以保護特定的 Azure 資源。 只需在建立新的虛擬網路時啟用保護即可。 您也可以在建立之後再啟用保護，而不需進行任何應用程式或資源變更。

![在保護客戶與虛擬網路不受攻擊者影響的層面上，Azure DDoS 保護所扮演的角色](media/azure-ddos-best-practices/image1.png)

DDoS 攻擊可分為三種類別：體積型，通訊協定、資源攻擊。

### <a name="volumetric-attacks"></a>體積型攻擊

體積型攻擊是最常見的 DDoS 攻擊類型。 體積型攻擊是目標為網路與傳輸層的暴力密碼破解攻擊。 這類攻擊會嘗試耗盡資源 (如網路連結)。 

這些攻擊通常會使用多個受感染的系統，以看似合法的流量填滿網路層。 在攻擊中會使用多種網路層通訊協定，例如網際網路控制訊息通訊協定 (ICMP)、使用者資料包通訊協定 (UDP)、傳輸控制通訊協定 (TCP)。

最常使用的網路層 DDoS 攻擊是 TCP SYN 洪水攻擊、ICMP 回應、UDP 洪水攻擊、DNS 及 NTP 放大攻擊。 此類型的攻擊不只可用來中斷服務，也可用來作為更惡毒且具針對性之網路入侵的煙幕。 近來體積型攻擊的其中一例是影響到 GitHub 的 [Memcached 漏洞](https://www.wired.com/story/github-ddos-memcached/)。 此攻擊的目標為 UDP 連接埠 11211，並產生 1.35 Tb/秒的攻擊量。

### <a name="protocol-attacks"></a>通訊協定攻擊

通訊協定攻擊的目標為應用程式通訊協定。 攻擊嘗試使用基礎結構裝置中的所有可用資源，例如防火牆、應用程式伺服器和負載平衡器。 通訊協定攻擊會使用格式不正確或包含通訊協定異常狀況的封包。 這些攻擊的運作方式是傳送大量開啟要求，讓伺服器和其他通訊裝置回應這些要求並等候封包回應。 目標會嘗試回應開啟要求，最終導致目標系統當機。

通訊協定型 DDoS 攻擊的最常見範例是 TCP SYN 洪水攻擊。 在這種攻擊中，會出現一連串的 TCP SYN 要求，嘗試癱瘓目標。 目標則是讓目標沒有回應。 除了視為應用程式層的攻擊，2016 年的 Dyn 中斷也被視為目標鎖定 Dyn DNS 伺服器連接埠 53 的 TCP SYN 洪水攻擊。

### <a name="resource-attacks"></a>資源攻擊

資源攻擊的目標為應用程式層。 它們會盡力觸發後端程序來癱瘓目標系統。 資源攻擊會濫用看似正常但會為伺服器帶來需要大量 CPU 查詢的流量。 耗盡資源所需的流量相對低於其他類型的攻擊。 難以分辨資源攻擊中的流量與合法流量，因此很難偵測到。 最常見的資源攻擊位於 HTTP/HTTPS 和 DNS 服務上。

## <a name="shared-responsibility-in-the-cloud"></a>雲端中共同承擔的責任

深入防禦策略有助於抵禦愈來愈千變萬化與複雜的攻擊。 安全性是客戶和 Microsoft 之間應共同承擔的責任。 Microsoft 將其稱為[共同承擔的責任模型](https://azure.microsoft.com/blog/microsoft-incident-response-and-shared-responsibility-for-cloud-computing/) (英文)。 下圖顯示了責任分工：

![客戶與 Azure 的責任](media/azure-ddos-best-practices/image2.png)

Azure 客戶可藉由檢閱 Microsoft 的最佳做法，以及全域建置針對失敗進行設計及測試的分散式應用程式，從中受益。

## <a name="fundamental-best-practices"></a>基本的最佳做法

下一節提供在 Azure 上建置 DDoS 復原服務的規範性指引。

### <a name="design-for-security"></a>安全性設計

確保安全性在應用程式的整個生命週期 (從設計和實作到部署與作業) 中具有優先順序。 應用程式可能存在一些錯誤 (bug)，可允許數量相對較低的要求過度使用大量資源，因而導致服務中斷。 

若要協助保護在 Microsoft Azure 上執行的服務，您應該對您應用程式架構有良好的了解，而且必須專注於[軟體品質的五大要素](https://docs.microsoft.com/azure/architecture/guide/pillars)。
您應該了解一般流量、應用程式與其他應用程式之間的連線模型，以及公開至公用網際網路的服務端點。

確保應用程式有足夠的彈性，能夠處理以應用程式本身為目標的阻斷服務，至關重要。 從[安全性開發週期 (SDL)](https://www.microsoft.com/sdl/default.aspx) 開始，安全性和隱私權便會內建於 Azure 平台之中。 SDL 會在每個開發階段處理安全性，並確保 Azure 會持續更新以使其更為安全。

### <a name="design-for-scalability"></a>延展性設計

延展性是指系統能夠處理負載增加的能力。 您必須將應用程式設計為可[水平調整](https://docs.microsoft.com/azure/architecture/guide/design-principles/scale-out)以滿足放大負載的需求，遭遇 DDoS 攻擊時尤其需要。 如果您的應用程式相依於服務的單一執行個體，它會建立單一失敗點。 佈建多個執行個體可讓系統更有彈性且更具延展性。

對於 [Azure App Service](../app-service/app-service-value-prop-what-is.md)，請選擇可提供多個執行個體的 [App Service 方案](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md)。 對於 Azure 雲端服務，設定您的每個角色以使用[多個執行個體](../cloud-services/cloud-services-choose-me.md)。 對於 [Azure 虛擬機器](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)，確保虛擬機器 (VM) 架構包含多個 VM，而且每個 VM 都包含於[可用性設定組](../virtual-machines/virtual-machines-windows-manage-availability.md)中。 建議您使用[虛擬機器擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)，自動調整功能。

### <a name="defense-in-depth"></a>深層防禦

深層防禦背後的概念是使用各種不同的防禦策略來管理風險。 將應用程式中的安全性防禦分層，可降低成功攻擊的機會。 建議您使用 Azure 平台的內建功能，為您的應用程式實作安全設計。

例如，攻擊的風險會隨著應用程式大小 (*介面區*) 而提高。 您可以使用白名單來關閉負載平衡器 ([Azure Load Balancer](../load-balancer/load-balancer-get-started-internet-portal.md) 和 [Azure 應用程式閘道](../application-gateway/application-gateway-create-probe-portal.md)) 上不需要的公開 IP 位址空間和接聽連接埠，來減少介面區。 [網路安全性群組 (NSG)](../virtual-network/security-overview.md) 是減少攻擊面的另一種方法。
您可以使用[服務標記](../virtual-network/security-overview.md#service-tags)和[應用程式安全性群組](../virtual-network/security-overview.md#application-security-groups)，將建立安全性規則與設定網路安全性的複雜性近可能降低，直到成為應用程式結構的自然延伸。

應盡可能將 Azure 服務部署於[虛擬網路](../virtual-network/virtual-networks-overview.md)上。 此種做法可讓服務資源透過私人 IP 位址進行通訊。 根據預設，來自虛擬網路的 Azure 服務流量會使用公用 IP 位址作為來源 IP 位址。 使用[服務端點](../virtual-network/virtual-network-service-endpoints-overview.md)，會在從虛擬網路存取 Azure 服務時，將服務流量切換為使用虛擬網路私人位址作為來源 IP 位址。

我們通常會看到客戶的內部部署資源連同其在 Azure 中的資源遭受攻擊。 如果您將內部部署環境連線到 Azure，建議盡量少讓內部部署資源暴露於公用網際網路上。 您可以藉由在 Azure 中部署已知的公用實體，來使用縮放和進階的 DDoS 保護功能。 由於這些可公開存取的實體通常是 DDoS 攻擊的目標，因此，將它們放置於 Azure，可降低對您內部部署資源的影響。

## <a name="azure-offerings-for-ddos-protection"></a>Azure 針對 DDoS 保護的供應項目

Azure 提供兩個 DDoS 服務供應項目，來保護您免於受到網路攻擊 (第 3 層和第4 層)：基本 DDoS 保護與標準 DDoS 保護。 

### <a name="ddos-protection-basic"></a>基本 DDoS 保護

基本保護預設會整合到 Azure，不需任何額外成本。 Azure 全球部署網路的規模與能力可提供防禦，透過一律啟動的流量監視和即時緩和，來抵禦常見的網路層攻擊。 基本 DDoS 保護不需要進行任何使用者設定或應用程式變更。 基本 DDoS 保護可協助保護所有的 Azure 服務 (包括像是 Azure DNS 的 PaaS 服務)。

![顯示 Azure 網路的地圖，上有「全球 DDoS 風險降低效果」和「領先的 DDoS 風險降低能力」的文字。](media/azure-ddos-best-practices/image3.png)

Azure 中的基本 DDoS 保護由軟體和硬體元件所組成。 軟體控制平面會決定何時、何處及何種類型的傳輸應該透過分析並移除攻擊流量的硬體裝置來操縱。 控制平面會根據整個基礎結構的 DDoS 保護*原則*做出決定。 這項原則是以靜態方式設定，並全域套用至所有的 Azure 客戶。

比方說，DDoS 保護原則會指定應該*觸發*哪個流量磁碟區的保護。 (亦即，應清除設備，藉以路由租用戶的流量。)接著，原則會指定清除設備應如何*減輕*攻擊。

Azure 基本 DDoS 保護服務已將目標設為基礎結構保護與 Azure 平台保護。 當流量明顯超過某個可能在多租用戶環境中影響多位客戶的速率時，即會降低流量。 它不會提供警示或為每位客戶自訂原則。

### <a name="ddos-protection-standard"></a>Azure 標準 DDoS 保護

標準保護提供增強的 DDoS 風險降低功能。 其會自動調整，以保護您在虛擬網路中特定的 Azure 資源。 只需在任何新的或現有的虛擬網路上啟用保護，而不需進行任何應用程式或資源變更。 它具有數個優於基本服務的優點，包括記錄、警示與遙測。 下列各節會概略說明標準 Azure DDoS 保護服務的主要功能。

#### <a name="adaptive-real-time-tuning"></a>調適性即時調整

Azure 基本 DDoS 保護服務可協助保護客戶，並防止對其他客戶產生影響。 例如，如果合法連入流量的正常流量小於涵蓋整個基礎結構範圍的 DDoS 保護原則*觸發率*，而且服務是針對該流量所佈建，則可能不會注意到對於該客戶資源的 DDoS 攻擊。 更普遍的說法是，最新攻擊 (例如多向量 DDoS) 的複雜本質，以及租用戶的應用程式專屬行為，會呼叫針對每位客戶自訂的保護原則。 服務會使用兩種見解來完成這項自訂：

- 自動學習每位客戶 (每個 IP) 的第 3 層與第 4 層流量模式。

- 考慮到 Azure 的規模會使之吸收大量的流量，進而降低誤判。

![標準 DDoS 保護運作方式圖解，圈起「原則產生」](media/azure-ddos-best-practices/image5.png)

#### <a name="ddos-protection-telemetry-monitoring-and-alerting"></a>DDoS 保護遙測、監視及警示

標準 DDoS 保護在遭受 DDoS 攻擊持續期間，透過 [Azure 監視器](../azure-monitor/overview.md)公開豐富的遙測。 可設定 DDoS 保護所用的任何 Azure 監視器度計量的警示。 您可以透過 Azure 監視器診斷介面來與 Splunk (Azure 事件中樞)、Azure Log Analytics 及 Azure 儲存體整合，以進行進階分析。

##### <a name="ddos-mitigation-policies"></a>DDoS 防護原則

在 Azure 入口網站中，選取 [監視] > [計量]。 在 [計量] 窗格中，選取資源群組、選取 [公用 IP 位址] 的資源類型，並選取您的 Azure 公用 IP 位址。 DDoS 計量會顯示在 [可用計量] 窗格中。

「標準 DDoS 保護」會在啟用 DDoS 的虛擬網路中，對受保護資源的每個公用 IP 套用三個自動調整的風險降低原則 (TCP SYN、TCP、UDP)。 可選取**要觸發 DDoS 防護的輸入封包**計量，藉此檢視原則閾值。

![可用的計量與計量圖表](media/azure-ddos-best-practices/image7.png)

原則閾值會透過機器學習架構的網路流量分析進行自動設定。 只有在超出原則閾值時，才會針對遭受攻擊的 IP 位址進行 DDoS 防護。

##### <a name="metric-for-an-ip-address-under-ddos-attack"></a>遭受 DDoS 攻擊的 IP 位址計量

如果公用 IP 位址遭受攻擊，則當 DDoS 保護在降低攻擊流量的風險時，[是否正遭受 DDoS 攻擊] 的計量值會變更為 1。

![「是否正遭受 DDoS 攻擊」計量與圖表](media/azure-ddos-best-practices/image8.png)

建議您針對此計量設定警示。 之後當公用 IP 位址上正在執行 DDoS 風險降低作業時，您便會收到通知。

如需詳細資訊，請參閱[使用 Azure 入口網站管理 Azure DDoS 保護標準](../virtual-network/ddos-protection-manage-portal.md)。

#### <a name="web-application-firewall-for-resource-attacks"></a>資源攻擊的 Web 應用程式防火牆

針對應用程式層上的資源攻擊，應該設定 Web 應用程式防火牆 (WAF) 來協助維護 Web 應用程式的安全。 WAF 會檢查傳入的 Web 流量，以封鎖 SQL 插入式攻擊、跨網站指令碼攻擊、DDoS 攻擊，以及其他第 7 層攻擊。 Azure 提供 [WAF 作為一項應用程式閘道功能](../application-gateway/application-gateway-web-application-firewall-overview.md)，用來進行 Web 應用程式的集中式保護，免於遭遇常見的攻擊和弱點。 透過 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?search=WAF&page=1)，您可以取得由 Azure 夥伴所提供且可能更適合您需求的其他 WAF 供應項目。

即使是 Web 應用程式防火牆，也很容易受到體積型與狀態耗盡攻擊。 強烈建議您在 WAF 虛擬網路上啟用標準 DDoS 保護，協助防禦體積型與通訊協定攻擊。 如需詳細資訊，請參閱 [DDoS 保護參考架構](#ddos-protection-reference-architectures)一節。

### <a name="protection-planning"></a>保護規劃

規劃與準備對於了解在發生 DDoS 攻擊時系統將如何執行，至關重要。 設計事件管理回應方案是這項工作的一部分。

如果您有標準 DDoS 保護，請務必在網際網路對向端點的虛擬網路上將之啟用。 設定 DDoS 警示可幫助您持續監看基礎結構上出現的任何潛在攻擊。 

您應該獨立監視各個應用程式， 並了解應用程式的正常行為。 如果應用程式在遭受 DDoS 攻擊時並未如預期般運作，請準備好採取行動。

#### <a name="testing-through-simulations"></a>透過模擬測試

最好的做法就是透過執行定期模擬，來測試您認為服務會如何回應攻擊的假設。 在測試期間，會驗證您的服務或應用程式是否會繼續如預期般運作，而且不會對使用者體驗產生任何中斷。 請以技術和處理程序的觀點，找出其中的差距，然後併入 DDoS 回應策略中。 建議在預備環境或非尖峰時間執行這類測試，將對生產環境的影響降到最低。

我們已與 [BreakingPoint Cloud](https://www.ixiacom.com/products/breakingpoint-cloud) 合作建立一個介面，Azure 客戶可以針對啟用 DDoS 保護的公用端點產生流量，以進行模擬。 您可以使用 [BreakingPoint Cloud](https://www.ixiacom.com/products/breakingpoint-cloud) 進行模擬，以達成以下效果：

- 驗證 Azure DDoS 保護會如何保護您的 Azure 資源免受 DDoS 攻擊。

- 使遭受 DDoS 攻擊時的事件回應程序達到最佳化。

- 記載 DDoS 合規性。

- 訓練網路安全性小組。

要維護網路安全，必須在防禦技術上不斷創新。 Azure 標準 DDoS 保護是最先進的技術，可提供有效的解決方案，為日益複雜的 DDoS 攻擊提供防護。

## <a name="components-of-a-ddos-response-strategy"></a>DDoS 回應策略的元件

從使用者的觀點來看，以 Azure 資源為目標的 DDoS 攻擊所需的介入最少。 儘管如此，仍要將 DDoS 風險降低納入為事件回應策略的一部分，如此有助於盡可能降低對營運能否持續運作的影響。

### <a name="microsoft-threat-intelligence"></a>Microsoft 威脅情報

Microsoft 擁有廣大的威脅情報網路。 此網路會使用延伸安全性社群的共同知識，來支援 Microsoft 線上服務、Microsoft 合作夥伴，以及網際網路安全性社群內的關聯性。 

Microsoft 身為重要的基礎結構提供者，會最先收到威脅的警告。 Microsoft 會從自己的線上服務及全球的客戶群收集威脅情報。 Microsoft 會將所有的威脅情報整併回其 Azure DDoS 保護產品中。

此外，Microsoft 數位犯罪防治中心 (DCU) 會對殭屍網路採取進攻策略。 殭屍網路是一種常見的 DDoS 攻擊指揮與控制來源。

### <a name="risk-evaluation-of-your-azure-resources"></a>Azure 資源的風險評估

請務必持續不斷地了解遭受 DDoS 攻擊的風險範圍。 請定期自問： 
- 有哪些公開使用的 Azure 新資源需要保護？

- 服務中有單一失敗點嗎？ 

- 如何將服務隔離以限制攻擊所產生的影響，同時仍能讓有效的客戶可以使用服務？

- 是否有虛擬網路應啟用標準 DDoS 保護，但尚未啟用？ 

- 我的服務是否會在跨多個區域進行容錯移轉時處於主動/主動狀態？

### <a name="customer-ddos-response-team"></a>客戶 DDoS 回應小組

成立 DDoS 回應小組是快速有效回應攻擊的關鍵步驟。 請找出組織中應負責監督規劃與執行的連絡人。 此 DDoS 回應小組應徹底了解 Azure 標準 DDoS 保護服務。 請確保小組能與內部和外部客戶 (包括 Microsoft 支援小組) 同心協力找出攻擊並降低風險。

對於您的 DDoS 回應小組，建議在您的服務可用性與持續性計劃中，常態納入模擬演練。 這些演練應該包含規模調整測試。

### <a name="alerts-during-an-attack"></a>攻擊時的警示

Azure 標準 DDoS 保護會識別 DDoS 攻擊並降低風險，無須使用者介入。 在受保護的公用 IP 上執行風險降低作業時，若想收到通知，可以在 [是否正遭受 DDoS 攻擊] 計量上[設定警示](../virtual-network/ddos-protection-manage-portal.md)。 可以選擇為其他 DDoS 計量建立警示，以了解攻擊的範圍、要卸除的流量及其他詳細資料。

#### <a name="when-to-contact-microsoft-support"></a>連絡 Microsoft 支援服務的時機

- 在遭受 DDoS 攻擊時，您發現受保護資源的效能嚴重降低或無法使用該資源。

- 您認為 DDoS 保護服務並未如預期般運作。 

  只有在 [觸發 DDoS 風險降低 (TCP/TCP SYN/UDP) 的原則] 計量值低於受保護公用 IP 資源上收到的流量時，DDoS 保護服務才會開始執行風險降低作業。

- 您正在計劃一個會大幅增加網路流量的病毒式事件。

- 有心人士威脅要針對您的資源發動 DDoS 攻擊。

對於會對業務造成重大影響的攻擊，請建立嚴重性 A 的[支援票證](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)。

### <a name="post-attack-steps"></a>攻擊發生後的步驟

遭受攻擊之後進行事後檢討，並視需要重新調整 DDoS 回應策略，永遠都是個不錯的策略。 需考量的事項：

- 是否因為缺少可調整規模的架構而使得服務或使用者體驗發生任何中斷？

- 哪些應用程式或服務發生最多？

- DDoS 回應策略的有效性如何，以及如何加以改善？

如果您懷疑遭到 DDoS 攻擊，請透過一般的 Azure 支援管道呈報。

## <a name="ddos-protection-reference-architectures"></a>DDoS 保護參考架構

標準 DDoS 保護是針對[部署於虛擬網路的服務](../virtual-network/virtual-network-for-azure-services.md)設計的。 對於其他服務，將適用預設的基本 DDoS 保護服務。 以下的參考架構依案例排列，並與架構模式組合在一起。

### <a name="virtual-machine-windowslinux-workloads"></a>虛擬機器 (Windows/Linux) 工作負載

#### <a name="application-running-on-load-balanced-vms"></a>在負載平衡的 VM 上執行的應用程式

此參考架構顯示一組經過證實的作法，可用來在負載平衡器後方的擴展集中執行多個 Windows VM，以提升可用性和延展性。 此架構可以用於任何無狀態的工作負載，例如網頁伺服器。

![在負載平衡的 VM 上執行的應用程式架構參考圖](media/azure-ddos-best-practices/image9.png)

在此架構中，會將工作負載散發到多個 VM 執行個體。 有單一公用 IP 位址，且網際網路流量是透過負載平衡器散發到 VM。 標準 DDoS 保護會在有相關聯公用 IP 的 Azure (網際網路) 負載平衡器其虛擬網路上啟用。

負載平衡器會將連入網際網路要求散發到 VM 執行個體。 虛擬機器擴展集允許以手動方式或自動根據預先定義的規則，相應縮小或放大 VM 數目。 如果資源受到 DDoS 攻擊，這就很重要。 如需此參考架構的詳細資訊，請參閱[這篇文章](https://docs.microsoft.com/azure/architecture/reference-architectures/virtual-machines-windows/multi-vm)。

#### <a name="application-running-on-windows-n-tier"></a>在 Windows 多層式架構 (N-Tier) 上執行的應用程式

有許多方式可實作多層式架構。 下圖顯示的是一般的三層式 Web 應用程式。 此架構是根據[執行負載平衡的 VM 以獲得延展性和可用性](https://docs.microsoft.com/azure/architecture/reference-architectures/virtual-machines-windows/multi-vm)一文所建置。 Web 及 Business 層會使用負載平衡的 VM。

![在 Windows 多層式架構 (N-Tier) 上執行的應用程式架構參考圖](media/azure-ddos-best-practices/image10.png)

在此架構中，標準 DDoS 保護會在虛擬網路上啟用。 虛擬網路中的所有公用 IP 都會得到第 3 層和第 4 層的 DDoS 保護。 至於第 7 層保護，則應在 WAF SKU 中部署應用程式閘道。 如需此參考架構的詳細資訊，請參閱[這篇文章](https://docs.microsoft.com/azure/architecture/reference-architectures/virtual-machines-windows/n-tier)。

#### <a name="paas-web-application"></a>PaaS Web 應用程式

此參考架構示範在單一區域中執行的 Azure App Service 應用程式。 針對使用 [Azure App Service](https://azure.microsoft.com/documentation/services/app-service/) 和 [Azure SQL Database](https://azure.microsoft.com/documentation/services/sql-database/) 的 Web 應用程式，此架構會示範一組經過證實的作法。
針對容錯移轉案例設定待命區域。

![PaaS Web 應用程式的架構參考圖](media/azure-ddos-best-practices/image11.png)

Azure 流量管理員會將連入要求路由到其中一個區域中的應用程式閘道。 正常作業期間，它會將要求路由到主要區域的應用程式閘道。 如果該區域變得無法使用，流量管理員就會容錯移轉到待命區域中的應用程式閘道。

所有來自網際網路且預定送往 Web 應用程式的流量都會透過流量管理員，路由到[應用程式閘道的公用 IP 位址](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-app-overview)。 在此案例中，應用程式服務 (Web 應用程式) 本身不會直接面向外部，且會受到應用程式閘道保護。 

建議您設定應用程式閘道 WAF SKU (防止模式) 來提供保護，以避免受到第 7 層 (HTTP/HTTPS/WebSocket) 攻擊。 此外，請將 Web 應用程式設定為[只接受來自應用程式閘道 IP 位址的流量](https://azure.microsoft.com/blog/ip-and-domain-restrictions-for-windows-azure-web-sites/) (英文)。

如需此參考架構的詳細資訊，請參閱這篇[文章](https://docs.microsoft.com/azure/architecture/reference-architectures/app-service-web-app/multi-region)。

### <a name="mitigation-for-non-web-paas-services"></a>適用於非 Web PaaS 服務的防護功能

#### <a name="hdinsight-on-azure"></a>Azure 上的 HDInsight

此參考架構示範如何針對 [Azure HDInsight 叢集](https://docs.microsoft.com/azure/hdinsight/)設定標準 DDoS 保護。 請確定 HDInsight 叢集會連結到虛擬網路，且該虛擬網路上的 DDoS 保護已啟用。

![含虛擬網路設定的 [HDInsight] 和 [進階設定] 窗格](media/azure-ddos-best-practices/image12.png)

![啟用 DDoS 保護的選取項目](media/azure-ddos-best-practices/image13.png)

在此架構中，來自網際網路且預定送往 HDInsight 叢集的流量，會路由到與 HDInsight 閘道負載平衡器相關聯的公用 IP。 閘道負載平衡器接著會將流量直接傳送到前端節點或背景工作節點。 由於已在 HDInsight 虛擬網路上啟用標準 DDoS 保護，因此虛擬網路中的所有公用 IP 都會受到第 3 層和第 4 層 DDoS 保護。 此參考架構可以與多層式 (N-Tier)/多區域參考架構結合。

如需此參考架構的詳細資訊，請參閱[使用 Azure 虛擬網路延伸 Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-extend-hadoop-virtual-network?toc=%2fazure%2fvirtual-network%2ftoc.json) 一文。


> [!NOTE]
> 在使用公用 IP 的虛擬網路中，Azure App Service 環境未能原生支援對於 PowerApps 或 API 的管理。

## <a name="next-steps"></a>後續步驟

* [Azure DDoS 保護產品頁面](https://azure.microsoft.com/services/ddos-protection/)

* [Azure DDoS 保護部落格](http://aka.ms/ddosblog) (英文)

* [Azure DDoS 保護文件](../virtual-network/ddos-protection-overview.md)
