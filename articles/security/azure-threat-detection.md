---
title: Azure 進階威脅偵測 | Microsoft Docs
description: 了解 Azure AD Identity Protection 和其功能。
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
ms.openlocfilehash: 9b688ca6faaa7e0d84dff0ae28e2a9b8b8279490
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37856873"
---
# <a name="azure-advanced-threat-detection"></a>Azure 進階威脅偵測

Azure 透過 Azure Active Directory (Azure AD)、Azure Operations Management Suite (OMS) 和 Azure 資訊安全中心這類服務，來提供內建進階威脅偵測功能。 這個安全性服務和功能集合提供簡單且快速的方式，來了解您 Azure 部署內的一舉一動。

Azure 提供各種選項來設定和自訂安全性，以符合您應用程式部署的需求。 本文討論如何符合這些需求。

## <a name="azure-active-directory-identity-protection"></a>Azure Active Directory Identity Protection

[Azure AD Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 是 [Azure Active Directory Premium P2](https://docs.microsoft.com/azure/active-directory/active-directory-editions) 版本功能，提供可影響組織身分識別之風險事件和潛在弱點的概觀。 Identity Protection 使用現有 Azure AD 異常偵測功能 (可透過 [Azure AD 異常活動報告](https://docs.microsoft.com/azure/active-directory/active-directory-view-access-usage-reports#anomalous-activity-reports)取得)，並引進可即時偵測異常的新風險事件類型。

![Azure AD Identity Protection 圖表](./media/azure-threat-detection/azure-threat-detection-fig1.png)

Identity Protection 會使用調適性機器學習演算法和啟發學習法，來偵測異常以及可能表示身分識別已遭入侵的風險事件。 Identity Protection 會使用此資料來產生報告和警示，讓您可以調查這些風險事件並採取適當的補救動作或緩和措施。

Azure Active Directory Identity Protection 不只是監視和報告工具而已。 Identity Protection 會根據風險事件，計算每位使用者的使用者風險層級，讓您設定風險原則來自動保護您組織的身分識別。

除了 Azure Active Directory 與 [EMS](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access) 所提供的其他[條件式存取控制](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access)以外，這些以風險為根據的原則可以自動封鎖或提供調適性補救動作，包括重設密碼，以及強制執行 Multi-Factor Authentication。

### <a name="identity-protection-capabilities"></a>Identity Protection 功能

Azure Active Directory Identity Protection 不只是監視和報告工具而已。 若要保護您組織的身分識別，您可以設定以風險為基礎的原則，當達到指定風險層級時自動回應偵測到的問題。 除了 Azure Active Directory 與 EMS 所提供的其他條件式存取控制以外，這些的原則可以自動封鎖或起始調適性補救動作，包括重設密碼以及強制 Multi-Factor Authentication。

Azure Identity Protection 可用以協助保護您的帳戶和身分識別的一些方法範例包括：

[偵測風險事件和有風險的帳戶](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection#detection)
-   使用機器學習和啟發式規則偵測六種風險事件類型。
-   計算使用者風險層級。
-   提供自訂建議，藉由醒目提示弱點來改善整體安全性狀態。

[調查風險事件](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection#investigation)
-   傳送風險事件的通知。
-   使用相關和內容資訊來調查風險事件。
-   提供基本工作流程來追蹤調查。
-   讓您輕鬆存取補救動作 (例如重設密碼)。

[風險條件式存取原則](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection#risky-sign-ins)
-   封鎖登入或要求 Multi-Factor Authentication 挑戰，以阻止高風險登入。
-   封鎖或保護有風險的使用者帳戶。
-   要求使用者註冊 Multi-Factor Authentication。

### <a name="azure-ad-privileged-identity-management"></a>Azure AD 特殊權限身分識別管理

您可以使用 [Azure Active Directory Privileged Identity Management (PIM)](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-configure) 來管理、控制和監視組織內的存取。 此功能包括存取 Azure AD 中的資源和其他 Microsoft 線上服務 (例如 Office 365 或 Microsoft Intune)。

![Azure AD Privileged Identity Management 圖表](./media/azure-threat-detection/azure-threat-detection-fig2.png)

PIM 可協助您：

-   取得 Azure AD 系統管理員以及對 Office 365 和 Intune 這類 Microsoft 線上服務之 Just-In-Time (JIT) 系統管理存取權的警示和報告。

-   取得有關系統管理員存取記錄與系統管理員指派變更的報告。

-   取得有關特殊權限角色存取的警示。

## <a name="operations-management-suite"></a>Operations Management Suite

[Operations Management Suite (OMS)](https://docs.microsoft.com/azure/operations-management-suite/operations-management-suite-overview) 是 Microsoft 雲端式 IT 管理解決方案，可協助您管理及保護內部部署和雲端基礎結構。 因為 OMS 實作為雲端式服務，所以您對基礎結構服務進行最小的投資就可以快速啟動並執行它。 會自動提供新的安全性功能，以節省持續維護和升級成本。

除了自行提供重要服務之外，OMS 還可以整合 System Center 元件 (例如 [System Center Operations Manager](https://blogs.technet.microsoft.com/cbernier/2013/10/23/monitoring-windows-azure-with-system-center-operations-manager-2012-get-me-started/))，以將現有的安全性管理投資擴充到雲端。 System Center 和 OMS 可以搭配運作，來提供完整的混合式管理經驗。

### <a name="holistic-security-and-compliance-posture"></a>整體安全性與合規性狀態

[OMS 安全性與稽核儀表板](https://docs.microsoft.com/azure/operations-management-suite/oms-security-getting-started)針對值得您注意的問題，使用內建的搜尋查詢，為您組織的 IT 安全性狀態提供全面檢視。 [安全性和稽核] 儀表板是 OMS 中所有安全性相關項目的主畫面。 它可讓您深入了解您的電腦的安全性狀態。 您也可以檢視過去 24 小時、7 天或任何其他自訂時間範圍內的所有事件。

OMS 儀表板可協助您快速且輕鬆地了解任何環境的整體安全性狀態，全部都在 IT 作業的內容中，包括軟體更新評定、反惡意程式碼評定和設定基準。 可立即存取安全性記錄資料，以簡化安全性與合規性稽核程序。

![OMS 安全性與稽核儀表板](./media/azure-threat-detection/azure-threat-detection-fig3.jpg)

[OMS 安全性與稽核] 儀表板分為四個主要類別︰

-   **安全性網域**︰可讓您進一步探索一段時間的安全性記錄；存取惡意程式碼評定；更新評定；檢視網路安全性、身分識別和存取資訊；檢視具有安全性事件的電腦，以及快速存取 Azure 資訊安全中心儀表板。

-   **值得注意的問題**︰可讓您快速識別作用中問題數目和問題嚴重性。

-   **偵測 (預覽)**︰可讓您顯示資源所發生的安全性警示，進而識別攻擊模式。

-   **威脅情報**：可讓您藉由下列方式來識別攻擊模式：顯示呈現具有惡意輸出 IP 流量的伺服器總數、惡意威脅類型，以及 IP 位置的地圖。

-   **常見安全性查詢**︰列出最常見的安全性查詢，以用來監視您的環境。 當您選取任何查詢時，即會開啟 [搜尋] 窗格，並顯示該查詢的結果。

### <a name="insight-and-analytics"></a>見解與分析
[Log Analytics](https://docs.microsoft.com/azure/log-analytics/log-analytics-overview) 的中心是 Azure 所裝載的 OMS 存放庫。

![見解和分析圖表](./media/azure-threat-detection/azure-threat-detection-fig4.png)

您可以藉由設定資料來源並將解決方案新增至訂用帳戶，將資料從連線的來源收集到存放庫。

![OMS 儀表板 ](./media/azure-threat-detection/azure-threat-detection-fig5.png)

資料來源和解決方案各會建立具有其專屬屬性集的不同記錄類型，但您仍然可以一起分析它們來查詢存放庫。 您可以使用相同的工具和方法，來處理各種來源所收集的各種資料。


與 Log Analytics 的大部分互動都會透過 OMS 入口網站，它可以在任何瀏覽器中執行，並讓您存取組態設定和多項工具來分析及處理所收集的資料。 從入口網站中，您可以使用：
* [記錄搜尋](https://docs.microsoft.com/azure/log-analytics/log-analytics-log-searches)，以建構查詢來分析收集的資料。
* [儀表板](https://docs.microsoft.com/azure/log-analytics/log-analytics-dashboards)，可使用最有價值搜尋的圖形檢視來自訂。
* [解決方案](https://docs.microsoft.com/azure/log-analytics/log-analytics-add-solutions)，提供額外的功能和分析工具。

![分析工具](./media/azure-threat-detection/azure-threat-detection-fig6.png)

方案會將功能加入 Log Analytics。 它們主要是在雲端執行，並提供 OMS 存放庫中所收集資料的分析。 解決方案也可能會定義要收集的新記錄類型，以使用記錄搜尋來進行分析，或藉由使用解決方案在 OMS 儀表板中所提供的其他使用者介面來進行分析。

[安全性與稽核] 儀表板是這類解決方案的範例。

### <a name="automation-and-control-alert-on-security-configuration-drifts"></a>自動化與控制︰有關安全性設定漂移的警示

Azure 自動化會使用根據 PowerShell 在雲端中執行的 Runbook，來自動化管理程序。 Runbook 也可以在本機資料中心的伺服器上執行，以管理本機資源。 Azure 自動化會使用 PowerShell Desired State Configuration (DSC) 來提供設定管理。

![Azure 自動化圖表](./media/azure-threat-detection/azure-threat-detection-fig7.png)

您可以建立和管理裝載於 Azure 的 DSC 資源，以及將它們套用到雲端和內部部署系統。 藉由這麼做，您可以定義和自動強制執行其設定或取得有關漂移的報告，以協助確保安全性設定保留在原則內。

## <a name="azure-security-center"></a>Azure 資訊安全中心

Azure 資訊安全中心可協助保護您的 Azure 資源。 它提供您 Azure 訂用帳戶之間的整合式安全性監視和原則管理。 在服務內，您可以定義對 Azure 訂用帳戶和[資源群組](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-portal)的原則，以取得更詳細的資料。

![Azure 資訊安全中心圖表](./media/azure-threat-detection/azure-threat-detection-fig8.png)

Microsoft 資訊安全研究人員會持續監視威脅。 他們可以存取從 Microsoft 在雲端和內部部署中的全域支援取得的一組廣泛遙測。 這組包羅萬象的資料集，可讓 Microsoft 探索其內部部署消費性和企業產品及其線上服務的新攻擊模式和趨勢。

因此，資訊安全中心可以在攻擊者發行新的和日益複雜的攻擊時，快速地更新其偵測演算法。 這種方法可協助您跟上瞬息萬變的威脅環境。

![資訊安全中心威脅偵測](./media/azure-threat-detection/azure-threat-detection-fig9.jpg)

資訊安全中心威脅偵測的運作方式如下：從您的 Azure 資源、網路及已連線的協力廠商解決方案自動收集安全性資訊。 它會分析這項資訊 (來自多個來源的相互關聯資訊) 以識別威脅。

資訊安全中心的安全性警示會排定優先順序，並提供如何補救威脅的建議。

資訊安全中心會運用進階安全性分析，其遠勝於以簽章為基礎的方法。 巨量資料和[機器學習](https://azure.microsoft.com/blog/machine-learning-in-azure-security-center/)技術中的突破用來評估整個雲端網狀架構中的所有事件。 進階分析可以偵測無法透過手動方法並預測攻擊進化的威脅。 下列各節涵蓋這些安全性分析類型。

### <a name="threat-intelligence"></a>威脅情報

Microsoft 可以存取大量全域威脅情報。

遙測會從多個來源流入，例如 Azure、Office 365、Microsoft CRM Online、Microsoft Dynamics AX、outlook.com、MSN.com、Microsoft 數位犯罪防治中心 (DCU) 和 Microsoft 安全性回應中心 (MSRC)。

![威脅情報結果](./media/azure-threat-detection/azure-threat-detection-fig10.jpg)

研究人員也會收到主要雲端服務提供者之間共用的威脅情報資訊，並訂閱來自協力廠商的威脅情報摘要。 Azure 資訊安全中心可以使用這項資訊來警示您來自已知不良執行者的威脅。 部分範例包括：

-   **運用機器學習服務的力量**：Azure 資訊安全中心可以存取雲端網路活動的大量資料，可用來偵測目標為您 Azure 部署的潛在威脅。

-   **暴力密碼破解偵測**：機器學習用來建立嘗試遠端存取的歷程記錄模式，使其得以偵測到對於安全殼層 (SSH)、遠端桌面通訊協定 (RDP) 和 SQL 連接埠的暴力密碼破解攻擊。

-   **輸出 DDoS 和 Botnet 偵測**：目標為雲端資源之攻擊的常見目標是使用這些資源的計算能力來執行其他攻擊。

-   **新的行為分析伺服器和 VM**：在伺服器或虛擬機器遭到入侵之後，攻擊者就能用各種不同的技術，在該系統上執行惡意程式碼，同時避開偵測、確保持續性，並迴避安全性控制。

-   **Azure SQL Database 威脅偵測**：適用於 Azure SQL Database 的威脅偵測，它會識別異常的資料庫活動，指出發生不尋常且可能有害的嘗試存取或惡意探索資料庫。

### <a name="behavioral-analytics"></a>行為分析

行為分析是一種可分析及比較資料與一組已知模式的技術。 不過，這些模式並非簡單的簽章。 它們會透過已套用至大型資料集的複雜機器學習演算法來決定。

![行為分析結果](./media/azure-threat-detection/azure-threat-detection-fig11.jpg)

這些模式也能透過專業分析師仔細分析惡意行為來判定。 Azure 資訊安全中心可以使用行為分析，根據虛擬機器記錄、虛擬網路裝置記錄、網狀架構記錄、毀損傾印和其他來源的分析，來識別遭到入侵的資源。

此外，模式會與其他訊號相互關聯，以檢查廣泛行銷活動的支援證明。 此相互關聯有助於識別與已確立危害指標一致的事件。

部分範例包括：
-   **可疑處理程序執行**︰攻擊者不需要偵測，即可運用數種技術來執行惡意軟體。 例如，攻擊者可能會讓惡意程式碼具有與合法系統檔案相同的名稱，但會將這些檔案放在替代位置、使用類似良性檔案名稱的名稱，或為檔案的真正副檔名加上遮罩。 資訊安全中心會建立處理序行為的模型，並監視處理序執行以偵測這類極端值。

-   **隱藏的惡意程式碼和弱點攻擊嘗試**︰複雜的惡意程式碼可藉由永遠不要寫入至磁碟或加密磁碟上儲存的軟體元件，來避開傳統的反惡意程式碼產品。 不過，可以使用記憶體分析來偵測這類惡意程式碼，因為惡意程式碼必須在記憶體中留下蹤跡才能運作。 當軟體損毀時，損毀傾印會在損毀時擷取部分的記憶體。 藉由分析損毀傾印中的記憶體，Azure 資訊安全中心可以偵測到用來惡意探索軟體中的弱點、存取機密資料，以及暗中保存於遭入侵電腦的技術，而不會影響您電腦的效能。

-   **橫向移動和內部偵察**︰為了保存於遭入侵的網路內並找出和獲取重要資料，攻擊者經常會試圖從遭入侵的電腦橫向移到相同網路內的其他電腦。 資訊安全中心會監視處理和登入活動，以探索在網路內展開攻擊者據點的嘗試，例如，遠端命令執行、網路探查和帳戶列舉。

-   **惡意 PowerShell 指令碼**︰攻擊者會針對各種目的，使用 PowerShell 在目標虛擬機器上執行惡意程式碼。 資訊安全中心會檢查 PowerShell 活動，以找到可疑活動的證明。

-   **傳出攻擊**︰攻擊者通常會以雲端資源為目標，目的在於使用這些資源來掛載其他攻擊。 例如，遭入侵的虛擬機器可用來對其他虛擬機器發動暴力密碼破解攻擊、傳送垃圾郵件，或掃描開啟的連接埠和網際網路上的其他裝置。 藉由將機器學習服務套用到網路流量，資訊安全中心可以偵測輸出網路通訊何時超出規範。 偵測到垃圾郵件時，資訊安全中心也會讓不尋常的電子郵件流量與 Office 365 提供的情報相互關聯，以判斷郵件是否可能有惡意或合法電子郵件行銷活動的結果。

### <a name="anomaly-detection"></a>異常偵測

Azure 資訊安全中心也會使用異常偵測來識別威脅。 相較於行為分析 (這取決於衍生自大型資料集的已知模式)，異常偵測更加「個人化」，且著重於您的部署專用的基準。 機器學習適用於判斷您部署的正常活動，然後產生規則來定義可能代表安全性事件的極端狀況。 範例如下：

-   **輸入 RDP/SSH 暴力密碼破解攻擊**︰您的部署中可能包含每天都有許多登入的繁忙虛擬機器，以及有少量 (如果有的話) 登入的其他虛擬機器。 Azure 資訊安全中心可以判斷這些虛擬機器的基準登入活動，並使用要定義於正常登入活動周圍的機器學習服務。 如果與針對登入相關特性所定義的基準有任何差異，則可能會產生警示。 同樣地，機器學習服務會判斷何者值得關注。

### <a name="continuous-threat-intelligence-monitoring"></a>連續威脅情報監視

Azure 資訊安全中心在世界各地設有資訊安全研究和資料科學小組，負責持續監視威脅態勢中的變化。 這包括下列計劃︰

-   **威脅情報監視**︰威脅情報包含關於現有或新興威脅的機制、指標、影響和可採取動作的建議。 安全性社群會共用此資訊，而 Microsoft 會持續監視來自內部和外部來源的威脅情報摘要。

-   **訊號共用**︰共用和分析 Microsoft 的資訊安全小組對於各種雲端和內部部署服務、伺服器及用戶端端點裝置組合所提供的見解。

-   **Microsoft 資訊安全專家**︰持續與擅長特殊資訊安全領域 (例如鑑識與 Web 攻擊偵測) 的 Microsoft 小組攜手合作。

-   **偵測微調**︰對真正的客戶資料集執行演算法，而資訊安全研究人員會與客戶一起驗證結果。 真肯定和誤判可用來縮小機器學習演算法的範圍。

結合上述努力終於獲得全新及改善的偵測功能，您因而立即受惠。 您不需採取任何動作。

## <a name="advanced-threat-detection-features-other-azure-services"></a>進階威脅偵測功能：其他 Azure 服務

### <a name="virtual-machines-microsoft-antimalware"></a>虛擬機器︰Microsoft Antimalware

適用於 Azure 的 [Microsoft Antimalware](https://docs.microsoft.com/azure/security/azure-security-antimalware) 是針對應用程式和租用戶環境所提供的單一代理程式解決方案，其設計可於無人為介入的情況下在背景中執行。 您可依據應用程式工作負載需求，選擇預設的基本安全性或進階的自訂組態 (包括反惡意程式碼監視) 來部署保護。 Azure Antimalware 是自動安裝在所有 Azure PaaS 虛擬機器之 Azure 虛擬機器的安全性選項。

#### <a name="microsoft-antimalware-core-features"></a>Microsoft Antimalware 核心功能

以下是部署和啟用您應用程式之 Microsoft Antimalware 的 Azure 功能：

-   **即時保護**：監視雲端服務和虛擬機器上的活動，以偵測和封鎖惡意程式碼執行。

-   **排程掃描**：定期執行目標掃描以偵測惡意程式碼，包括主動執行程式。

-   **惡意程式碼補救**：自動處理偵測到的惡意程式碼，例如刪除或隔離惡意檔案及清除惡意的登錄項目。

-   **簽章更新**：自動安裝最新的保護簽章 (病毒定義) 以確保依預定頻率維持最新的保護狀態。

-   **Antimalware 引擎更新**：自動更新 Microsoft Antimalware 引擎。

-   **Antimalware 平台更新**：自動更新 Microsoft Antimalware 平台。

-   **主動保護**：向 Microsoft Azure 報告有關偵測到的威脅和可疑資源的遙測中繼資料，以確保能針對不斷演變的威脅型態做出快速的回應，並透過 Microsoft Active Protection System 啟用即時同步簽章傳遞。

-   **範例報告**：將範例提供並報告至 Microsoftt Antimalware 服務，以協助改善服務並啟用疑難排解。

-   **排除項目**：可讓應用程式和服務管理員設定特定的檔案、處理序及磁碟機，以因應效能和其他原因將其從保護和掃描中排除。

-   **反惡意程式碼軟體事件收集**：記錄作業系統事件記錄檔中反惡意程式碼軟體服務健康狀態、可疑的活動以及其所採取的補救動作，並將它們收集至客戶的 Azure 儲存體帳戶。

### <a name="azure-sql-database-threat-detection"></a>Azure SQL Database 威脅偵測

[Azure SQL Database 威脅偵測](https://azure.microsoft.com/blog/azure-sql-database-threat-detection-your-built-in-security-expert/)是內建於 Azure SQL Database 服務的新安全性智慧型功能。 Azure SQL Database 威脅偵測可藉由全天候學習、分析及偵測異常資料庫活動，來識別資料庫的潛在威脅。

資訊安全人員或其他指定的系統管理員可以在發生可疑的資料庫活動時立即取得通知。 每個通知都會提供可疑活動的詳細資料，以及建議如何進一步調查並減輕威脅。

目前，Azure SQL Database 威脅偵測會偵測潛在的弱點與 SQL 插入式攻擊，以及異常的資料庫存取模式。

在收到威脅偵測電子郵件通知時，使用者可以透過電子郵件中的深層連結來巡覽和檢視相關稽核記錄。 連結會開啟稽核檢視器或預先設定的稽核 Excel 範本，以根據下表顯示可疑事件發生時間前後的相關稽核記錄：

-   適用於具有資料庫異常活動之資料庫/伺服器的稽核儲存體。

-   將事件寫入稽核記錄時所使用的相關稽核儲存體資料表。

-   緊接在發生事件後該小時的稽核記錄。

-   事件發生時具有類似事件識別碼的稽核記錄 (對於某些偵測器而言是選擇性的)。

SQL Database 威脅偵測器會使用下列其中一種偵測方法：

-   **具決定性的偵測**：偵測 SQL 用戶端查詢中符合已知攻擊的可疑模式 (規則)。 這種方法具有高偵測度和低誤判率，但其涵蓋範圍有限，因為它屬於「不可部分完成的偵測」類別。

-   **行為偵測**：偵測異常活動，此為資料庫中未曾在最近 30 天期間看到的異常行為。 SQL 用戶端異常活動的範例可以是失敗的登入或查詢數目突然增加、擷取大量的資料、不尋常的標準查詢，或用來存取資料庫的陌生 IP 位址。

### <a name="application-gateway-web-application-firewall"></a>應用程式閘道 Web 應用程式防火牆

[Web 應用程式防火牆 (WAF)](../app-service/environment/app-service-app-service-environment-web-application-firewall.md) 是 [Azure 應用程式閘道](https://docs.microsoft.com/azure/application-gateway/application-gateway-webapplicationfirewall-overview)的功能，可保護使用應用程式閘道執行標準[應用程式傳遞控制](https://kemptechnologies.com/in/application-delivery-controllers)功能的 Web 應用程式。 Web 應用程式防火牆的做法是保護應用程式，以防範 [Open Web Application Security Project (OWASP) top 10 common web vulnerabilities](https://www.owasp.org/index.php/Top_10_2010-Main) (Open Web Application Security Project (OWASP) 前 10 個最常見的 Web 弱點)。

![應用程式閘道 Web 應用程式防火牆圖表](./media/azure-threat-detection/azure-threat-detection-fig13.png)

保護包括：

-   SQL 插入保護。

-   跨網站指令碼保護。

-   常見 Web 攻擊保護，例如命令插入、HTTP 要求走私、HTTP 回應分割和遠端檔案包含攻擊。

-   防範 HTTP 通訊協定違規。

-   防範 HTTP 通訊協定異常 (例如遺漏主機使用者代理程式和接受標頭)。

-   防範 Bot、編目程式和掃描器。

-   偵測一般應用程式錯誤設定 (即 Apache、IIS 等)。

在應用程式閘道上設定 WAF 可提供下列優點：

-   不需修改後端程式碼就能保護 Web 應用程式不受 Web 弱點和攻擊的威脅。

-   在應用程式閘道背後同時保護多個 Web 應用程式。 應用程式閘道支援裝載最多 20 個網站。

-   使用應用程式閘道 WAF 記錄所產生的即時報告，監視 Web 應用程式對抗攻擊。

-   協助符合合規性需求。 某些合規性控制項需要由 WAF 解決方案保護所有網際網路對向端點。

### <a name="anomaly-detection-api-built-with-azure-machine-learning"></a>異常偵測 API：使用 Azure Machine Learning 所建置

異常偵測 API 可用於偵測時間序列資料中的各種異常模式。 API 會為時間序列中的每個資料點指派異常分數，可用來產生警示、透過儀表板監視或與您的票證系統連線。

[異常偵測 API](https://docs.microsoft.com/azure/machine-learning/machine-learning-apps-anomaly-detection-api) 可在時間序列資料上偵測下列異常類型：

-   **尖峰和下降**：當您監視服務的登入失敗數目或電子商務網站的簽出數目時，不尋常的尖峰或下降可能表示遭到安全性攻擊或服務中斷。

-   **正面和負面趨勢**：當您監視運算中的記憶體使用量時，縮小可用的記憶體大小表示記憶體可能流失。 針對服務佇列長度監視，持續增加的趨勢可能表示基礎軟體問題。

-   **層級變更和動態值範圍中的變更**：服務升級後的服務延遲層級變更或升級後的例外狀況層級降低，都是值得監視的重點。

機器學習 API 能夠進行下列動作：

-   **具彈性且健全的偵測**：異常偵測模型允許使用者設定具敏感性的設定，並偵測季節性和非季節性資料集之間的異常行為。 使用者可以根據自己的需求來調整異常偵測模型，以降低或提高偵測 API 的敏感度。 這表示會在包含和不含季節性模式的資料中偵測較不常見或較常見的異常。

-   **可調整的即時偵測**：使用透過專家網域知識所設定之現有閾值進行監視的傳統方式既昂貴，又無法調整為數百萬個動態變更的資料集。 會學習此 API 中的異常偵測模型，而且會從歷程記錄和即時資料自動調整模型。

-   **主動且可採取動作的偵測**：緩慢的趨勢和層級變更偵測可應用於早期異常偵測。 偵測到的早期異常訊號可用來引導使用者調查及處理問題區域。 此外，還可以在此異常偵測 API 服務上，開發根本原因分析模型和警示工具。

異常偵測 API 是適用於各種案例之有效且有效率的解決方案，例如服務健康狀態和 KPI 監視、IoT、效能監視以及網路流量監視。 以下提供一些常見案例，此 API 在這類案例中非常實用：

- IT 部門需要工具來即時追蹤事件、錯誤碼、使用量記錄及效能 (CPU、記憶體等)。

-   線上電子商務網站想要追蹤客戶活動、頁面檢視、點擊次數等項目。

-   公用事業公司想要追蹤水、天然氣、電力和其他資源的耗用量。

-   設備或大樓管理服務想要監視溫度、濕度、流量等項目。

-   IoT/製造商想要使用時間序列中的感應器資料來監視工作流程、品質等項目。

-   服務提供者 (例如話務中心) 需要監視服務需求趨勢、事件量、等候佇列長度等項目。

-   商務分析群組想要即時監視商務 KPI (例如銷售量、客戶人氣或定價) 的異常移動。

### <a name="cloud-app-security"></a>Cloud App Security

[Cloud App Security](https://docs.microsoft.com/cloud-app-security/what-is-cloud-app-security) 是 Microsoft Cloud 安全性堆疊的一個重要元件。 它是全方位的解決方案，可協助您的組織在您移動時能夠充分運用雲端應用程式的承諾。 它透過提高對活動的可見度來保持控制。 它也有助於跨雲端應用程式提高對重要資料的保護。

利用有助於揭露影子 IT、評估風險、強制執行原則、調查活動及停止威脅的工具，您的組織可以更安全的移至雲端，同時仍能控制重要資料。

| | |
|---|---|
| 探索 | 利用 Cloud App Security 來揭露影子 IT。 藉由探索雲端環境中的應用程式、活動、使用者、資料和檔案，來取得可見度。 探索連接到您雲端的協力廠商應用程式。|
|調查 | 使用雲端鑑識工具深入探討有風險的應用程式、特定的使用者及您網路中的檔案，藉以調查您的雲端應用程式。 在收集自您雲端的資料中尋找模式。 產生報告來監視您的雲端。 |
| 控制 | 藉由設定原則和警示以便能充分控制網路雲端流量來降低風險。 使用 Cloud App Security，將您的使用者移轉至安全且獲批准的雲端應用程式替代項目。 |
| Protect | 使用 Cloud App Security 來批准或禁止應用程式、強制執行資料損失防範措施、控制權限和共用，並產生自訂報告和警示。 |
| 控制 | 藉由設定原則和警示以便能充分控制網路雲端流量來降低風險。 使用 Cloud App Security，將您的使用者移轉至安全且獲批准的雲端應用程式替代項目。 |
| | |


![Cloud App Security 圖表](./media/azure-threat-detection/azure-threat-detection-fig14.png)

Cloud App Security 會透過下列方式來整合雲端的可見性：

-   使用 Cloud Discovery 來對應和識別您的雲端環境，以及您組織所使用的雲端應用程式。

-   批准和禁止您雲端中的應用程式。

-   使用易於部署的應用程式連接器，利用提供者 API 來取得您所連接之應用程式的可見度和控管。

-   透過設定，接著持續微調原則，協助您持續進行控制。

從這些來源收集資料時，Cloud App Security 會對資料執行複雜分析。 它會在發生異常活動時立即警示您，並讓您能深入檢視您的雲端環境。 您可以在 Cloud App Security 中設定原則，並使用它來保護您雲端環境中的一切。

## <a name="third-party-advanced-threat-detection-capabilities-through-the-azure-marketplace"></a>透過 Azure Marketplace 的協力廠商進階威脅偵測功能

### <a name="web-application-firewall"></a>Web 應用程式防火牆

Web 應用程式防火牆會檢查輸入的 Web 流量和並封鎖 SQL 插入、跨網站指令碼、惡意程式碼上傳和應用程式 DDoS 攻擊，以及目標為您 Web 應用程式的其他攻擊。 它也會針對資料外洩防護 (DLP) 檢查來自後端 Web 伺服器的回應。 整合式存取控制引擎讓系統管理員能夠建立細微的存取控制原則以用於驗證、授權和帳戶處理 (AAA)，其可為組織提供增強式驗證和使用者控制。

Web 應用程式防火牆提供下列優點：

-   偵測並封鎖 SQL 插入、跨網站指令碼、惡意程式碼上傳、應用程式 DDoS 或針對您應用程式的任何其他攻擊。

-   驗證和存取控制。

-   掃描輸出流量以偵測敏感性資料，並且可為資訊加上遮罩或封鎖以防止外洩。

-   使用快取、壓縮及其他流量最佳化等功能，來加速 Web 應用程式內容的傳遞。

如需 Azure Marketplace 中可用 Web 應用程式防火牆的範例，請參閱 [Barracuda WAF, Brocade virtual web application firewall (vWAF), Imperva SecureSphere, and the ThreatSTOP IP firewall](https://azuremarketplace.microsoft.com/marketplace/apps/barracudanetworks.waf) (Barracuda WAF、Brocade 虛擬 Web 應用程式防火牆 (vWAF)、Imperva SecureSphere 和 ThreatSTOP IP 防火牆)。

## <a name="next-steps"></a>後續步驟

- [Azure 資訊安全中心偵測功能](https://docs.microsoft.com/azure/security-center/security-center-detection-capabilities)：協助識別以您的 Azure 資源為目標的作用中威脅，並提供您快速回應所需的見解。

- [Azure SQL Database 威脅偵測](https://azure.microsoft.com/blog/azure-sql-database-threat-detection-your-built-in-security-expert/)：協助解決您對資料庫潛在威脅的疑慮。
