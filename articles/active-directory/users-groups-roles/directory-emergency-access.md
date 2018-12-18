---
title: 在 Azure AD 中管理緊急存取系統管理帳戶 | Microsoft Docs
description: 本文會說明如何使用緊急存取帳戶，協助組織限制現有 Azure Active Directory 環境內的特殊權限存取。
services: active-directory
author: markwahl-msft
ms.author: billmath
ms.date: 12/13/2017
ms.topic: article-type-from-white-list
ms.service: active-directory
ms.workload: identity
ms.custom: it-pro
ms.reviewer: markwahl-msft
ms.openlocfilehash: 4f3772abc1cdbd3b35b8b1f16e7a47c0f1a17783
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38595649"
---
# <a name="manage-emergency-access-administrative-accounts-in-azure-ad"></a>在 Azure AD 中管理緊急存取系統管理帳戶 

對於大部分的日常活動，您的使用者不需要「全域管理員」權限。 使用者不應永久指派給該角色，因為使用者可能會不慎執行所需權限高於他們應有權限的工作。 當使用者不需作為全域管理員時，應使用 Azure Active Directory (Azure AD) Privileged Identity Management (PIM) 在其自己的帳戶或其他系統管理帳戶上啟動角色指派。

除了使用者可為本身取得系統管理存取權限外，您也需要避免不慎鎖定 Azure AD 租用戶的系統管理，因為您可能會無法以管理員身分您登入和啟動現有的個別使用者帳戶。 您可以藉由在租用戶中儲存兩個或多個「緊急存取帳戶」，來減緩不慎失去系統管理存取權的影響。

緊急存取帳戶可協助組織限制現有 Azure Active Directory 環境內的特殊權限存取。 此類帳戶具有高度權限，不會指派給特定個人。 緊急存取帳戶僅限於緊急或「急用」狀況，且這些狀況無法使用一般系統管理帳戶。 組織必須限制只有在必要期間才能使用緊急帳戶，並維護此限制目標。

組織在下列情況中可能需要使用緊急存取帳戶：

 - 使用者帳戶為同盟，但由於行動網路中斷或身分識別提供者運作中斷，同盟目前停用。 例如，如果您環境中的識別提供者主機已中斷，那麼當 Azure AD 重新導向至使用者的識別提供者時，使用者可能無法登入。 
 - 管理員已透過 Azure Multi-Factor Authentication 進行註冊，但他們的所有個別裝置皆無法使用。 使用者可能無法完成 Multi-Factor Authentication 來啟動角色。 例如，行動網路的中斷會讓使用者無法接聽來電或接收文字簡訊 (唯一為裝置註冊的兩種驗證機制)。 
 - 具有最新全域系統管理存取權的人員已離開組織。 Azure AD 可防止最新的「全域管理員」帳戶遭到刪除，但無法防止該帳戶在內部部署環境中遭到刪除或停用。 其中任一情況皆可能造成組織無法復原帳戶。

## <a name="initial-configuration"></a>初始設定

建立兩個或以上的緊急存取帳戶。 這些帳戶應該是使用 \*.onmicrosoft.com 網域的雲端限定帳戶，而且未與內部部署環境同盟或同步。 

這些帳戶不應與組織中的任何個別使用者相關聯。 組織需要確保這些帳戶的認證保持在安全狀態，且僅有獲得授權使用的個人知道該認證內容。 

> [!NOTE]
> 緊急存取帳戶的帳戶密碼通常會分成兩個或三個部分、記錄在不同的紙上，以及儲存在不同安全地點的安全防火保險箱中。 
>
> 請確定緊急存取帳戶未連線至任何員工提供的行動電話、會與個別員工一同移動的硬體權杖，或其他員工專屬的認證。 此預防措施適用於需要認證時卻聯絡不到個別員工的狀況。 

### <a name="initial-configuration-with-permanent-assignments"></a>使用永久指派的初始設定

其中一個方法是讓使用者成為「全域管理員」角色的永久成員。 此方法適用於沒有 Azure AD Premium P2 訂用帳戶的組織。

若要降低因密碼洩漏而遭到攻擊的風險，Azure AD 建議您要求所有個別使用者使用 Multi-Factor Authentication。 這個群組應包含管理員和所有其他人員 (例如財務人員)，其遭洩漏的帳戶會造成重大影響。 

