---
title: Azure 安全性簡介 | Microsoft Docs
description: 了解 Azure 安全性、它的服務及運作方式。
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
ms.openlocfilehash: 81eb23a0955bbc6a5f9dcd95c6874f0c5a8c286c
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434023"
---
# <a name="introduction-to-azure-security"></a>Azure 安全性簡介
## <a name="overview"></a>概觀
我們知道安全性是雲端中的首要工作和其重要性，因為您可在其中找到精確且及時的 Azure 安全性資訊。 針對您的應用程式和服務使用 Azure 的最佳原因之一是可以利用它的各種安全性工具和功能。 這些工具和功能可協助您在安全的 Azure 平台上建立安全的解決方案。 Microsoft Azure 提供客戶資料的機密性、完整性和可用性，同時也能釐清責任。

為了協助您從客戶和 Microsoft 作業的觀點深入了解如何在 Microsoft Azure 內實作一組安全性控制，因而編寫了本技術白皮書＜Azure 安全性簡介＞來提供可透過 Microsoft Azure 取得之安全性的完整介紹。

### <a name="azure-platform"></a>Azure 平台
Azure 是一個公用雲端服務平台，支援廣泛的作業系統、程式設計語言、架構、工具、資料庫及裝置等選擇。 它可以透過 Docker 整合執行 Linux 容器；使用 JavaScript、Python、.NET、PHP、Java 及 Node.js 建置應用程式；為 iOS、Android 及 Windows 裝置建置後端。

Azure 公用雲端服務支援數百萬名開發人員和 IT 專家早已仰賴和信任的相同技術。 當您建置 IT 資產或將其移轉至公用雲端服務提供者時，您正是依賴該組織的能力，利用他們提供來管理您雲端式資產安全性的服務與控制機制，來保護您的應用程式和資料。

Azure 的基礎結構設計涵蓋設備與應用程式，可同時裝載數以百萬計的客戶，並提供值得信任的基礎以使企業可符合其安全性需求。

此外，Azure 也為您提供各式各樣可設定的安全性選項及控制它們的功能，讓您能夠自訂安全性以符合組織部署的特殊需求。 此文件有助於您了解 Azure 安全性功能如何協助您滿足這些需求。

