---
title: 保護 PaaS 部署 | Microsoft Docs
description: " 了解 PaaS 與其他雲端服務模型相較之下的安全性優點，以及了解保護 Azure PaaS 部署的建議做法。 "
services: security
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: techlake
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/21/2018
ms.author: terrylan
ms.openlocfilehash: 35650eec65fa9181d035c52e6b466985b483500c
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47036502"
---
# <a name="securing-paas-deployments"></a>保護 PaaS 部署

此文章提供的資訊可協助您：

- 了解將應用程式裝載在雲端的安全性優點
- 評估平台即服務 (PaaS) 與其他雲端服務模型相較之下的安全性優點
- 將您的安全性焦點從以網路為中心變更成以身分識別為中心的周邊安全性方法
- 實作一般 PaaS 安全性最佳做法建議

## <a name="cloud-security-advantages"></a>雲端安全性優點
在雲端環境中有一些安全性優點。 在內部部署環境中，組織可能責任重大但可投資在安全性上的資源卻相當有限，導致創造出一種攻擊者能夠利用所有層級弱點的環境。

![雲端時代的安全性優點][1]

組織能夠透過使用提供者的雲端型安全性功能和雲端智慧，改進其威脅偵測和回應時間。  透過將責任轉移給雲端提供者，組織便可擴大安全性涵蓋範圍，而能夠將安全性資源和預算重新配置給其他業務優先順序項目。

## <a name="division-of-responsibility"></a>責任劃分
了解您與 Microsoft 之間的責任劃分相當重要。 在內部部署環境中，您擁有整個堆疊，但是當您移到雲端時，部分責任就會轉移給 Microsoft。 以下責任矩陣圖顯示 SaaS、PaaS 及 IaaS 部署中由您負責和由 Microsoft 負責的堆疊領域。

![責任區][2]

就所有雲端部署類型而言，您擁有您的資料和身分識別。 您需負責保護您資料和身分識別、內部部署資源及您所控制之雲端元件 (因服務類型而異) 的安全性。

不論部署類型為何，一律由您承擔責任的對象包括：

- 資料
- 端點
- 帳戶
- 存取管理

## <a name="security-advantages-of-a-paas-cloud-service-model"></a>PaaS 雲端服務模型的安全性優點
讓我們使用相同的責任矩陣圖，來看看 Azure PaaS 部署與內部部署相較之下的安全性優點。

![PaaS 的安全性優點][3]

Microsoft 是從堆疊底部的實體基礎結構開始來減輕常見的風險和責任。 由於 Microsoft 雲端受到 Microsoft 持續不斷的監視，因此難以對它進行攻擊。 對攻擊者來說，以 Microsoft 雲端作為目標並非明智之舉。 除非攻擊者擁有許多資金和資源，否則攻擊者就可能轉移到另一個目標。  

在攻擊當中，PaaS 部署與內部部署之間並沒有差異。 在應用程式層以及帳戶和存取管理層，您都有類似的風險。 在此文章的＜後續步驟＞一節中，我們將引導您進行將這些風險消除或降到最低的最佳做法。

在堆疊頂端的資料控管和權限管理，您需承擔一項可由金鑰管理降低的風險。 (金鑰管理涵蓋在最佳做法中)。雖然金鑰管理是一個額外的責任，但在 PaaS 部署中有些領域已不再需要由您管理，因此您可以將資源轉移到金鑰管理。

Azure 平台也透過使用各種網路型技術，提供您增強式 DDoS 保護。 不過，所有類型的網路型 DDoS 保護方法在每一連結和每一資料中心上都有其限制。 若要協助避免大型 DDoS 攻擊所帶來的影響，您可以利用可讓您快速且自動相應放大規模來防禦 DDoS 攻擊的 Azure 核心雲端功能。 我們將在建議的做法文章中，更詳細地深入探討如何這麼做。

## <a name="modernizing-the-defenders-mindset"></a>讓防禦者的心態邁向現代化
伴隨 PaaS 部署而來的就是您整體安全性方法的轉變。 您會從需要全部自行控制轉變成與 Microsoft 分享責任。

PaaS 與傳統內部部署的另一個重大差異在於一個新觀點，就是定義主要安全性周邊的是什麼。 在過去，主要內部部署安全性周邊是您的網路，而大多數內部部署安全性設計皆使用網路作為其主要安全性樞紐。 就 PaaS 部署而言，將身分識別視為主要安全性周邊可為您提供較佳的服務。

