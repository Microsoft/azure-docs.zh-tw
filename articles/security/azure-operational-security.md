---
title: Azure 作業安全性 | Microsoft Docs
description: 了解 Microsoft Operations Management Suite (OMS)、它的服務及運作方式。
services: security
documentationcenter: na
author: UnifyCloud
manager: mbaldwin
editor: TomSh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: TomSh
ms.openlocfilehash: 60260d7bcff07a9ce2d680c84119d11271579e7d
ms.sourcegitcommit: 4597964eba08b7e0584d2b275cc33a370c25e027
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37342264"
---
# <a name="azure-operational-security"></a>Azure 作業安全性
## <a name="introduction"></a>簡介

### <a name="overview"></a>概觀
我們知道安全性是雲端中的首要工作和其重要性，因為您可在其中找到精確且及時的 Azure 安全性資訊。 針對您的應用程式和服務使用 Azure 的其中一個最佳原因是可以利用各種可用的安全性工具和功能。 這些工具和功能可協助您在安全的 Azure 平台上建立安全的解決方案。 Microsoft Azure 必須提供客戶資料的機密性、完整性和可用性，同時也能釐清責任。

為了協助客戶從客戶和 Microsoft 作業的觀點深入了解如何在 Microsoft Azure 內實作一系列安全性控制，因而編寫了本技術白皮書＜Azure 作業安全性＞來提供可透過 Windows Azure 取得之作業安全性的完整介紹。

### <a name="azure-platform"></a>Azure 平台
Azure 是一個公用雲端服務平台，支援廣泛的作業系統、程式設計語言、架構、工具、資料庫及裝置等選擇。 它可以透過 Docker 整合執行 Linux 容器；使用 JavaScript、Python、.NET、PHP、Java 及 Node.js 建置應用程式；為 iOS、Android 及 Windows 裝置建置後端。 Azure 雲端服務支援數百萬名開發人員和 IT 專家早已仰賴和信任的相同技術。

當您建置 IT 資產或將其移轉至公用雲端服務提供者時，您正是依賴該組織的能力，利用他們提供來管理您雲端式資產安全性的服務與控制機制，來保護您的應用程式和資料。

Azure 的基礎結構設計涵蓋設備與應用程式，可同時裝載數以百萬計的客戶，並提供可靠的基礎供企業符合其安全性需求。 此外，Azure 也為您提供各式各樣可設定的安全性選項及控制它們的功能，讓您能夠自訂安全性以符合組織部署的特殊需求。 本文件有助於您了解 Azure 安全性功能如何協助您滿足這些需求。

### <a name="abstract"></a>摘要
Azure 作業安全性是指使用者可在 Microsoft Azure 中用來保護其資料、應用程式和其他資產的服務、控制及功能。 Azure 作業安全性的基礎架構涵蓋透過 Microsoft 特有之各種功能獲得的知識，包括 Microsoft 安全性開發週期 (SDL)、Microsoft Security Response Center 方案，以及對網路安全性威脅型態的深層認知。