> [!Note]
> 此文件的主要焦點是客戶面對的控制，您可以使用這些控制來自訂並提升應用程式和服務的安全性。
>
> 我們會提供一些概觀資訊，但如需 Microsoft 如何保護 Azure 平台本身的詳細資訊，請參閱 [Microsoft 信任中心 (英文)](https://www.microsoft.com/TrustCenter/default.aspx) 中提供的資訊。

### <a name="abstract"></a>摘要
最初是基於節省成本與靈活運用創新而衍生出公用雲端移轉。 經過一段時間之後，才將安全性視為公用雲端移轉的主要考量，甚至是個關鍵因素。 不過，公用雲端安全性已從主要考量轉換為雲端移轉的驅策因素之一。 這背後的原理是大型公用雲端服務提供者的卓越能力，可保護雲端式資產的應用程式和資料。

Azure 的基礎結構設計涵蓋設備與應用程式，可同時裝載數以百萬計的客戶，並提供可靠的基礎供企業符合其安全性需求。 此外，Azure 也為您提供各式各樣可設定的安全性選項及控制它們的功能，讓您能夠自訂安全性來符合部署的特殊需求，以符合 IT 控制原則並遵循外部法規。

本白皮書概述 Microsoft 處理 Microsoft Azure 雲端平台安全性的方法：
* Microsoft 實作來保護 Azure 基礎結構、客戶資料和應用程式的安全性功能。
* Azure 服務和安全性功能，可讓您用來管理服務及 Azure 訂用帳戶中資料的安全性。

## <a name="summary-azure-security-capabilities"></a>Azure 安全性功能摘要
下表提供 Microsoft 實作來保護 Azure 基礎結構、客戶資料和安全應用程式之安全性功能的簡短描述。
### <a name="security-features-implemented-to-secure-the-azure-platform"></a>實作來保護 Azure 平台的安全性功能：
您可以檢閱下列功能，以保證會透過安全的方式來管理 Azure 平台。 其中提供了連結，讓您可以進一步深入了解 Microsoft 如何在下列四個領域中處理客戶信任問題：安全的平台、隱私權與控制、合規性及透明度。


| [安全的平台](https://www.microsoft.com/en-us/trustcenter/Security/default.aspx)  | [隱私權與控制](https://www.microsoft.com/en-us/trustcenter/Privacy/default.aspx)  |[合規性](https://www.microsoft.com/en-us/trustcenter/Compliance/default.aspx)   | [透明度](https://www.microsoft.com/en-us/trustcenter/Transparency/default.aspx) |
| :-- | :-- | :-- | :-- |
| [安全性開發週期 (英文)](https://www.microsoft.com/en-us/sdl/)、內建稽核 | [隨時管理您的資料 (英文)](https://www.microsoft.com/en-us/trustcenter/Privacy/You-own-your-data) | [信任中心](https://www.microsoft.com/en-us/trustcenter/default.aspx) |[Microsoft 如何保護 Azure 服務中的客戶資料 (英文)](https://www.microsoft.com/en-us/trustcenter/Transparency/default.aspx) |
| [必要的安全性訓練、背景檢查 (英文)](https://downloads.cloudsecurityalliance.org/star/self-assessment/StandardResponsetoRequestforInformationWindowsAzureSecurityPrivacy.docx) |  [資料位置控制 (英文)](https://www.microsoft.com/en-us/trustcenter/Privacy/Where-your-data-is-located) |  [Common Controls Hub (英文)](https://www.microsoft.com/en-us/trustcenter/Common-Controls-Hub) |[Microsoft 如何管理 Azure 服務中的資料位置 (英文)](http://azuredatacentermap.azurewebsites.net/)|
| [滲透測試 (英文)](https://downloads.cloudsecurityalliance.org/star/self-assessment/StandardResponsetoRequestforInformationWindowsAzureSecurityPrivacy.docx)、[入侵偵測、DDoS (英文)](https://www.microsoft.com/en-us/trustcenter/Security/ThreatManagement)、[稽核和記錄(英文)](https://www.microsoft.com/en-us/trustcenter/Security/AuditingAndLogging) | [根據您的條款提供資料存取 (英文)](https://www.microsoft.com/en-us/trustcenter/Privacy/Who-can-access-your-data-and-on-what-terms) |  [雲端服務審查評鑑檢查表 (英文)](https://www.microsoft.com/en-us/trustcenter/Compliance/Due-Diligence-Checklist) |[Microsoft 內部的哪些人員可根據哪些條款存取您的資料 (英文)](https://www.microsoft.com/en-us/trustcenter/Privacy/Who-can-access-your-data-and-on-what-terms)|
| [技術資料中心的狀態 (英文)](https://www.microsoft.com/en-us/cloud-platform/global-datacenters)、實體安全性、[安全網路](https://docs.microsoft.com/azure/security/security-network-overview) | [回應執法機關 (英文)](https://www.microsoft.com/en-us/trustcenter/Privacy/Responding-to-govt-agency-requests-for-customer-data) |  [依服務、位置和產業的合規性 (英文)](https://www.microsoft.com/en-us/trustcenter/Compliance/default.aspx) |[Microsoft 如何保護 Azure 服務中的客戶資料 (英文)](https://www.microsoft.com/en-us/trustcenter/Transparency/default.aspx)|
|  [安全性事件回應 (英文)](http://aka.ms/SecurityResponsepaper)[共同責任 (英文)](http://aka.ms/sharedresponsibility) |[嚴格的隱私權標準 (英文)](https://www.microsoft.com/en-us/TrustCenter/Privacy/We-set-and-adhere-to-stringent-standards) |  | [檢閱 Azure 服務的憑證、透明度中樞 (英文)](https://www.microsoft.com/en-us/trustcenter/Compliance/default.aspx)|



### <a name="security-features-offered-by-azure-to-secure-data-and-application"></a>Azure 提供來保護資料和應用程式的安全性功能
負責管理應用程式或服務安全性之人員應負的責任會根據雲端服務模型而不同。 Azure 平台中提供一些功能，可協助您透過內建功能，以及透過可部署到 Azure 訂用帳戶的協力廠商解決方案，來達成這些職責。

內建功能分成六 (6) 個功能領域︰作業、應用程式、儲存體、網路功能、計算及身分識別。 在這六 (6) 個領域中，適用於 Azure 平台之特性與功能的其他詳細資料，會透過摘要資訊來提供。

## <a name="operations"></a>作業
本節提供關於安全性作業中主要功能的其他資訊，以及這些功能的摘要資訊。

### <a name="operations-management-suite-security-and-audit-dashboard"></a>Operations Management Suite 安全性與稽核儀表板
[OMS 安全性和稽核解決方案](https://docs.microsoft.com/azure/operations-management-suite/oms-security-getting-started)針對值得您注意的問題，使用[內建的搜尋查詢 (英文)](https://blogs.technet.microsoft.com/msoms/2016/01/21/easy-microsoft-operations-management-suite-search-queries/)，為您組織的 IT 安全性狀況提供全面性檢視。 [安全性和稽核](https://technet.microsoft.com/library/mt484091.aspx) 儀表板是 OMS 中所有安全性相關項目的主畫面。 它可讓您深入了解電腦的安全性狀態。 它還能夠檢視過去 24 小時、7 天或任何其他自訂時間範圍內的所有事件。

此外，您還能設定 OMS 安全性與合規性，以便在偵測到特定事件時[自動執行特定動作 (英文)](https://blogs.technet.microsoft.com/robdavies/2016/04/20/simple-look-at-oms-alert-remediation-with-runbooks-part-1/)。

### <a name="azure-resource-manager"></a>Azure Resource Manager
[Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-deployment-model) 可讓您以群組方式使用解決方案中的資源。 您可以透過單一、協調的作業來部署、更新或刪除方案的所有資源。 您會使用 [Azure Resource Manager 範本 (英文)](https://blogs.technet.microsoft.com/canitpro/2015/06/29/devops-basics-infrastructure-as-code-arm-templates/) 部署，該範本可用於不同的環境，例如測試、預備和生產環境。 Resource Manager 會提供安全性、稽核和標記功能，以協助您在部署後管理您的資源。

以 Azure Resource Manager 範本為基礎的部署，有助於提升部署於 Azure 中之解決方案的安全性 (因為標準的安全性控制設定)，並且可整合至以標準化範本為基礎的部署中。 這會降低可能需要在手動部署期間執行的安全性設定錯誤風險。

### <a name="application-insights"></a>Application Insights
[Application Insights](https://docs.microsoft.com/azure/application-insights/) 是適用於 Web 開發人員的可延伸「應用程式效能管理」(APM) 服務。 使用 Application Insights，您可以即時監視 Web 應用程式，並自動偵測效能異常。 其中包括強大的分析工具可協助您診斷問題，並了解使用者實際如何運用您的應用程式。 它會在您的應用程式執行時全程監視，包括測試期間，以及您發佈或部署之後。

Application Insights 會建立圖表和資料表為您顯示多種資訊，例如，您在一天中的哪些時間有最多使用者、應用程式的回應性如何，以及它所依存的任何外部服務是否順暢地為其提供服務。

如果有當機、失敗或效能問題，您可以搜尋詳細的遙測資料，以診斷原因。 此外，如果應用程式的可用性和效能有任何變更，服務會傳送電子郵件給您。 Application Insight 因而成為一個非常實用的安全性工具，因為它有助於提供機密性、完整性和可用性安全性三部曲中的「可用性」。

### <a name="azure-monitor"></a>Azure 監視器
[Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/)針對來自 Azure 基礎結構 ([活動記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)) 及每個個別的 Azure 資源 ([診斷記錄](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)) 的資料，提供視覺效果、查詢、路由、警示、自動調整及自動化功能。 您可以使用 Azure 監視器，在 Azure 記錄中產生安全性相關事件時接收警示通知。

### <a name="log-analytics"></a>Log Analytics
[Operations Management Suite](https://www.microsoft.com/cloud-platform/operations-management-suite) 中的 [Log Analytics](https://azure.microsoft.com/documentation/services/log-analytics/)：為內部部署和協力廠商雲端式基礎結構 (如 AWS) 提供 Azure 資源之外的 IT 管理解決方案。 Azure 監視器中的資料可以直接路由至 Log Analytics，以便您可以在同一個地方看到整個環境的度量與記錄。

Log Analytics 在鑑識和其他安全性分析方面是一個非常實用的工具，因為此工具可讓您利用彈性查詢方法快速搜尋大量的安全性相關項目。 此外，內部部署的[防火牆和 Proxy 記錄可匯出到 Azure，並使用 Log Analytics 來分析它們。](https://docs.microsoft.com/azure/log-analytics/log-analytics-proxy-firewall)

### <a name="azure-advisor"></a>Azure Advisor
[Azure Advisor](https://docs.microsoft.com/azure/advisor/) 是個人化雲端顧問，可協助您將 Azure 部署最佳化。 它會分析您的資源及用量遙測， 接著會建議解決方案，以協助改善資源的[效能](https://docs.microsoft.com/azure/advisor/advisor-performance-recommendations)、[安全性](https://docs.microsoft.com/azure/advisor/advisor-security-recommendations)及[高可用性](https://docs.microsoft.com/azure/advisor/advisor-high-availability-recommendations)，同時尋找[降低整體 Azure 費用](https://docs.microsoft.com/azure/advisor/advisor-cost-recommendations)的機會。 Azure Advisor 提供安全性建議，讓您能夠大幅改善您部署於 Azure 中之解決方案的整體安全性狀態。 這些建議均取自 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)所執行的安全性分析。

### <a name="azure-security-center"></a>Azure 資訊安全中心
[Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)利用加強對您 Azure 資源的能見度及安全性控制權，來協助您預防、偵測及回應威脅。 它提供您 Azure 訂用帳戶之間的整合式安全性監視和原則管理，協助您偵測可能會忽略的威脅，且適用於廣泛的安全性解決方案生態系統。

此外，Azure 資訊安全中心可協助進行安全性作業，方法是提供單一儀表板來顯示可立即採取行動的警示和建議。 通常，您可以在 Azure 資訊安全中心主控台內透過按一下的方式來補救問題。
## <a name="applications"></a>[應用程式]
本節提供關於應用程式安全性中主要功能的其他資訊，以及這些功能的摘要資訊。

### <a name="web-application-vulnerability-scanning"></a>Web 應用程式弱點掃描
開始測試您 [App Service 應用程式](https://docs.microsoft.com/azure/app-service/app-service-web-overview)上是否有弱點的最簡單方式之一是使用[與 Tinfoil Security 的整合 (英文)](https://azure.microsoft.com/blog/web-vulnerability-scanning-for-azure-app-service-powered-by-tinfoil-security/)，以在您的應用程式上執行單鍵弱點掃描。 您可以在容易了解的報告中檢視測試結果，並且了解如何以逐步指示修正每個弱點。

### <a name="penetration-testing"></a>滲透測試
如果您想要執行專屬的滲透測試，或想要使用另一個掃描器套件或提供者，您必須依照 [Azure 滲透測試核准程序](https://docs.microsoft.com/azure/security/azure-security-pen-testing ) 並取得先前核准，才能執行所需的滲透測試。

### <a name="web-application-firewall"></a>Web 應用程式防火牆
[Azure 應用程式閘道](https://azure.microsoft.com/services/application-gateway/)中的 Web 應用程式防火牆 (WAF) 可協助保護 Web 應用程式，以免於常見的 Web 型攻擊，例如 SQL 插入式攻擊、跨網站指令碼攻擊和工作階段攔截。 其已預先設定 [Open Web Application Security Project (OWASP) 認定為前 10 大常見漏洞](https://msdn.microsoft.com/library/)的威脅防護。

### <a name="authentication-and-authorization-in-azure-app-service"></a>Azure App Service 中的驗證與授權
[App Service 驗證/授權](https://docs.microsoft.com/azure/app-service/app-service-authentication-overview)是可讓應用程式接受使用者登入的一種功能，而不需要您在應用程式後端變更程式碼。 它提供簡單的方法來保護您的應用程式，以及使用每位使用者的資料。

### <a name="layered-security-architecture"></a>分層式安全性架構
由於 [App Service 環境](https://docs.microsoft.com/azure/app-service/environment/app-service-app-service-environment-intro)提供部署至 [Azure 虛擬網路](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)的隔離執行階段環境，因此開發人員能夠建立分層式安全性架構，針對每個應用程式層提供不同層級的網路存取。 常見的需求之一，是要隱藏對 API 後端的一般網際網路存取，而只允許由上游 Web 應用程式呼叫 API。 [網路安全性群組 (NSG)](https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/) 可用於包含 App Service 環境的 Azure 虛擬網路子網路，以限制對 API 應用程式的公用存取。

### <a name="web-server-diagnostics-and-application-diagnostics"></a>Web 伺服器診斷和應用程式診斷
App Service Web 應用程式會針對來自 Web 伺服器和 Web 應用程式的記錄資訊提供診斷功能。 這些資訊邏輯上可區分為 [Web 伺服器診斷](https://docs.microsoft.com/azure/app-service/web-sites-enable-diagnostic-log)與[應用程式診斷](https://technet.microsoft.com/library/hh530058(v=sc.12).aspx)。 Web 伺服器在針對網站和應用程式進行診斷及疑難排解方面包含了兩個重大進展。

第一個新功能是關於應用程式集區、背景工作處理序、網站、應用程式定義域和執行中要求的即時狀態資訊。 第二個新優點是詳細的追蹤事件，可在整個完成要求與回應程序中追蹤要求。

若要能夠收集這些追蹤事件，您可以設定 IIS 7，針對任何以經過時間或錯誤回應碼為基礎的特定要求，自動擷取完整的追蹤記錄 (XML 格式)。

#### <a name="web-server-diagnostics"></a>Web 伺服器診斷
您可以啟用或停用下列各種記錄：

-   詳細的錯誤記錄：對於表示失敗的 HTTP 狀態碼 (狀態碼 400 或更大) 的詳細錯誤資訊。 這當中包含的資訊可協助您判斷為何伺服器傳回錯誤碼。

-   失敗要求的追蹤：關於失敗要求的詳細資訊，包括追蹤用來處理要求的 IIS 元件及每個元件所花費的時間。 如果您嘗試提升網站效能或是想要從傳回的特定 HTTP 錯誤中找到發生原因，這個方法非常實用。

-   Web 伺服器記錄：使用 W3C 擴充記錄檔格式的 HTTP 交易相關資訊。 當您需要判斷整體網站指標 (例如，處理的要求數量，或者有多少要求來自特定的 IP 位址) 時，這非常實用。

#### <a name="application-diagnostics"></a>應用程式診斷
[應用程式診斷](https://docs.microsoft.com/azure/app-service/web-sites-enable-diagnostic-log)可讓您擷取 Web 應用程式所產生的資訊。 ASP.NET 應用程式會使用 [System.Diagnostics.Trace](https://msdn.microsoft.com/library/system.diagnostics.trace) 類別將資訊記錄到應用程式診斷記錄。 在應用程式診斷中，有兩種主要的事件類型：與應用程式效能相關的類型，以及與應用程式失敗和錯誤相關的類型。 失敗和錯誤可進一步細分為連線、安全性和失敗問題。 失敗問題通常與應用程式程式碼的問題有關。

在應用程式診斷中，您可以檢視以下列方式分組的事件：

-   全部 (顯示所有事件)
-   應用程式錯誤 (顯示例外狀況事件)
-   效能 (顯示效能事件)

## <a name="storage"></a>儲存體
本節提供關於 Azure 儲存體安全性中主要功能的其他資訊，以及這些功能的摘要資訊。

### <a name="role-based-access-control-rbac"></a>角色型存取控制 (RBAC)
您可以使用角色型存取控制 (RBAC) 來保護儲存體帳戶。 對於想要強制執行資料存取安全性原則的組織，根據[需要知道 (英文)](https://en.wikipedia.org/wiki/Need_to_know) 和[最低權限 (英文)](https://en.wikipedia.org/wiki/Principle_of_least_privilege) 安全性主體限制存取權限是必須做的事。 在特定範圍將適當的 RBAC 角色指派給群組和應用程式，即可授與這些存取權限。 您可以使用 [內建的 RBAC 角色](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)(例如儲存體帳戶參與者) 將權限指派給使用者。 使用 [Azure Resource Manager](https://docs.microsoft.com/azure/storage/storage-security-guide) 模型來存取儲存體帳戶的儲存體金鑰，可以透過角色型存取控制 (RBAC) 來控制。

### <a name="shared-access-signature"></a>共用存取簽章
[共用存取簽章 (SAS)](https://docs.microsoft.com/azure/storage/storage-dotnet-shared-access-signature-part-1) 可提供您儲存體帳戶中資源的委派存取。 SAS 意謂著您可以將儲存體帳戶中物件的有限權限授與用戶端，讓該用戶端可以在一段指定期間內使用一組指定的權限進行存取。 您可以在不須分享您帳戶存取金鑰的情況下，授與這些有限的權限。

### <a name="encryption-in-transit"></a>傳輸中加密
傳輸中加密是透過網路傳輸資料時用來保護資料的機制。 透過 Azure 儲存體，您可以使用下列各項來保護資料：
-   [傳輸層級加密](https://docs.microsoft.com/azure/storage/storage-security-guide#encryption-in-transit)，例如從 Azure 儲存體傳入或傳出資料時的 HTTPS。

-   [連線加密](https://docs.microsoft.com/azure/storage/storage-security-guide#using-encryption-during-transit-with-azure-file-shares)，例如 [Azure 檔案共用](https://docs.microsoft.com/azure/storage/storage-dotnet-how-to-use-files)的 [SMB 3.0 加密](https://docs.microsoft.com/azure/storage/storage-security-guide)。

-   用戶端加密，在將資料傳輸至儲存體之前加密資料，以及自儲存體傳出後解密資料。

### <a name="encryption-at-rest"></a>待用加密
對許多組織來說，待用資料加密是達到資料隱私權、合規性及資料主權的必要步驟。 有三個 Azure 儲存體安全性功能可提供「待用」資料的加密：

-   [儲存體服務加密](https://docs.microsoft.com/azure/storage/storage-service-encryption)可讓您要求儲存體服務在將資料寫入 Azure 儲存體時自動加密資料。

-   [用戶端加密](https://docs.microsoft.com/azure/storage/storage-client-side-encryption) 也會提供待用加密的功能。

-   [Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) 允許您加密 IaaS 虛擬機器所使用的作業系統磁碟和資料磁碟。

### <a name="storage-analytics"></a>儲存體分析
[Azure 儲存體分析](https://docs.microsoft.com/rest/api/storageservices/fileservices/storage-analytics)會執行記錄，並提供儲存體帳戶的計量資料。 您可以使用此資料來追蹤要求、分析使用量趨勢，以及診斷儲存體帳戶的問題。 儲存體分析會記錄對儲存體服務之成功和失敗要求的詳細資訊。 此資訊可用來監視個別要求，並診斷儲存體服務的問題。 系統會以最佳方式來記錄要求。 系統將記錄下列類型的驗證要求：
-   成功的要求。

-   失敗的要求，包括逾時、節流、網路、授權和其他錯誤。

-   使用共用存取簽章 (SAS) 的要求，包括失敗和成功的要求。

-   分析資料的要求。

### <a name="enabling-browser-based-clients-using-cors"></a>使用 CORS 啟用瀏覽器型用戶端
[跨原始來源資源共用 (CORS)](https://docs.microsoft.com/rest/api/storageservices/fileservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services) 這個機制可讓網域能夠為彼此提供權限來存取彼此的資源。 使用者代理程式會傳送額外的標頭，以確保允許從特定網域載入的 JavaScript 程式碼存取位於另一個網域的資源。 第二個網域接著會利用額外的標頭回覆，以允許或拒絕對其資源的原始網域存取。

Azure 儲存體服務目前支援 CORS，因此，一旦您設定服務的 CORS 規則之後，即會評估從不同網域對服務所提出的適當驗證要求，以判斷是否可根據您指定的規則來允許它。
## <a name="networking"></a>網路功能
本節提供關於 Azure 網路安全性中主要功能的其他資訊，以及這些功能的摘要資訊。

### <a name="network-layer-controls"></a>網路層控制
網路存取控制是指限制與特定裝置或子網路間之連線的動作，並代表網路安全性的核心。 網路存取控制的目的是確定只有您想要它們存取的使用者和裝置，才能存取您的虛擬機器和服務。

#### <a name="network-security-groups"></a>網路安全性群組
[網路安全性群組 (NSG)](https://www.techopedia.com/definition/28190/5-tuple) 是基本可設定狀態封包過濾防火牆，並可讓您根據 [5 個 Tuple (英文)](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) 來控制存取權。 NSG 未提供應用程式層級檢查或已驗證的存取控制。 它們可用來控制在 Azure 虛擬網路內子網路之間移動的流量，以及在 Azure 虛擬網路與網際網路之間的流量。

#### <a name="route-control-and-forced-tunneling"></a>路由控制和強制通道
控制您 Azure 虛擬網路上路由行為的能力是重大網路安全性和存取控制功能。 例如，如果您想要確定進出 Azure 虛擬網路的所有流量都會經過該虛擬安全性設備，您需要能夠控制和自訂路由行為。 做法是在 Azure 中設定使用者定義的路由。

[使用者定義的路由](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview)可讓您自訂移入和移出個別虛擬機器或子網路之流量的連入和連出路徑，盡可能確保最安全的路由。 [強制通道處理](https://www.petri.com/azure-forced-tunneling)這個機制可用來確保不允許您的服務起始與網際網路上裝置的連線。

這與能夠接受連入連線，然後回應它們不同。 前端 Web 伺服器需要回應來自網際網路主機的要求，因此允許來自網際網路的流量傳入到這些 Web 伺服器，而 Web 伺服器可以回應。

強制通道處理通常用來強制傳至網際網路的連出流量通過內部部署安全性 Proxy 和防火牆。

#### <a name="virtual-network-security-appliances"></a>虛擬網路安全性應用裝置
雖然網路安全性群組、使用者定義路由和強制通道處理提供 [OSI 模型 (英文)](https://en.wikipedia.org/wiki/OSI_model) 之網路和傳輸層的安全性層級，但是您有時可能想要啟用較高堆疊層級的安全性。 您可以使用 Azure 合作夥伴網路安全性設備解決方案，來存取這些增強的網路安全性功能。 您可以瀏覽 [Azure Marketplace](https://azure.microsoft.com/marketplace/) 並搜尋「安全性」和「網路安全性」，以尋找最新的 Azure 合作夥伴網路安全性解決方案。

### <a name="azure-virtual-network"></a>Azure 虛擬網路

Azure 虛擬網路 (VNet) 是您的網路在雲端中的身分。 它是專屬於您訂用帳戶的 Azure 網路網狀架構邏輯隔離。 您可以完全控制此網路內的 IP 位址區塊、DNS 設定、安全性原則和路由表。 您可以將 VNet 分成數個子網路，並在 Azure 虛擬網路上放置 Azure IaaS 虛擬機器 (VM) 和/或[雲端服務 (PaaS 角色執行個體)](https://docs.microsoft.com/azure/cloud-services/cloud-services-choose-me)。

另外，您也可以使用 Azure 中提供的其中一個[連線選項](https://docs.microsoft.com/azure/vpn-gateway/)將虛擬網路連線到內部部署網路。 基本上，您可以將您的網路延伸至 Azure，透過 Azure 提供的企業級好處完整控制 IP 位址區塊。

Azure 網路功能支援各種安全遠端存取案例。 其中包含：

-   [將個別工作站連線到 Azure 虛擬網路](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps)

-   [使用 VPN 將內部部署網路連線到 Azure 虛擬網路](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-plan-design)

-   [使用專用 WAN 連結將內部部署網路連線到 Azure 虛擬網路](https://docs.microsoft.com/azure/expressroute/expressroute-introduction)

-   [將 Azure 虛擬網路彼此連線](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps)

### <a name="vpn-gateway"></a>VPN 閘道
若要在 Azure 虛擬網路和您的內部部署網站之間傳送網路流量，就必須為 Azure 虛擬網路建立 VPN 閘道。 [VPN 閘道](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways)是一種虛擬網路閘道，可透過公用連接傳送加密的流量。 您也可以使用 VPN 閘道，透過 Azure 網路網狀架構傳送 Azure 虛擬網路之間的流量。

### <a name="express-route"></a>ExpressRoute
Microsoft Azure [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 是專用的 WAN 連結，可讓您透過連線提供者所提供的專用私人連接，將內部部署網路擴充至 Microsoft 雲端。

![ExpressRoute](./media/azure-security/azure-security-fig1.png)

透過 ExpressRoute，您可以建立 Microsoft 雲端服務的連線，例如 Microsoft Azure、Office 365 和 CRM Online。 從任意點對任意點 (IP VPN) 網路、點對點乙太網路，或在共置設施上透過連線提供者的虛擬交叉連接，都可以進行連線。

ExpressRoute 連接不會經過公用網際網路，因此可視為比 VPN 型解決方案更安全。 相較於一般網際網路連線，這可讓 ExpressRoute 連線提供更可靠、更快速、延遲更短和更安全的連線。


### <a name="application-gateway"></a>應用程式閘道
Microsoft [Azure 應用程式閘道](https://docs.microsoft.com/azure/application-gateway/application-gateway-introduction)會以服務形式提供[應用程式傳遞控制器 (ADC) (英文)](https://en.wikipedia.org/wiki/Application_delivery_controller)，為您的應用程式提供各種第 7 層負載平衡功能。

![應用程式閘道](./media/azure-security/azure-security-fig2.png)

它會將 CPU 密集 SSL 終止卸載至應用程式閘道 (亦稱為「SSL 卸載」或「SSL 橋接」)，讓您能夠將 Web 伺服陣列的產能最佳化。 它也提供其他第 7 層路由功能，包括循環配置傳入流量、以 Cookie 為基礎的工作階段同質性、URL 路徑型路由，以及在單一應用程式閘道背後代管多個網站的能力。 Azure 應用程式閘道是第 7 層負載平衡器。

不論是在雲端或內部部署中，此閘道均提供在不同伺服器之間進行容錯移轉及效能路由傳送 HTTP 要求。

應用程式提供許多應用程式傳遞控制器 (ADC) 功能，包括 HTTP 負載平衡、以 Cookie 為基礎的工作階段同質性、[安全通訊端層 (SSL)](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-powershell) 卸載、自訂健康狀態探查、支援多網站，以及許多其他功能。

### <a name="web-application-firewall"></a>Web 應用程式防火牆
Web 應用程式防火牆是 [Azure 應用程式閘道](https://docs.microsoft.com/azure/application-gateway/application-gateway-introduction)的一個功能，可保護使用應用程式閘道執行標準應用程式傳遞控制 (ADC) 功能的 Web 應用程式。 Web 應用程式防火牆的做法是保護應用程式以防範 OWASP 前 10 個最常見的 Web 弱點。

![Web 應用程式防火牆](./media/azure-security/azure-security-fig1.png)

-   SQL 插入式攻擊保護

-   常見 Web 攻擊保護，例如命令插入式攻擊、HTTP 要求走私、HTTP 回應分割和遠端檔案包含攻擊

-   防範 HTTP 通訊協定違規

-   防範 HTTP 通訊協定異常行為，例如遺漏主機使用者代理程式和接受標頭

-   防範 Bot、編目程式和掃描器

-   偵測一般應用程式錯誤組態 (也就是 Apache、IIS 等)


防止 Web 攻擊的集中式 Web 應用程式防火牆可簡化安全性管理作業，並更加確保應用程式能夠對抗入侵威脅。 相較於保護每個個別的 Web 應用程式，WAF 方案還可透過在中央位置修補已知弱點，更快地因應安全性威脅。 現有的應用程式閘道可以輕易地轉換成具有 Web 應用程式防火牆的應用程式閘道。
### <a name="traffic-manager"></a>流量管理員
Microsoft [Azure 流量管理員](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-overview)可讓您控制使用者流量，將流量分散到不同資料中心的服務端點。 流量管理員支援的服務端點包括 Azure VM、Web Apps 和雲端服務。 您也可以將流量管理員使用於外部、非 Azure 端點。 流量管理員會使用網域名稱系統 (DNS)，根據[流量路由方法](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-routing-methods)和端點的健全狀況，將用戶端要求導向到最適當的端點。

流量管理員提供各種流量路由方法，以符合不同的應用程式需求、端點健全狀況[監視](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-monitoring)、及自動容錯移轉。 流量管理員可針對失敗彈性應變，包括整個 Azure 區域失敗。
### <a name="azure-load-balancer"></a>Azure Load Balancer
[Azure Load Balancer](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview) 可為您的應用程式提供高可用性和網路效能。 這是 Layer 4 (TCP、UDP) 負載平衡器，可將連入流量分配到負載平衡集中所定義服務的狀況良好執行個體。 Azure Load Balancer 可以設定為：

-   對虛擬機器的連入網際網路流量進行負載平衡。 這個組態稱為 [網際網路面向的負載平衡](https://docs.microsoft.com/azure/load-balancer/load-balancer-internet-overview)。

-   平衡虛擬網路中的虛擬機器之間、雲端服務中的虛擬機器之間，或內部部署電腦與跨單位部署虛擬網路中的虛擬機器之間的流量負載。 這個組態稱為 [內部負載平衡](https://docs.microsoft.com/azure/load-balancer/load-balancer-internal-overview)。 

- 將外部流量轉送到特定的虛擬機器

### <a name="internal-dns"></a>內部 DNS
您可以在管理入口網站或網路組態檔中，管理用於 VNet 的 DNS 伺服器清單。 客戶可以為每個 VNet 新增最多 12 部 DNS 伺服器。 指定 DNS 伺服器時，請務必確認您會針對客戶環境以正確順序列出客戶的 DNS 伺服器。 DNS 伺服器清單不會使用循環配置資源， 而會依其指定的順序來使用。 如果可以連接至清單上的第一部 DNS 伺服器，用戶端就會使用該 DNS 伺服器，而無論該 DNS 伺服器是否運作正常。 若要變更客戶虛擬網路的 DNS 伺服器順序，請從清單中移除 DNS 伺服器，然後以客戶想要的順序將其重新加入。 DNS 支援 “CIA” 安全性三部曲的可用性層面。

### <a name="azure-dns"></a>Azure DNS
[網域名稱系統](https://technet.microsoft.com/library/bb629410.aspx) (或 DNS) 負責將網站或服務名稱轉譯 (或解析) 為其 IP 位址。 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-overview) 是 DNS 網域的主機服務，採用 Microsoft Azure 基礎結構提供名稱解析。 只要將您的網域裝載於 Azure，就可以像管理其他 Azure 服務一樣，使用相同的認證、API、工具和計費方式來管理 DNS 記錄。 DNS 支援 “CIA” 安全性三部曲的可用性層面。
### <a name="log-analytics-nsgs"></a>Log Analytics NSG
您可以啟用下列 NSG 的診斷記錄類別︰
-   事件︰包含要將 NSG 規則套用到以 MAC 位址為基礎的 VM 和執行個體角色的項目。 每隔 60 秒會收集一次這些規則的狀態。

-   規則計數器：包含套用每個 NSG 規則以拒絕或允許流量之次數的項目。

### <a name="azure-security-center"></a>Azure 資訊安全中心
資訊安全中心可協助您預防、偵測和回應威脅，並加強對 Azure 資源安全性的能見度及控制權。 它提供您 Azure 訂用帳戶之間的整合式安全性監視和原則管理、協助偵測可能忽略的威脅，並適用於廣泛的安全性解決方案生態系統。 網路建議集中圍繞在防火牆、網路安全性群組、設定輸入流量規則等等。

可用的網路建議如下：

-   [新增新一代防火牆](https://docs.microsoft.com/azure/security-center/security-center-add-next-generation-firewall)：建議您新增由 Microsoft 合作夥伴提供的新一代防火牆 (NGFW)，以提升您的安全防護。

-   [僅透過 NGFW 路由傳送流量](https://docs.microsoft.com/azure/security-center/security-center-add-next-generation-firewall#route-traffic-through-ngfw-only)：建議您設定網路安全性群組 (NSG) 規則，強制透過您的 NGFW 將輸入流量傳送到 VM。

-   [啟用子網路/虛擬機器上的網路安全性群組](https://docs.microsoft.com/azure/security-center/security-center-enable-network-security-groups)：建議您在子網路或 VM 上啟用 NSG。

-   [透過網際網路面向端點限制存取](https://docs.microsoft.com/azure/security-center/security-center-restrict-access-through-internet-facing-endpoints)：建議您為 NSG 設定輸入流量規則。


## <a name="compute"></a>計算

本節提供關於這個領域中主要功能的其他資訊，以及這些功能的摘要資訊。

### <a name="antimalware--antivirus"></a>反惡意程式碼與防毒軟體
運用 Azure IaaS，您可以使用來自安全性廠商 (例如 Microsoft、Symantec、Trend Micro、McAfee 和 Kaspersky) 的反惡意程式碼軟體，以保護您的虛擬機器來抵禦惡意檔案、廣告軟體和其他威脅。 適用於 Azure 雲端服務和虛擬機器的 [Microsoft Antimalware](https://docs.microsoft.com/azure/security/azure-security-antimalware) 是一個保護功能，有助於識別和移除病毒、間諜軟體和其他惡意軟體。 Microsoft Antimalware 會提供可設定的警示，在已知的惡意或垃圾軟體嘗試自行安裝或在您的 Azure 系統上執行時發出警示。 您也可以使用 Azure 資訊安全中心來部署 Microsoft Antimalware。

### <a name="hardware-security-module"></a>硬體安全性模型
加密和驗證不會改善安全性，除非金鑰本身也受到保護。 您可以透過將關鍵密碼和金鑰存放在 [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis)，來簡化其管理與安全性。 Key Vault 讓您能選擇將金鑰存放在通過 FIPS 140-2 Level 2 標準認證的硬體安全性模組 (HSM) 中。 備份或 [透明資料加密](https://msdn.microsoft.com/library/bb934049.aspx) 的 SQL Server 加密金鑰都能與應用程式的任何金鑰或密碼一起存放在金鑰保存庫中。 這些受保護項目的權限和存取權是透過 [Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/)來管理。

### <a name="virtual-machine-backup"></a>虛擬機器備份
[Azure 備份](https://docs.microsoft.com/azure/backup/backup-introduction-to-azure-backup)是一種解決方案，可以不需成本地保護您的應用程式資料，以及將操作成本降到最低。 應用程式錯誤可能導致資料損毀，而人為錯誤可能會將 Bug 導入應用程式，因而引發安全性問題。 使用 Azure 備份，您執行 Windows 與 Linux 的虛擬機器會受到保護。

### <a name="azure-site-recovery"></a>Azure Site Recovery
組織之[商務持續性/災害復原 (BCDR)](https://docs.microsoft.com/azure/best-practices-availability-paired-regions) 策略的一個重要部分是，找出在發生計劃中和非計劃中的中斷時讓企業工作負載和應用程式保持啟動並執行的方法。 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 有助於協調工作負載和應用程式的複寫、容錯移轉及復原，因此能夠在主要位置發生故障時，透過次要位置提供工作負載和應用程式。

### <a name="sql-vm-tde"></a>SQL VM TDE
[透明資料加密 (TDE)](https://docs.microsoft.com/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-keyvault)和資料行層級加密 (CLE) 都是 SQL Server 加密功能。 此形式的加密需要客戶管理和儲存您用來加密的密碼編譯金鑰。

Azure 金鑰保存庫 (AKV) 服務是設計來改善這些金鑰在安全且高度可用位置的安全性和管理。 SQL Server 連接器讓 SQL Server 可以從 Azure Key Vault 使用這些金鑰。

如果您使用內部部署機器執行 SQL Server，則可遵循一些步驟來從內部部署 SQL Server 機器存取 Azure Key Vault。 但是對於 Azure VM 中的 SQL Server，您可以使用 Azure Key Vault 整合功能來節省時間。 使用一些 Azure PowerShell Cmdlet 來啟用此功能，您可以自動化 SQL VM 存取您的金鑰保存庫所需的組態。

### <a name="vm-disk-encryption"></a>VM 磁碟加密
[Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)是一個新功能，可協助您加密 Windows 和 Linux IaaS 虛擬機器磁碟。 它運用 Windows 的業界標準 BitLocker 功能和 Linux 的 DM-Crypt 功能，為 OS 和資料磁碟提供磁碟區加密。 此解決方案會與 Azure Key Vault 整合，以協助您控制及管理 Key Vault 訂用帳戶中的磁碟加密金鑰與密碼。 此解決方案也可確保虛擬機器磁碟上的所有待用資料都會在您的 Azure 儲存體中加密。

### <a name="virtual-networking"></a>虛擬網路
虛擬機器需要遠端連線。 為了支援該需求，Azure 需要虛擬機器連接到 Azure 虛擬網路。 Azure 虛擬網路是以實體 Azure 網路網狀架構為基礎所建置的邏輯建構。 每個邏輯 [Azure 虛擬網路](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)都會與其他所有 Azure 虛擬網路隔離。 此隔離可協助確保其他 Microsoft Azure 客戶無法存取您部署中的網路流量。

### <a name="patch-updates"></a>修補程式更新
修補程式更新提供尋找及修正潛在問題的基礎並簡化軟體更新管理程序，方法是減少您必須在企業中部署的軟體更新數目，以及增強您監視合規性的能力。

### <a name="security-policy-management-and-reporting"></a>安全性原則管理和報告
[Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)可協助您預防、偵測和回應威脅，並加強提供對 Azure 資源安全性的能見度及控制權。 它提供您 Azure 訂用帳戶之間的整合式安全性監視和原則管理、協助偵測可能忽略的威脅，並適用於廣泛的安全性解決方案生態系統。

### <a name="azure-security-center"></a>Azure 資訊安全中心
資訊安全中心利用加強對您 Azure 資源的能見度及安全性控制權，來協助您預防、偵測及回應威脅。 它提供您 Azure 訂用帳戶之間的整合式安全性監視和原則管理，協助您偵測可能會忽略的威脅，且適用於廣泛的安全性解決方案生態系統。

## <a name="identify-and-access-management"></a>身分識別和存取管理

保護系統、應用程式及資料是從以身分識別為基礎的存取控制開始。 內建於 Microsoft 商務產品和服務的身分識別與存取管理功能，可協助保護您的組織和個人資訊免於遭受未經授權的存取，同時讓合法的使用者隨時隨地都能視需要來使用它。

### <a name="secure-identity"></a>安全的身分識別
Microsoft 在其產品與服務上使用多個安全性作法與技術來管理身分識別與存取。
-   [Multi-Factor Authentication](https://azure.microsoft.com/services/multi-factor-authentication/) 需要使用者在內部部署和雲端中使用多種方法進行存取。 它使用一些簡單驗證選項來提供堅固的驗證，同時透過簡易登入程序來因應使用者。

-   [Microsoft Authenticator (英文)](https://aka.ms/authenticator) 提供易於使用的 Multi-Factor Authentication 體驗，可搭配 Microsoft Azure Active Directory 和 Microsoft 帳戶一起使用，並包括對於穿戴式裝置與指紋式核准的支援。

-   [密碼原則強制執行](https://azure.microsoft.com/documentation/articles/active-directory-passwords-policy/)透過加強長度和複雜度需求、強制定期循環，以及在失敗的驗證嘗試之後鎖定帳戶，來提高傳統密碼的安全性。

-   [權杖型驗證](https://azure.microsoft.com/documentation/articles/active-directory-authentication-scenarios/)可透過 Azure Active Directory 啟用驗證。

-   [角色型存取控制 (RBAC)](https://azure.microsoft.com/documentation/articles/role-based-access-built-in-roles/) 可讓您根據使用者指派的角色來授與存取權限，以便輕鬆地只為使用者提供執行其作業內容所需的存取權限。 您可以針對每個組織的商務模型和風險承受度自訂 RBAC。

-   [整合式身分識別管理 (混合式身分識別)](https://azure.microsoft.com/documentation/articles/active-directory-hybrid-identity-design-considerations-overview/) 可讓您維持控制使用者在內部資料中心和雲端平台上的存取權，建立單一使用者身分識別，以便對所有資源進行驗證與授權。

### <a name="secure-apps-and-data"></a>保護應用程式和資料
[Azure Active Directory](https://azure.microsoft.com/services/active-directory/) 是全面性的身分識別和存取管理雲端解決方案，可協助保護對現場與雲端中應用程式內資料的存取，並簡化使用者和群組的管理。 它結合了核心目錄服務、進階身分識別控管、安全性，以及應用程式存取管理，並讓開發人員能夠輕鬆地將以原則為基礎的身分識別管理建置到他們的應用程式中。 若要增強您的 Azure Active Directory，您可以使用 Azure Active Directory Basic、Premium P1 及 Premium P2 版本來新增付費功能。

| 免費/常用功能     | 基本功能    |Premium P1 功能 |Premium P2 功能 | Azure Active Directory Join – 僅適用於 Windows 10 的相關功能|
| :------------- | :------------- |:------------- |:------------- |:------------- |
|   [目錄物件](https://docs.microsoft.com/azure/active-directory/active-directory-editions#directory-objects)、[使用者/群組管理 (新增/更新/刪除)/以使用者為基礎的佈建、裝置註冊](https://docs.microsoft.com/azure/active-directory/active-directory-editions#usergroup-management-addupdatedelete-user-based-provisioning-device-registration)、[單一登入 (SSO)](https://docs.microsoft.com/azure/active-directory/active-directory-editions#single-sign-on-sso)、[雲端使用者的自助式密碼變更](https://docs.microsoft.com/azure/active-directory/active-directory-editions#self-service-password-change-for-cloud-users)、[連線 (針對將內部部署目錄延伸至 Azure Active Directory 的引擎進行同步)](https://docs.microsoft.com/azure/active-directory/active-directory-editions#connect-sync-engine-that-extends-on-premises-directories-to-azure-active-directory)、[安全性/使用量報告](https://docs.microsoft.com/azure/active-directory/active-directory-editions#securityusage-reports)       |   [以群組為基礎的存取管理/佈建](https://docs.microsoft.com/azure/active-directory/active-directory-editions#group-based-access-managementprovisioning)、[雲端使用者的自助式密碼重設](https://docs.microsoft.com/azure/active-directory/active-directory-editions#self-service-password-reset-for-cloud-users)、[公司商標 (登入頁面/存取面板自訂)](https://docs.microsoft.com/azure/active-directory/active-directory-editions#company-branding-logon-pagesaccess-panel-customization)、[應用程式 Proxy](https://docs.microsoft.com/azure/active-directory/active-directory-editions#application-proxy)、[SLA 99.9%](https://docs.microsoft.com/azure/active-directory/active-directory-editions#sla-999) |  [自助式群組和應用程式管理/自助式應用程式新增/動態群組](https://docs.microsoft.com/azure/active-directory/active-directory-editions#self-service-group)、[使用內部部署回寫來進行的自助式密碼重設/變更/解除鎖定](https://docs.microsoft.com/azure/active-directory/active-directory-editions#self-service-password-resetchangeunlock-with-on-premises-write-back)、[Multi-Factor Authentication (雲端與內部部署 (MFA Server))](https://docs.microsoft.com/azure/active-directory/active-directory-editions#multi-factor-authentication-cloud-and-on-premises-mfa-server)、[MIM CAL + MIM 伺服器](https://docs.microsoft.com/azure/active-directory/active-directory-editions#mim-cal-mim-server)、[Cloud App Discovery](https://docs.microsoft.com/azure/active-directory/active-directory-editions#cloud-app-discovery)、[Connect Health](https://docs.microsoft.com/azure/active-directory/active-directory-editions#connect-health)、[群組帳戶的自動密碼變換](https://docs.microsoft.com/azure/active-directory/active-directory-editions#automatic-password-rollover-for-group-accounts)|   [Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection)、[Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-configure)|  [將裝置加入 Azure AD、Desktop SSO、適用於 Azure AD 的 Microsoft Passport、系統管理員 Bitlocker 復原](https://docs.microsoft.com/azure/active-directory/active-directory-editions#join-a-device-to-azure-ad-desktop-sso-microsoft-passport-for-azure-ad-administrator-bitlocker-recovery)、[MDM 自動註冊、自助式 Bitlocker 復原、Windows 10 裝置透過 Azure AD Join 取得的其他本機系統管理員](https://docs.microsoft.com/azure/active-directory/active-directory-editions#mdm-auto-enrollment)|


- [Cloud App Discovery](https://docs.microsoft.com/azure/active-directory/active-directory-cloudappdiscovery-whatis) 是 Azure Active Directory 的一個高階功能，可讓您識別組織中的員工所使用的雲端應用程式。

- [Azure Active Directory Identity Protection](https://azure.microsoft.com/documentation/articles/active-directory-identityprotection/) 是一種安全性服務，會使用 Azure Active Directory 異常偵測功能來提供可能影響組織身分識別之風險事件和潛在弱點的合併檢視。

- [Azure Active Directory Domain Services](https://azure.microsoft.com/services/active-directory-ds/) 可讓您將 Azure VM 加入至網域，而不需部署網域控制站。 使用者利用其公司的 Active Directory 認證登入這些 VM，並可順暢地存取資源。

- [Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) 是一個高可用性的全域身分識別管理服務，適用於可處理數億個身分識別並跨行動裝置與 Web 平台整合的消費者端應用程式。 您的客戶可以透過可自訂的體驗 (現有的社交媒體帳戶) 登入您所有的應用程式，或者您可以建立新的獨立認證。

- [Azure Active Directory B2B 共同作業](https://aka.ms/aad-b2b-collaboration)是一個安全的合作夥伴整合解決方案，可支援公司間的關係，方法則是讓合作夥伴使用由其自行管理的身分識別，選擇性地存取您的公司應用程式和資料。

- [Azure Active Directory Join](https://azure.microsoft.com/documentation/articles/active-directory-azureadjoin-overview/) 可讓您將雲端功能擴充至 Windows 10 裝置以進行集中管理。 它可讓使用者透過 Azure Active Directory 連接到公司或組織雲端，並簡化對應用程式和資源的存取。

- [Azure Active Directory 應用程式 Proxy](https://azure.microsoft.com/documentation/articles/active-directory-application-proxy-get-started/) 為內部部署裝載的 Web 應用程式提供 SSO 及安全的遠端存取。

## <a name="next-steps"></a>後續步驟
- [開始使用 Microsoft Azure 安全性](https://docs.microsoft.com/azure/security/azure-security-getting-started)

您可以用來保護 Azure 中的服務和資料的 Azure 服務與功能

- [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)

加強 Azure 資源的可見度及安全性控制，達到預防、偵測及回應威脅的目的

- [Azure 資訊安全中心的安全性健康情況監視](https://docs.microsoft.com/azure/security-center/security-center-monitoring)

Azure 資訊安全中心內的監視功能，透過原則監視合規性。
