---
title: 適用于 Azure 防火牆管理員的 Azure 安全性基準
description: Azure 防火牆管理員安全性基準提供的程式指引和資源，可讓您執行 Azure 安全性基準測試中所指定的安全性建議。
author: msmbaldwin
ms.service: firewall-manager
ms.topic: conceptual
ms.date: 11/24/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: eca243a9a83ad5d2bfa006b46499470cf06e808a
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98203026"
---
# <a name="azure-security-baseline-for-azure-firewall-manager"></a>適用于 Azure 防火牆管理員的 Azure 安全性基準

此安全性基準會將 [Azure 安全性基準測試版本 2.0](../security/benchmarks/overview.md) 的指引套用至 Azure 防火牆管理員。 Azure 安全性基準提供如何在 Azure 上保護雲端解決方案的建議。 內容會依 Azure 安全性基準測試所定義的 **安全性控制** ，以及適用于 Azure 防火牆管理員的相關指引來分組。 已排除不適用 Azure 防火牆管理員的 **控制項**。

若要查看 Azure 防火牆管理員如何完全對應至 Azure 安全性基準測試，請參閱 [完整的 Azure 防火牆管理員安全性基準對應](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines)檔案。

## <a name="identity-management"></a>身分識別管理

如需詳細資訊，請參閱 [Azure 安全性效能評定：身分識別管理](/azure/security/benchmarks/security-controls-v2-identity-management)。

### <a name="im-1-standardize-azure-active-directory-as-the-central-identity-and-authentication-system"></a>IM-1：將 Azure Active Directory 標準化為中央身分識別與驗證系統

**指導** 方針： Azure 防火牆管理員使用 Azure Active Directory (Azure AD) 作為預設身分識別和存取管理服務。 您應該將 Azure AD 標準化，以管理組織的身分識別和存取管理：

- Microsoft 雲端資源，例如 Azure 入口網站、Azure 儲存體、Azure 虛擬機器 (Linux 和 Windows) 、Azure Key Vault、PaaS 和 SaaS 應用程式。

- 您的組織資源，例如 Azure 上的應用程式或您的公司網路資源。

保護 Azure AD 在組織的雲端安全性實務中，應具有較高的優先順序。 Azure AD 提供身分識別安全分數，協助您評定相對於 Microsoft 最佳做法建議的身分識別安全性狀態。 請使用此分數來測量設定符合最佳做法建議的程度，並改善您的安全性狀態。

Azure AD 支援外部身分識別，讓沒有 Microsoft 帳戶的使用者可以使用其外部身分識別登入其應用程式和資源。

- [Azure Active Directory 中的租用](../active-directory/develop/single-and-multi-tenant-apps.md)

