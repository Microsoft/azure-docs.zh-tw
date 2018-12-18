---
title: Azure AD 自助式密碼重設原則
description: 設定 Azure AD 自助式密碼重設原則選項
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: ee30ee4fa89ce47e8441845b088919b26ce32b31
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434261"
---
# <a name="password-policies-and-restrictions-in-azure-active-directory"></a>密碼原則和 Azure Active Directory 中的限制

本文說明與儲存在 Azure Active Directory (Azure AD) 租用戶中的使用者帳戶相關聯的密碼原則和複雜性需求。

## <a name="administrator-reset-policy-differences"></a>系統管理員重設原則差異

**Microsoft 會對任何 Azure 系統管理員角色強制執行預設的強式*雙閘道*密碼重設原則。** 此原則可能會不同於您已為使用者定義的原則，且無法加以變更。 請一律要以未指派任何 Azure 系統管理員角色的使用者身分測試密碼重設功能。

利用雙閘道原則，**系統管理員就不需要有使用安全性問題的能力**。

 兩個閘道原則需要兩項驗證資料，例如電子郵件地址*和*電話號碼。 兩個閘道原則適用於下列情況：

* 下列 Azure 系統管理員角色會受到影響：
  * 服務台管理員
  * 服務支援管理員
  * 計費管理員
  * 合作夥伴第 1 層支援
  * 合作夥伴第 2 層支援
  * Exchange 服務管理員
  * Lync 服務管理員
  * 使用者帳戶管理員
  * 目錄寫入器
  * 全域管理員或公司系統管理員
  * SharePoint 服務管理員
  * 規範管理員
  * 應用程式管理員
  * 安全性系統管理員
  * 特殊權限角色管理員
  * Microsoft Intune 服務管理員
  * 應用程式 Proxy 服務管理員
  * CRM 服務管理員
  * Power BI 服務管理員

* 如果試用版訂用帳戶已經過 30 天

  或

* 已存在虛名網域，例如 contoso.com

  或

* Azure AD Connect 正在同步處理內部部署目錄中的身分識別

### <a name="exceptions"></a>例外狀況

一個閘道原則需要一項驗證資料，例如電子郵件地址*或*電話號碼。 一個閘道原則適用於下列情況：

* 處於試用訂用帳戶的第一個 30 天內

  或

* 不存在虛名網域 (*.onmicrosoft.com)

  和

  Azure AD Connect 未同步身分識別

## <a name="userprincipalname-policies-that-apply-to-all-user-accounts"></a>適用於所有使用者帳戶的 UserPrincipalName 原則

每個需要登入 Azure AD 的使用者帳戶都必須具有與其帳戶相關聯的唯一使用者主體名稱 (UPN) 屬性值。 下表列出適用於內部部署的 Active Directory 使用者帳戶 (已同步至雲端) 和僅適用於雲端使用者帳戶的原則：

| 屬性 | UserPrincipalName 需求 |
| --- | --- |
| 允許的字元 |<ul> <li>A – Z</li> <li>a - z</li><li>0 – 9</li> <li> \. - \_ ! \# ^ \~</li></ul> |
| 不允許的字元 |<ul> <li>任何不是用來分隔使用者名稱和網域的 "\@\" 字元。</li> <li>"\@\" 符號前面不可包含緊鄰的句點字元 "."</li></ul> |
| 長度限制 |<ul> <li>總長度不得超過 113 個字元</li><li>"\@\" 符號前面最多可以有 64 個字元</li><li>"\@\" 符號後面最多可以有 48 個字元</li></ul> |

## <a name="password-policies-that-only-apply-to-cloud-user-accounts"></a>僅適用於雲端使用者帳戶的密碼原則

下表描述可套用至在 Azure AD 中建立及管理的使用者帳戶的可用密碼原則設定：

| 屬性 | 需求 |
| --- | --- |
| 允許的字元 |<ul><li>A – Z</li><li>a - z</li><li>0 – 9</li> <li>@ # $ % ^ & * - _ ! + = [ ] { } &#124; \ : ‘ , . ? / ` ~ " ( ) ;</li></ul> |
| 不允許的字元 |<ul><li>Unicode 字元。</li><li>空格。</li><li> 僅限強式密碼</li></ul> |
| 密碼限制 |<ul><li>最少 8 個字元和最多 16 個字元。</li><li>僅限使用增強式密碼︰需要下列四種字元中的三種：<ul><li>小寫字元。</li><li>大寫字元。</li><li>數字 (0-9)。</li><li>符號 (請參閱先前的密碼限制)。</li></ul></li></ul> |
| 密碼到期時間 |<ul><li>預設值：**90** 天。</li><li>可透過適用於 Windows PowerShell 的 Azure Active Directory 模組使用 `Set-MsolPasswordPolicy` Cmdlet 設定該值。</li></ul> |
| 密碼到期通知 |<ul><li>預設值：**14** 天 (密碼到期之前)。</li><li>可使用 `Set-MsolPasswordPolicy` Cmdlet 設定此值。</li></ul> |
| 密碼到期 |<ul><li>預設值︰**false** 天 (表示已啟用密碼到期)。</li><li>可以使用 `Set-MsolUser` Cmdlet 針對個別使用者帳戶設定該值。</li></ul> |
| 密碼變更歷程記錄 |當使用者變更密碼時，*不能*重複使用上次的密碼。 |
| 密碼重設歷程記錄 | 當使用者重設忘記密碼時，*不能*重複使用上次的密碼。 |
| 帳戶鎖定 |使用錯誤的密碼於 10 次失敗的登入嘗試之後，使用者會被封鎖一分鐘。 後續錯誤的登入嘗試會增加使用者被封鎖的時間。 |

