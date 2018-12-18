---
title: 混合式身分識別設計 - Azure 資料保護策略 | Microsoft Docs
description: 您可為混合式身分識別解決方案定義資料保護策略，以符合您已定義的商務需求。
documentationcenter: ''
services: active-directory
author: billmath
manager: mtillman
editor: ''
ms.assetid: e76fd1f4-340a-492a-84d9-e05f3b7cc396
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/13/2017
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: 7999dc15fbe1af9d23b47971db03cf6b197f110f
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306107"
---
# <a name="define-data-protection-strategy-for-your-hybrid-identity-solution"></a>定義混合式身分識別解決方案的資料保護策略
在這項工作中，您將為混合式身分識別解決方案定義資料保護策略，以符合您已定義的商務需求：

* [判斷資料保護需求](plan-hybrid-identity-design-considerations-dataprotection-requirements.md)
* [判斷內容管理需求](plan-hybrid-identity-design-considerations-contentmgt-requirements.md)
* [判斷存取控制需求](plan-hybrid-identity-design-considerations-accesscontrol-requirements.md)
* [判斷事件因應需求](plan-hybrid-identity-design-considerations-incident-response-requirements.md)

## <a name="define-data-protection-options"></a>定義資料保護選項
如[判斷目錄同步處理需求](plan-hybrid-identity-design-considerations-directory-sync-requirements.md)中所述，Microsoft Azure AD 可以與您內部部署的 Active Directory Domain Services (AD DS) 同步處理。 這項整合可讓組織在使用者嘗試存取公司資源時，利用 Azure AD 來驗證使用者的認證。 您可以針對下列兩種案例使用此作法：內部部署和雲端中的待用資料。 要存取 Azure AD 中的資料，必須透過 Security Token Service (STS) 進行使用者驗證。

驗證之後，可從驗證權杖讀取使用者主體名稱 (UPN)。 然後，授權系統會判斷對應到使用者網域的複寫分割區和容器。 使用者的存在情形、啟用狀態和角色的相關資訊，可協助授權系統判斷這名使用者在此工作階段中對目標租用戶的存取是否已獲授權。 特定的授權動作 (具體而言，如建立使用者和密碼重設) 會建立可供租用戶管理員用來管理法規遵循工作或調查的稽核記錄。