- [如何建立及設定 Azure AD 執行個體](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

- [使用應用程式的外部識別提供者](/azure/active-directory/b2b/identity-providers) (機器翻譯)

- [Azure Active Directory 中的身分識別安全分數為何](../active-directory/fundamentals/identity-secure-score.md) (機器翻譯)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="im-3-use-azure-ad-single-sign-on-sso-for-application-access"></a>IM-3：使用 Azure AD 單一登入 (SSO) 進行應用程式存取

**指導** 方針： Azure 防火牆管理員使用 Azure Active Directory 為 Azure 資源、雲端應用程式和內部部署應用程式提供身分識別和存取管理。 這包括企業身分識別 (例如員工)，以及外部身分識別 (例如合作夥伴、廠商和供應商)。 這可讓單一登入 (SSO) 管理及保護內部部署和雲端中的組織資料與資源存取。 請將您的所有使用者、應用程式和裝置連線到 Azure AD，以進行順暢且安全的存取，並提供更高的可見度和控制。

- [了解 Azure AD 的應用程式 SSO](../active-directory/manage-apps/what-is-single-sign-on.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="im-4-use-strong-authentication-controls-for-all-azure-active-directory-based-access"></a>IM-4：針對所有以 Azure Active Directory 為基礎的存取，使用增強式驗證控制項

**指導** 方針： Azure 防火牆管理員使用 Azure Active Directory，透過多重要素驗證 (MFA) 和強式無密碼方法，來支援強式驗證控制項。
- 多重要素驗證 - 啟用 Azure AD MFA，並遵循 Azure 資訊安全中心身分識別與存取權管理建議，以取得 MFA 設定中的一些最佳做法。 MFA 可以根據登入條件和風險因素，在所有、選取使用者或每個使用者層級上施行。
- 無密碼驗證 – 有三種無密碼驗證選項可供使用：Windows Hello 企業版、Microsoft Authenticator 應用程式和內部部署驗證方法 (例如智慧卡)。

針對系統管理員和特殊許可權的使用者，請確定已使用強式驗證方法的最高層級，接著向其他使用者推出適當的增強式驗證原則。

- [如何在 Azure 中啟用 MFA](../active-directory/authentication/howto-mfa-getstarted.md) 

- [Azure Active Directory 的無密碼驗證選項簡介](../active-directory/authentication/concept-authentication-passwordless.md)

- [Azure AD 預設密碼原則](../active-directory/authentication/concept-sspr-policy.md#password-policies-that-only-apply-to-cloud-user-accounts)

- [使用 Azure AD 密碼保護排除不正確的密碼](../active-directory/authentication/concept-password-ban-bad.md)

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="im-5-monitor-and-alert-on-account-anomalies"></a>IM-5：監視及警示帳戶異常

**指導** 方針： Azure 防火牆管理員與 Azure Active Directory 整合，其中提供下列資料來源：
- 登入 - 登入報表會提供受控應用程式和使用者登入活動的使用情況相關資訊。
- 稽核記錄 - 可針對各種功能在 Azure AD 內進行的所有變更，提供記錄追蹤功能。 稽核記錄的範例包括對 Azure AD 中任何資源所做的變更，像是新增或移除使用者、應用程式、群組、角色和原則。
- 有風險的登入 - 有風險的登入表示非使用者帳戶合法擁有者的某人嘗試登入。
- 標幟為有風險的使用者 - 有風險的使用者表示可能被盜用的使用者帳戶。

這些資料來源可以與 Azure 監視器、Azure Sentinel 或協力廠商 SIEM 系統整合。

活動記錄檔不支援目前關於防火牆原則規則集合群組的動作，這是已知的問題，在未來的更新中已解決。

Azure 資訊安全中心也可對特定的可疑活動發出警示，例如驗證嘗試失敗次數過多，訂用帳戶中已淘汰的帳戶。

Azure 進階威脅防護 (ATP) 是一項安全性解決方案，可使用 Active Directory 訊號來識別、偵測及調查進階威脅、遭入侵的身分識別，以及惡意的內部人員動作。

- [Azure Active Directory 中的稽核活動報表](../active-directory/reports-monitoring/concept-audit-logs.md)

- [如何檢視有風險的 Azure AD 登入](/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

- [如何在 Azure 資訊安全中心監視使用者的身分識別和存取活動](../security-center/security-center-identity-access.md)

- [如何將 Azure 活動記錄整合到 Azure 監視器中](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

- [Azure 防火牆管理員的已知問題](overview.md#known-issues)

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="im-6-restrict-azure-resource-access-based-on-conditions"></a>IM-6：根據條件限制 Azure 資源存取

**指導** 方針：根據使用者定義的條件，Azure 防火牆管理員支援 Azure Active Directory (Azure AD) 條件式存取，例如需要使用 MFA 進行登入的特定 IP 範圍的使用者登入。 細微的驗證工作階段管理原則也可用於不同的使用案例。

- [Azure 條件式存取概觀](../active-directory/conditional-access/overview.md)

- [一般條件式存取原則](../active-directory/conditional-access/concept-conditional-access-policy-common.md) (機器翻譯)

- [使用條件式存取來設定驗證工作階段管理](../active-directory/conditional-access/howto-conditional-access-session-lifetime.md) (機器翻譯)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

## <a name="privileged-access"></a>特殊權限存取

如需詳細資訊，請參閱 [Azure 安全性效能評定：特殊權限存取](/azure/security/benchmarks/security-controls-v2-privileged-access)。

### <a name="pa-1-protect-and-limit-highly-privileged-users"></a>PA-1：保護及限制高權限使用者

**指導** 方針： Azure 防火牆管理員使用 Azure Active Directory (Azure AD) 進行身分識別和存取。 最重要的內建角色是 Azure AD 全域管理員和特殊許可權角色管理員，因為指派給這兩個角色的使用者可以委派系統管理員角色：
- 全域管理員/公司系統管理員：具備此角色的使用者可以存取 Azure AD 中的所有管理功能，以及使用 Azure AD 身分識別的服務。
- 特殊許可權角色管理員：具備此角色的使用者可以在 Azure AD 中管理角色指派，也可以在 Azure AD Privileged Identity Management (PIM) 內管理。 此外，此角色可讓您管理 PIM 和管理單位的所有層面。

如果您使用自訂角色搭配指派的特定特殊許可權許可權，您可能會有其他重要角色需要進行控管。 此外，您也可能想要將類似的控制項套用至重要商務資產的系統管理員帳戶。

您可以使用 Azure AD Privileged Identity Management (PIM)，啟用對 Azure 資源和 Azure AD 的 Just-In-Time (JIT) 特殊權限存取。 JIT 只有在使用者需要時，才會授與暫時性權限以執行特殊權限的工作。 當您的 Azure AD 組織中有可疑或不安全的活動時，PIM 也會產生安全性警示。

- [Azure AD 中的系統管理員角色權限](/azure/active-directory/users-groups-roles/directory-assign-admin-roles)

- [使用 Azure Privileged Identity Management 安全性警示](../active-directory/privileged-identity-management/pim-how-to-configure-security-alerts.md)

- [在 Azure AD 中保護混合式部署和雲端部署的特殊權限存取](/azure/active-directory/users-groups-roles/directory-admin-roles-secure)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="pa-2-restrict-administrative-access-to-business-critical-systems"></a>PA-2：限制對業務關鍵系統的系統管理存取

**指導** 方針：使用自訂 azure RBAC 角色來隔離 Azure 防火牆原則的規則集合群組存取權。 使用 Azure 自訂角色定義，以防止意外移除基底原則，並在訂用帳戶或資源群組內提供選擇性存取規則集合群組。

確定您也會限制對您的業務關鍵系統具有系統管理存取權的管理、身分識別和安全性系統的存取權。 入侵這些管理和安全性系統的攻擊者，可以立即 weaponize 它們來入侵商務關鍵資產。

所有類型的存取控制都應符合您的企業分割策略，以確保存取控制的一致性。

- [使用 Azure 防火牆原則來定義規則階層](rule-hierarchy.md)

- [Azure 元件和參考模型](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151)

- [管理群組存取](../governance/management-groups/overview.md#management-group-access)

- [Azure 訂用帳戶管理員](../cost-management-billing/manage/add-change-subscription-administrator.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="pa-3-review-and-reconcile-user-access-regularly"></a>PA-3：定期檢閱及協調使用者存取

**指導** 方針： Azure 防火牆管理員使用 Azure Active Directory (Azure AD) 帳戶來管理其資源。 定期審核使用者帳戶和存取指派，以確保帳戶及其存取權都有效。 您可以使用 Azure AD 存取評論來查看群組成員資格、企業應用程式的存取權，以及角色指派。 Azure AD 報告可以提供記錄以協助探索過時的帳戶。 您也可以使用 Azure AD Privileged Identity Management 建立存取權審核報表工作流程，以加速審核程式。

此外，Azure Privileged Identity Management 也可以設定為在建立過多的系統管理員帳戶時發出警示，以及識別已過時或設定不正確的系統管理員帳戶。

某些 Azure 服務支援無法透過 Azure AD 管理的本機使用者和角色。 您將需要分開管理這些使用者。

- [在 Privileged Identity Management (PIM) 中建立 Azure 資源角色的存取權審核 ](../active-directory/privileged-identity-management/pim-resource-roles-start-access-review.md) 

- [如何使用 Azure AD 身分識別和存取權檢閱](../active-directory/governance/access-reviews-overview.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="pa-4-set-up-emergency-access-in-azure-ad"></a>PA-4：在 Azure AD 中設定緊急存取

**指導** 方針： Azure 防火牆管理員使用 Azure Active Directory 來驗證管理服務的使用者。 若要避免不小心遭到 Azure AD 的組織封鎖，請在無法使用一般系統管理帳戶時，設定緊急存取帳戶以進行存取。 緊急存取帳戶通常具有高權限，不應將其指派給特定個人。 緊急存取帳戶僅限用於無法使用一般系統管理帳戶的緊急或「急用」狀況。

您應該確保緊急存取帳戶的認證 (例如密碼、憑證或智慧卡) 受到保護，而且只有在緊急情況下有權使用這些認證的個人才會知道。

- [在 Azure AD 中管理緊急存取帳戶](/azure/active-directory/users-groups-roles/directory-emergency-access)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="pa-5-automate-entitlement-management"></a>PA-5：自動化權利管理 

**指導** 方針： Azure 防火牆管理員與 Azure Active Directory 整合以管理其資源。 使用 Azure AD 權利管理功能來自動化存取要求工作流程，包括存取權指派、評論和到期日。 也支援雙重或多階段核准。

- [什麼是 Azure AD 存取權評論](../active-directory/governance/access-reviews-overview.md)

- [什麼是 Azure AD 權利管理](../active-directory/governance/entitlement-management-overview.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="pa-6-use-privileged-access-workstations"></a>PA-6：使用特殊權限存取工作站

**指引**：安全、隔離的工作站對於敏感性角色 (例如系統管理員、開發人員及重要服務操作員) 的安全性來說至關重要。 使用高度安全的使用者工作站，在生產環境中使用您的 Azure 防火牆管理員資源來執行系統管理管理工作。 請使用 Azure Active Directory、Microsoft Defender 進階威脅防護 (ATP) 和/或 Microsoft Intune，以部署安全且受控的使用者工作站來進行系統管理工作。 受保護的工作站可以集中管理以強制執行安全的設定，包括強式驗證、軟體和硬體基準，以及受限的邏輯和網路存取。

- [瞭解特殊許可權的存取工作站](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [部署特殊權限存取工作站](../active-directory/devices/howto-azure-managed-workstation.md) (機器翻譯)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="pa-7-follow-just-enough-administration-least-privilege-principle"></a>PA-7：遵循恰到好處的系統管理 (最低權限原則) 

**指導** 方針： Azure 防火牆管理員已與 Azure 角色型存取控制整合 (RBAC) 來管理其資源。 Azure RBAC 可讓您透過角色指派來管理 Azure 資源存取。 您可以將這些角色指派給使用者、群組服務主體和受控識別。 某些資源有預先定義的內建角色，這些角色可透過 Azure CLI、Azure PowerShell 或 Azure 入口網站之類的工具進行清查或查詢。 透過 Azure RBAC 指派給資源的權限，應一律限定為角色所需的權限。 這類權限可補強 Azure AD Privileged Identity Management (PIM) 的 Just-In-Time (JIT) 方法，且您應定期加以檢閱。

請使用內建角色來配置權限，且僅在必要時才建立自訂角色。

- [什麼是 Azure 角色型存取控制 (Azure RBAC) ](../role-based-access-control/overview.md) 

- [如何在 Azure 中設定 RBAC](../role-based-access-control/role-assignments-portal.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

## <a name="data-protection"></a>資料保護

如需詳細資訊，請參閱 [Azure 安全性效能評定：資料保護](/azure/security/benchmarks/security-controls-v2-data-protection)。

### <a name="dp-2-protect-sensitive-data"></a>DP-2：保護敏感性資料

**指導** 方針：使用 azure 角色型存取控制來限制 Azure 防火牆原則的設定資料等機密資料， (azure RBAC) 、網路型存取控制，以及 azure 服務中的特定控制項。

為確保存取控制的一致性，所有類型的存取控制都應與您的企業分割策略相符。 企業分割策略也應傳達至敏感性或業務關鍵資料和系統的所在位置。

針對 Microsoft 管理的基礎平台，Microsoft 會將所有客戶內容視為敏感性資訊，並防範客戶資料外洩和暴露。 為確保 Azure 中的客戶資料安全無虞，Microsoft 已實作某些預設的資料保護控制項和功能。

- [Azure 角色型存取控制 (RBAC)](../role-based-access-control/overview.md)

- [瞭解 Azure 中的客戶資料保護](../security/fundamentals/protection-customer-data.md)

**Azure 資訊安全中心監視**：不適用

**責任**：共用

### <a name="dp-3-monitor-for-unauthorized-transfer-of-sensitive-data"></a>DP-3：監視未經授權的敏感性資料傳輸

**指導** 方針：只有經過驗證的使用者才能存取 Azure 防火牆原則資源。 客戶必須確定只有獲得授權的使用者才能存取資料。

- [建立自訂角色以存取規則集合群組](rule-hierarchy.md#create-custom-roles-to-access-the-rule-collection-groups)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="dp-4-encrypt-sensitive-information-in-transit"></a>DP-4：加密傳輸中的敏感性資訊

**指導** 方針：若要補充存取控制，傳輸中的資料應該受到保護以防止「頻外」攻擊 (例如，使用加密的流量捕捉) ，以確保攻擊者無法輕易地讀取或修改資料。

Azure 防火牆管理員支援以 TLS 1.2 或更新版本傳輸的資料加密。

雖然這對私人網路的流量是選擇性的，但對於外部和公用網路上的流量而言是不可或缺的。 針對 HTTP 流量，請確定任何連線至 Azure 資源的用戶端都可以協調 TLS 1.2 或更新版本。 若要進行遠端系統管理，請使用適用于 Linux) 的 SSH (或適用于 Windows) 的 RDP/TLS (，而不是未加密的通訊協定。 已淘汰的 SSL、TLS 和 SSH 版本和通訊協定，以及弱式密碼都應停用。

根據預設，Azure 會為 Azure 資料中心之間傳輸中的資料提供加密功能。

- [瞭解 Azure 中的傳輸加密](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit) 

- [TLS 安全性的資訊](/security/engineering/solving-tls1-problem) 

- [Azure 資料傳輸中的雙重加密](../security/fundamentals/double-encryption.md#data-in-transit)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：共用

## <a name="asset-management"></a>資產管理

如需詳細資訊，請參閱 [Azure 安全性效能評定：資產管理](/azure/security/benchmarks/security-controls-v2-asset-management)。

### <a name="am-1-ensure-security-team-has-visibility-into-risks-for-assets"></a>AM-1：確保安全性小組能夠看到資產的風險

**指引**：確保安全性小組會獲得 Azure 租用戶和訂用帳戶中的「安全性讀取者」權限，而能夠使用 Azure 資訊安全中心來監視安全性風險。 

根據安全性小組的權責架構，監視安全性風險可能是由中央安全性小組或區域小組負責。 然而，安全性見解和風險務必要在組織內部集中彙總。 

「安全性讀取者」權限可以廣泛套用至整個租用戶 (根管理群組)，或將範圍限定於管理群組或特定訂用帳戶。 

可能需要額外的許可權，才能看到工作負載和服務。 

- [安全性讀取者角色概觀](../role-based-access-control/built-in-roles.md#security-reader)

- [Azure 管理群組概觀](../governance/management-groups/overview.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="am-2-ensure-security-team-has-access-to-asset-inventory-and-metadata"></a>AM-2：確保安全性小組可以存取資產清查和中繼資料

**指導** 方針：確定安全性小組可以存取 Azure 上 Azure 防火牆管理員資產的持續更新清查。 他們可以使用 Azure Resource Graph 來查詢及探索訂用帳戶中的所有 Azure 防火牆資源，包括 Azure 服務、應用程式和網路資源。

將標籤套用至您的 Azure 資源、資源群組和訂用帳戶，以邏輯方式將這些標記組織成分類法。 每個標記都是由一個名稱和一個值配對組成。 例如，您可以將「環境」名稱和「生產」值套用至生產環境中的所有資源。

- [如何使用 Azure Resource Graph Explorer 建立查詢](../governance/resource-graph/first-query-portal.md) 

- [Azure 資訊安全中心資產庫存管理](../security-center/asset-inventory.md) 

- [如需標記資產的詳細資訊，請參閱資源命名和標記決策指南](https://docs.microsoft.com/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="am-3-use-only-approved-azure-services"></a>AM-3：僅使用已核准的 Azure 服務

**指導** 方針：使用 Azure 原則來審核和限制使用者可以在您的環境中布建的服務，這包括能夠允許或拒絕部署 Azure 防火牆資源。 使用 Azure Resource Graph 則可查詢及探索其訂用帳戶內的資源。 您也可以使用 Azure 監視器來建立規則，以在偵測到未核准的服務時觸發警示。

- [如何設定和管理 Azure 原則](../governance/policy/tutorials/create-and-manage.md) 

- [如何使用 Azure 原則拒絕特定的資源類型](../governance/policy/samples/built-in-policies.md#general) 

- [如何使用 Azure Resource Graph Explorer 建立查詢](../governance/resource-graph/first-query-portal.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="am-4-ensure-security-of-asset-lifecycle-management"></a>AM-4：確保資產生命週期管理的安全性

**指導** 方針：移除不再需要的 Azure 防火牆管理員資源，以將受攻擊面降至最低。 使用者可以透過 Azure 入口網站、CLI 或 REST Api 來管理其 Azure 防火牆管理員資源。

- [Azure 防火牆原則 CLI](https://docs.microsoft.com/cli/azure/ext/azure-firewall/network/firewall/policy?view=azure-cli-latest&amp;preserve-view=true)

- [Azure 網路 CLI](https://docs.microsoft.com/powershell/module/az.network/?view=azps-5.1.0#networking&amp;preserve-view=true)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="am-5-limit-users-ability-to-interact-with-azure-resource-manager"></a>上午-5：限制使用者與 Azure Resource Manager 互動的能力

**指導** 方針： Azure 防火牆管理員已與 Azure Active Directory 的 (Azure AD) 整合，以進行身分識別和驗證。 您可以使用 Azure 條件式存取，藉由設定「Microsoft Azure 管理」應用程式的「封鎖存取」，來限制使用者與 Azure 資源管理員互動的能力。

- [如何設定條件式存取以封鎖對 Azure 資源管理員的存取](../role-based-access-control/conditional-access-azure-management.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

## <a name="logging-and-threat-detection"></a>記錄與威脅偵測

如需詳細資訊，請參閱 [Azure 安全性效能評定：記錄與威脅偵測](/azure/security/benchmarks/security-controls-v2-logging-threat-detection)。

### <a name="lt-1-enable-threat-detection-for-azure-resources"></a>LT-1：啟用 Azure 資源的威脅偵測

**指導** 方針：將由或相關的防火牆原則所產生的活動記錄轉送到您的 SIEM，可用來設定自訂威脅偵測。 確定您正在監視不同類型的 Azure 資產，以找出潛在的威脅和異常。 專注于取得高品質的警示，以減少因分析師排序的誤報。 警示可能源自于記錄資料、代理程式或其他資料。

- [建立自訂分析規則來偵測威脅](../sentinel/tutorial-detect-threats-custom.md) 

- [使用 Azure Sentinel 的網路威脅情報](/azure/architecture/example-scenario/data/sentinel-threat-intelligence)

- [Azure 防火牆記錄和計量](../firewall/firewall-diagnostics.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="lt-2-enable-threat-detection-for-azure-identity-and-access-management"></a>LT-2：啟用 Azure 身分識別和存取管理的威脅偵測

**指引**：Azure AD 提供下列使用者記錄，可在 Azure AD 報告中檢視，或與 Azure 監視器、Azure Sentinel 或其他 SIEM/監視工具整合，以用於較複雜的監視和分析使用案例：
- 登入 – 登入報告會提供受控應用程式和使用者登入活動的使用情況相關資訊。

- 稽核記錄 - 可針對各種功能在 Azure AD 內進行的所有變更，提供記錄追蹤功能。 稽核記錄的範例包括對 Azure AD 中任何資源所做的變更，像是新增或移除使用者、應用程式、群組、角色和原則。
- 有風險的登入 - 有風險的登入表示非使用者帳戶合法擁有者的某人嘗試登入。
- 標幟為有風險的使用者 - 有風險的使用者表示可能被盜用的使用者帳戶。

Azure 資訊安全中心也可對特定的可疑活動發出警示，例如驗證嘗試失敗次數過多，訂用帳戶中已淘汰的帳戶。 除了基本的安全性檢查監視以外，Azure 資訊安全中心的威脅防護模組也可以從個別的 Azure 計算資源 (虛擬機器、容器、App Service)、資料資源 (SQL DB 和儲存體) 與 Azure 服務層收集更深入的安全性警示。 這項功能可讓您查看個別資源內的帳戶異常。

活動記錄檔不支援目前關於防火牆原則規則集合群組的動作，這是已知的問題，在未來的更新中已解決。

- [Azure Active Directory 中的稽核活動報表](../active-directory/reports-monitoring/concept-audit-logs.md)

- [啟用 Azure Identity Protection](../active-directory/identity-protection/overview-identity-protection.md)

- [Azure 資訊安全中心內的威脅防護](/azure/security-center/threat-protection)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="lt-4-enable-logging-for-azure-resources"></a>LT-4：啟用 Azure 資源的記錄功能

**指導** 方針：自動提供的活動記錄，會包含 (PUT、POST、DELETE) 防火牆原則資源的所有寫入作業， (取得) 以外的讀取作業。 當您進行疑難排解時，可以使用活動記錄來尋找錯誤，或是監視組織中的使用者修改資源的方式。

活動記錄檔不支援目前關於防火牆原則規則集合群組的動作，這是已知的問題，在未來的更新中已解決。

- [如何使用 Azure 監視器收集平臺記錄和計量](../azure-monitor/platform/diagnostic-settings.md)

- [瞭解 Azure 中的記錄和不同的記錄類型](../azure-monitor/platform/platform-logs-overview.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="lt-5-centralize-security-log-management-and-analysis"></a>LT-5：將安全性記錄的管理與分析集中

**指導** 方針：集中記錄儲存體和分析以啟用相互關聯。 針對每個記錄來源，請確定您已指派資料擁有者、存取指引、儲存位置、使用哪些工具來處理和存取資料，以及資料保留需求。

確定您要將 Azure 活動記錄整合到您的集中記錄。 透過 Azure 監視器內嵌記錄，以匯總端點裝置、網路資源和其他安全性系統所產生的安全性資料。 在 Azure 監視器中，請使用 Log Analytics 工作區來查詢和執行分析，並使用 Azure 儲存體帳戶來長期和封存儲存體。

此外，請啟用記錄資料，並將其上架到 Azure Sentinel 或協力廠商 SIEM。

許多組織選擇使用 Azure Sentinel 「經常性存取」資料，而這些資料經常使用，並 Azure 儲存體使用較不常使用的「冷」資料。

- [如何使用 Azure 監視器收集平臺記錄和計量](../azure-monitor/platform/diagnostic-settings.md)

- [如何使 Azure Sentinel 上線](../sentinel/quickstart-onboard.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="lt-6-configure-log-storage-retention"></a>LT-6：設定記錄儲存保留期

**指導** 方針：確定用來儲存防火牆原則記錄的任何儲存體帳戶或 log Analytics 工作區，都已根據您組織的合規性法規設定記錄保留期限。

在 Azure 監視器中，您可以根據組織的合規性法規來設定 Log Analytics 工作區保留期限。 使用 Azure 儲存體、Data Lake 或 Log Analytics 工作區帳戶來取得長期和封存儲存體。

- [如何設定 Log Analytics 工作區保留期限](../azure-monitor/platform/manage-cost-storage.md)

- [在 Azure 儲存體帳戶中儲存資源記錄](/azure/azure-monitor/platform/resource-logs-collect-storage)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

## <a name="incident-response"></a>事件回應

如需詳細資訊，請參閱 [Azure 安全性效能評定：事件回應](/azure/security/benchmarks/security-controls-v2-incident-response)。

### <a name="ir-1-preparation--update-incident-response-process-for-azure"></a>IR-1：準備 – 更新 Azure 的事件回應流程

**指引**：確定您的組織具有回應安全性事件的流程、已更新 Azure 的這些處理序，而且會定期執行以確保就緒。

- [在企業環境中實作安全性](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud) (機器翻譯)

- [事件回應參考指南](/microsoft-365/downloads/IR-Reference-Guide.pdf) (英文)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="ir-2-preparation--setup-incident-notification"></a>IR-2：準備 – 設定事件通知

**指引**：在 Azure 資訊安全中心中設定安全性事件連絡人資訊。 當 Microsoft 安全回應中心 (MSRC) 發現您的資料已遭非法或未經授權的合作對象存取時，Microsoft 會使用此連絡人資訊來與您連絡。 您也可以選擇根據事件回應需求，在不同的 Azure 服務中自訂事件警示和通知。 

- [如何設定 Azure 資訊安全中心的安全性連絡人](../security-center/security-center-provide-security-contact-details.md)

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="ir-3-detection-and-analysis--create-incidents-based-on-high-quality-alerts"></a>IR-3：偵測與分析 – 根據高品質警示建立事件

**指引**：確定您具有建立高品質警示並測量警示品質的流程。 這可讓您從過去的事件吸取教訓，並為分析師設定警示優先順序，這樣他們就不會浪費時間在誤報上。 

您可以根據過去事件的經驗、已驗證的社區來源，以及設計成透過融合和相互關聯不同信號來源以產生和清除警示的工具，來建置高品質警示。 

Azure 資訊安全中心在許多 Azure 資產之間提供高品質的警示。 您可以使用 ASC 資料連接器將警示串流至 Azure Sentinel。 Azure Sentinel 可讓您建立進階警示規則，以自動產生事件供調查之用。 

使用匯出功能匯出 Azure 資訊安全中心警示和建議，以利找出 Azure 資源的風險。 以手動或持續不斷的方式匯出警示和建議。

- [如何設定匯出](../security-center/continuous-export.md)

- [如何將警示串流至 Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="ir-4-detection-and-analysis--investigate-an-incident"></a>IR-4：偵測和分析 – 調查事件

**指引**：確定分析師在調查潛在的事件時可查詢和使用各種資料來源，以全面觀照發生的狀況。 您應收集各種記錄，以追蹤整個攻擊鏈中潛在攻擊者的活動，而避免產生盲點。  您也應確實獲取深入解析和知識，以供其他分析師和未來的歷程記錄參考使用。  

調查的資料來源包括從範圍內的服務和執行中的系統收集到的集中式記錄來源，但也可包括：

- 網路資料 – 使用網路安全性群組的流量記錄、Azure 網路監看員和 Azure 監視器，擷取網路流量記錄和其他分析資訊。 

- 執行中系統的快照集： 

    - 使用 Azure 虛擬機器的快照集功能，建立執行中系統磁碟的快照集。 

    - 使用作業系統的原生記憶體傾印功能，建立執行中系統記憶體的快照集。

    - 使用 Azure 服務的快照集功能或軟體本身的功能，建立執行中系統的快照集。

Azure Sentinel 可讓您對絕大多數的記錄來源進行廣泛的資料分析，並提供案例管理入口網站來管理事件的完整生命週期。 調查期間的情報資訊可以與事件相關聯，以供追蹤和報告之用。 

- [為 Windows 機器的磁碟建立快照集](../virtual-machines/windows/snapshot-copy-managed-disk.md)

- [為 Linux 機器的磁碟建立快照集](../virtual-machines/linux/snapshot-copy-managed-disk.md)

- [Microsoft Azure 支援診斷資訊和記憶體傾印收集](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) 

- [使用 Azure Sentinel 調查事件](../sentinel/tutorial-investigate-cases.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="ir-5-detection-and-analysis--prioritize-incidents"></a>IR-5：偵測與分析 – 設定事件的優先順序

**指引**：根據警示嚴重性和資產敏感度，為分析師提供應優先關注哪些事件的內容。 

Azure 資訊安全中心會指派每個警示的嚴重性，以協助您設定應優先調查哪些警示。 嚴重性是以「資訊安全中心」在「尋找」或用來發出警示的分析，以及導致警示的活動背後有惡意意圖的信賴等級為基礎。

此外，請使用標籤來標示資源，並建立命名系統以識別及分類 Azure 資源，尤其是處理敏感資料的資源。  您需負責根據發生事件的 Azure 資源和環境的重要性，設定警示的補救優先順序。

- [Azure 資訊安全中心的安全性警示](../security-center/security-center-alerts-overview.md)

- [使用標記來組織 Azure 資源](/azure/azure-resource-manager/resource-group-using-tags)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="ir-6-containment-eradication-and-recovery--automate-the-incident-handling"></a>IR-6：圍堵、根除及復原 – 將事件處理自動化

**指引**：將手動重複的工作自動化，以縮短回應時間並降低分析師的負擔。 手動工作需要較長的時間來執行，進而降低每個事件的速度，以及減少分析師可以處理的事件數目。 手動工作也會使分析師更加疲勞，而增加人為錯誤導致延遲的風險，並降低分析師有效專注處理複雜工作的能力。 請使用 Azure 資訊安全中心和 Azure Sentinel 中的工作流程自動化功能來自動觸發動作，或執行劇本以回應傳入的安全性警示。 劇本會採取動作，例如傳送通知、停用帳戶，以及隔離有問題的網路。 

- [在資訊安全中心設定工作流程自動化](../security-center/workflow-automation.md)

- [在 Azure 資訊安全中心設定自動化威脅回應](../security-center/tutorial-security-incident.md#triage-security-alerts)

- [在 Azure Sentinel 中設定自動化威脅回應](../sentinel/tutorial-respond-threats-playbook.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

## <a name="posture-and-vulnerability-management"></a>狀況和弱點管理

如需詳細資訊，請參閱 [Azure 安全性效能評定：狀況和弱點管理](/azure/security/benchmarks/security-controls-v2-posture-vulnerability-management)。

### <a name="pv-1-establish-secure-configurations-for-azure-services"></a>PV-1：建立 Azure 服務的安全設定 

**指導** 方針：使用下列機制（例如： Azure Resource Manager 範本、Azure RBAC 控制項和 Azure 原則）將部署和設定環境中的 Azure 防火牆管理員資源自動化並加以保護。 在部署期間定義 Azure 防火牆管理員資源的安全設定，並使用 ' Azure. Network ' 命名空間中的別名定義自訂 Azure 原則定義，以進行這些設定的審核和強制執行。

- [Azure 防火牆原則範本參考](/azure/templates/microsoft.network/firewallpolicies)

- [Azure 防火牆原則 CLI](https://docs.microsoft.com/cli/azure/ext/azure-firewall/network/firewall/policy?view=azure-cli-latest&amp;preserve-view=true)

- [企業規模登陸區域中的護欄實行圖](/azure/cloud-adoption-framework/ready/enterprise-scale/architecture#landing-zone-expanded-definition)

- [建立和管理原則以強制執行合規性](../governance/policy/tutorials/create-and-manage.md)

**Azure 資訊安全中心監視**：是

**責任**：客戶

### <a name="pv-2-sustain-secure-configurations-for-azure-services"></a>PV-2：維持 Azure 服務的安全設定

**指導** 方針： Azure 防火牆管理員支援以 Azure Resource Manager 為基礎的範本，以及透過 Azure 原則強制執行設定。 定義自訂 Azure 原則定義，以在「Azure. 網路」命名空間中使用別名來審核和強制執行 Azure 防火牆管理員資源設定。

- [瞭解 Azure 原則效果](../governance/policy/concepts/effects.md)

- [Azure 防火牆原則範本參考](/azure/templates/microsoft.network/firewallpolicies)

- [企業規模登陸區域中的護欄實行圖](/azure/cloud-adoption-framework/ready/enterprise-scale/architecture#landing-zone-expanded-definition)

- [建立和管理原則以強制執行合規性](../governance/policy/tutorials/create-and-manage.md)

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="pv-3-establish-secure-configurations-for-compute-resources"></a>PV-3：建立計算資源的安全設定

**指導** 方針：不適用;Azure 防火牆管理員是防火牆控制平面管理服務，而且不會公開基礎服務的計算基礎結構供客戶設定。

**Azure 資訊安全中心監視**：目前無法使用

**責任**：客戶

### <a name="pv-8-conduct-regular-attack-simulation"></a>PV-8：執行一般攻擊模擬

**指引**：如有需要，請對您的 Azure 資源執行滲透測試或紅隊活動，並確保補救所有重要的安全性結果。
請遵循 Microsoft 雲端滲透測試參與規則，以確保您的滲透測試不會違反 Microsoft 原則。 針對 Microsoft 管理的雲端基礎結構、服務和應用程式，使用 Microsoft 對於紅隊和即時網站滲透測試的策略和執行方法。

- [Azure 中的滲透測試](../security/fundamentals/pen-testing.md) (機器翻譯)

- [滲透測試運作規則](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Azure 資訊安全中心監視**：不適用

**責任**：共用

## <a name="backup-and-recovery"></a>備份及修復

如需詳細資訊，請參閱 [Azure 安全性效能評定：備份和復原](/azure/security/benchmarks/security-controls-v2-backup-recovery)。

### <a name="br-1-ensure-regular-automated-backups"></a>BR-1：確保定期自動備份

**指導** 方針： Azure 防火牆管理員沒有面向客戶系統備份的概念，Microsoft 會處理基礎結構。

若為資源設定備份，請使用 Azure Resource Manager 在 JavaScript 物件標記法 (JSON) 範本中匯出防火牆原則和相關資源，該範本可作為設定備份。 您也可以從 Azure 入口網站使用 Azure 防火牆的 [匯出範本] 功能來匯出防火牆原則設定。 使用 Azure 自動化來執行任何自訂的備份腳本，以自動捕捉 Azure 防火牆管理員資源的資源設定。

- [使用範本部署安全虛擬中樞](quick-secure-virtual-hub.md)

- [Microsoft 防火牆原則範本參考](/azure/templates/microsoft.network/firewallpolicies)

- [關於 Azure 自動化](../automation/automation-intro.md)

**Azure 資訊安全中心監視**：不適用

**責任**：共用

### <a name="br-3-validate-all-backups-including-customer-managed-keys"></a>BR-3：驗證所有備份，包括客戶自控金鑰

**指導** 方針： Azure 防火牆管理員沒有面向客戶系統備份的概念。 對於任何匯出的 Azure 防火牆管理員資源範本，會定期使用這些 Azure Resource Manager 範本檔案來執行還原。

- [使用 Azure Resource Manager 範本部署安全虛擬中樞](quick-secure-virtual-hub.md)

- [Microsoft 防火牆原則範本參考](/azure/templates/microsoft.network/firewallpolicies)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

## <a name="governance-and-strategy"></a>控管與策略

如需詳細資訊，請參閱 [Azure 安全性效能評定：治理和策略](/azure/security/benchmarks/security-controls-v2-governance-strategy)。

### <a name="gs-1-define-asset-management-and-data-protection-strategy"></a>GS-1：定義資產管理和資料保護策略 

**指引**：務必記載並傳達明確的策略，以持續監視及保護系統和資料。 請優先探索、評估、保護及監視業務關鍵資料和系統。 

此策略應該包含下列項目的已記載指引、原則和標準： 

-   符合商業風險的資料分類標準

-   安全性組織對風險和資產清查的可見度 

-   安全性組織核准 Azure 服務以供使用的程序 

-   資產在其生命週期中的安全性

-   符合組織資料分類的必要存取控制策略

-   使用 Azure 原生和協力廠商資料保護功能

-   傳輸中和待用使用案例的資料加密需求

-   適當的密碼編譯標準

如需詳細資訊，請參閱下列參考資料：
- [Azure 安全性架構建議 - 儲存體、資料和加密](https://docs.microsoft.com/azure/architecture/framework/security/storage-data-encryption?toc=/security/compass/toc.json&amp;bc=/security/compass/breadcrumb/toc.json) (機器翻譯)

- [Azure 安全性基礎觀念 - Azure 資料安全性、加密和儲存體](../security/fundamentals/encryption-overview.md) (機器翻譯)

- [雲端採用架構 - Azure 資料安全性和加密最佳做法](https://docs.microsoft.com/azure/security/fundamentals/data-encryption-best-practices?toc=/azure/cloud-adoption-framework/toc.json&amp;bc=/azure/cloud-adoption-framework/_bread/toc.json) (機器翻譯)

- [Azure 安全性效能評定 - 資產管理](/azure/security/benchmarks/security-controls-v2-asset-management)

- [Azure 安全性效能評定 - 資料保護](/azure/security/benchmarks/security-controls-v2-data-protection)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="gs-2-define-enterprise-segmentation-strategy"></a>GS-2：定義企業分割策略 

**指引**：使用身分識別、網路、應用程式、訂用帳戶、管理群組和其他控制項的組合，建立企業範圍的策略來分割對產的存取。

審慎權衡安全性區隔的需求，以及需要與彼此通訊並存取資料的啟用每日作業需求。

確保在各種控制項類型上一致地實作分割策略，這些控制項類型包括網路安全性、身分識別和存取模型，以及應用程式權限/存取模型與人力流程控制。

- [Azure 中的分割策略指引 (影片)](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151) (機器翻譯)

- [Azure 中的分割策略指引 (文件)](/security/compass/governance#enterprise-segmentation-strategy) (機器翻譯)

- [使用企業分割策略來調整網路分割](/security/compass/network-security-containment#align-network-segmentation-with-enterprise-segmentation-strategy) (機器翻譯)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="gs-3-define-security-posture-management-strategy"></a>GS-3：定義安全性狀態管理策略

**指引**：持續測量並降低個別資產及其裝載環境的風險。 優先處理高價值資產和高度公開的攻擊面，例如已發佈的應用程式、網路輸入和輸出點、使用者和系統管理員端點等等。

- [Azure 安全性效能評定 - 狀態和弱點管理](/azure/security/benchmarks/security-controls-v2-posture-vulnerability-management)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="gs-4-align-organization-roles-responsibilities-and-accountabilities"></a>GS-4：協調組織角色、責任及權責

**指引**：務必為安全性組織中的角色和責任記載並傳達清楚的策略。 優先為安全性決策提供清楚的權責、讓每個人熟知共同責任模型，並讓技術團隊熟知保護雲端的技術。

- [Azure 安全性最佳做法 1 – 人員：讓小組熟知雲端安全性旅程](/azure/cloud-adoption-framework/security/security-top-10#1-people-educate-teams-about-the-cloud-security-journey) (機器翻譯)

- [Azure 安全性最佳做法 2 - 人員：讓小組熟知雲端安全性技術](/azure/cloud-adoption-framework/security/security-top-10#2-people-educate-teams-on-cloud-security-technology) (機器翻譯)

- [Azure 安全性最佳做法 3 - 流程：指派雲端安全性決策的權責](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud) (機器翻譯)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="gs-5-define-network-security-strategy"></a>GS-5：定義網路安全性策略

**指引**：建立 Azure 網路安全性方法，作為組織整體安全性存取控制策略的一部分。  

此策略應該包含下列項目的已記載指引、原則和標準： 

-   集中式網路管理和安全性責任

-   與企業分割策略一致的虛擬網路分割模型

-   不同威脅和攻擊案例的補救策略

-   網際網路邊緣和輸入與輸出策略

-   混合式雲端和內部部署互連能力策略

-   最新的網路安全性成品 (例如網路圖、參考網路架構)

如需詳細資訊，請參閱下列參考資料：
- [Azure 安全性最佳做法 11 - 架構。單一整合的安全性策略](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy) (機器翻譯)

- [Azure 安全性效能評定 - 網路安全性](/azure/security/benchmarks/security-controls-v2-network-security)

- [Azure 網路安全性概觀](../security/fundamentals/network-overview.md)

- [企業網路架構策略](/azure/cloud-adoption-framework/ready/enterprise-scale/architecture) (機器翻譯)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="gs-6-define-identity-and-privileged-access-strategy"></a>GS-6：定義身分識別和特殊權限存取策略

**指引**：建立 Azure 身分識別和特殊權限存取方法，作為組織整體安全性存取控制策略的一部分。  

此策略應該包含下列項目的已記載指引、原則和標準： 

-   集中式身分識別和驗證系統，以及其與其他內部和外部身分識別系統的互連能力

-   不同使用案例和條件中的增強式驗證方法

-   高權限使用者的保護

-   異常使用者活動監視和處理  

-   使用者身分識別、存取權檢閱與核對流程

如需詳細資訊，請參閱下列參考資料：

- [Azure 安全性效能評定 - 身分識別管理](/azure/security/benchmarks/security-controls-v2-identity-management)

- [Azure 安全性效能評定 - 特殊權限存取](/azure/security/benchmarks/security-controls-v2-privileged-access)

- [Azure 安全性最佳做法 11 - 架構。單一整合的安全性策略](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy) (機器翻譯)

- [Azure 身分識別管理安全性概觀](../security/fundamentals/identity-management-overview.md)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

### <a name="gs-7-define-logging-and-threat-response-strategy"></a>GS-7：定義記錄和威脅回應策略

**指引**：建立記錄和威脅回應策略，以快速偵測並補救威脅，同時符合合規性需求。 優先為分析師提供高品質的警示和順暢的體驗，使其能夠全心處理威脅，而非整合和手動步驟。 

此策略應包含針對下列元素而記載的指引、原則和標準： 

-   安全性作業 (SecOps) 組織的角色和責任 

-   妥善定義且與 NIST 或其他產業架構一致的事件回應流程 

-   記錄擷取和保留，以支援威脅偵測、事件回應及合規性需求

-   使用 SIEM、原生 Azure 功能及其他來源，集中顯示及相互關聯威脅的相關資訊 

-   與客戶、供應商和相關公開合作對象之間的溝通和通知計畫

-   使用 Azure 原生和協力廠商平台進行事件處理，例如記錄和威脅偵測、鑑定，以及攻擊補救和根除

-   處理事件和事件後活動的流程，例如吸取的經驗和證據保留

如需詳細資訊，請參閱下列參考資料：

- [Azure 安全性效能評定 - 記錄與威脅偵測](/azure/security/benchmarks/security-controls-v2-logging-threat-detection)

- [Azure 安全性效能評定 - 事件回應](/azure/security/benchmarks/security-controls-v2-incident-response)

- [Azure 安全性最佳做法 4 - 流程。更新雲端的事件回應流程](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud) (機器翻譯)

- [Azure 採用架構、記錄與報告決策指南](/azure/cloud-adoption-framework/decision-guides/logging-and-reporting/)

- [Azure 企業規模、管理與監視](/azure/cloud-adoption-framework/ready/enterprise-scale/management-and-monitoring) (機器翻譯)

**Azure 資訊安全中心監視**：不適用

**責任**：客戶

## <a name="next-steps"></a>後續步驟

- 請參閱 [Azure 安全性效能評定 V2 概觀](/azure/security/benchmarks/overview)
- 深入了解 [Azure 資訊安全性基準](/azure/security/benchmarks/security-baselines-overview)
