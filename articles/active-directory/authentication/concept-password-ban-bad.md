---
title: Azure AD 中的動態禁用密碼
description: 使用 Azure AD 動態禁用密碼而在環境中禁用弱式密碼
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: rogoya
ms.openlocfilehash: 1ad499e2703ff8376c063d933c0cc1f03765fc23
ms.sourcegitcommit: 17fe5fe119bdd82e011f8235283e599931fa671a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/11/2018
ms.locfileid: "42142078"
---
# <a name="eliminate-bad-passwords-in-your-organization"></a>避免在組織中使用不當密碼

|     |
| --- |
| Azure AD 密碼保護和自訂的禁用密碼清單是 Azure Active Directory 的公開預覽版功能。 如需預覽版的詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)|
|     |

業界領導者指示您不應在多處使用相同的密碼，且應使用複雜的密碼，而不要使用 Password123 這種簡單的密碼。 組織要如何確保使用者會遵循指引？ 他們要如何確定使用者不會使用常見的密碼或已知包含在近期資料外洩中的密碼？

## <a name="global-banned-password-list"></a>全域禁用密碼清單

Microsoft 一直努力在網路罪犯發生前加以防範。 因此，Azure AD Identity Protection 小組會持續尋找常用和遭入侵的密碼。 然後，他們會在所謂的全域禁用密碼清單中封鎖這些會被視為太常見的密碼。 網路罪犯也會在其攻擊中使用類似的策略，因此 Microsoft 不會公開發佈此清單的內容。 這些易受攻擊的密碼會在對 Microsoft 的客戶形成實際的威脅之前就遭到封鎖。 如需現行安全性成果的詳細資訊，請參閱 [Microsoft 安全性情資報告](https://www.microsoft.com/security/intelligence-report)。

## <a name="preview-custom-banned-password-list"></a>預覽：自訂禁用密碼清單

有些組織可能會想要在全域禁用密碼清單之上，在 Microsoft 所謂的自訂禁用密碼清單中加上其本身的自訂，以進一步提高安全性。 企業客戶 (如 Contoso) 可再選擇封鎖其品牌名稱、公司特定詞彙或其他項目的變異形式。

自訂禁用密碼清單和啟用內部部署 Active Directory 整合的能力，都可使用 Azure 入口網站來管理。

![在 Azure 入口網站中的 [驗證方法] 下修改自訂禁用密碼清單](./media/concept-password-ban-bad/authentication-methods-password-protection.png)

## <a name="on-premises-hybrid-scenarios"></a>內部部署混合式案例

保護僅限雲端帳戶有其效用，但許多組織仍會維護包含內部部署 Windows Server Active Directory 的混合式案例。 您可以內部部署安裝 Windows Server Active Directory (預覽) 代理程式的 Azure AD 密碼保護，以將禁用密碼清單擴充至現有的基礎結構。 現在，在內部部署變更、設定或重設密碼的使用者和系統管理員，都必須遵循與僅限雲端使用者相同的密碼原則。

## <a name="how-does-the-banned-password-list-work"></a>禁用密碼清單的運作方式

禁用密碼清單會將字串轉換為小寫，並使用編輯距離在 1 以內的模糊比對將其與已知的禁用密碼比較，以比對清單中的密碼。

範例：組織使用 password 這個字時會遭到封鎖
   - 使用者若嘗試將其密碼設定為 "P@ssword"，由於這個字會轉換為 "password"，並且是 password 的變異形式，因此會遭到封鎖。
   - 系統管理員若嘗試將使用者密碼設定為 "Password123!"， 由於這個字會轉換為 "password123!"， 並且是 password 的變異形式，因此會遭到封鎖。

每當使用者重設或變更其 Azure AD 密碼時，密碼都須經過此程序的處理，以確認它不在禁用密碼清單上。 此檢查會使用自助式密碼重設、密碼雜湊同步和傳遞驗證包含在混合式案例中。

## <a name="license-requirements"></a>授權需求

|   | 使用全域禁用密碼清單的 Azure AD 密碼保護 | 使用自訂禁用密碼清單的 Azure AD 密碼保護|
| --- | --- | --- |
| 僅限雲端使用者 | Azure AD Free | Azure AD Basic |
| 從內部部署 Windows Server Active Directory 同步處理的使用者 | Azure AD Premium P1 或 P2 | Azure AD Premium P1 或 P2 |

在 [Azure Active Directory 價格網站](https://azure.microsoft.com/pricing/details/active-directory/)上可以找到其他授權資訊 (包括成本)。

## <a name="what-do-users-see"></a>使用者看到的內容

當使用者嘗試將密碼重設為會禁用的密碼時，便會看到下列錯誤訊息：

不幸的是，您的密碼包含單字、片語或模式，可輕易猜到您的密碼。 請使用不同的密碼再試一次。

## <a name="next-steps"></a>後續步驟

* [設定自訂禁用密碼清單](howto-password-ban-bad.md)
* [啟用內部部署的 Azure AD 密碼保護代理程式](howto-password-ban-bad-on-premises.md)