## <a name="adopt-a-policy-of-identity-as-the-primary-security-perimeter"></a>採用以身分識別作為主要安全性周邊的原則
雲端運算的五個基本特性之一是廣泛的網路存取，這使得以網路為中心的思維不是那麼重要。 雲端運算的大部分目標在於讓使用者不論身在哪個位置都能存取資源。 就大多數使用者而言，他們的位置將是網際網路上的某一處。

下圖說明安全性周邊如何從網路周邊發展到身分識別周邊。 安全性變得較不著重於防禦您的網路，而是較著重於防禦您的資料，以及管理您應用程式和使用者的安全性。 主要的差異在於您想要讓安全性更貼近於您公司所看重的方面。

![以身分識別作為新的安全性周邊][4]

一開始，Azure PaaS 服務 (例如 Web 角色和 Azure SQL) 提供極少或未提供任何傳統網路周邊防禦。 在認知上，元素的用途是對網際網路公開 (Web 角色)，而驗證則提供新的周邊 (例如 BLOB 或 Azure SQL)。

新式安全性做法是假設敵人已經突破網路周邊。 因此，新式防禦做法已經轉移到身分識別。 組織必須以增強式驗證與授權防疫 (最佳做法) 建立身分識別型安全性周邊。

網路周邊的原則和模式已經存在數十年。 對照之下，業界在使用身分識別作為主要安全性周邊方面就相對較無經驗。 儘管如此，我們也已累積足夠的經驗來提供一些一般性的建議，這些建議已經過實地驗證且適用於幾乎所有 PaaS 服務。

以下是管理身分識別周邊的最佳做法。

**最佳做法**：保護金鑰和認證來保護 PaaS 部署。   
**詳細資料**：遺失金鑰和認證是相當常見的問題。 您可以使用集中式解決方案，將金鑰和密碼存放在硬體安全性模組 (HSM) 中。 Azure 透過 [Azure Key Vault](../key-vault/key-vault-whatis.md) 提供您一個雲端 HSM。

**最佳做法**：不要將認證與其他祕密放在原始程式碼或 GitHub 中。   
**詳細資料**：唯一比遺失金鑰和認證更糟的情況就是讓未經授權的一方能夠存取這些機密資料。 攻擊者能夠利用 Bot 技術來尋找存放在程式碼存放庫 (例如 GitHub) 中的金鑰和密碼。 請勿將金鑰和密碼放在這些公用程式碼存放庫中。