不過，如果您的組織沒有共用的裝置，這些緊急存取帳戶可能無法使用 Multi-Factor Authentication。 如果您要針對 Azure AD 和其他連線的 SaaS 應用程式設定條件式存取原則，以要求[每位管理員使用 Multi-Factor Authentication 註冊](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-get-started-user-states)，則您需要設定原則排除，將緊急存取帳戶從此要求中排除。

### <a name="initial-configuration-with-approvals"></a>使用核准的初始設定

另一個方法是將使用者設定為可啟動「全域管理員」角色的合格者和核准者。 此方法會要求您的組織需有 Azure AD Premium P2 訂用帳戶。 這也需要有適合在多個個人和網路環境之間共用的 Multi-Factor Authentication 選項。 會有這些需求，是因為啟用「全域管理員」角色會要求使用者先執行 Multi-Factor Authentication。 如需詳細資訊，請參閱[如何在 Azure AD Privileged Identity Management 中要求 Multi-Factor Authentication](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-how-to-require-mfa)。

針對緊急存取帳戶，不建議使用與個人裝置相關聯的 Multi-Factor Authentication。 在實際的緊急情況下，需要存取已註冊 Multi-Factor Authentication 裝置的人員未必是擁有此個人裝置的人員。 

也請考量環境威脅。 例如，可能發生未預期的緊急情況，例如自然災害，在其間行動電話或其他網路可能無法使用。 請務必確保任何已註冊的裝置可處於已知且安全的位置，並有多個方法可與 Azure AD 通訊。

## <a name="ongoing-monitoring"></a>持續監控

監控 [Azure AD 登入和稽核記錄](https://docs.microsoft.com/azure/active-directory/active-directory-reporting-activity-sign-ins)，可了解緊急存取帳戶的任何登入和稽核活動。 通常這些帳戶應該不會登入，且應該不會進行變更，因此帳戶的使用很可能是出現異常，需要安全性檢查。

## <a name="account-check-validation-must-occur-at-regular-intervals"></a>帳戶檢查驗證必須定期實行

若要驗證帳戶，請最少於以下時間點執行下列步驟：
- 每 90 天。
- 當 IT 人員在最近發生變動時，例如工作變更、離職或雇用新人員。
- 當組織中的 Azure AD 訂用帳戶發生變更時。

若要訓練組織成員使用緊急存取帳戶，請執行下列動作：

* 確定安全性監控人員了解帳戶檢查活動須持續進行。
* 驗證雲端使用者帳戶是否可以登入並啟動其角色，以及在發生緊急狀況時需要執行這些步驟的使用者是否已接受流程訓練。
* 確定他們尚未將 Multi-Factor Authentication 或自助式密碼重設 (SSPR) 註冊至任何個別使用者的裝置或個人詳細資料。 
* 如果帳戶已在裝置上註冊以使用 Multi-Factor Authentication (在角色啟用時使用)，請確定在發生緊急狀況時可能需要使用裝置的所有管理員皆可存取裝置。 另外也請確認裝置已透過至少兩項失敗模式不同的機制來註冊。 例如，裝置可以透過設施的無線網路和電信業者提供的網路來與網際網路通訊。
* 更新帳戶認證。

## <a name="next-steps"></a>後續步驟
- [新增雲端式使用者](../fundamentals/add-users-azure-active-directory.md)並[將新使用者指派為全域管理員角色](../fundamentals/active-directory-users-assign-role-azure-portal.md)。
- 請[註冊 Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md) (如果您尚未註冊)。
- [要求指派為管理員的個別使用者使用 Azure Multi-Factor Authentication](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-get-started-user-states)。
- 如果您使用 Office 365，[在 Office 365 中為全域管理員設定額外保護](https://support.office.com/article/Protect-your-Office-365-global-administrator-accounts-6b4ded77-ac8d-42ed-8606-c014fd947560)。
- [執行全域管理員的存取權檢閱](../privileged-identity-management/pim-how-to-start-security-review.md)並[將現有的全域管理員轉換為更具體的管理員角色](directory-assign-admin-roles.md)。