本技術白皮書概述 Microsoft 在 Microsoft Azure 雲端平台內處理 Azure 作業安全性的方法，並涵蓋以下服務：
1.  [Azure Operations Management Suite](https://docs.microsoft.com/azure/operations-management-suite/operations-management-suite-overview)

2.  [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)

3.  [Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview)

4.  [Azure 網路監看員](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)

5.  [Azure 儲存體分析](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics)

6.  [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-whatis)


## <a name="microsoft-operations-management-suite"></a>Microsoft Operations Management Suite

Microsoft Operations Management Suite (OMS) 是適用於混合式雲端的 IT 管理解決方案。 透過單獨使用或擴充現有的 System Center 部署，OMS 可為您基礎結構的雲端式管理提供最大的彈性和控制。

![Microsoft Operations Management Suite](./media/azure-operational-security/azure-operational-security-fig1.png)

透過 OMS，您就能以低於競爭解決方案的成本來管理任何雲端中的任何執行個體，包括內部部署、Azure、AWS、Windows Server、Linux、VMware 和 OpenStack。 由於 OMS 是針對雲端優先的世界建置的，因此，可提供一種新方法來管理您的企業，此為最快速且最符合成本效益的方式，可符合新的商業挑戰並適應新的工作負載、應用程式及雲端環境。

### <a name="oms-services"></a>OMS 服務

OMS 的核心功能是由在 Azure 中執行的一組服務所提供。 每個服務會提供特定的管理功能，您可以合併服務以達成不同的管理案例。

| 服務  | 說明|
| :------------- | :-------------|
| Log Analytics | 監視及分析不同資源 (包括實體和虛擬機器) 的可用性和效能。 |
|自動化 | 讓手動程序自動化，並強制設定實體和虛擬機器。 |
| Backup  | 備份及還原重要資料。 |
| Site Recovery | 為重要應用程式提供高可用性。 |

### <a name="log-analytics"></a>Log Analytics

[Log Analytics](http://azure.microsoft.com/documentation/services/log-analytics) 藉由將受控資源中的資料收集到中央儲存機制，以提供 OMS 監視服務。 此資料可能包含事件、效能資料或透過 API 提供的自訂資料。 所收集的資料即可用於警示、分析和匯出。


這個方法可讓您合併各種來源的資料，以便結合來自 Azure 服務與現有內部部署環境的資料。 它也可清楚劃分資料的收集與對該資料採取的動作，而讓所有類型的資料可以使用所有的動作。


![Log Analytics](./media/azure-operational-security/azure-operational-security-fig2.png)

Log Analytics 服務會使用下列方法安全地管理您以雲端為基礎的資料：
-   資料隔離
-   資料保留
-   實體安全性
-   事件管理
-   法規遵循
-   安全性標準認證

### <a name="azure-backup"></a>Azure 備份

[Azure 備份](http://azure.microsoft.com/documentation/services/backup)提供資料備份及還原服務，其為產品和服務之 OMS 套件的一部分。
它可保護您的應用程式資料並保留數年，而您完全不必投入資本投資，並且只需要最少的營運成本。 它可以備份實體和虛擬 Windows 伺服器中的資料，以及應用程式工作負載 (例如 SQL Server 和 SharePoint)。 [System Center Data Protection Manager (DPM) (英文)](https://en.wikipedia.org/wiki/System_Center_Data_Protection_Manager) 也可以使用它，將受保護資料複寫到 Azure 以進行備援與長期儲存。


Azure 備份中受保護的資料會儲存在位於特定地理區域的備份保存庫中。 資料是在相同的區域內進行複寫，而且根據保存庫類型，也可能會複寫到另一個區域，以供進一步復原。

### <a name="management-solutions"></a>管理解決方案
[Microsoft Operations Management Suite (OMS)](https://docs.microsoft.com/azure/operations-management-suite/oms-security-getting-started) 是 Microsoft 的雲端式 IT 管理解決方案，可協助您管理並保護內部部署和雲端基礎結構。


[管理解決方案](https://docs.microsoft.com/azure/operations-management-suite/operations-management-suite-solutions)是預先封裝的邏輯集合，可使用一或多項 OMS 服務實作特定管理案例。 Microsoft 和合作夥伴會提供不同的解決方案，您可以輕鬆地將其新增至您的 Azure 訂用帳戶，以提升您的 OMS 投資價值。 身為合作夥伴的您可以建立自己的解決方案，來支援您的應用程式和服務，並透過 Azure Marketplace 或快速入門範本將其提供給使用者。


![管理解決方案](./media/azure-operational-security/azure-operational-security-fig4.png)

[更新管理解決方案](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-update-management)就是使用多項服務來提供其他功能的良好解決方案範例。 這個解決方案使用 Windows 和 Linux 適用的 [Log Analytics](https://docs.microsoft.com/azure/log-analytics/log-analytics-overview) 代理程式，來收集每個代理程式上必要更新的相關資訊。 它會將此資料寫入至 Log Analytics 儲存機制，以便透過內含的儀表板分析資料。

當您建立部署時，可以使用 [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-intro)中的 Runbook 來安裝必要的更新。 您可在入口網站中管理這整個程序，而不需擔心基礎詳細資料。

## <a name="azure-security-center"></a>Azure 資訊安全中心

Azure 資訊安全中心可協助保護您的 Azure 資源。 它提供您 Azure 訂用帳戶之間的整合式安全性監視和原則管理。 在服務中，您不只可針對 Azure 訂用帳戶定義原則，也能針對[資源群組](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups)加以定義，如此便能以更細微的方式進行。

### <a name="security-policies-and-recommendations"></a>安全性原則和建議

安全性原則定義了針對指定之訂用帳戶或資源群組內的資源，所建議的一組控制項。

在資訊安全中心，您可以根據公司的安全性需求，以及應用程式的類型或資料的敏感性來定義原則。

![安全性原則和建議](./media/azure-operational-security/azure-operational-security-fig5.png)


在訂用帳戶層級啟用的原則將會自動傳播至訂用帳戶中的所有資源群組，如圖中的右側所示：


### <a name="data-collection"></a>資料收集

資訊安全中心會收集虛擬機器 (VM) 的資料，以便評估其安全性狀態、提供安全性建議，並對您發出威脅警示。 當您第一次存取資訊安全中心時，訂用帳戶中的所有 VM 都會啟用資料收集。 建議啟用資料收集，但您可以在資訊安全中心原則中關閉資料收集來選擇退出。

### <a name="data-sources"></a>資料來源

- Azure 資訊安全中心會分析來自下列來源的資料，以掌握您的安全性狀態、識別弱點並提供建議防護功能，並偵測作用中的威脅︰

-   Azure 服務︰與 Azure 服務的資源提供者通訊，以使用您所部署之 Azure 服務的組態相關資訊。

- 網路流量︰使用從 Microsoft 的基礎結構取樣的網路流量中繼資料，例如來源/目的地 IP/連接埠、封包大小和網路通訊協定。

-   合作夥伴解決方案：使用整合式合作夥伴解決方案 (例如防火牆和反惡意程式碼解決方案) 的安全性警示。

-   您的虛擬機器：使用組態資訊以及安全性事件的相關資訊，例如來自您虛擬機器的 Windows 事件和稽核記錄檔、IIS 記錄檔、syslog 訊息和損毀傾印檔。

### <a name="data-protection"></a>資料保護

為了協助客戶防範、偵測和回應威脅，Azure 資訊安全中心會收集和處理安全性的相關資料，包括組態資訊、中繼資料、事件記錄檔、損毀傾印檔等等。 Microsoft 從撰寫程式碼到運作服務均遵守嚴格的規範與安全性指導方針。

-   **資料隔離**：資料會以邏輯方式分開保存在服務的每個元件上。 每個組織加上標記的所有資料。 這項標記作業在整個資料生命週期持續發生，它會強制執行服務的每個層級。

-   **資料存取**︰為了提供安全性建議和調查潛在的安全性威脅，Microsoft 人員可能會存取 Azure 服務所收集或分析的資訊，包含損毀傾印檔案、處理建立事件、VM 磁碟快照集和構件，這些資訊可能會在無意中包含客戶資料或您虛擬機器中的個人資料。 我們會遵守 [Microsoft Online Services Terms 和隱私權聲明 (英文)](http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31)，其中陳述 Microsoft 不會使用客戶資料或從中衍生資訊作為任何廣告或類似的商業用途。

-   **資料使用**：Microsoft 使用可見於多個租用戶的模式和威脅智慧來加強我們的防護和偵測功能；我們會根據[隱私權聲明](https://www.microsoft.com/en-us/privacystatement/OnlineServices/)中所述的隱私權承諾進行。

### <a name="data-location"></a>資料位置

Azure 資訊安全中心會收集損毀傾印檔案的暫時複本並加以分析，以取得惡意探索嘗試和成功入侵的證明。 Azure 資訊安全中心會在與工作區相同的地理區域內執行這項分析，並且在分析完成時刪除暫複本。 機器構件會集中儲存在與 VM 相同的區域。

-   **儲存體帳戶**︰針對執行虛擬機器的每個區域指定儲存體帳戶。 這可讓您將資料儲存在與從中收集資料的虛擬機器相同的區域中。

-   **Azure 資訊安全中心儲存體**：安全性警示的相關資訊 (包含合作夥伴警示、建議和安全性健康狀態) 會集中儲存 (目前位於美國)。 此資訊可能包括視需要從您的虛擬機器收集的相關組態資訊和安全性事件，以提供給您安全性警示、建議或安全性健康狀態。


## <a name="azure-monitor"></a>Azure 監視器

[OMS 安全性](https://docs.microsoft.com/azure/operations-management-suite/oms-security-monitoring-resources)和稽核解決方案可讓 IT 人員主動監視所有資源，此舉有助於將安全性事件的影響降至最低。 OMS 安全性和稽核具備可用來監視資源的安全性網域。 安全性網域提供特定選項的快速存取方式，並基於安全性監視目的，更詳盡地涵蓋下列網域：

-   惡意程式碼評估
-   更新評估
-   身分識別與存取。

Azure 監視器提供特定資源類型詳細資訊的指標。 它會針對來自 Azure 基礎結構 (活動記錄) 及每個個別的 Azure 資源 (診斷記錄) 的資料，提供視覺效果、查詢、路由、警示、自動調整及自動化功能。

![Azure 監視器](./media/azure-operational-security/azure-operational-security-fig6.png)


雲端應用程式相當複雜，且具有許多移動組件。 監視會提供資料，以確保應用程式持續運作並以健全的狀態執行。 它也可協助您預防潛在問題，或是針對過去所發生的問題進行疑難排解。

除此之外，您還可以使用監視資料來取得應用程式的深入解析。 這些知識可協助您提升應用程式效能或維護性，或是將原本需要手動介入的動作自動化。

### <a name="azure-activity-log"></a>Azure 活動記錄檔


它是一個記錄，能讓您了解訂用帳戶中的資源所執行之作業。 活動記錄檔之前叫做「稽核記錄」或「作業記錄」，因為它會報告訂用帳戶中控制層面的事件。

![Azure 活動記錄檔](./media/azure-operational-security/azure-operational-security-fig7.png)

您可以使用活動記錄檔來判斷訂用帳戶中的資源上任何寫入作業 (PUT、POST、DELETE) 的「內容、對象和時間」。 您也可以了解作業的狀態和其他相關屬性。 活動記錄不包含讀取 (GET) 作業，或是使用傳統模型之資源的作業。

### <a name="azure-diagnostic-logs"></a>Azure 診斷記錄

這些記錄是由資源發出的，提供有關該資源之作業的豐富、經常性資料。 這些記錄的內容會依資源類型而有所不同。

例如，Windows 事件系統記錄是適用於 VM、Blob 和資料表的一個診斷記錄類別，而佇列記錄則是適用於儲存體帳戶的診斷記錄類別。

診斷記錄不同於[活動記錄 (之前稱為稽核記錄或作業記錄)](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)。 活動記錄能讓您了解訂用帳戶中的資源所執行之作業。 診斷記錄能讓您了解資源自行執行的作業。

### <a name="metrics"></a>度量

Azure 監視器可讓您取用遙測來查看您 Azure 工作負載的效能與健全狀況。 Azure 遙測資料最重要的類型是由大多數 Azure 資源所發出的度量 (也稱為效能計數器)。 Azure 監視器提供數種方式，來設定及取用這些[計量](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics)以進行監視與疑難排解。 度量是遙測的寶貴來源，可讓您執行下列工作：

-   **追蹤資源** (例如 VM、網站或邏輯應用程式) 的效能，方法是在入口網站圖表上繪製其度量，並將該圖表釘選到儀表板。

-   **取得問題的通知**，該問題在度量超出特定的閾值時會影響資源的效能。

-   **設定自動化動作**，例如自動調整資源，或在計量超出特定閾值時觸發 Runbook。

-   **執行進階分析**或報告您資源的效能或使用量趨勢。

-   **封存**您資源的效能或健全狀況歷程記錄，以用於合規性或稽核用途。

### <a name="azure-diagnostics"></a>Azure 診斷

它是 Azure 中可對部署的應用程式啟用診斷資料收集的功能。 您可以使用來自各種不同來源的診斷延伸模組。 目前支援 [Azure 雲端服務 Web 和背景工作角色](https://docs.microsoft.com/azure/vs-azure-tools-configure-roles-for-cloud-service)、執行 Microsoft Windows 的 [Azure 虛擬機器](https://docs.microsoft.com/azure/virtual-machines/windows/overview)，以及 [Service Fabric](https://docs.microsoft.com/azure/monitoring-and-diagnostics/azure-diagnostics)。 其他 Azure 服務都有自己個別的診斷。

## <a name="azure-network-watcher"></a>Azure 網路監看員

在偵測網路漏洞，並確認您的 IT 安全性和法規治理模型的合規性時，稽核您的網路安全性非常重要。 透過 [安全性群組] 檢視，您可以擷取已設定的網路安全性群組和安全性規則，以及有效的安全性規則。 在套用的規則清單中，您可以決定已開啟的連接埠，並評估網路弱點。

[網路監看員](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview#network-watcher)是一項區域性服務，可讓您監視與診斷位於和進出 Azure 的網路層級條件。 網路監看員提供的網路診斷和視覺效果工具，可幫助您了解、診斷及洞悉您在 Azure 中的網路。 這項服務包括封包擷取、下一個躍點、IP 流量驗證、安全性群組檢視、NSG 流量記錄。 案例層級監視可提供端對端的網路資源檢視，而非個別的網路資源監視。

![Azure 網路監看員](./media/azure-operational-security/azure-operational-security-fig8.png)

網路監看員目前具有下列功能︰

-   **<a href="https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview">稽核記錄</a>**：會記錄做為網路組態一部分所執行的作業。 這些記錄可在 Azure 入口網站中檢視，或使用 Power BI 之類的 Microsoft 工具或協力廠商工具來擷取。 稽核記錄可透過入口網站、PowerShell、CLI 和 Rest API 來取得。 如需稽核記錄的詳細資訊，請參閱＜使用 Resource Manager 來稽核作業＞。 針對所有網路資源所進行的作業都會有稽核記錄。


-   **<a href="https://docs.microsoft.com/azure/network-watcher/network-watcher-ip-flow-verify-overview">IP 流程驗證</a>**：根據流程資訊 5 個 Tuple 封包參數 (目的地 IP、來源 IP、目的地連接埠、來源連接埠和通訊協定) 檢查是否允許或拒絕封包。 如果封包遭到網路安全性群組拒絕，則會傳回拒絕封包的規則和網路安全性群組。

-   **<a href="https://docs.microsoft.com/azure/network-watcher/network-watcher-next-hop-overview">下一個躍點</a>** - 決定在 Azure 網路網狀架構中路由傳送之封包的下一個躍點，讓您得以診斷任何設定錯誤的使用者定義路由。

-   **<a href="https://docs.microsoft.com/azure/network-watcher/network-watcher-security-group-view-overview">安全性群組檢視</a>** - 取得套用至 VM 的有效和已套用安全性規則。

-   **<a href="https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-overview">NSG 流量記錄</a>** - 網路安全性群組的流量記錄可讓您擷取群組中的安全性規則所允許或拒絕之流量的相關記錄。 流程是由 5個 Tuple 資訊定義的，這些資訊分別是來源 IP、目的地 IP、來源連接埠、目的地連接埠和通訊協定。

## <a name="azure-storage-analytics"></a>Azure 儲存體分析

[儲存體分析](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics)可以儲存包含與儲存體服務要求相關之彙總的交易統計資料及容量資料的計量。 報告的交易是屬於 API 作業層級和儲存體服務層級，而報告的容量則是屬於儲存體服務層級。 度量資料可用來分析儲存體服務使用量、診斷針對儲存體服務提出之要求的問題，以及提升使用服務的應用程式效能。

[Azure 儲存體分析](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics)會執行記錄，並提供儲存體帳戶的計量資料。 您可以使用此資料來追蹤要求、分析使用量趨勢，以及診斷儲存體帳戶的問題。 儲存體分析記錄適用於 [Blob、佇列及表格服務](https://docs.microsoft.com/azure/storage/storage-introduction)。 儲存體分析會記錄對儲存體服務之成功和失敗要求的詳細資訊。

這項資訊可用來監視個別要求，並診斷儲存體服務的問題。 系統會以最佳方式來記錄要求。 只有在對服務端點提出要求時，才會建立記錄項目。 例如，如果儲存體帳戶在其 Blob 端點中有活動，而不是在其表格或佇列端點中，則只會建立關於 Blob 服務的記錄。

若要使用儲存體分析，您必須針對想要監視的每個服務個別啟用它。 您可以在 [Azure 入口網站](https://portal.azure.com/)中啟用它；如需詳細資訊，請參閱[在 Azure 入口網站中監視儲存體帳戶](https://docs.microsoft.com/azure/storage/storage-monitor-storage-account)。 您也可以利用程式設計方式，透過 REST API 或用戶端程式庫來啟用儲存體分析。 使用 [設定服務屬性] 作業，分別為各個服務啟用儲存體分析。

彙總的資料會儲存於已知的 Blob (用於記錄) 和已知的資料表 (用於度量) 中，您可以使用 Blob 服務和資料表服務 API 來存取。

儲存體分析在儲存的資料量上有 20 TB 的限制，但此限制與儲存體帳戶的總限制無關。 所有記錄都會儲存在名為 $logs 之容器內的[區塊 Blob](https://docs.microsoft.com/azure/storage/storage-analytics) 中，該容器是在針對儲存體帳戶啟用儲存體分析時自動建立的。

儲存體分析所執行的下列動作會列入計費：

-   建立 Blob 以用於記錄的要求
-   針對度量建立資料表實體的要求。

> [!Note]
> 如需計費和資料保留原則的詳細資訊，請參閱[儲存體分析和計費](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics-and-billing)。
> 為達最佳效能，您需要限制連接至虛擬機器之高度使用的磁碟數目，以避免可能的節流。 只要不是所有磁碟都會在同一時間高度使用，儲存體帳戶就能支援更大的磁碟數目。

> [!Note]
> 如需儲存體帳戶限制的詳細資訊，請參閱 [Azure 儲存體延展性和效能目標](https://docs.microsoft.com/azure/storage/storage-scalability-targets)。


系統會記錄下列類型的已驗證與匿名要求。

| 已驗證  | 匿名|
| :------------- | :-------------|
| 成功的要求 | 成功的要求 |
|失敗的要求，包括逾時、節流、網路、授權和其他錯誤 | 使用共用存取簽章 (SAS) 的要求，包括失敗和成功的要求 |
| 使用共用存取簽章 (SAS) 的要求，包括失敗和成功的要求 |用戶端與伺服器的逾時錯誤 |
|   分析資料的要求 |    失敗的 GET 要求，錯誤碼為 304 (未修改) |
| 系統不會記錄儲存體分析本身所提出的要求 (例如，記錄檔的建立或刪除)。 記錄資料的完整清單記錄於[儲存體分析記錄作業和狀態訊息](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics-logged-operations-and-status-messages)及[儲存體分析記錄檔格式](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics-log-format)主題中。 | 系統不會記錄所有其他失敗的匿名要求。 記錄資料的完整清單記錄於[儲存體分析記錄作業和狀態訊息](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics-logged-operations-and-status-messages)及[儲存體分析記錄格式](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics-log-format)中。 |
## <a name="azure-active-directory"></a>Azure Active Directory

Azure AD 也包含一組完整的身分識別管理功能，包括多重要素驗證、裝置註冊、自助式密碼管理、自助式群組管理、特殊權限的帳戶管理、角色型存取控制、應用程式使用量監視、豐富的稽核，以及安全性監視和警示。

-   利用 Azure AD 多重要素驗證和條件式存取，改善應用程式的安全性。

-   利用安全性報告和監視，監視應用程式使用量並保護您的企業免於受到嚴重的威脅。

Azure Active Directory (Azure AD) 包括您的目錄的安全性、活動和稽核報告。 [Azure Active Directory 稽核報告](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-guide)可協助客戶識別其 Azure Active Directory 中發生的特殊權限動作。 特殊權限動作包括提高權限變更 (例如，角色建立或密碼重設)、原則設定變更 (例如密碼原則) 或目錄設定變更 (例如，網域同盟設定變更)。

報告會提供的稽核記錄包括事件名稱、執行動作的執行者、受變更影響的目標資源，以及日期和時間 (UTC)。 客戶能透過 [Azure 入口網站](https://portal.azure.com/)擷取其 Azure Active Directory 的稽核事件清單，如[檢視您的稽核記錄](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-azure-portal)中所述。 以下是包含的報告清單：

| 安全性報告  | 活動報告| 稽核報告 |
| :------------- | :-------------| :-------------|
|從不明來源登入 | 應用程式使用情況：摘要 | 目錄稽核報告 |
|在多次失敗後登入 | 應用程式使用情況：詳細 |   |
|從多個地理區域登入 | 應用程式儀表板 |  |
|從具有可疑活動的 IP 位址登入 |帳戶佈建錯誤 |  |
|異常的登入活動 |個別使用者裝置 |  |
|從可能受感染的裝置登入 |個別使用者活動 |   |
|具有異常登入活動的使用者 |群組活動報告 |   |
| |密碼重設登錄活動報告 |   |
| |密碼重設活動 |   | |



這些報告的資料對您的應用程式 (例如 SIEM 系統、稽核和商業智慧工具) 非常有用。 Azure AD 報告 [API](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-api-getting-started) 透過一組以 REST 為基礎的 API 提供資料的程式設計方式存取。 您可以從各種程式設計語言和工具呼叫這些 API。

Azure AD 稽核報告中的事件會保留 180 天。

> [!Note]
> 如需保留報告的詳細資訊，請參閱 [Azure Active Directory 報告保留原則](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-retention)。

若是對於儲存其[稽核事件](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-audit-events)較長的保留時間感興趣的客戶，報告 API 可用來定期將稽核事件提取至不同的資料存放區。

## <a name="summary"></a>總結

本文摘要說明保護您的隱私權和保護資料安全，同時提供軟體和服務，幫助您管理組織的 IT 基礎結構。 Microsoft 了解當他們將資料委託給他人管理時，該信任需要嚴格的安全性。 Microsoft 從撰寫程式碼到運作服務均遵守嚴格的規範與安全性指導方針。 保全和保護資料在 Microsoft 是第一要務。

本文說明

-   Operations Management Suite (OMS) 中收集、處理和保護資料的方式。

-   快速分析多個資料來源中的事件。 識別安全性風險，並了解威脅和攻擊的範圍和影響，以減輕安全性缺口的損害。

-   藉由將輸出的惡意 IP 流量和惡意威脅類型視覺化，進而識別攻擊模式。 了解您整個環境的安全性情勢，無論平台為何皆可。

-   擷取安全性或合規性稽核所需的所有記錄和事件資料。 透過完整、可搜尋且可匯出的記錄與事件資料集，來降低提供安全性稽核所需的時間和資源。

<ul>
<li>收集安全性相關的事件、稽核及缺口分析，以密切注意您的資產：</li>
<ul>
<li>安全性狀態</li>
<li>值得注意的問題</li>
<li>摘要說明威脅</li>
</ul>
</ul>

## <a name="next-steps"></a>後續步驟

- [設計與作業安全性 (英文)](https://www.microsoft.com/trustcenter/security/designopsecurity)

Microsoft 在設計其服務與軟體時已將安全性納入考量，以協助確保其雲端基礎結構有彈性且可抵禦攻擊。

- [Operations Management Suite | 安全性與合規性](https://www.microsoft.com/cloud-platform/security-and-compliance)

使用 Microsoft 安全性資料和分析，來執行更聰明且有效的威脅偵測。

- [Azure 資訊安全中心規劃和操作](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide)：一組步驟和工作，讓您能夠遵循以根據組織的安全性需求和雲端管理模型，將您的資訊安全中心使用最佳化。