**最佳做法**：使用可讓您以遠端直接管理這些 VM 的管理介面，保護混合式 PaaS 和 IaaS 服務上的 VM 管理介面。   
**詳細資料**：可以使用遠端管理通訊協定，例如 [SSH](https://en.wikipedia.org/wiki/Secure_Shell)、[RDP](https://support.microsoft.com/kb/186607) 及 [PowerShell 遠端處理](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.core/enable-psremoting)。 一般而言，建議您不要啟用從網際網路直接遠端存取 VM 的功能。

可能的話，請使用替代的方法，例如在 Azure 虛擬網路中使用虛擬私人網路。 如果沒有替代方法可用，則請務必使用複雜密碼和雙因素驗證 (例如 [Azure Multi-Factor Authentication](../active-directory/authentication/multi-factor-authentication.md))。

**最佳做法**：使用增強式驗證與授權平台。   
**詳細資料**：使用 Azure AD 中的同盟身分識別，而不要使用自訂使用者存放區。 使用同盟身分識別時，您可以利用平台型方法並將已授權之身分識別的管理委派給您的合作夥伴。 在員工已被解雇而該資訊必須透過多個身分識別與授權系統來反映時，同盟身分識別方法尤其重要。

使用平台提供的驗證與授權機制，而不要使用自訂程式碼。 原因在於開發自訂驗證程式碼可能容易出錯。 您的大多數開發人員都不是安全性專家，因此可能不是很清楚驗證與授權方面的微妙細節和最新發展。 商業程式碼 (例如，來自 Microsoft) 通常都經過廣泛的安全性檢閱。

使用雙因素驗證。 雙因素驗證是現行的驗證與授權標準，因為它可避免使用者名稱與密碼型驗證中固有的安全性弱點。 您應該將 Azure 管理 (入口網站/遠端 PowerShell) 介面和面向客戶之服務的存取方式都設計並設定成使用 [Azure Multi-Factor Authentication](../active-directory/authentication/multi-factor-authentication.md)。

使用標準驗證通訊協定，例如 OAuth2 和 Kerberos。 這些通訊協定已經過廣泛的對等檢閱，而可能作為您驗證與授權平台程式庫的一部分來實作。

## <a name="use-threat-modeling-during-application-design"></a>在應用程式設計期間使用威脅模型化
Microsoft [安全性開發週期](https://www.microsoft.com/en-us/sdl)指定小組應在進行設計階段時，參與稱為威脅模型化的程序。 為了加快此程序，Microsoft 已建立 [SDL Threat Modeling Tool](../security/azure-security-threat-modeling-tool.md)。 將應用程式設計模型化，並跨所有信任界限列舉 [STRIDE](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxzZWN1cmVwcm9ncmFtbWluZ3xneDo0MTY1MmM0ZDI0ZjQ4ZDMy) 威脅，可以盡早攔截設計錯誤。

下表列出 STRIDE 威脅，並提供一些使用 Azure 功能的範例風險降低措施。 這些風險降低措施並非能在各個情況下運作。

| Threat | 安全性屬性 | 潛在的 Azure 平台移轉 |
| --- | --- | --- |
| 詐騙 | 驗證 | 需要 HTTPS 連線。 |
| 竄改 | 完整性 | 驗證 SSL 憑證。 |
| 否認性 | 不可否認性 | 啟用 Azure [監視和診斷](https://docs.microsoft.com/azure/architecture/best-practices/monitoring)。 |
| 資訊洩漏 | 保密 | 使用[服務憑證](https://docs.microsoft.com/rest/api/appservice/certificates)將待用的敏感性資料加密。 |
| Denial of service (拒絕服務) | 可用性 | 監視潛在拒絕服務狀況的效能計量。 實作 IP 連線篩選。 |
| 權限提高 | Authorization | 使用[具特殊權限身分識別管理](../active-directory/privileged-identity-management/subscription-requirements.md)。 |

## <a name="develop-on-azure-app-service"></a>在 Azure App Service 上開發
[Azure App Service](../app-service/app-service-web-overview.md) 是一個 PaaS 供應項目，可讓您為任何平台或裝置建立 Web 與行動應用程式，以及連線到雲端或內部部署環境中任何位置的資料。 App Service 包含先前以 Azure 網站和 Azure 行動服務形式個別提供的 Web 和行動功能。 此外，它也包含可用來自動執行商務程序及裝載雲端 API 的新功能。 App Service 會以單一整合式服務形式，為 Web、行動及整合案例提供一組豐富的功能。

以下是使用 App Service 的最佳做法。

**最佳做法**：[透過 Azure Active Directory 進行驗證](../app-service/app-service-authentication-overview.md)。   
**詳細資料**：App Service 可為您的識別提供者提供 OAuth 2.0 服務。 OAuth 2.0 既將焦點放在為用戶端開發人員提供簡易性，同時又為 Web 應用程式、傳統型應用程式及行動電話提供特定授權流程。 Azure AD 使用 OAuth 2.0 來讓您能夠授與對行動與 Web 應用程式的存取權。

**最佳做法**：根據必要知悉和最低權限安全性原則來限制存取。   
**詳細資料**：對於想要強制執行資料存取安全性原則的組織來說，限制存取是必須做的事。 您可以使用 RBAC 來將權限指派給特定範圍的使用者、群組及應用程式。 若要深入了解授與使用者的應用程式存取權，請參閱[開始使用存取管理](../role-based-access-control/overview.md)。

**最佳做法**：保護您的金鑰。   
**詳細資料**：Azure Key Vault 可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和祕密。 您可以使用金鑰保存庫加密金鑰和密碼 (例如驗證金鑰、儲存體帳戶金鑰、資料加密金鑰、.PFX 檔案和密碼)，方法是使用受硬體安全模組 (HSM) 保護的金鑰。 為了加強保證，您可以在 HSM 中匯入或產生金鑰。 若要深入了解，請參閱 [Azure Key Vault](../key-vault/key-vault-whatis.md)。 您可以使用 Key Vault 透過自動更新管理 TLS 憑證。

**最佳做法**：限制連入來源 IP 位址。   
**詳細資料**：[App Service 環境](../app-service/environment/intro.md)具有虛擬網路整合功能，可協助您透過網路安全性群組限制連入來源 IP 位址。 虛擬網路可讓您將 Azure 資源，放在您控制存取權的非網際網路可路由網路中。 若要深入了解，請參閱[將您的應用程式與 Azure 虛擬網路整合](../app-service/web-sites-integrate-with-vnet.md)。

**最佳做法**：監視 App Service 環境的安全性狀態。   
**詳細資料**：使用 Azure 資訊安全中心來監視 App Service 環境。 當資訊安全中心發現潛在的安全性弱點時會建立[建議](../security-center/security-center-virtual-machine-recommendations.md)，引導您完成設定所需控制項的程序。

> [!NOTE]
> 監視 App Service 是預覽功能，僅適用於資訊安全中心的[標準層](../security-center/security-center-pricing.md)。
>
>

## <a name="install-a-web-application-firewall"></a>安裝 Web 應用程式防火牆
Web 應用程式已逐漸成為利用常見已知弱點的惡意攻擊目標。 這些攻擊中最常見的是 SQL 插入式攻擊、跨網站指令碼攻擊等等。 想要防止應用程式的程式碼受到這類攻擊會非常困難，而且可能需要對許多層次的應用程式拓撲執行嚴格的維護、修補和監視工作。 集中式 Web 應用程式防火牆有助於簡化安全性管理作業，且更加確保應用程式管理員能夠對抗威脅或入侵。 相較於保護每個個別的 Web 應用程式，WAF 方案還可透過在中央位置修補已知弱點，更快地因應安全性威脅。 現有的應用程式閘道可以輕易地轉換成已啟用 Web 應用程式防火牆的應用程式閘道。

[Web 應用程式防火牆 (WAF)](../application-gateway/waf-overview.md) 是一個應用程式閘道功能，可提供 Web 應用程式的集中式保護，免於遭遇常見的攻擊和弱點。 WAF 會根據 [Open Web Application Security Project (OWASP) 核心規則集](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 3.0 或 2.2.9 中的規則提供保護。

## <a name="monitor-the-performance-of-your-applications"></a>監視應用程式的效能
監視係指收集和分析資料來判斷應用程式之效能、健康情況和可用性的行為。 有效的監視策略可協助您了解您應用程式元件的詳細操作。 它可透過通知重大問題來協助您增加運作時間，以便您在這些問題發生之前予以解決。 它也可協助您偵測可能和安全性相關的異常。

使用 [Azure Application Insights](http://azure.microsoft.com/documentation/services/application-insights) 來監視應用程式的可用性、效能及使用情況 (不論該應用程式是裝載在雲端還是內部部署環境)。 使用 Application Insights，您可以快速識別並診斷應用程式的錯誤，不必等使用者回報。 有了所收集的資訊之後，您便可以針對應用程式的維護和改善，進行資訊充分的選擇。

Application Insights 具有廣泛的工具，能與它所收集的資料進行互動。 Application Insights 會將其資料儲存在一般存放庫中。 它可以利用共用功能，例如警示、儀表板，以及採用 Log Analytics 查詢語言的深入分析。



## <a name="next-steps"></a>後續步驟
在此文章中，我們是將焦點放在 Azure PaaS 部署的安全性優點和雲端應用程式的安全性最佳做法。 接下來，請了解使用特定 Azure 服務保護 PaaS Web 和行動解決方案的建議做法。 我們將從 Azure App Service、Azure SQL Database、Azure SQL 資料倉儲及 Azure 儲存體開始著手。 當有適用於其他 Azure 服務的建議做法文章推出時，就會在以下清單中提供連結：

- [Azure App Service](security-paas-applications-using-app-services.md)
- [Azure SQL Database 和 Azure SQL Data Warehouse](security-paas-applications-using-sql.md)
- [Azure 儲存體](security-paas-applications-using-storage.md)
- Azure REDIS Cache
- Azure 服務匯流排
- Web 應用程式防火牆

如需更多安全性最佳做法，請參閱 [Azure 安全性最佳做法與模式](security-best-practices-and-patterns.md)，以便在使用 Azure 設計、部署和管理雲端解決方案時使用。

下列資源可提供更多有關 Azure 安全性和相關 Microsoft 服務的一般資訊：
* [Azure 安全性部落格](https://blogs.msdn.microsoft.com/azuresecurity/) - Azure 安全性的最新資訊
* [Microsoft 安全性回應中心](https://technet.microsoft.com/library/dn440717.aspx) - 可在其中回報 Microsoft 安全性弱點 (包括 Azure 的問題) 或透過電子郵件傳送給 secure@microsoft.com

<!--Image references-->
[1]: ./media/security-paas-deployments/advantages-of-cloud.png
[2]: ./media/security-paas-deployments/responsibility-zones.png
[3]: ./media/security-paas-deployments/advantages-of-paas.png
[4]: ./media/security-paas-deployments/identity-perimeter.png