## <a name="set-password-expiration-policies-in-azure-ad"></a>在 Azure AD 中設定密碼到期原則

Microsoft 雲端服務的全域管理員可以使用「適用於 Windows PowerShell 的 Microsoft Azure AD 模組」，將使用者密碼設為不會到期。 您亦可使用 Windows PowerShell Cmdlet 移除永不到期組態，或是查看哪些使用者密碼設為永不到期。 

本指引適用於其他提供者 (例如 Intune 和 Office 365)，它們也仰賴 Azure AD 提供身分識別和目錄服務。 密碼到期是原則中唯一可變更的部分。

> [!NOTE]
> 您僅可將未透過目錄同步作業執行同步處理的使用者帳戶密碼，設定為不會到期。 如需目錄同步作業的詳細資訊，請參閱 [Connect AD 與 Azure AD](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect)。
>

## <a name="set-or-check-the-password-policies-by-using-powershell"></a>使用 PowerShell 設定或檢查密碼原則

首先，您必須[下載並安裝 Azure AD PowerShell 模組](https://docs.microsoft.com/powershell/module/Azuread/?view=azureadps-2.0)。 安裝之後，您可以使用下列步驟來設定每個欄位。

### <a name="check-the-expiration-policy-for-a-password"></a>檢查密碼的到期原則

1. 使用您公司的管理員認證連線至 Windows PowerShell。
2. 執行下列其中一個命令：

   * 若要查看單一使用者的密碼是否設為永久有效，請透過使用 UPN (例如 *aprilr@contoso.onmicrosoft.com*)，或是您想要檢查之使用者的使用者識別碼，來執行下列 Cmdlet：`Get-MSOLUser -UserPrincipalName <user ID> | Select PasswordNeverExpires`
   * 若要查看所有使用者的**密碼永久有效**設定，請執行下列 Cmdlet：`Get-MSOLUser | Select UserPrincipalName, PasswordNeverExpires`

### <a name="set-a-password-to-expire"></a>設定密碼到期

1. 使用您公司的管理員認證連線至 Windows PowerShell。
2. 執行下列其中一個命令：

   * 若要將某位使用者的密碼設為會到期，請透過使用 UPN 或使用者的使用者識別碼，執行下列 Cmdlet：`Set-MsolUser -UserPrincipalName <user ID> -PasswordNeverExpires $false`
   * 若要將組織中所有使用者的密碼設為會到期，請使用下列 Cmdlet：`Get-MSOLUser | Set-MsolUser -PasswordNeverExpires $false`

### <a name="set-a-password-to-never-expire"></a>設定密碼為永久有效

1. 使用您公司的管理員認證連線至 Windows PowerShell。
2. 執行下列其中一個命令：

   * 若要將某位使用者的密碼設為永不過期，請透過使用使用者主體名稱 (UPN) 或使用者的使用者識別碼，來執行下列 Cmdlet：`Set-MsolUser -UserPrincipalName <user ID> -PasswordNeverExpires $true`
   * 若要將組織中所有使用者的密碼設為永久有效，請執行下列 Cmdlet： `Get-MSOLUser | Set-MsolUser -PasswordNeverExpires $true`

   > [!WARNING]
   > 設定為 `-PasswordNeverExpires $true` 的密碼仍然會根據 `pwdLastSet` 屬性計算時效。 如果將使用者密碼設定為永不過期，那麼經過 90 天後，密碼會過期。 根據 `pwdLastSet` 屬性，如果您將到期變更為 `-PasswordNeverExpires $false`，`pwdLastSet` 早於 90 天的所有密碼會要求使用者在下次登入時變更密碼。 這項變更可能會影響大量使用者。 

## <a name="next-steps"></a>後續步驟

下列文章提供有關透過 Azure AD 重設密碼的其他資訊：

* [如何完成 SSPR 成功首度發行？](howto-sspr-deployment.md)
* [重設或變更您的密碼](../user-help/active-directory-passwords-update-your-own-password.md)。
* [註冊自助式密碼重設](../user-help/active-directory-passwords-reset-register.md)。
* [您有授權問題嗎？](concept-sspr-licensing.md)
* [SSPR 使用哪些資料，以及您應該為使用者填入哪些資料？](howto-sspr-authenticationdata.md)
* [哪些驗證方法可供使用者使用？](concept-sspr-howitworks.md#authentication-methods)
* [什麼是密碼回寫，且為什麼我需要了解它？](howto-sspr-writeback.md)
* [如何回報 SSPR 中的活動？](howto-sspr-reporting.md)
* [SSPR 中的所有選項有哪些，以及它們有何意義？](concept-sspr-howitworks.md)
* [我認為有中斷。如何針對 SSPR 進行疑難排解？](active-directory-passwords-troubleshoot.md)
* [在其他某處並未涵蓋我的問題](active-directory-passwords-faq.md)
