---
title: 在雲端中開始使用 Azure MFA
description: 透過條件式存取開始使用 Microsoft Azure Multi-Factor Authentication
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 09/01/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 0408b26e687dd31c408dbccc68f56e8198016c8f
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43664783"
---
# <a name="deploy-cloud-based-azure-multi-factor-authentication"></a>部署雲端式 Azure Multi-Factor Authentication

開始使用 Azure Multi-Factor Authentication (Azure MFA) 是一個簡單的程序。

在開始之前，請先確定您已滿足下列先決條件：

* Azure AD 租用戶中的全域系統管理員帳戶。 如果您需要完成這個步驟的說明，請參閱[開始使用 Azure AD](../get-started-azure-ad.md) 一文。
* 指派給使用者的正確授權。 如需詳細資訊，請參閱[如何取得 Azure Multi-Factor Authentication](concept-mfa-licensing.md) 主題。

## <a name="choose-how-to-enable"></a>選擇啟用方式

**由條件式存取原則啟用** - 本文會討論這個方法。 這是最具彈性的方法，可為您的使用者啟用雙步驟驗證。 啟用條件式存取原則，只適用於雲端 Azure MFA，而且是 Azure AD 的進階功能。

由 Azure AD Identity Protection 啟用 - 這個方法使用 Azure AD Identity Protection 風險原則，要求所有雲端應用程式進行只根據登入風險的雙步驟驗證。 這個方法需要 Azure Active Directory P2 授權。 如需這個方法的詳細資訊，請參閱 [Azure Active Directory Identity Protection](../identity-protection/overview.md#risky-sign-ins)。

藉由變更使用者狀態來啟用 - 這是要求使用雙步驟驗證的傳統方法。 這種方法適用於雲端 Azure MFA 和 Azure MFA Server。 如果使用這種方法，則會要求使用者在**每次**登入時執行雙步驟驗證，並且會覆寫條件式存取原則。 這個方法的詳細資訊可在[如何要求使用者使用雙步驟驗證](howto-mfa-userstates.md)中找到。

> [!Note]
> 如需授權和定價的詳細資訊，請參閱 [Azure AD](https://azure.microsoft.com/pricing/details/active-directory/
) 和 [Multi-Factor Authentication](https://azure.microsoft.com/pricing/details/multi-factor-authentication/) 定價頁面。

## <a name="choose-authentication-methods"></a>選擇驗證方法

請根據組織的需求，為使用者啟用至少一個驗證方法。 我們發現，當您為使用者啟用時，Microsoft Authenticator 應用程式可提供最佳使用者體驗。 如果您需要了解有哪些方法可供使用以及如何設定這些方式，請參閱[驗證方法有哪些](concept-authentication-methods.md)一文。

## <a name="get-users-to-enroll"></a>讓使用者註冊

啟用條件式存取原則後，系統會強制讓使用者在下一次使用受原則保護的應用程式時進行註冊。 如果您啟用的原則會要求所有雲端應用程式上的所有使用者使用 MFA，這個動作可能會讓使用者和技術服務人員感到困擾。 建議您要求使用者事先使用註冊入口網站 (網址：[https://aka.ms/mfasetup](https://aka.ms/mfasetup)) 註冊驗證方法。 許多組織發現，製作海報、桌卡和電子郵件訊息會有助於推行此政策。

## <a name="enable-multi-factor-authentication-with-conditional-access"></a>透過條件式存取啟用 Multi-Factor Authentication

使用全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com)。

### <a name="choose-verification-options"></a>選擇驗證選項

在啟用 Azure Multi-factor Authentication 之前，組織必須先決定要允許哪些驗證選項。 為了進行這項練習，您會啟用電話通話和電話簡訊，原因是大部分使用者都能夠使用這兩種泛用選項。 關於驗證方法及其使用方式的詳細資訊，可在[驗證方法有哪些？](concept-authentication-methods.md)一文中找到

1. 瀏覽至 [Azure Active Directory]、[使用者]、[Multi-Factor Authentication]
   ![從 Azure 入口網站中的 [Azure AD 使用者] 刀鋒視窗存取 Multi-Factor Authentication 入口網站](media/howto-mfa-getstarted/users-mfa.png) 
2. 在開啟的新索引標籤中，瀏覽至 [服務設定]
3. 在 [驗證選項] 底下，核取下列使用者可用方法的方塊
   * 電話通話
   * 電話簡訊

   ![在 [Multi-Factor Authentication 服務設定] 索引標籤中設定驗證方法](media/howto-mfa-getstarted/mfa-servicesettings-verificationoptions.png)

4. 按一下 [儲存] 
5. 關閉 [服務設定] 索引標籤

### <a name="create-conditional-access-policy"></a>建立條件式存取原則

1. 使用全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com)。
1. 瀏覽至 [Azure Active Directory]、[條件式存取]
1. 選取 [新增原則]
1. 為原則提供有意義的名稱
1. 在 [使用者和群組] 底下
   * 在 [包含] 索引標籤上，選取 [所有使用者] 選項按鈕
   * 建議︰在 [排除] 索引標籤上，核取 [使用者和群組] 的方塊，然後選擇當使用者沒有其驗證方法的存取權時，要用於排除項目的群組。
   * 按一下 [完成]
1. 在 [雲端應用程式] 下方，選取 [所有雲端應用程式] 選項按鈕
   * 選擇性：在 [排除] 索引標籤上，選擇組織不要要求使用 MFA 的雲端應用程式。
   * 按一下 [完成]
1. 在 [條件] 區段底下
   * 選擇性：如果您已啟用 Azure Identity Protection，您可以選擇要在原則中評估登入風險。
   * 選擇性：如果您已設定信任的位置或具名位置，您可以指定要在原則中包含或排除這些位置。
1. 在 [授與] 下方，確定已選取 [授與存取權] 選項按鈕
    * 核取 [需要多重要素驗證] 的方塊
    * 按一下 [選取]
1. 略過 [工作階段] 區段
1. 將 [啟用原則] 切換為 [開啟]
1. 按一下 [建立] 

![建立條件式存取原則，以對試驗群組中的 Azure 入口網站使用者啟用 MFA](media/howto-mfa-getstarted/conditionalaccess-newpolicy.png)

### <a name="test-azure-multi-factor-authentication"></a>測試 Azure Multi-Factor Authentication

若要確認條件式存取原則有作用，請對不應該要求使用 MFA 的資源測試登入動作，然後對會要求使用 MFA 的 Azure 入口網站測試登入動作。

1. 在 InPrivate 或 Incognito 模式中開啟新的瀏覽器視窗，並瀏覽至 [https://account.activedirectory.windowsazure.com](https://account.activedirectory.windowsazure.com)。
   * 使用在本文的先決條件一節中建立的測試使用者進行登入，並留意系統應該不會要求您完成 MFA。
   * 關閉瀏覽器視窗
2. 在 InPrivate 或 Incognito 模式中開啟新的瀏覽器視窗，並瀏覽至 [https://portal.azure.com](https://portal.azure.com)。
   * 使用在本文的先決條件一節中建立的測試使用者進行登入，並留意您現在應該必須註冊並使用 Azure Multi-Factor Authentication。
   * 關閉瀏覽器視窗

## <a name="next-steps"></a>後續步驟

恭喜，您已在雲端中設定 Azure Multi-Factor Authentication。

若要設定信任的 IP、自訂語音訊息及詐騙警示等額外設定，請參閱[設定 Azure Multi-Factor Authentication 設定](howto-mfa-mfasettings.md)

如需管理 Azure Multi-Factor Authentication 使用者設定的詳細資訊，請參閱[在雲端中管理 Azure Multi-Factor Authentication 的使用者設定](howto-mfa-userdevicesettings.md)一文

[啟用 Azure Multi-Factor Authentication 的聚合式註冊和 Azure AD 自助密碼重設](concept-registration-mfa-sspr-converged.md)
