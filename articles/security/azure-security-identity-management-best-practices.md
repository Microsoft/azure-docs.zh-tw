---
title: Azure 身分識別和存取安全性的最佳做法 | Microsoft Docs
description: 本文提供使用內建 Azure 功能的一些身分識別管理和存取控制最佳作法。
services: security
documentationcenter: na
author: barclayn
manager: mbaldwin
editor: TomSh
ms.assetid: 07d8e8a8-47e8-447c-9c06-3a88d2713bc1
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/26/2018
ms.author: barclayn
ms.openlocfilehash: e3fe033de05ed42d221795159461048790e1cec8
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39493297"
---
# <a name="azure-identity-management-and-access-control-security-best-practices"></a>Azure 身分識別管理和存取控制安全性最佳作法

許多人認為身分識別是安全性的新界限層，並從傳統以網路為中心的觀點來接收該角色。 此安全性注意和投資進展的主要關鍵是由於網路周邊的漏洞日益增加，而且周邊防禦的效果不如 [BYOD](http://aka.ms/byodcg) 裝置和雲端應用程式暴增前的效果。

本文會討論一系列的 Azure 身分識別管理和存取控制安全性最佳做法。 這些最佳作法衍生自我們的 [Azure AD](../active-directory/fundamentals/active-directory-whatis.md) 經驗和客戶的經驗。

針對每個最佳做法，我們會說明︰

* 最佳作法是什麼
* 您為何想要啟用該最佳作法
* 如果無法啟用最佳作法，結果可能為何
* 最佳作法的可能替代方案
* 如何學習啟用最佳作法

這篇「Azure 身分識別管理和存取控制安全性最佳作法」是以共識意見及 Azure 平台功能和特性集 (因為在撰寫本文時已存在) 為基礎。 意見和技術會隨著時間改變，這篇文章將會定期進行更新以反映這些變更。

本文討論的 Azure 身分識別管理和存取控制安全性最佳作法包括：

* 集中管理您的身分識別
* 啟用單一登入 (SSO)
* 部署密碼管理
* 對使用者強制執行 Multi-Factor Authentication (MFA)
* 使用角色型存取控制 (RBAC)
* 使用 Resource Manager 控制資源的建立位置
* 引導開發人員運用 SaaS 應用程式的身分識別功能
* 主動監視可疑的活動

## <a name="centralize-your-identity-management"></a>集中管理您的身分識別

保護您的身分識別的一個重要步驟，就是確保 IT 可以從一個有關此帳戶建立所在的單一位置管理帳戶。 雖然大部分的企業 IT 組織都會在內部部署主要帳戶目錄，但混合式雲端部署逐漸增加，因此，您務必了解如何整合內部部署和雲端目錄並為使用者提供順暢的體驗。

若要完成此[混合式身分識別](../active-directory/active-directory-hybrid-identity-design-considerations-overview.md)案例，我們建議兩個選項：

* 使用 Azure AD Connect 同步處理內部部署目錄與雲端目錄。
* 使用[密碼雜湊同步處理](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-hash-synchronization)或[傳遞驗證](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication-faq)來啟用單一登入，或者使用 [Active Directory 同盟服務](https://docs.microsoft.com/windows-server/identity/ad-fs/deployment/deploying-federation-servers) (AD FS) \(機器翻譯\)，使內部部署身分識別與雲端目錄結成同盟

無法整合內部部署身分識別與雲端身分識別的組織會面臨用於管理帳戶的系統管理額外負荷增加，這會使錯誤和安全性缺口的可能性提高。

如需 Azure AD 同步處理的詳細資訊，請參閱[整合內部部署身分識別與 Azure Active Directory](../active-directory/active-directory-aadconnect.md) 一文。

## <a name="enable-single-sign-on-sso"></a>啟用單一登入 (SSO)

當您有多個目錄要管理時，這不只會成為 IT 的系統管理問題，對於必須記住多組密碼的使用者而言也是個問題。 使用 [SSO](https://azure.microsoft.com/documentation/videos/overview-of-single-sign-on/) \(英文\)，為使用者提供使用同一組認證來登入和存取所需資源的能力，而不論此資源位於內部部署或雲端。

使用 SSO 讓使用者根據其在 Azure AD 中的組織帳戶存取其 [SaaS 應用程式](../active-directory/manage-apps/what-is-single-sign-on.md)。 這不只適用於 Microsoft SaaS 應用程式，也適用於其他應用程式，例如 [Google Apps](../active-directory/saas-apps/google-apps-tutorial.md) 和 [Salesforce](../active-directory/saas-apps/salesforce-tutorial.md)。 您的應用程式可以設定為使用 Azure AD 作為 [SAML 型身分識別提供者](../active-directory/fundamentals-identity.md)。 為了控制安全性，Azure AD 不會核發允許他們登入應用程式的權杖，除非他們已使用 Azure AD 獲得存取存取權。 您可以直接授與存取權，或透過其所屬的群組授與。

> [!NOTE]
> 使用 SSO 的決策會影響您整合內部部署目錄與雲端目錄的方式。 如果您想要 SSO，則必須使用同盟，因為目錄同步處理只會提供[相同的登入體驗](../active-directory/active-directory-aadconnect.md)。
>
>

未對使用者和應用程式強制執行 SSO 的組織更容易遭遇使用者有多組密碼的情況，以致於會直接提高使用者重複使用密碼或使用弱式密碼的可能性。

若要深入了解 Azure AD SSO，請閱讀[使用 Azure AD Connect 管理和自訂 AD FS](../active-directory/active-directory-aadconnect-federation-management.md) 一文。

## <a name="deploy-password-management"></a>部署密碼管理

在您有多個租用戶或想要讓使用者[重設其密碼](../active-directory/user-help/active-directory-passwords-update-your-own-password.md)的案例中，請務必使用適當的安全性原則來防止不當使用。 在 Azure 中，您可以運用自助密碼重設功能，並自訂安全性選項以符合您的商務需求。

務必向這些使用者取得意見反應，並從他們嘗試執行這些步驟時的體驗中學習。 根據這些經驗進行詳細計劃，以緩和在大型群組部署期間可能發生的潛在問題。 此外，也建議您使用[密碼重設登錄活動報告](../active-directory/active-directory-passwords-get-insights.md)來監視進行註冊的使用者。

想要避免密碼變更支援電話但可讓使用者重設其密碼的組織，其服務中心比較容易接到較多有關密碼問題的電話量。 在有多個租用戶的組織中，務必實作此類型的功能，並可讓使用者在安全性原則中建立的安全範圍內執行密碼重設。

您可以閱讀[部署密碼管理和訓練使用者使用密碼](../active-directory/authentication/howto-sspr-deployment.md)一文，進一步了解密碼重設。

## <a name="enforce-multi-factor-authentication-mfa-for-users"></a>對使用者強制執行 Multi-Factor Authentication (MFA)

對於需要符合業界標準 (例如 [PCI DSS 3.2 版](http://blog.pcisecuritystandards.org/preparing-for-pci-dss-32)) 的組織，Multi-Factor Authentication 是驗證使用者的必備功能。 除了符合業界標準以外，強制執行 MFA 來驗證使用者，也可協助組織緩和認證竊取攻擊類型，例如[傳遞雜湊 (Pass-the-Hash，PtH)](http://aka.ms/PtHPaper)。

為您的使用者啟用 Azure MFA，您可為使用者登入和交易增加第二層安全性。 在此情況下，交易可能會存取位於檔案伺服器或 SharePoint Online 中的文件。 Azure MFA 也會協助 IT 降低遭盜用的認證能夠存取公司資料的可能性。

例如︰對您的使用者強制執行 Azure MFA，並將它設定為使用電話或簡訊進行驗證。 如果使用者的認證遭到盜用，攻擊者就無法存取任何資源，因為他們無法存取使用者的電話。 未新增額外身分識別保護層的組織會更容易受到認證竊取攻擊，這可能會導致資料洩漏。

想要將整個驗證控制保留於內部部署的組織有一個替代方法，就是使用 [Azure Multi-factor Authentication Server](../active-directory/authentication/howto-mfaserver-deploy.md) (也稱為 MFA 內部部署)。 使用此方法，您仍可強制執行 Multi-Factor Authentication，同時保留 MFA 伺服器內部部署。

如需 Azure MFA 的詳細資訊，請參閱[開始在雲端中使用 Azure Multi-Factor Authentication](../active-directory/authentication/howto-mfa-getstarted.md) 一文。

## <a name="use-role-based-access-control-rbac"></a>使用角色型存取控制 (RBAC)

對於想要強制執行資料存取安全性原則的組織，根據[需要知道](https://en.wikipedia.org/wiki/Need_to_know)和[最低權限](https://en.wikipedia.org/wiki/Principle_of_least_privilege)安全性原則限制存取權限是必須做的事。 Azure 角色型存取控制 (RBAC) 可用來指派權限給特定範圍的使用者、群組和應用程式。 角色指派的範圍可以是訂用帳戶、資源群組或單一資源。

您可以利用 Azure 中[內建的 RBAC](../role-based-access-control/built-in-roles.md) 角色指派權限給使用者。 請考慮將*儲存體帳戶參與者*用於需要管理儲存體帳戶的雲端操作者，以及使用*傳統儲存體帳戶參與者*角色來管理傳統儲存體帳戶。 對於需要管理 VM 和儲存體帳戶的雲端操作者，請考慮將他們新增至*虛擬機器參與者*角色。

未利用諸如 RBAC 等功能來強制執行資料存取控制的組織，可能會對其使用者提供超過所需的權限。 一開始就讓使用者存取他們不應具備的特定資料類型 (例如，高度業務衝擊)，可能會導致資料洩露。

若要深入了解 Azure RBAC，請閱讀 [Azure 角色型存取控制](../role-based-access-control/role-assignments-portal.md)一文。

## <a name="control-locations-where-resources-are-created-using-resource-manager"></a>使用 Resource Manager 控制資源的建立位置

讓雲端操作者能夠執行工作，同時防止他們違反管理組織資源所需的慣例，極為重要。 想要控制資源建立位置的組織應將這些位置硬式編碼。

若要達成此目標，組織可以建立安全性原則，其定義會描述遭拒的動作或資源。 在所需範圍內指派那些原則定義，例如訂用帳戶、資源群組或是個別的資源。

> [!NOTE]
> 這與 RBAC 不同，其實際利用 RBAC 來驗證使用者是否有建立這些資源的權限。
>
>

利用 [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md) 來建立同樣適用於以下案例的自訂原則：組織只想在與適當的成本中心相關聯時允許作業，否則會拒絕要求。

不控制資源建立方式的組織比較容易遇到使用者因建立超過所需的資源而濫用服務。 強化資源建立程序是保護多租用戶案例的重要步驟。

您可以閱讀[什麼是 Azure 原則？](../azure-policy/azure-policy-introduction.md)一文，深入了解如何使用 Azure Resource Manager 建立原則。

## <a name="guide-developers-to-leverage-identity-capabilities-for-saas-apps"></a>引導開發人員運用 SaaS 應用程式的身分識別功能

當使用者存取可與內部部署或雲端目錄整合的 [SaaS 應用程式](https://azure.microsoft.com/marketplace/active-directory/all/)時，可在許多案例中運用使用者身分識別。 最重要的是，我們建議開發人員使用安全的方法來開發這些應用程式，例如 [Microsoft 安全性開發生命週期 (SDL)](https://www.microsoft.com/sdl/default.aspx)。 Azure AD 提供身分識別作為服務，支援業界標準通訊協定 (例如 [OAuth 2.0](http://oauth.net/2/) \(英文\) 和 [OpenID Connect](http://openid.net/connect/) \(英文\))，以及適用於不同平台的開放原始碼程式庫，來簡化開發人員的驗證工作。

務必註冊將驗證外包給 Azure AD 的任何應用程式，這是必要的程序。 其背後的理由是因為 Azure AD 在處理登入 (SSO) 或交換權杖時，需要座標才能與應用程式通訊。 當 Azure AD 所簽發的權杖存留期到期時，使用者工作階段就過期。 一定要評估您的應用程式是否應該使用此時間，或者可以縮短此時間。 縮短存留期可做為安全措施，以根據一段閒置時間強制使用者登出。

未強制身分識別控制存取應用程式且不會引導其開發人員如何安全地整合應用程式與其身分識別管理系統的組織，可能更容易遭受認證竊取攻擊類型，例如 [Open Web Application Security Project (OWASP) Top 10 中所述的弱式驗證與工作階段管理](https://www.owasp.org/index.php/OWASP_Top_Ten_Cheat_Sheet)。

您可以閱讀 [Azure AD 的驗證案例](../active-directory/develop/authentication-scenarios.md)，進一步了解 SaaS 應用程式的驗證案例。

## <a name="actively-monitor-for-suspicious-activities"></a>主動監視可疑的活動

根據 [Verizon 2016 資料缺口報告](http://www.verizonenterprise.com/verizon-insights-lab/dbir/2016/)，認證洩漏仍在上升中，並成為網路罪犯的最有利潤業務之一。 基於這個理由，請務必備妥可快速偵測可疑行為活動並觸發警示的主動身分識別監視系統，以便進一步調查。 Azure AD 有兩項主要功能可協助組織監視其身分識別：Azure AD Premium [異常報告](../active-directory/active-directory-view-access-usage-reports.md)與 [Azure AD 身分識別保護](../active-directory/active-directory-identityprotection.md)功能。

務必使用異常報告來找出[不被追蹤](../active-directory/active-directory-reporting-sign-ins-from-unknown-sources.md)的登入嘗試、對特定帳戶的[暴力密碼破解](../active-directory/active-directory-reporting-sign-ins-after-multiple-failures.md)攻擊、來自多個位置的登入嘗試，以及來自受感染裝置和可疑 IP 位址的登入嘗試。 請記住，這些都是報告。 換句話說，您必須備妥相關處理和程序，以便 IT 系統管理員每天或依需求執行這些報告 (通常出現在事件回應案例)。

相反地，Azure AD Identity Protection 是主動監視系統，它會在自己的儀表板上標示目前的風險。 除此之外，您也會透過電子郵件收到每日摘要通知。 我們建議根據您的業務需求調整風險層級。 風險事件的風險層級可表示風險事件的嚴重性 (高、中或低)。 風險層級可協助 Identity Protection 使用者排定為了降低組織風險而必須採取之行動的優先順序。

未主動監視其身分識別系統的組織有洩漏使用者認證的風險。 若不知道有使用這些認證進行的可疑活動，組織將無法減輕這類型的威脅。
您可以閱讀 [Azure Active Directory 身分識別保護](../active-directory/active-directory-identityprotection.md)，進一步了解 Azure 身分識別保護。
