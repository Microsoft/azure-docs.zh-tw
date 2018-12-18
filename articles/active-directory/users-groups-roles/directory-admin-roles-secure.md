---
title: 在 Azure AD 中保護系統管理存取的最佳做法 | Microsoft Docs
description: 請確定組織的系統管理存取和管理帳戶安全無虞。 適用於設定 Azure AD、Azure 和 Microsoft 線上服務的系統架構設計人員和 IT 專業人員。
services: active-directory
keywords: ''
author: curtand
manager: mtillman
ms.author: curtand
ms.date: 06/25/2018
ms.topic: article
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.custom: it-pro
ms.reviewer: martincoetzer, MarkMorow
ms.openlocfilehash: 55ce6398d89e56f92874d0ec135377d2a08cf075
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46297804"
---
# <a name="securing-privileged-access-for-hybrid-and-cloud-deployments-in-azure-ad"></a>在 Azure AD 中保護混合式部署和雲端部署的特殊權限存取

在現今的組織中，大部分甚或所有企業資產的安全性都取決於負責管理 IT 系統的特殊權限帳戶是否完整。 包括網路攻擊者在內的惡意人士通常會以管理帳戶和特殊權限存取的其他元素為目標，嘗試使用認證竊取攻擊快速取得敏感性資料和系統的存取權。 就雲端服務而言，預防和回應是雲端服務提供者和客戶的共同責任。 若想進一步了解端點和雲端有哪些最新威脅，請參閱 [Microsoft Security Intelligence Report](https://www.microsoft.com/security/sir/default.aspx)。 本文可協助您擬定適當計劃，以期消弭您目前的計劃與此處所述的指導方針之間的落差。

> [!NOTE] 
> Microsoft 致力於達到最高水準的信任、透明度、標準一致性和法規遵循性。 請經由 [Microsoft 信任中心 - 安全性](https://www.microsoft.com/trustcenter/security)深入了解 Microsoft 全球事件回應小組如何降低對雲端服務的攻擊所造成的影響，所及 Microsoft 商業產品和雲端服務內建的安全性，並經由 [Microsoft 信任中心 - 合規性](https://www.microsoft.com/trustcenter/compliance)了解 Microsoft 的合規性目標。

<!--## Risk management, incident response, and recovery preparation

A cyber-attack, if successful, can shut down operations not just for a few hours, but in some cases for days or even weeks. The collateral damage, such as legal ramifications, information leaks, and media coverage, could potentially continue for years. To ensure effective company-wide risk containment, cybersecurity and IT pros must align their response and recovery processes. To reduce the risk of business disruption due to a cyber-attack, industry experts recommend you do the following:

* As part of your risk management operations, establish a crisis management team for your organization that is responsible for managing all types of business disruptions.

* Compare your current risk mitigations, incident response, and recovery plan with industry best practices for managing a business disruption before, during, and after a cyber-attack.

* Develop and implement a roadmap for closing the gaps between your current plans and the best practices described in this document.


## Securing privileged access for hybrid and cloud deployments

does the article really start here?-->
就大部分的組織而言，企業資產的安全性都取決於負責管理 IT 系統的特殊權限帳戶是否完整。 網路攻擊者著眼於基礎結構系統 (例如 Active Directory 和 Azure Active Directory) 的特殊權限存取，以設法取得組織中敏感性資料的存取權。 

隨著在網際網路上使用 SaaS 應用程式和個人裝置的情形日漸普及，過去聚焦於將網路的進入和退出點視為主要安全性周邊而加以保護的傳統方法，已逐漸失去效用。 在現今複雜的企業中，網路安全性周邊已自然取代為組織的身分識別層中的驗證與授權控制。 

具特殊權限的系統管理帳戶可有效控管這個新的「安全性周邊」。 保護特殊權限存取至關重要，無論您的環境是內部部署、雲端，還是混合的內部部署和雲端託管服務。 面對難纏的對手，為了保護系統管理存取，您必須運用完整而縝密的方法將風險阻隔於組織的系統外。 

要保護特殊權限的存取須進行以下變更
* 程序、系統管理實作和知識管理
* 技術元件，例如主機防禦、帳戶保護和身分識別管理

本文主要說明如何擬定適當的計劃，以保護 Azure AD、Microsoft Azure、Office 365 和其他雲端服務所管理或報告的身分識別和存取。 如果您的組織具有內部部署系統管理帳戶，請經由[保護特殊權限存取](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/securing-privileged-access)，參閱從 Active Directory 管理的內部部署和混合式特殊權限存取所適用的指引。 

> [!NOTE] 
> 本文中的指引主要參照 Azure Active Directory Premium 計劃 P1 和 P2 所包含的 Azure Active Directory 功能。 Azure Active Directory Premium P2 隨附於 EMS E5 套件和 Microsoft 365 E5 套件。 本指引假設您的組織已為使用者購買 Azure AD Premium P2 授權。 如果您沒有這些授權，本指引可能有某些內容不適用於您的組織。 此外，在這整篇文章中，全域系統管理員 (或全域管理員) 皆與「公司系統管理員」或「租用戶系統管理員」同義。

## <a name="develop-a-roadmap"></a>擬定計劃 

Microsoft 建議您擬定並遵循適當計劃以保護特殊權限存取，使網路攻擊者無法取得。 您可以根據組織內的現有功能和特定需求，隨時調整您的計劃。 計劃中的每個階段均應提高對手對您的內部部署、雲端和混合式資產發動特殊權限存取攻擊的成本和難度。 Microsoft 建議您採用下列四個計劃階段：這項建議計劃首先根據 Microsoft 處理網路攻擊事件與回應的實務經驗，規劃了最有效且最快速的實作方式。 此計劃的時程是大略推估的。

![具時程的各個計劃階段](./media/directory-admin-roles-secure/roadmap-timeline.png)

* 階段 1 (24-48 小時)：建議您立即執行的重大事項

* 階段 2 (2-4 週)：減少最常用的攻擊技巧

* 階段 3 (1-3 個月)：建置可見性以及對管理活動的全面掌控

* 階段 4 (六個月後)：繼續建置防禦機制以進一步強化安全性平台

此計劃架構只在充分運用您已部署的 Microsoft 技術。 您也可以利用目前和未來的重要安全性技術，並整合其他廠商所提供、且您已部署或考慮部署的安全性工具。 

## <a name="stage-1-critical-items-that-we-recommend-you-do-right-away"></a>階段 1：建議您立即執行的重大事項

![階段 1](./media/directory-admin-roles-secure/stage-one.png)

計劃的階段 1 著重於可快速輕鬆實作的工作。 我們建議您立即在前 24-48 小時內完成這幾項工作，以確保特殊權限存取能夠得到基本層級的保護。 「受保護的特殊權限存取」計劃的這個階段包含下列動作：

### <a name="general-preparation"></a>一般準備工作

#### <a name="turn-on-azure-ad-privileged-identity-management"></a>開啟 Azure AD Privileged Identity Management

如果您尚未開啟 Azure AD Privileged Identity Management (PIM)，請在生產租用戶中加以開啟。 開啟 Privileged Identity Management 之後，您會收到有關於特殊權限存取角色有所變更的通知電子郵件訊息。 您目錄中的高特殊權限角色新增了其他使用者時，這些通知將會提供早期警告。

Azure AD Privileged Identity Management 隨附於 Azure AD Premium P2 或 EMS E5 中。 這些解決方案可協助您保護對內部部署環境與雲端中的應用程式和資源的存取。 如果您還沒有 Azure AD Premium P2 或 EMS E5，而想要評估此計劃中參考的更多功能，請註冊 [Enterprise Mobility + Security 的 90 天免費試用版](https://www.microsoft.com/cloud-platform/enterprise-mobility-security-trial)。 請使用這些授權試用 Azure AD Privileged Identity Management 和 Azure AD Identity Protection，透過 Azure AD 的進階安全性報告、稽核和警示來監視活動。

在您開啟 Azure AD Privileged Identity Management 後：

1. 使用具備生產租用戶之目錄全域管理員身分的帳戶，登入 [Azure 入口網站](https://portal.azure.com/)。

2. 若要選取要使用 Privileged Identity Management 的租用戶，請選取 Azure 入口網站右上角的使用者名稱。

3. 選取 [所有服務]，並篩選 [Azure AD Privileged Identity Management] 的清單。

4. 從 [所有服務] 清單中開啟 Privileged Identity Management，並將其釘選至儀表板。

在您的租用戶中使用 Azure AD Privileged Identity Management 的第一人，會自動被指派租用戶中的「安全性系統管理員」和「特殊權限角色管理員」角色。 只有特殊權限角色管理員才能管理使用者的 Azure AD Directory 角色指派。 此外，在新增 Azure AD Privileged Identity Management 後，畫面上會顯示安全性精靈引導您完成初始探索和指派體驗。 您此時可以結束精靈而不進行任何其他變更。 

#### <a name="identify-and-categorize-accounts-that-are-in-highly-privileged-roles"></a>識別及分類高特殊權限角色中的帳戶 

在開啟 Azure AD Privileged Identity Management 之後，請檢視全域系統管理員、特殊權限角色系統管理員、Exchange Online 系統管理員和 SharePoint Online 系統管理員等目錄角色中的使用者。 如果您的租用戶中沒有 Azure AD PIM，您可以使用 [PowerShell API](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)。 一開始請使用全域管理員角色，因為此角色是通用的：無論是在 Office 365 入口網站還是 Azure 入口網站中指派角色，或是使用適用於 Microsoft PowerShell 的 Azure AD 模組來指派角色，被指派此管理員角色的使用者在您組織所訂閱的所有雲端服務中都具有相同的權限。 

請移除這些角色中不再需要的任何帳戶，並將指派給管理員角色的其餘帳戶分類：

* 個別指派給系統管理使用者，並且也可用於非系統管理用途 (例如個人電子郵件)
* 個別指派給系統管理使用者，且指定為僅供系統管理之用
* 在多個使用者之間共用
* 用於急用緊急存取案例
* 用於自動化指令碼
* 用於外部使用者

#### <a name="define-at-least-two-emergency-access-accounts"></a>定義至少兩個緊急存取帳戶 

請務必控管好相關狀況，以避免使用者因為無法以系統管理員身分登入或啟動現有個別使用者的帳戶，而不慎在您的 Azure AD 租用戶中遭到鎖定而無法管理。 例如，如果組織與內部部署身分識別提供者同盟，使用者可能會因為該身分識別提供者無法使用而無法登入內部部署。 您可以藉由在租用戶中儲存兩個或多個緊急存取帳戶，來減輕不慎失去系統管理存取權的影響。

緊急存取帳戶可協助組織限制現有 Azure Active Directory 環境內的特殊權限存取。 這些帳戶具有高特殊權限，不會指派給特定個人。 緊急存取帳戶僅限用於無法使用一般系統管理帳戶的「急用」狀況。 組織必須控制並減少使用緊急帳戶，並明訂只有在必要時才能使用。 

請評估已指派或適用於全域管理員角色的帳戶。 如果您未看到任何僅限雲端帳戶使用 *.onmicrosoft.com 網域 (供「急用」緊急存取使用)，請加以建立。 如需詳細資訊，請參閱[在 Azure AD 中管理緊急存取系統管理帳戶](directory-emergency-access.md)。

#### <a name="turn-on-multi-factor-authentication-and-register-all-other-highly-privileged-single-user-non-federated-admin-accounts"></a>開啟多重要素驗證，並註冊所有其他高特殊權限的單一使用者非同盟管理員帳戶 

所有永久指派給一或多個 Azure AD 管理員角色的個別使用者在進行登入時，都必須進行 Azure Multi-Factor Authentication (MFA)：全域系統管理員、特殊權限角色系統管理員、Exchange Online 系統管理員和 SharePoint Online 系統管理員。 請使用指南啟用[管理員帳戶的 Multi-factor Authentication (MFA)](../authentication/howto-mfa-userstates.md)，並確定這些使用者全都已在 [https://aka.ms/mfasetup](https://aka.ms/mfasetup) 註冊。 您可以在[保護 Office 365 中的資料和服務存取](https://support.office.com/article/Protect-access-to-data-and-services-in-Office-365-a6ef28a4-2447-4b43-aae2-f5af6d53c68e)指南的步驟 2 和步驟 3 下找到詳細資訊。 

## <a name="stage-2-mitigate-the-most-frequently-used-attack-techniques"></a>階段 2：減少最常用的攻擊技巧

![階段 2](./media/directory-admin-roles-secure/stage-two.png)

計劃的階段 2 著重於降低認證竊取和濫用最常用的攻擊技巧，依設計應在大約 2-4 週內實作。 「受保護的特殊權限存取」計劃的這個階段包含下列動作。

### <a name="general-preparation"></a>一般準備工作

#### <a name="conduct-a-inventory-of-services-owners-and-admins"></a>進行服務、擁有者和系統管理員的清查

隨著「攜帶您自己的裝置 (BYOD)」和在家工作政策的普及，以及企業日漸採用無線連線，監視哪些人連線至您的網路，已成為不可或缺的工作。 有效的安全性稽核常會顯示在您的網路上執行的裝置、應用程式和程式未受到 IT 支援，因此可能不安全。 如需詳細資訊，請參閱 [Azure 安全性管理和監視概觀](../../security/security-management-and-monitoring-overview.md)。 請務必在您的清查程序中納入下列所有工作。 

* 識別具有系統管理角色的使用者，以及可供其進行管理的服務。
* 使用 Azure AD PIM 找出您的組織中可對 Azure AD 進行管理員存取的使用者，包括未在階段 1 中列出的其他角色。
* 除了在 Azure AD 中定義的角色以外，Office 365 也隨附一組您可以指派給組織使用者的管理員角色。 每個管理員角色都對應至常用的商務功能，並且可為組織中的成員提供在 Office 365 管理中心執行特定工作的權限。 使用 Office 管理中心找出您的組織中可對 Office 365 進行管理員存取的使用者，包括透過未受 Azure AD 管理的角色進行的存取。 如需詳細資訊，請參閱[關於 Office 365 管理員角色](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d)和 [Office 365 的安全性最佳做法](https://support.office.com/article/Security-best-practices-for-Office-365-9295e396-e53d-49b9-ae9b-0b5828cdedc3)。
* 在您的組織仰賴的其他服務中執行清查，例如 Azure、Intune 或 Dynamics 365。
* 確定您的管理員帳戶 (用於管理用途的帳戶，而不只是使用者的日常工作帳戶) 已有連結的工作電子郵件地址，並已註冊 Azure MFA 或使用 MFA 內部部署。
* 要求使用者提供他們需要系統管理存取權的業務理由。
* 對於不需要管理員存取權的服務與個人，應移除其存取權。

#### <a name="identify-microsoft-accounts-in-administrative-roles-that-need-to-be-switched-to-work-or-school-accounts"></a>識別系統管理角色中需要切換至工作或學校帳戶的 Microsoft 帳戶 

在某些情況下，組織的初始全域系統管理員在開始使用 Azure AD 時，會重複使用其現有的 Microsoft 帳戶認證。 這些 Microsoft 帳戶應取代為個別的雲端或同步處理的帳戶。 

#### <a name="ensure-separate-user-accounts-and-mail-forwarding-for-global-administrator-accounts"></a>確認全域系統管理員帳戶的個別使用者帳戶和郵件轉寄 

全域系統管理員帳戶不應有個人電子郵件地址，因為網路攻擊者會定期執行個人電子郵件帳戶的網路釣魚。 為了協助系統管理權限杜絕網際網路風險 (網路釣魚攻擊、無意間的網頁瀏覽)，請為每個具有系統管理權限的使用者建立專用帳戶。 

如果您尚未這樣做，請為執行全域管理工作的使用者建立個別帳戶，以確定他們不會不慎開啟或執行與其管理員帳戶相關聯的電子郵件或程式。 請確定這些帳戶會將其電子郵件轉寄至工作信箱。  

#### <a name="ensure-the-passwords-of-administrative-accounts-have-recently-changed"></a>確訂系統管理帳戶的密碼近期做過變更

請確定所有使用者都曾在過去 90 天內登入其系統管理帳戶並變更密碼至少一次。 此外，請確定任何有多個使用者知道密碼的共用帳戶近期都曾變更過密碼。

#### <a name="turn-on-password-hash-synchronization"></a>開啟密碼雜湊同步處理

密碼雜湊同步處理功能可用來將使用者密碼雜湊從內部部署 Active Directory 執行個體，同步到雲端式 Azure AD 執行個體。 即使您選擇使用與 Active Directory 同盟服務 (AD FS) 或其他身分識別提供者的同盟，仍可選擇性地設定密碼雜湊同步處理作為備用方式，以防 AD 或 ADFS 伺服器之類的內部部署基礎結構失敗或暫時無法使用。 這可讓使用者透過他們登入內部部署 AD 執行個體時所使用的相同密碼來登入服務。 此外，如果有使用者在未連線至 Azure AD 的其他服務上使用相同的電子郵件地址和密碼，Identity Protection 也將能夠藉由比較這些密碼雜湊與已知遭到破解的密碼，來偵測遭破解的認證。  如需詳細資訊，請參閱[使用 Azure AD Connect 同步實作密碼雜湊同步處理](../hybrid/how-to-connect-password-hash-synchronization.md)。

#### <a name="require-multi-factor-authentication-mfa-for-users-in-all-privileged-roles-as-well-as-exposed-users"></a>所有具有特殊權限角色的使用者和公開的使用者，都必須進行多重要素驗證 (MFA)

Azure AD 建議您要求所有使用者進行多重要素驗證 (MFA)，包括系統管理員，以及一旦帳戶遭到盜用則會產生重大衝擊的所有其他使用者 (例如財務人員)。 這可降低密碼遭入侵所導致的攻擊風險。

開啟：

* [高度曝險帳戶的 MFA](../authentication/multi-factor-authentication-security-best-practices.md)，例如組織中行政人員的帳戶 
* [與個別使用者相關聯的每個管理員帳戶的 MFA](../authentication/howto-mfa-userstates.md) (針對其他已連線的 SaaS 應用程式) 
* Microsoft SaaS 應用程式之所有管理員的 MFA，包括在 Exchange Online 和 Office 入口網站中受管理之角色中的系統管理員

如果您使用 Windows Hello 企業版，您可以使用 Windows Hello 登入體驗達到 MFA 需求。 如需詳細資訊，請參閱 [Windows Hello](https://docs.microsoft.com/windows/uwp/security/microsoft-passport)。 

#### <a name="configure-identity-protection"></a>設定 Identity Protection 

Azure AD Identity Protection 是一種以演算法為基礎的監視和報告工具，可用來偵測會影響組織身分識別的潛在弱點。 您可以設定自動化機制來回應這些偵測到的可疑活動，並採取適當動作加以解決。 如需詳細資訊，請參閱 [Azure Active Directory Identity Protection](../active-directory-identityprotection.md)。

#### <a name="obtain-your-office-365-secure-score-if-using-office-365"></a>取得您的 Office 365 安全分數 (如果使用 Office 365)

安全分數可指出您所使用的 Office 365 服務 (例如 OneDrive、SharePoint 和 Exchange)，然後查看您的設定和活動，並將其與 Microsoft 所建立的基準比較。 您的得分將會以您與最佳安全性實作的相符程度為準。 任何人只要具有 Office 365 商務進階版或企業版訂用帳戶的管理員權限 (全域管理員或自訂系統管理員角色)，即可在 [https://securescore.office.com](https://securescore.office.com/) 存取安全分數。

#### <a name="review-the-office-365-security-and-compliance-guidance-if-using-office-365"></a>檢閱 Office 365 安全性與合規性指引 (如果使用 Office 365)

[安全性與合規性計劃](https://support.office.com/article/Plan-for-security-and-compliance-in-Office-365-dc4f704c-6fcc-4cab-9a02-95a824e4fb57)概略說明了 Office 365 客戶應如何設定 Office 365 及使用其他 EMS 功能的方法。 接著，請檢閱如何[保護 Office 365 中的資料和服務存取](https://support.office.com/article/Protect-access-to-data-and-services-in-Office-365-a6ef28a4-2447-4b43-aae2-f5af6d53c68e)的步驟 3-6，以及如何[監視 Office 365 中的安全性和合規性](https://support.office.com/article/Monitor-security-and-compliance-in-Office-365-b62f1722-fd39-44eb-8361-da61d21509b6)的指南。


#### <a name="configure-office-365-activity-monitoring-if-using-office-365"></a>設定 Office 365 活動監視 (如果使用 Office 365)

您可以監視您的組織成員使用 Office 365 服務的情形，以識別具有系統管理帳戶的使用者，以及因為未登入這些入口網站而可能不需要 Office 365 存取的人員。 如需詳細資訊，請參閱 [Office 365 管理中心的活動報告](https://support.office.com/article/Activity-Reports-in-the-Office-365-admin-center-0d6dfb17-8582-4172-a9a9-aed798150263)。

#### <a name="establish-incidentemergency-response-plan-owners"></a>建立事件/緊急回應計劃擁有者

有效執行事件回應是複雜的作業。 因此，要建立成功的事件回應能力，有賴於充分的規劃和資源。 持續監視網路攻擊，並建立相關程序以排定處理事件的優先順序，是很重要的。 必須要有收集、分析和報告資料的有效方法，才能產生關聯性，並建立與其他內部群組和計劃擁有者之間的通訊。 如需詳細資訊，請參閱 [Microsoft 安全性回應中心](https://technet.microsoft.com/security/dn440717)。 

#### <a name="secure-on-premises-privileged-administrative-accounts-if-not-already-done"></a>保護內部部署的特殊權限系統管理帳戶 (如果尚未這麼做)

如果您的 Azure Active Directory 租用戶與內部部署 Active Directory 同步處理，請遵循[安全性特殊權限存取計劃](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/securing-privileged-access)：階段 1 中的指引。 其中包括為需要執行內部部署系統管理工作的使用者建立個別的管理員帳戶、為 Active Directory 系統管理員部署特殊權限存取工作站，以及為工作站和伺服器建立唯一的本機管理員密碼。

### <a name="additional-steps-for-organizations-managing-access-to-azure"></a>組織管理 Azure 存取的其他步驟

#### <a name="complete-an-inventory-of-subscriptions"></a>完成訂用帳戶的清查

使用企業版入口網站和 Azure 入口網站，識別您的組織中裝載生產應用程式的訂用帳戶。 

#### <a name="remove-microsoft-accounts-from-admin-roles"></a>從管理員角色移除 Microsoft 帳戶

來自其他程式 (例如 Xbox、Live 和 Outlook) 的 Microsoft 帳戶，不應作為組織訂用帳戶的系統管理員帳戶。 請從所有 Microsoft 帳戶中移除管理員狀態，並取代為 Active Directory (例如 chris@contoso.com) 工作或學校帳戶。

#### <a name="monitor-azure-activity"></a>監視 Azure 活動

「Azure 活動記錄」提供 Azure 中的訂用帳戶層級事件的記錄。 它提供關於何人建立、更新及刪除了什麼資源，以及這些事件於何時發生的資訊。 如需詳細資訊，請參閱[稽核和接收 Azure 訂用帳戶中關於重要動作的通知](../../monitoring-and-diagnostics/monitor-quick-audit-notify-action-in-subscription.md)。


### <a name="additional-steps-for-organizations-managing-access-to-other-cloud-apps-via-azure-ad"></a>組織透過 Azure AD 對其他雲端應用程式的存取進行管理的其他步驟 

#### <a name="configure-conditional-access-policies"></a>設定條件式存取原則

準備內部部署和雲端代管應用程式的條件式存取原則。 如果您有已加入使用者工作場所的裝置，請從[使用 Azure Active Directory 裝置註冊設定內部部署條件式存取](../active-directory-device-registration-on-premises-setup.md)取得詳細資訊。


## <a name="stage-3-build-visibility-and-take-full-control-of-admin-activity"></a>階段 3：建置可見性以及全面掌控管理活動

![階段 3](./media/directory-admin-roles-secure/stage-three.png)

階段 3 以階段 2 的緩解作業作為建置基礎，依設計應在大約 1-3 個月內實作。 「受保護的特殊權限存取」計劃的這個階段包含下列要項。

### <a name="general-preparation"></a>一般準備工作

#### <a name="complete-an-access-review-of-users-in-administrator-roles"></a>在系統管理員角色中完成使用者的存取權檢閱

有愈來愈多的公司使用者透過雲端服務取得特殊權限存取，這可能會導致未受管理的平台隨之增加。 其中包括將成為 Office 365 全域管理員的使用者、Azure 訂用帳戶系統管理員，以及可對 VM 或透過 SaaS 應用程式進行管理員存取的使用者。 此時，組織應讓所有員工 (尤其是管理員) 以不具特殊權限的使用者身分處理日常業務交易，而僅在必要時才採用管理員權限。 在初次採用管理員角色後具備此角色的使用者數目可能會增加，因此應完成存取權檢閱，以識別並確認每個適合啟動管理員權限的使用者。 

執行下列動作：

* 確認哪些使用者是 Azure AD 管理員，隨需啟用 Just-In-Time 管理員存取和角色型安全性控制。
* 將沒有正當理由可取得管理員特殊權限存取權的使用者轉換為不同角色 (如果沒有符合資格的角色，則將其移除)。

#### <a name="continue-rollout-of-stronger-authentication-for-all-users"></a>繼續對所有使用者施行更嚴格的驗證 

要求最高管理層、高階經理、重要 IT 人員和安全人員，以及其他高度公開的使用者進行最新的強式驗證，例如 Azure MFA 或 Windows Hello。 

#### <a name="use-dedicated-workstations-for-administration-for-azure-ad"></a>以專用的工作站處理 Azure AD 的管理工作

攻擊者可能嘗試以特殊權限帳戶作為目標來取得組織資料和系統的存取，進而能夠透過會更改程式邏輯或對輸入認證的管理員進行刺探的惡意程式碼，來干擾資料的完整性和真實性。 特殊權限存取工作站 (PAW) 提供敏感性工作的專用作業系統，以免遭受網際網路攻擊和威脅載體攻擊。 將這些敏感性工作和帳戶與日常使用的工作站和裝置分隔，可提供非常強大的防護功能，以免遭受網路釣魚攻擊、應用程式和作業系統弱點、各種模擬攻擊以及認證盜用攻擊，例如按鍵輸入記錄、傳遞雜湊 (Pass-the-Hash) 和傳遞票證 (Pass-The-Ticket) 等攻擊。 藉由部署特殊權限存取工作站，您可以降低管理員在未經強化的桌面環境中輸入管理員認證的風險。 如需詳細資訊，請參閱[特殊權限存取工作站](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations)。

#### <a name="review-national-institute-of-standards-and-technology-recommendations-for-handling-incidents"></a>檢閱美國國家標準技術局的事件處理建議 

美國國家標準技術局 (NIST) 提供了事件處理的指導方針，特別是在分析事件相關資料和判斷每個事件的適當回應方面，有更明確的指示。 如需詳細資訊，請參閱 [NIST Computer Security Incident Handling Guide (SP 800-61，修訂 2)](http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)。

#### <a name="implement-privileged-identity-management-pim-for-jit-to-additional-administrative-roles"></a>對其他系統管理角色實作適用於 JIT 的 Privileged Identity Management (PIM)

對於 Azure Active Directory，請使用 [Azure AD Privileged Identity Management](../privileged-identity-management/pim-configure.md) 功能。 您可以透過下列方式在時間限制內啟用特殊權限角色：

* 啟動執行特定工作的管理員權限
* 在啟用程序期間強制執行 MFA
* 使用警示通知管理員有頻外變更
* 讓使用者可在預先設定的一段時間內保留特定權限
* 允許安全性管理員探索所有特殊權限身分識別、檢視稽核報告以及建立存取權檢閱，以識別每個有資格啟動管理員權限的使用者

如果您已在使用 Azure AD Privileged Identity Management，請視需要調整時限性權限的時間範圍 (例如維護期間)。

#### <a name="determine-exposure-to-password-based-sign-in-protocols-if-using-exchange-online"></a>判斷密碼式登入通訊協定的風險 (如果使用 Exchange Online)

在過去，通訊協定會假設使用者名稱/密碼組合已內嵌在裝置、電子郵件帳戶、電話等項目中。 但現在由於雲端中潛藏網路攻擊的風險，我們建議您識別每個可能因為認證遭破解而對組織造成嚴重危害的使用者，並藉由實作強式驗證需求和條件式存取，使其無法透過使用者名稱/密碼登入電子郵件。 

#### <a name="complete-a-roles-review-assessment-for-office-365-roles-if-using-office-365"></a>完成 Office 365 角色的角色檢閱評估 (如果使用 Office 365)

評估所有的管理員使用者是否都屬於正確的角色 (根據這項評估刪除並重新指派)。

#### <a name="review-the-security-incident-management-approach-used-in-office-365-and-compare-with-your-own-organization"></a>檢閱 Office 365 中使用的安全性事件管理方法，並與您自己的組織進行比較

您可以從 [Microsoft Office 365 中的安全性事件管理](https://www.microsoft.com/download/details.aspx?id=54302)下載此報告。

#### <a name="continue-to-secure-on-premises-privileged-administrative-accounts"></a>繼續保護內部部署的特殊權限系統管理帳戶

如果您的 Azure Active Directory 連線至內部部署 Active Directory，請遵循[安全性特殊權限存取計劃](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/securing-privileged-access)：階段 2 中的指引。 這包括部署所有系統管理員的特殊權限存取工作站、要求 MFA，使用恰到好處的管理進行 DC 維護、減少網域的攻擊面、部署 ATA 以進行攻擊偵測。

### <a name="additional-steps-for-organizations-managing-access-to-azure"></a>組織管理 Azure 存取的其他步驟

#### <a name="establish-integrated-monitoring"></a>建立整合式監視

[Azure 資訊安全中心](../../security-center/security-center-intro.md)能提供 Azure 訂用帳戶之間的整合式安全性監視和原則管理，協助偵測原本可能被忽略的威脅，且適用於廣泛的安全性解決方案生態系統。

#### <a name="inventory-your-privileged-accounts-within-hosted-virtual-machines"></a>清查您在代管虛擬機器內的特殊權限帳戶

在大部分情況下，您並不需要為使用者提供對您所有 Azure 訂用帳戶或資源的無限制權限。 您可以使用 Azure AD 管理員角色來區隔組織內的職責，而僅授與使用者執行特定作業所需的存取權。 例如，使用 Azure AD 系統管理員角色讓一個管理員僅管理某個訂用帳戶中的 VM，而讓另一個管理員管理相同訂用帳戶中的 SQL 資料庫。 如需詳細資訊，請參閱[在 Azure 入口網站中開始使用角色型存取控制](../../role-based-access-control/overview.md)。

#### <a name="implement-pim-for-azure-ad-administrator-roles"></a>為 Azure AD 系統管理員角色實作 PIM

透過具有 Azure AD 系統管理員角色的 Privileged Identity Management，可管理、控制及監視對 Azure 資源的存取。 使用 PIM 可降低權限的曝光時間，並透過報告和警示提升您對其使用情況的了解，以保護特殊權限帳戶免受網路攻擊的威脅。 如需詳細資訊，請參閱[使用 Privileged Identity Management 管理對 Azure 資源的 RABC 存取](../../role-based-access-control/pim-azure-resource.md)。

#### <a name="use-azure-log-integrations-to-send-relevant-azure-logs-to-your-siem-systems"></a>使用 Azure 記錄整合將相關 Azure 記錄傳送至您的 SIEM 系統 

Azure 記錄整合可讓您將來自 Azure 資源的原始記錄整合到組織現有的安全性資訊與事件管理 (SIEM) 系統。 [Azure 記錄整合](../../security/security-azure-log-integration-overview.md)會從 Windows 事件檢視器記錄收集 Windows 事件，並從 Azure 活動記錄、Azure 資訊安全中心警示和 Azure 診斷記錄收集 Azure 資源。 


### <a name="additional-steps-for-organizations-managing-access-to-other-cloud-apps-via-azure-ad"></a>組織透過 Azure AD 對其他雲端應用程式的存取進行管理的其他步驟

#### <a name="implement-user-provisioning-for-connected-apps"></a>為連線的應用程式實作使用者佈建

Azure AD 可讓您自動化在雲端 (SaaS) 應用程式 (例如 Dropbox、Salesforce、ServiceNow 等等) 中建立、維護和移除使用者身分識別的作業。 如需詳細資訊，請參閱[自動化使用 Azure AD 對於 SaaS 應用程式的使用者佈建和解除佈建](../manage-apps/user-provisioning.md)。

#### <a name="integrate-information-protection"></a>整合資訊保護

MCAS 可讓您調查檔案，並根據 Azure 資訊保護分類標籤設定原則，讓您能夠進一步檢視及控制雲端中的資料。 掃描及分類雲端中的檔案，並套用 Azure 資訊保護標籤。 如需詳細資訊，請參閱 [Azure 資訊保護整合](https://docs.microsoft.com/cloud-app-security/azip-integration)。

#### <a name="configure-conditional-access"></a>設定條件式存取

根據群組、位置和應用程式敏感性。設定 [SaaS 應用程式](https://azure.microsoft.com/overview/what-is-saas/)和 Azure AD 連線應用程式的條件式存取。 

#### <a name="monitor-activity-in-connected-cloud-apps"></a>監視連線的雲端應用程式中的活動

為了確保使用者的存取在連線的應用程式中也會受到保護，我們建議您使用 [Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/what-is-cloud-app-security)。 這可以讓您透過下列方式保護企業對雲端應用程式的存取，以及保護您的管理員帳戶：

* 擴充對雲端應用程式的可見性和控制
* 建立存取、活動和資料共用的原則
* 自動識別風險的活動、異常行為和威脅
* 防止資料外洩
* 盡可能降低風險，並將威脅防護和原則強制執行自動化

Cloud App Security SIEM 代理程式可整合 Cloud App Security 與您的 SIEM 伺服器，讓您集中監視 Office 365 警示和活動。 它會在您的伺服器上執行，並從 Cloud App Security 提取警示和活動，再將其串流至 SIEM 伺服器。 如需詳細資訊，請參閱 [SIEM 整合](https://docs.microsoft.com/cloud-app-security/siem)。

## <a name="stage-4-continue-building-defenses-to-a-more-proactive-security-posture"></a>階段 4：繼續建置防禦機制以達到更為主動的安全性狀態


![階段 4](./media/directory-admin-roles-secure/stage-four.png)

計劃的階段 4 以階段 3 的可見性作為建置基礎，依設計應在六個月後實作。 完成計劃可協助您擬定健全的特殊權限存取保護，以抵禦目前已知和新興的潛在攻擊。 壞消息是，安全性威脅不斷演化和變換，因此，建議您將安全性視為設法提高對手對您的環境展開攻擊的成本及降低其成功率的持續過程。

保護特殊權限存取，是為現今組織中的企業資產建立安全保障的首要步驟，但這並不是完整安全計劃的唯一要件。在計劃中還必須考量其他要素，才能提供持續的安全保障，例如原則、作業、資訊安全性、伺服器、應用程式、PC、裝置、雲端網狀架構和其他元件。 

除了管理您的特殊權限存取帳戶以外，建議您也應持續檢討下列事項：

* 確定管理員以不具特殊權限的使用者身分執行其日常業務。
* 在必要時才授與特殊權限存取權，且不再需要時即應移除 (Just-In-Time)。
* 保留並檢閱與特殊權限帳戶有關的稽核活動。

如需關於如何建置完整安全性計劃的詳細資訊，請參閱 [Microsoft 雲端 IT 架構資源](https://docs.microsoft.com/office365/enterprise/microsoft-cloud-it-architecture-resources)。 若想進一步了解如何預約 Microsoft 服務以取得前述任一主題的相關協助，請連絡您的 Microsoft 代表，或參閱[建置重要網路防禦機制以保護您的企業](https://www.microsoft.com/microsoftservices/campaigns/cybersecurity-protection.aspx)。

「受保護的特殊權限存取」計劃的這個最後階段包含下列要項。

### <a name="general-preparation"></a>一般準備工作

#### <a name="review-admin-roles-in-azure-active-directory"></a>檢閱 Azure Active Directory 中的管理員角色 

確認目前的內建 Azure AD 管理員角色是否仍保持最新狀態，並確定使用者是否僅具備其對應權限所需的角色和委派。 透過 Azure AD，您可以指定個別的系統管理員來執行不同的功能。 如需詳細資訊，請參閱[在 Azure Active Directory 中指派系統管理員角色](directory-assign-admin-roles.md)。

#### <a name="review-users-who-have-administration-of-azure-ad-joined-devices"></a>檢閱有哪些使用者可管理已加入 Azure AD 的裝置

如需詳細資訊，請參閱[如何設定已加入 Azure Active Directory 的混合式裝置](../device-management-hybrid-azuread-joined-devices-setup.md)。

#### <a name="review-members-of-built-in-office-365-admin-roleshttpssupportofficecomarticleabout-office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d"></a>檢閱[內建 Office 365 管理員角色](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d)的成員
如果您使用 Office 365。
‎
#### <a name="validate-incident-response-plan"></a>驗證事件回應計劃

若要改善您的計劃，Microsoft 建議您定期驗證您的計劃如預期運作：

* 檢查您現有的計劃是否有缺失
* 根據事後分析，修訂現有的最佳做法或定義新的
* 確實您已更新的事件回應計劃和最佳做法發佈到您整個組織


### <a name="additional-steps-for-organizations-managing-access-to-azure"></a>組織管理 Azure 存取的其他步驟 

確認您是否需要[將 Azure 訂用帳戶的擁有權轉移給另一個帳戶](../../billing/billing-subscription-transfer.md)。
‎

## <a name="break-glass-what-to-do-in-an-emergency"></a>「急用」：緊急狀況的因應措施

![緊急狀況](./media/directory-admin-roles-secure/emergency.jpeg)

1. 將事件的相關資訊告知有關的經理和安全人員。

2. 檢閱您的攻擊腳本。 

3. 存取您的「急用」帳戶使用者名稱/密碼組合，以登入 Azure AD。 

4. [提出 Azure 支援要求](../../azure-supportability/how-to-create-azure-support-request.md)以取得 Microsoft 的協助。

5. 查看 [Azure AD 登入報告](../reports-monitoring/overview-reports.md)。 事件在發生後可能需要一段延遲時間才會包含在報告中。

6. 在混合式環境中，如果同盟和您的 AD FS 伺服器無法使用，您可能需要暫時從同盟驗證切換為使用密碼雜湊同步。這會使網域同盟還原為受控驗證，直到 AD FS 伺服器恢復可用性為止。

7. 監控特殊權限帳戶的電子郵件。

8. 確實儲存相關記錄的備份，以供可能的鑑識和法律調查使用。

如需關於 Microsoft Office 365 如何處理安全性事件的詳細資訊，請參閱 [Microsoft Office 365 中的安全性事件管理](http://aka.ms/Office365SIM)。

## <a name="faq-common-questions-we-receive-regarding-securing-privileged-access"></a>常見問題集：我們在特殊權限存取的保護方面經常收到的問題  


**問：** 如果我尚未實作任何安全存取元件，應該怎麼辦？

**答：** 定義至少兩個急用帳戶、將 MFA 指派給您的特殊權限管理員帳戶，並將使用者帳戶與全域管理員帳戶區隔開來。


**問：** 遭到入侵後，應最先處理的問題為何？

**答：** 確定您對高度公開的個人要求最嚴格的驗證。


**問：** 如果我們的特殊權限管理員已停用，將會如何？

**答：** 建立永遠保持最新狀態的全域管理員帳戶。


**問：** 如果只剩下一個全域管理員，且無法加以聯繫，將會如何？ 

**答：** 使用您的其中一個急用帳戶立即取得特殊權限存取。


**問：** 如何保護組織內的管理員？

**答：** 讓管理員一律以「不具特殊權限的」標準使用者身分執行日常業務。
 

**問：** 在 Azure AD 中建立管理員帳戶的最佳做法為何？

**答：** 將特殊權限存取保留給特定管理工作。


**問：** 有哪些工具可減少持續性管理員存取？

**答：** Privileged Identity Management (PIM) 和 Azure AD 管理員角色。


**問：** 將管理員帳戶同步至 Azure AD 時，Microsoft 的定位為何？

**答：** 第 0 層管理員帳戶 (包括帳戶、群組，以及其他可直接或間接對 AD 樹系、網域或網域控制站和所有資產進行系統管理控制的資產) 只會用於內部部署 AD 帳戶，且通常不會針對雲端的 Azure AD 進行同步處理。 


**問：** 如何防止管理員在入口網站中指派隨機的管理員存取？

**答：** 對所有使用者和大部分的管理員使用非特殊權限帳戶。 首先應確認組織的使用量，以判斷哪幾個管理員帳戶應具有特殊權限。 接著必須監視新建立的系統管理使用者。


## <a name="next-steps"></a>後續步驟

* [Microsoft 信任中心 (產品安全性)](https://www.microsoft.com/trustcenter/security) – Microsoft 雲端產品和服務的安全性功能

* [Microsoft 信任中心 - 合規性](https://www.microsoft.com/trustcenter/compliance/complianceofferings) – Microsoft 針對雲端服務提供的一組完整的合規性供應項目

* [如何執行風險評定的指引](https://www.microsoft.com/trustcenter/guidance/risk-assessment) - 管理 Microsoft 雲端服務的安全性與合規性需求

### <a name="other-ms-online-services"></a>其他 MS Online Services 

* [Microsoft Intune 安全性](https://www.microsoft.com/trustcenter/security/intune-security) – Intune 可從雲端提供行動裝置管理、行動應用程式管理，和電腦管理能力。

* [Microsoft Dynamics 365 安全性](https://www.microsoft.com/trustcenter/security/dynamics365-security) – Dynamics 365 是 Microsoft 雲端式解決方案，可統合客戶關係管理 (CRM) 與企業資源規劃 (ERP) 功能。

 
