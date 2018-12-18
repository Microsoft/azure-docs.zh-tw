---
title: Azure 作業安全性最佳做法 | Microsoft Docs
description: 此文章提供一組 Azure 作業安全性的最佳做法。
services: security
documentationcenter: na
author: TerryLanfear
manager: mbaldwin
editor: tomsh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/20/2018
ms.author: terrylan
ms.openlocfilehash: d005dd01de0dff0136c0a4e9775001dbe018228f
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47035278"
---
# <a name="azure-operational-security-best-practices"></a>Azure 作業安全性最佳做法
Azure 作業安全性是指使用者可在 Azure 中用來保護其資料、應用程式和其他資產的服務、控制措施與功能。 Azure 作業安全性的基礎架構涵蓋透過 Microsoft 特有功能獲得的知識，包括[安全性開發週期 (SDL)](https://www.microsoft.com/sdl)、[Microsoft Security Response Center](https://www.microsoft.com/msrc?rtc=1) 方案，以及對網路安全性威脅型態的深層認知。

此文章討論安全性最佳做法的集合。 這些最佳做法衍生自我們的 Azure 資料庫安全性以及如您本身的客戶體驗。

針對每個最佳做法，我們會說明︰
-   最佳作法是什麼
-   您為何想要啟用該最佳作法
-   如果無法啟用最佳作法，結果可能為何
- 如何學習啟用最佳作法

這篇＜Azure 作業安全性最佳做法＞文章是以共識意見以及 Azure 平台功能和特性集 (因為在撰寫此文章時已存在) 作為基礎。 意見和技術會隨著時間改變，這篇文章將會定期進行更新以反映這些變更。

## <a name="monitor-storage-services-for-unexpected-changes-in-behavior"></a>監視儲存體服務在行為上是否有非預期的變更
與傳統環境相比，在雲端環境託管的分散式應用程式一旦發生問題，無論要為其進行診斷或疑難排解，都更加複雜。 應用程式可以在 PaaS 或 IaaS 基礎架構、內部部署環境、行動裝置或是這幾種環境的組合上部署。 您應用程式的網路流量可能會周遊公用及私人網路，而且您的應用程式可能會使用多個儲存技術。

您應該持續監視您應用程式使用的儲存服務在行為上是否有任何非預期的變更 (例如回應時間更慢)。 使用記錄收集更多詳細的資料，並深入分析問題。 透過監視與記錄取得的診斷資訊將有助於判斷應用程式所遭遇問題的根本原因。 之後，您才能針對問題進行疑難排解，並決定該採取哪些合宜的步驟來矯正它。

[Azure 儲存體分析](../storage/storage-analytics.md)會執行記錄，並提供 Azure 儲存體帳戶的計量資料。 建議您使用此資料來追蹤要求、分析使用量趨勢，以及診斷儲存體帳戶的問題。

## <a name="prevent-detect-and-respond-to-threats"></a>預防、偵測及回應威脅
[Azure 資訊安全中心](../security-center/security-center-intro.md)有助於您利用加強對 Azure 資源安全性的可見度 (和控制權)，以預防、偵測並回應威脅。 它會在您的 Azure 訂用帳戶之間提供整合式安全性監視和原則管理，協助您偵測可能會忽略的威脅，且適用於各種安全性解決方案。

資訊安全中心的免費層僅為 Azure 資源提供有限的安全性。 標準層可將這些功能擴充到內部部署及其他雲端。 資訊安全中心的標準層可協助您尋找和修正安全性弱點、套用存取和應用程式控制項以封鎖惡意活動、使用分析和情報來偵測威脅，以及在遭受攻擊時迅速回應。 前 60 天可以免費試用資訊安全中心標準定價層。 建議您[將 Azure 訂用帳戶上架到資訊安全中心的標準層](../security-center/security-center-get-started.md)。

使用資訊安全中心，以集中檢視所有 Azure 資源的安全性狀態。 只需看一眼，便可確認安全性控制項是否已就緒並正確設定，並快速找出任何需要注意的資源。

## <a name="monitor-end-to-end-scenario-based-network-monitoring"></a>監視以端對端案例為基礎的網路監視
客戶可以結合網路資源 (例如虛擬網路、ExpressRoute、應用程式閘道及負載平衡器)，在 Azure 中建置端對端網路。 每個網路資源都可進行監視。

[Azure 網路監看員](../network-watcher/network-watcher-monitoring-overview.md)是一個區域性服務。 使用其診斷和視覺化工具可監視並診斷位於和進出 Azure 的網路案例層級條件。

以下是網路監視及可用工具的最佳作法。

**最佳做法**：利用封包擷取自動化遠端網路監視作業。   
**詳細資料**：使用網路監看員無須登入您的 VM，就能監視及診斷網路問題。 您可以設定警示來觸發[封包擷取](../network-watcher/network-watcher-alert-triggered-packet-capture.md)，並能存取封包層級的即時效能資訊。 當您發現問題時，可以詳加調查，從而進行更好的診斷。

**最佳做法**：使用流量記錄獲取對網路流量的見解。   
**詳細資料**：使用[網路安全性群組流量記錄](../network-watcher/network-watcher-nsg-flow-logging-overview.md)，更深入了解您的網路流量模式。 流量記錄中的資訊可協助您收集合規性資料，並稽核及監視您的網路安全性設定檔。

**最佳做法**：診斷 VPN 連線問題。   
**詳細資料**：使用網路監看員[診斷最常見的 VPN 閘道及連線問題](../network-watcher/network-watcher-diagnose-on-premises-connectivity.md)。 您不僅可以找出問題，還可以使用詳細的記錄進一步調查。

## <a name="secure-deployment-by-using-proven-devops-tools"></a>使用經證實的 DevOps 工具進行安全的部署
使用下列 DevOps 最佳做法以確保您企業和團隊的生產力與效率。

**最佳做法**：自動建置及部署服務。   
**詳細資料**：[基礎架構即程式碼](https://en.wikipedia.org/wiki/Infrastructure_as_Code)是一組技術和實作，可協助 IT 專業人員減輕每日建置及管理模組化基礎架構的負擔。 它可讓 IT 專業人員建置並維護其新式伺服器環境，就像是軟體開發人員建置並維護應用程式程式碼一樣。

您可以使用 [Azure Resource Manager](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/) 搭配宣告式範本佈建應用程式。 在單一的範本中，您可以部署多個服務及其相依性。 您可以使用相同的範本，在應用程式生命週期的每個階段重複部署應用程式。

**最佳做法**：自動建置並部署至 Azure Web 應用程式或雲端服務。   
**詳細資料**：您可以將 Visual Studio Team Services (VSTS) 小組專案設定為[自動建置並部署](https://www.visualstudio.com/docs/build/overview)至 Azure Web 應用程式或雲端服務。 VSTS 會在對 Azure 執行組建之後，將二進位檔自動部署至每個程式碼簽入後面。 套件建置程序等同於 Visual Studio 中的 [套件] 命令，而發佈步驟等同於 Visual Studio 中的 [發佈] 命令。

**最佳做法**：自動化發行管理。   
**詳細資料**：Visual Studio [Release Management](https://msdn.microsoft.com/library/vs/alm/release/overview) 是用於自動化多階段部署及管理發行程序的解決方案。 建立受控持續部署管線以快速、輕鬆且經常發行。 您可以使用 Release Management，將發行程序自動化，而且可以擁有預先定義的核准工作流程。 在內部部署及部署至雲端、擴充，並視需要自訂。

**最佳做法**：在您啟動應用程式或將更新部署至生產環境之前，請先檢查該應用程式的效能。   
**詳細資料**：使用 VSTS 執行雲端式[負載測試](https://www.visualstudio.com/docs/test/performance-testing/getting-started/getting-started-with-performance-testing)，以：

- 尋找您應用程式中的效能問題。
- 提升部署品質。
- 確定您的應用程式仍可使用。
- 確定您的應用程式可以處理後續上市或行銷活動的流量。

**最佳做法**：監視應用程式效能。   
**詳細資料**：[Azure Application Insights](../application-insights/app-insights-overview.md) 是多個平台上的 Web 開發人員所適用的可延伸「應用程式效能管理」(APM) 服務。 使用 Application Insights 監視您的即時 Web 應用程式。 它會自動偵測效能異常。 其中包括分析工具可協助您診斷問題，並了解使用者實際如何運用您的應用程式。 它是設計來協助您持續改善效能和可用性。

## <a name="mitigate-and-protect-against-ddos"></a>減少並防範 DDoS
分散式阻斷服務 (DDoS) 是一種嘗試耗盡應用程式資源的攻擊類型。 目標是影響應用程式的可用性，及其處理合法要求的能力。 這些攻擊會變得越來越複雜，而且大小和影響越來越大。 它們可以鎖定可透過網際網路公開觸達的任何端點。

設計和建置 DDoS 復原需要規劃與設計各種不同的失敗模式。

以下是在 Azure 上建置 DDoS 復原服務的最佳做法。

**最佳做法**：確保安全性在應用程式的整個生命週期 (從設計和實作到部署與作業) 中具有優先順序。 應用程式可能存在一些錯誤 (bug)，讓非常小量的要求使用大量資源，因而導致服務中斷。    
**詳細資料**：若要協助保護在 Microsoft Azure 上執行的服務，您應該對您應用程式架構有良好的了解，而且必須專注於[軟體品質的五大要素](https://docs.microsoft.com/azure/architecture/guide/pillars)。 您應該了解一般流量、應用程式與其他應用程式之間的連線模型，以及公開至公用網際網路的服務端點。

確保應用程式有足夠的彈性，能夠處理以應用程式本身為目標的阻斷服務，至關重要。 從[安全性開發週期 (SDL)](https://www.microsoft.com/en-us/sdl) 開始，安全性和隱私權便會內建於 Azure 平台之中。 SDL 會在每個開發階段處理安全性，並確保 Azure 會持續更新以使其更為安全。

**最佳做法**：將應用程式設計為可[水平調整](https://docs.microsoft.com/azure/architecture/guide/design-principles/scale-out)以滿足放大負載的需求，遭遇 DDoS 攻擊時尤其需要。 如果您的應用程式相依於服務的單一執行個體，它會建立單一失敗點。 佈建多個執行個體可讓系統更有彈性且更具延展性。   
**詳細資料**：若是 [Azure App Service](../app-service/app-service-value-prop-what-is.md)，選取可提供多個執行個體的 [App Service 方案](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md)。

對於 Azure 雲端服務，設定您的每個角色以使用[多個執行個體](../cloud-services/cloud-services-choose-me.md)。

對於 [Azure 虛擬機器](../virtual-machines/windows/overview.md)，確保 VM 架構包含多個 VM，而且每個 VM 都包含於[可用性設定組](../virtual-machines/virtual-machines-windows-manage-availability.md)中。 建議您使用虛擬機器擴展集，自動調整功能。

**最佳做法**：將應用程式中的安全性防禦分層，可降低成功攻擊的機會。 使用 Azure 平台的內建功能，為您的應用程式實作安全設計。   
**詳細資料**：攻擊的風險會隨著應用程式大小 (介面區) 而提高。 您可以使用白名單來關閉負載平衡器 ([Azure Load Balancer](../load-balancer/load-balancer-get-started-internet-portal.md) 和 [Azure 應用程式閘道](../application-gateway/application-gateway-create-probe-portal.md)) 上不需要的公開 IP 位址空間和接聽連接埠，來減少介面區。

[網路安全性群組](../virtual-network/security-overview.md)是減少攻擊面的另一種方法。 您可以使用[服務標記](../virtual-network/security-overview.md#service-tags)和[應用程式安全性群組](../virtual-network/security-overview.md#application-security-groups)，將建立安全性規則與設定網路安全性的複雜性近可能降低，直到成為應用程式結構的自然延伸。

應盡可能將 Azure 服務部署於[虛擬網路](../virtual-network/virtual-networks-overview.md)上。 此種做法可讓服務資源透過私人 IP 位址進行通訊。 根據預設，來自虛擬網路的 Azure 服務流量會使用公用 IP 位址作為來源 IP 位址。

使用[服務端點](../virtual-network/virtual-network-service-endpoints-overview.md)會在從虛擬網路存取 Azure 服務時，將服務流量切換為使用虛擬網路私人位址作為來源 IP 位址。

我們通常會看到客戶的內部部署資源連同其在 Azure 中的資源遭受攻擊。 如果您將內部部署環境連線到 Azure，請將內部部署資源在公用網際網路上的曝光率降至最低。

Azure 有兩個 DDoS [服務供應項目](../virtual-network/ddos-protection-overview.md)，可防止網路攻擊：

- 基本保護預設會整合到 Azure 中，不需任何額外成本。 Azure 全球部署網路的規模與能力可提供防禦，透過一律啟動的流量監視和即時緩和，來抵禦常見的網路層攻擊。 基本保護不需要設定使用者也不需要變更應用程式，而且有助於保護所有 Azure 服務，包括像是 Azure DNS 的 PaaS 服務。
- 標準保護則針對網路攻擊，提供進階的 DDoS 防護功能。 其會自動調整以保護特定的 Azure 資源。 只需在建立虛擬網路時啟用保護即可。 您也可以在建立之後再啟用保護，而不需進行任何應用程式或資源變更。

## <a name="next-steps"></a>後續步驟
如需更多安全性最佳做法，請參閱 [Azure 安全性最佳做法與模式](security-best-practices-and-patterns.md)，以便在使用 Azure 設計、部署和管理雲端解決方案時使用。

下列資源可提供更多有關 Azure 安全性和相關 Microsoft 服務的一般資訊：
* [Azure 安全性部落格](https://blogs.msdn.microsoft.com/azuresecurity/) - Azure 安全性的最新資訊
* [Microsoft 安全性回應中心](https://technet.microsoft.com/library/dn440717.aspx) -- 可在其中回報 Microsoft 安全性弱點 (包括 Azure 的問題) 或透過電子郵件傳送給 secure@microsoft.com