由於資料數量、頻寬可用性或其他考量，透過網際網路連線將資料從您的內部部署資料中心移至 Azure 儲存體中，並不一定永遠可行。 [Azure 儲存體匯入/匯出服務](../../storage/common/storage-import-export-service.md) 提供以硬體為基礎的選項，可讓您在 Blob 儲存體中放置/擷取大量資料。 它可讓您將 [BitLocker 加密](https://technet.microsoft.com/library/dn306081#BKMK_BL2012R2)硬碟直接傳送至 Azure 資料中心，而雲端操作員會在此處將內容上傳至您的儲存體帳戶，或是將您的 Azure 資料下載到您的磁碟機歸還給您。 在此程序中只能使用加密磁碟 (使用服務本身在工作設定期間產生的 BitLocker 金鑰)。 BitLocker 金鑰會個別提供給 Azure，藉此提供頻外金鑰共用。

由於傳輸中的資料可能在不同情況下進行，因此也需要了解 Microsoft Azure 會使用 [虛擬網路](https://azure.microsoft.com/documentation/services/virtual-network/) 隔離租用戶彼此之間的流量，它所採用的機制有可能是主機和客體層級防火牆、IP 封包篩選、連接埠封鎖和 HTTPS 端點等。 不過，Azure 的內部通訊大多也會加密，包括基礎結構對基礎結構和基礎結構對客戶 (內部部署) 的通訊。 另一個重要的案例是 Azure 資料中心內的通訊；Microsoft 會管理網路，以確保沒有 VM 可以模擬或竊聽其他 VM 的 IP 位址。 在存取 Azure 儲存體或 SQL Database，或是在連接到雲端服務時，會使用 TLS/SSL。 在此情況下，客戶的系統管理員會負責取得 TLS/SSL 憑證，並將其部署至其租用戶基礎結構。 透過 Microsoft Azure 虛擬網路在相同部署中的虛擬機器之間或單一部署中的租用戶之間移動的資料流量，可透過加密的通訊協定 (例如 HTTPS、SSL/TLS 或其他通訊協定) 受到保護。

根據您在[判斷資料保護需求](plan-hybrid-identity-design-considerations-dataprotection-requirements.md)中回答問題的情形，您應該能夠判斷如何保護您的資料，以及混合式身分識別解決方案如何協助進行該處理程序。 下表顯示 Azure 所支援並且可用於各種資料保護案例的選項。

| 資料保護選項 | 在雲端中待用 | 在內部部署中待用 | 傳輸中 |
| --- | --- | --- | --- |
| BitLocker 磁碟機加密 |X |X | |
| 要加密資料庫的 SQL Server |X |X | |
| VM 對 VM 加密 | | |X |
| SSL/TLS | | |X |
| VPN | | |X |

> [!NOTE]
> 請閱讀 [Microsoft Azure 信任中心](https://azure.microsoft.com/support/trust-center/)的依功能[遵循法規](https://azure.microsoft.com/support/trust-center/services/)，以深入了解每個 Azure 服務的相容憑證。
> 由於資料保護的選項採用多層式方法，因此在執行這項工作時無法比較這些選項。 請確實針對資料所處的各種狀態，運用所有可用的選項。
>
>

## <a name="define-content-management-options"></a>定義內容管理選項

<a name="one-advantage-of-using-azure-ad-to-manage-a-hybrid-identity-infrastructure-is-that-the-process-is-fully-transparent-from-the-end-users-perspective-the-user-tries-to-access-a-shared-resource-the-resource-requires-authentication-the-user-has-to-send-an-authentication-request-to-azure-ad-in-order-to-obtain-the-token-and-access-the-resource-this-entire-process-happens-in-the-background-without-user-interaction"></a>使用 Azure AD 來管理混合式身分識別基礎結構的優點之一，是整個程序對使用者而言是完全透明的。 使用者會嘗試存取共用資源，而資源需要驗證，因此使用者必須將驗證要求傳送至 Azure AD，才能夠取得權杖並存取資源。 整個程序都會在背景執行，使用者無須動作。 
=======
使用 Azure AD 來管理混合式身分識別基礎結構的優點之一，是整個程序對使用者而言是完全透明的。 使用者會嘗試存取共用資源，而資源需要驗證，因此使用者必須將驗證要求傳送至 Azure AD，才能夠取得權杖並存取資源。 整個程序都會在背景執行，使用者無須動作。 

有資料隱私權顧慮的組織通常需要對其解決方案進行資料分類。 如果其目前的內部部署基礎結構已使用資料分類，就能夠利用 Azure AD 作為使用者身分識別的主要儲存機制。 在 Windows Server 2012 R2 中，內部部署中用於資料分類的常用工具稱為 [資料分類工具組](https://msdn.microsoft.com/library/Hh204743.aspx) 。 這項工具有助於您對私人雲端中的檔案伺服器進行資料的識別、分類及保護。 您也可利用 Windows Server 2012 中的[自動檔案分類](https://technet.microsoft.com/library/hh831672.aspx)來完成這項工作。

如果您的組織目前沒有資料分類機制，但需要在內部部署不新增伺服器的情況下保護敏感檔案，他們可以使用 Microsoft [Azure Rights Management Service](https://technet.microsoft.com/library/JJ585026.aspx)。  Azure RMS 會使用加密、身分識別和授權原則協助您保護檔案和電子郵件，並且可跨多個裝置運作 — 手機、平板電腦和 PC。 Azure RMS 屬於雲端服務，因此您無須明確設定對其他組織的信任，即可與他們共用受保護的內容。 如果他們已有 Office 365 或 Azure AD 目錄，則會自動支援跨組織的共同作業。 您也可以使用 Azure Active Directory 同步處理服務 (AAD 同步) 或 Azure AD Connect，僅同步處理 Azure RMS 支援您內部部署 Active Directory 帳戶的一般身分識別所需的目錄屬性。

了解誰正在存取哪個資源，是內容管理很重要的一部分，因此強大的記錄功能對於身分識別管理解決方案而言，是很重要的。 Azure AD 提供超過 30 天的記錄，其中包括：

* 角色成員資格的變更 (例如：新增至全域管理員角色的使用者)
* 認證更新 (例如：密碼變更)
* 網域管理 (例如：驗證自訂網域、移除網域)
* 新增或移除應用程式
* 使用者管理 (例如：新增、移除、更新使用者)
* 新增或移除授權

> [!NOTE]
> 請閱讀 [Microsoft Azure 安全性和稽核記錄管理](http://download.microsoft.com/download/B/6/C/B6C0A98B-D34A-417C-826E-3EA28CDFC9DD/AzureSecurityandAuditLogManagement_11132014.pdf) ，以深入了解 Azure 中的記錄功能。
> 根據您在 [判斷內容管理需求](plan-hybrid-identity-design-considerations-contentmgt-requirements.md)中回答問題的情形，您應該能夠判斷在混合式身分識別解決方案中要如何保護您的內容。 雖然表 6 中顯示的所有選項都都能夠與 Azure AD 整合，但請務必定義何者較適用於您的商務需求。
>
>

| 內容管理選項 | 優點 | 缺點 |
| --- | --- | --- |
| 集中式的內部部署 (Active Directory Rights Management Server) |完全掌控負責分類資料的伺服器基礎結構  <br> Windows Server 中內建功能，不需要取得額外授權或訂用帳戶 <br> 可以與混合式案例中的 Azure AD 整合 <br> 支援 Microsoft Online Services，例如 Exchange Online 和 SharePoint Online 以及 Office 365 中的資訊版權管理 (IRM) 功能 <br> 支援內部部署 Microsoft 伺服器產品，例如 Exchange Server、SharePoint Server，以及執行 Windows Server 和檔案分類基礎結構 (FCI) 的檔案伺服器。 |需要較多維護 (隨時進行更新、設定與潛在升級)，因為 IT 人員擁有伺服器 <br> 需要內部部署的伺服器基礎結構<br> 在原生狀態下不會使用 Azure 功能 |
| 集中於雲端 (Azure RMS) |比內部部署解決方案容易管理  <br> 可以與混合式案例中的 AD DS 整合 <br>  完全與 Azure AD 整合 <br> 不需要內部部署伺服器，即可部署服務 <br> 支援內部部署 Microsoft 伺服器產品，例如 Exchange Server、SharePoint Server，以及執行 Windows Server 和檔案分類基礎結構 (FCI) 的檔案伺服器 <br> IT 人員可透過 BYOK 功能完全控制其租用戶的金鑰。 |組織必須具有支援 RMS 的雲端訂用帳戶  <br> 組織必須具有 Azure AD 目錄以支援 RMS 的使用者驗證 |
| 混合式 (Azure RMS 與內部部署 Active Directory Rights Management Server 整合) |這種情況下結合了集中式內部部署和雲端兩者的優點。 |組織必須具有支援 RMS 的雲端訂用帳戶  <br> 組織必須具有 Azure AD 目錄以支援 RMS 的使用者驗證， <br> Azure 雲端服務與內部部署基礎結構之間必須要有連線 |

## <a name="define-access-control-options"></a>定義存取控制選項
利用 Azure AD 中提供的驗證、授權和存取控制功能，您將能夠讓公司使用中央身分識別存放庫，同時允許使用者和合作夥伴使用單一登入 (SSO)，如下圖所示：

![](./media/plan-hybrid-identity-design-considerations/centralized-management.png)

集中式管理以及與其他目錄的完全整合

Azure Active Directory 可為數千個 SaaS 應用程式和內部部署 Web 應用程式提供單一登入。 如需經過 Microsoft 測試之 SSO 協力廠商的詳細資訊，請參閱 [Azure Active Directory 同盟相容性清單：可用來實作單一登入的協力廠商識別提供者](https://msdn.microsoft.com/library/azure/jj679342.aspx)一文。 這項功能可讓組織實作各種 B2B 案例，同時得以控制身分識別和存取管理。 不過，在 B2B 期間，要了解合作夥伴所將使用的驗證方法驗證，並驗證 Azure 是否支援此方法，設計程序是很重要的。 目前，Azure AD 支援下列方法：

* 安全性判斷提示標記語言 (SAML)
* OAuth
* Kerberos
* 權杖
* 憑證

> [!NOTE]
> 請閱讀 [Azure Active Directory 驗證通訊協定](https://msdn.microsoft.com/library/azure/dn151124.aspx) ，以深入了解 Azure 中的每個通訊協定及其功能。
>
>

藉由 Azure AD 支援，行動商業應用程式可使用相同的簡單行動服務驗證功能，讓員工利用公司的 Active Directory 認證登入其行動應用程式。 透過這項功能，Azure AD 可受到支援作為行動服務中的識別提供者，與已支援的其他識別提供者 (包括 Microsoft 帳戶、Facebook ID、Google ID 和 Twitter ID) 搭配運作。 如果內部部署應用程式使用位於公司 AD DS 的使用者認證，則來自雲端的協力廠商和使用者所做的存取應該是透明的。 您可以管理使用者對 (以雲端為基礎的) Web 應用程式、Web API、Microsoft 雲端服務、協力廠商 SaaS 應用程式和原生 (行動) 用戶端應用程式的條件式存取控制，並且可因為安全性、稽核和報告全都集中在同一處而獲益。 不過，建議您在非生產環境中或使用者數量有限的環境中驗證此實作。

> [!TIP]
> 務必要提到的是，Azure AD 不像 AD DS 一樣具有群組原則。 若要為裝置強制執行原則，您需要有行動裝置管理解決方案，例如 [Microsoft Intune](https://technet.microsoft.com/library/jj676587.aspx)。
>
>

使用 Azure AD 驗證使用者之後，務必要評估使用者所將擁有的存取層級。 使用者對資源的存取層級可能有所不同。 由於 Azure AD 可藉由控制對某些資源的存取而增加額外的安全性層級，因此請記住，資源本身也可能有其本身的存取控制清單，例如，對位於檔案伺服器中之檔案的存取控制。 下圖摘要說明您在混合式案例中可以有的存取控制層級：

![](./media/plan-hybrid-identity-design-considerations/accesscontrol.png)

圖 X 中顯示的每個互動，分別代表 Azure AD 可以涵蓋的一個存取控制案例。 每個案例的說明如下：

  1. 內部部署裝載之應用程式的條件式存取：您可以使用已註冊的裝置搭配適用於已設定為使用 AD FS with Windows Server 2012 R2 之應用程式的存取原則。

  2. Azure 入口網站的存取控制：Azure 也可讓您使用角色型存取控制 (RBAC) 來控制對入口網站的存取。 此方法可讓公司限制個人可以在 Azure 入口網站中執行的作業數目。 使用 RBAC 來控制存取入口網站時，IT 系統管理員可以利用下列存取管理方法來委派存取：

   - 群組型角色指派：您可以指派存取權給可從本機 Active Directory 同步處理的 Azure AD 群組。 這可讓您運用組織目前在群組管理工具和程序方面所做的投資。 您也可以使用 Azure AD Premium 的委派群組管理功能。
   - 運用 Azure 中內建的角色：您可以使用三個角色 — 擁有者、參與者和讀者，以確保使用者和群組只擁有他們執行工作所需的權限。
   -  細微控制資源的存取：您可以針對特定的訂用帳戶、資源群組或個別 Azure 資源 (例如網站或資料庫)，將角色指派給使用者和群組。 如此一來，您可以確保使用者能夠存取他們需要的資源，但不能存取他們不需要管理的資源。

   > [!NOTE]
   > 如果您要建置應用程式並想要自訂其存取控制，可以使用 Azure AD 應用程式角色來進行授權。 請參閱 [WebApp-RoleClaims-DotNet 範例](https://github.com/AzureADSamples/WebApp-RoleClaims-DotNet) ，了解如何建置您的應用程式以使用這項功能。


  3. 適用於包含 Microsoft Intune 的 Office 365 應用程式條件式存取 ：IT 管理員可以佈建條件式存取裝置原則來保護公司資源，同時允許符合規範之裝置上的資訊工作者存取服務。 
  
  4. Saas 應用程式的條件式存取：[此功能](https://cloudblogs.microsoft.com/enterprisemobility/2015/06/25/azure-ad-conditional-access-preview-update-more-apps-and-blocking-access-for-users-not-at-work/)可讓您設定每個應用程式的多重要素驗證存取規則，且能夠封鎖不在受信任網路上的使用者存取。 您可以將多因素驗證規則套用至所有已指派給應用程式的使用者，或只套用至指定的安全性群組內的使用者。 如果使用者是從組織網路內的 IP 位址存取應用程式，則可從多因素驗證需求中排除這些使用者。

由於存取控制的選項採用多層式方法，因此在執行這項工作時無法比較這些選項。 請確實針對每個要求您控制資源存取權的案例，使用所有適用的選項。

## <a name="define-incident-response-options"></a>定義事件回應選項
Azure AD 可藉由監視使用者活動，協助 IT 人員識別環境中潛在的安全性風險。 IT 人員可以使用 Azure AD 的存取和使用情況報告來了解貴組織的目錄完整性和安全性。 利用此資訊，IT 管理員更能夠判斷可能發生安全性風險的位置，以便適當地規劃來減輕這些風險。  [Azure AD Premium 訂用帳戶](../fundamentals/active-directory-get-started-premium.md) 有一組安全性報告可讓 IT 人員取得這項資訊。 [Azure AD 報告](../reports-monitoring/overview-reports.md) 歸類如下：

* **異常報告**：包含發現異常的登入事件。 目標在於使您注意這類活動，並讓您能夠判斷事件是否可疑。
* **整合式應用程式報告**：可供深入了解雲端應用程式在組織中的使用方式。 Azure Active Directory 提供與數千個雲端應用程式的整合。
* **錯誤報告**：指出將帳戶佈建至外部應用程式時可能發生的錯誤。
* **使用者特定報告**：顯示特定使用者的裝置/登入活動資料。
* **活動記錄檔**：包含過去 24 小時、過去 7 天或過去 30 天內所有稽核事件的記錄，以及群組活動變更、密碼重設和登錄活動。

> [!TIP]
> 也有助於事件回應小組處理案例的另一份報告，是 [認證外洩的使用者](https://cloudblogs.microsoft.com/enterprisemobility/2015/06/15/azure-active-directory-premium-reporting-now-detects-leaked-credentials/) 報告。 此報告會呈現外洩的認證清單與您的租用戶之間的任何相符項目。
>


Azure AD 中還有其他可在事件回應調查期間使用的重要內建報告，包括：

* **密碼重設活動**：可讓系統管理員深入了解組織中使用密碼重設的頻率。
* **密碼重設註冊活動**：可讓您深入了解哪些使用者註冊了密碼重設方法，以及他們選取了哪些方法。
* **群組活動**：提供在 [存取面板] 中對群組起始之變更的歷程記錄 (例如：新增或移除使用者)。

除了可在事件回應調查程序期間使用之 Azure AD Premium 的核心報告功能以外，IT 人員也可利用稽核報告取得資訊，例如：

* 角色成員資格的變更 (例如，新增至全域管理員角色的使用者)
* 認證更新 (例如，密碼變更)
* 網域管理 (例如，驗證自訂網域、移除網域)
* 新增或移除應用程式
* 使用者管理 (例如，新增、移除、更新使用者)
* 新增或移除授權

由於事件回應的選項採用多層式方法，因此在執行這項工作時無法比較這些選項。 請確實針對每個要求您在公司的事件回應程序中使用 Azure AD 報告功能的案例，使用所有適用的選項。

## <a name="next-steps"></a>後續步驟
[判斷混合式身分識別管理工作](plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md)

## <a name="see-also"></a>另請參閱
[設計考量概觀](plan-hybrid-identity-design-considerations-overview.md)
