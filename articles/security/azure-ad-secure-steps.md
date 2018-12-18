---
title: 可在 Azure Active Directory 中保護身分識別基礎結構的五個步驟
description: 本文件概述系統管理員所應實作以便有助自身使用 Azure AD 功能來保護組織的重要動作清單
services: active-directory
author: martincoetzer
manager: manmeetb
tags: azuread
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.date: 06/18/2018
ms.author: martincoetzer
ms.openlocfilehash: 2a6c94d902c639a2529e501347876d63a3f56d7e
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39505604"
---
# <a name="five-steps-to-securing-your-identity-infrastructure"></a>可保護身分識別基礎結構的五個步驟

如果您正在閱讀本文件，想必您已了解安全性有多重要。 您可能已承擔起保護組織的重任。 如果您需要說服他人，讓其了解安全性有多重要，請讓其閱讀最新的 [安全情報報告](https://www.microsoft.com/security/intelligence-report)。

本文件會協助您使用 Azure Active Directory 的功能，透過有五個步驟的檢查清單讓組織預防網路攻擊來獲得更安全的狀態。

這份檢查清單會藉由說明如何執行下列作業，協助您快速部署重要的建議動作來立即保護組織：

* 強化認證。
* 減少受攻擊面。
* 將威脅回應自動化。
* 增加對稽核和監視的認識。
* 透過自助服務實現更具預測性且更完整的使用者安全性。

> [!NOTE]
> 這份文件中有諸多建議，僅對設定為以 Azure Active Directory 作為其識別提供者的應用程式有效。 為應用程式設定單一登入，可確保應用程式會享有認證原則、威脅偵測、稽核、記錄，以及其他已新增至這些應用程式的功能。 [透過 Azure Active Directory 的單一登入](https://docs.microsoft.com/azure/active-directory/active-directory-enterprise-apps-manage-sso)是實現這些建議的基礎。

## <a name="before-you-begin-protect-privileged-accounts-with-mfa"></a>開始之前：使用 MFA 保護特殊權限帳戶

在開始進行此檢查清單之前，請確定您不會在閱讀此檢查清單時遭到入侵。 您必須先保護特殊權限帳戶。

攻擊者若掌控了特殊權限帳戶，將會造成極大的破壞，因此請務必先保護這些帳戶。 對組織中使用[基準保護](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-baseline-protection)的所有系統管理員，啟用並要求 [Azure Multi-Factor Authentication](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication) (MFA)。 如果您尚未實作 MFA，請立即實作！ 這一點非常重要。

全都準備好了嗎？ 那就開始進行此檢查清單吧。

## <a name="step-1---strengthen-your-credentials"></a>步驟 1 - 強化認證 

企業的安全性缺口，大多是因為帳戶遭到少數幾種入侵方法 (例如，密碼噴灑、入侵重播或網路釣魚) 入侵。 若要深入了解這些攻擊，請觀看這段影片：
> [!VIDEO https://channel9.msdn.com/events/Ignite/Microsoft-Ignite-Orlando-2017/BRK3016/player]

如果身分識別系統中的使用者使用弱式密碼，而且未使用多重要素驗證來予以強化，那麼遭到入侵就不是有沒有可能或是何時的問題，而是會有「多頻繁」。

### <a name="make-sure-your-organization-use-strong-authentication"></a>確定組織使用的是強式驗證

有鑑於入侵者經常會猜測、誘騙或以惡意程式碼竊取密碼，使用者也經常會重複使用密碼，所以請務必使用某種形式的強式認證來支援密碼 - 詳情請參閱 [Azure Multi-Factor Authentication](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication)。

### <a name="turn-off-traditional-complexity-expiration-rules-and-start-banning-commonly-attacked-passwords-instead"></a>停止使用傳統的複雜性規則、到期規則，改為開始禁用常常受到攻擊的密碼

許多組織使用傳統的複雜性規則 (例如，特殊字元) 和密碼到期規則。 Microsoft 的研究顯示，這些有害原則會導致使用者選擇較容易猜到的密碼。

Microsoft 的建議與 [NIST 指導方針](https://pages.nist.gov/800-63-3/sp800-63b.html)一致，也就是要實作下列三點：

1. 要求密碼至少要有 8 個字元。 密碼並非越長越好，密碼越長會導致使用者選擇可預測的密碼、將密碼儲存在檔案中或寫下來。
2. 停用到期規則，到期規則會讓使用者想要使用容易猜測的密碼，例如 **Summer2018!**。
3. 停用字元組合規定，防止使用者選擇常常受到攻擊的密碼，原因是這些規定會導致使用者在密碼中選擇可預測的替代字元。

如果您直接在 Azure AD 中建立身分識別，則可以使用 [PowerShell 來防止使用者的密碼到期](https://docs.microsoft.com/azure/active-directory/authentication/concept-sspr-policy)。 搭配使用內部部署 AD 與 Azure AD Connect 將身分識別同步到 Azure AD (也稱為混合式部署) 的組織，則應該使用[網域群組原則設定](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/hh994572(v%3dws.10))或 [Windows PowerShell](https://docs.microsoft.com/powershell/module/addsadministration/set-addefaultdomainpasswordpolicy) 來實作內部部署[智慧型密碼原則](https://aka.ms/passwordguidance)。

Azure Active Directory 的[動態禁用密碼](https://docs.microsoft.com/azure/active-directory/active-directory-secure-passwords)功能會使用目前的攻擊者行為來防止使用者設定可輕易猜到的密碼。 這項功能會永遠開啟，使用混合式部署的組織可以啟用[密碼回寫](https://docs.microsoft.com/azure/active-directory/authentication/howto-sspr-writeback)而從這項功能受益，也可以部署 [Windows Server Active Directory 的 Azure AD 密碼保護](https://docs.microsoft.com/azure/active-directory/authentication/concept-password-ban-bad-on-premises)。 Azure AD 密碼保護會阻止使用者選擇一般的常見密碼以及您可以設定的自訂密碼。

### <a name="protect-against-leaked-credentials-and-add-resilience-against-outages"></a>防止認證外洩並新增遇到中斷時的復原能力

如果組織使用混合式身分識別解決方案，則基於下列兩個原因，您應該啟用密碼雜湊同步處理：

* Azure AD 管理中的[認證外洩的使用者](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-risk-events)報告會就「暗網 (Dark Web)」上已公開的使用者名稱和密碼組向您發出警告。 數量驚人的密碼是透過網路釣魚、惡意程式碼，以及在第三方網站上重複使用密碼並於之後遭到入侵而外洩的。 Microsoft 發現許多這樣的外洩認證，而且如果與您組織的認證相符，則會在此報告中告訴您，但前提是您已[啟用密碼雜湊同步處理](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-hash-synchronization)！
* 發生內部中斷 (例如，遭遇勒索軟體攻擊) 時，您可以改為[使用密碼雜湊同步處理進行雲端驗證](https://docs.microsoft.com/azure/security/azure-ad-choose-authn)。 此備用驗證方法可讓您繼續存取已設定為使用 Azure Active Directory 進行驗證的應用程式，包括 Office 365。

深入了解[密碼雜湊同步處理](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-hash-synchronization)的運作方式。

### <a name="implement-ad-fs-extranet-lockout"></a>實作 AD FS 外部網路鎖定

將應用程式設定為直接向 Azure AD 進行驗證的組織，可受益於 [Azure AD 智慧鎖定](https://docs.microsoft.com/azure/active-directory/active-directory-secure-passwords)。 如果您是在 Windows Server 2012R2 中使用 AD FS，請實作 AD FS [外部網路鎖定保護](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection) \(英文\)。 如果您是在 Windows Server 2016 上使用 AD FS，請實作[外部網路智慧鎖定](https://support.microsoft.com/en-us/help/4096478/extranet-smart-lockout-feature-in-windows-server-2016) \(機器翻譯\)。 AD FS 智慧外部網路鎖定可防範以 AD FS 目標的暴力密碼破解攻擊，並防止系統在 Active Directory 中鎖定使用者。

### <a name="take-advantage-of-intrinsically-secure-easier-to-use-credentials"></a>利用真正安全、更容易使用的認證

使用 [Windows Hello](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-identity-verification)，即可在 PC 和行動裝置上將密碼改為強式雙因素驗證。 這項驗證是由與裝置繫結的新型使用者認證所組成的，而且會使用生物特徵辨識或 PIN 碼。

## <a name="step-2---reduce-your-attack-surface"></a>步驟 2 - 減少受攻擊面

由於密碼入侵問題無所不在，所以盡量減少組織的受攻擊面就相當重要。 不要使用老舊且不安全的通訊協定、限制存取進入點，以及更加嚴格地控制資源的系統管理存取權，可協助減少受攻擊面。

### <a name="block-legacy-authentication"></a>封鎖舊式驗證

使用自有舊式方法來向 Azure AD 進行驗證和存取公司資料的應用程式，會對組織造成其他風險。 使用舊式驗證的應用程式範例包括 POP3、IMAP4 或 SMTP 用戶端。 舊式驗證應用程式會代表使用者進行驗證，而讓 Azure AD 無法進行進階的安全性評估。 作為替代的新式驗證可支援多重要素驗證和條件式存取，所以會減少安全性風險。 建議您進行下列三個動作：

1. 封鎖[舊式驗證 (如果您使用 AD FS)](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/access-control-policies-w2k12)。
2. 將 [SharePoint Online 和 Exchange Online 設定為使用新式驗證](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-no-modern-authentication)。
3. 使用[條件式存取原則來封鎖舊式驗證](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)。

### <a name="block-invalid-authentication-entry-points"></a>封鎖無效的驗證進入點

使用假想入侵心態，應該就能在使用者認證遭到入侵時降低其影響。 對於環境中的每個應用程式，請考慮有效的使用案例：哪些群組、哪些網路、哪些裝置和其他元素有獲得授權，然後將其餘項目封鎖。 在限制使用[高特殊權限帳戶或服務帳戶](https://docs.microsoft.com/azure/active-directory/admin-roles-best-practices)時請小心。 透過 [Azure AD 條件式存取](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)，即可根據所定義的特定條件，控制獲得授權的使用者要如何存取其應用程式和資源。

請特別注意服務帳戶 (用來以自動化方式進行工作的帳戶)。 使用條件式存取，即可確定這類帳戶只能對服務執行、從 IP 執行，以及在一天當中的某個時間執行 (視何者適當)。

### <a name="implement-azure-ad-privileged-identity-management"></a>實作 Azure AD Privileged Identity Management

「假想入侵」的另一個影響是需要將遭到入侵的帳戶可使用特殊權限角色進行操作的可能性降到最低。 

[Azure AD Privileged Identity Management (PIM)](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-configure) 可協助您進行下列作業，讓您將帳戶特殊權限最小化：

* 識別和管理指派給系統管理角色的使用者。
* 了解您應該移除的未用或多餘特殊權限角色。
* 建立規則以確定特殊權限角色會受到多重要素驗證保護。
* 建立規則以確定所授與的特殊權限角色有效時間僅足以完成特殊權限工作。

啟用 Azure AD PIM，然後檢視已獲派系統管理角色的使用者，並將這些角色中的不必要帳戶移除。 對於其餘特殊權限使用者，則將它們從永久性角色改為合適角色。 最後，建立適當原則以確定當使用者需要存取其特殊權限角色時，可安全地進行。

在部署特殊權限帳戶程序時，請遵循[最佳做法建立至少兩個緊急帳戶](https://docs.microsoft.com/azure/active-directory/admin-roles-best-practices)，以確定您自己在遭到鎖定時，仍可存取 Azure AD。

## <a name="step-3---automate-threat-response"></a>步驟 3 - 將威脅回應自動化

Azure Active Directory 有許多會自動攔截攻擊的功能，可讓偵測與回應之間沒有延遲。 若能減少罪犯可供用來將自身埋藏到您環境的時間，您就能降低成本和風險。 您可以採取的具體步驟如下。

### <a name="implement-user-risk-security-policy-using-azure-ad-identity-protection"></a>使用 Azure AD Identity Protection 來實作使用者風險安全性原則

使用者風險可指出使用者的身分識別已遭到入侵的可能性，並會根據與使用者身分識別相關聯的[使用者風險事件](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection)來算出。 使用者風險原則是條件式存取原則，可評估特定使用者或群組的風險層級。 根據低、中、高風險層級，原則可以設定為封鎖存取，也可以設定為必須使用多重要素驗證來進行安全密碼變更。 Microsoft 會建議您要求高風險使用者必須進行安全密碼變更。

![標示有風險的使用者](media/azure-ad/azure-ad-sec-steps1.png)

### <a name="implement-sign-in-risk-policy-using-azure-ad-identity-protection"></a>使用 Azure AD Identity Protection 來實作登入風險原則

登入風險是指帳戶擁有者以外人士嘗試使用身分識別來登入的可能性。 [登入風險原則](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection)是條件式存取原則，可評估特定使用者或群組的風險層級。 根據風險層級 (高/中/低)，原則可以設定為封鎖存取，也可以設定為強制使用多重要素驗證。 請確定您有對中等以上風險的登入，強制使用多重要素驗證。

![從匿名 IP 登入](media/azure-ad/azure-ad-sec-steps2.png)

## <a name="step-4---increase-your-awareness"></a>步驟 4 - 增加認識

安全性相關事件的稽核和記錄及相關警示是有效保護策略的重要元件。 安全性記錄和報告會提供可疑活動的電子記錄，並協助您偵測可能指出從外部嘗試或成功滲透網路以及內部攻擊的模式。 您可以使用稽核來監視使用者活動、文件合規性、執行鑑識分析等等。 警示會提供安全性事件通知。

### <a name="monitor-azure-ad"></a>監視 Azure AD

Microsoft Azure 服務和功能提供可設定的安全性稽核和記錄選項，以協助您識別安全性原則和機制間的差距，並解決這些差距以協助防止破壞。 您可以使用 [Azure 記錄與稽核](https://docs.microsoft.com/azure/security/azure-log-audit)以及使用 [Azure Active Directory 入口網站中的稽核活動報告](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-activity-audit-logs)。

### <a name="monitor-azure-ad-connect-health-in-hybrid-environments"></a>在混合式環境中監視 Azure AD Connect Health

[使用 Azure AD Connect Health 監視 AD FS](https://docs.microsoft.com/azure/active-directory/connect-health/active-directory-aadconnect-health-adfs) 可讓您更加深入地了解潛在問題並看到 ADFS 基礎結構所受到的攻擊。 Azure AD Connect Health 會提供警示與詳細資料、解決步驟，以及相關文件的連結；提供數個驗證流量相關計量的使用情況分析；提供效能監控和報告。

![Azure AD Connect Health](media/azure-ad/azure-ad-sec-steps4.png)

### <a name="monitor-azure-ad-identity-protection-events"></a>監視 Azure AD Identity Protection 事件

[Azure AD Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) 是一種通知、監視和報告工具，可用來偵測會影響組織身分識別的潛在弱點。 它會偵測風險事件，例如外洩的認證、不可能的行進路線、源自受感染裝置的登入、匿名 IP 位址、與可疑活動相關聯的 IP 位址，以及不明的位置。 請啟用通知警示，以接收有風險使用者的電子郵件和/或每週摘要電子郵件。

![標示有風險的使用者](media/azure-ad/azure-ad-sec-steps3.png)

## <a name="step-5---enable-end-user-self-help"></a>步驟 5 - 啟用使用者自助服務

請盡可能地在安全性與生產力之間取得平衡。 和運用打下長遠安全性基礎的心態來著手處理旅途的思路相同，您可以藉由既讓使用者獲得能力同時又保持警覺的方式，去除組織中的分歧意見。 

### <a name="implement-self-service-password-reset"></a>實作自助密碼重設

Azure 的[自助密碼重設 (SSPR)](https://docs.microsoft.com/azure/active-directory/active-directory-passwords-getting-started) 提供簡單的方法，讓 IT 系統管理員允許使用者重設或解除鎖定其密碼或帳戶，而不必系統管理員介入處理。 系統包含詳細的報告，可追蹤使用者何時存取系統與通知來警示您誤用或濫用。 

### <a name="implement-self-service-group-management"></a>實作自助群組管理

Azure AD 可讓您使用安全性群組和 Office 365 群組來管理資源的存取權。 這些群組可由群組擁有者 (而不是 IT 系統管理員) 負責管理。 這項功能又稱為[自助群組管理](https://docs.microsoft.com/azure/active-directory/active-directory-accessmanagement-self-service-group-management)，可讓未獲得系統管理角色的群組擁有者建立和管理群組，而不需要依賴系統管理員來處理其要求。

### <a name="implement-azure-ad-access-reviews"></a>實作 Azure AD 存取權檢閱

透過 [Azure AD 存取權檢閱](https://docs.microsoft.com/azure/active-directory/active-directory-azure-ad-controls-access-reviews-overview)功能，您可以管理群組成員資格、企業應用程式的存取權和特殊權限角色指派，以確保能維持安全性標準，不讓使用者在不需要時獲得更長時間的存取權。

## <a name="summary"></a>總結

安全的身分識別基礎結構涉及許多層面，但這份有五個步驟的檢查清單會協助您快速完成更安全可靠的身分識別基礎結構：

* 強化認證。
* 減少受攻擊面。
* 將威脅回應自動化。
* 增加對稽核和監視的認識。
* 透過自助服務實現更具預測性且更完整的使用者安全性。

我們非常感謝您如此嚴肅地看待身分識別安全性，希望這份文件有助於讓貴組織達成更安全的狀態。

## <a name="next-steps"></a>後續步驟
如果您需要協助來規劃和部署各項建議，請參閱 [Azure AD 專案部署方案](http://aka.ms/deploymentplans)中的說明。
