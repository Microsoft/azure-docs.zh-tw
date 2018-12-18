---
title: 使用安全性資訊重設您的密碼 - Azure Active Directory | Microsoft Docs
description: 如何在您忘記密碼時，使用安全性資訊和雙步驟驗證來重設自己的密碼。
services: active-directory
author: eross-msft
manager: mtillman
ms.reviewer: sahenry
ms.service: active-directory
ms.workload: identity
ms.component: user-help
ms.topic: conceptual
ms.date: 07/30/2018
ms.author: lizross
ms.openlocfilehash: a95d5adb4ee0eb8aa71f7759e3f2b5520bda5ff4
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39347701"
---
# <a name="reset-your-work-or-school-password"></a>重設您的工作密碼或學校密碼

如果您忘記密碼、公司支援人員不曾提供密碼給您，或您的帳戶遭鎖定，您可以使用安全性資訊和行動裝置來重設密碼。

>[!NOTE]
>如果您知道密碼，但只想變更它，您可以前往本文中的[變更您的密碼](#how-to-change-your-password)步驟。<br><br>
>如果您正嘗試重新進入個人帳戶 (例如 Xbox、hotmail.com 或 outlook.com)，請試試[當您無法登入 Microsoft 帳戶時](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)一文中所提供的建議。

## <a name="how-to-reset-or-unlock-your-password-for-a-work-or-school-account"></a>如何將公司或學校帳戶的密碼重設或解除鎖定

如果您無法存取 Azure Active Directory (Azure AD) 帳戶，可能是下列其中一個因素所導致：

- 密碼沒有作用，所以您想要予以重設，或者

- 您知道密碼，但帳戶已遭鎖定，因此需要將帳戶解除鎖定。

### <a name="to-reset-your-password-and-get-back-into-your-account"></a>重設密碼並重新進入您的帳戶

1. 在 [輸入密碼] 畫面上，選取 [忘記密碼]。

2. 在 [重新進入您的帳戶] 畫面中，輸入公司或學校的**使用者識別碼** (例如您的電子郵件地址)、輸入您在畫面上看到的字元以證明您不是機器人，然後選取 [下一步]。

   ![[重新進入您的帳戶] 畫面](media/security-info/security-info-back-into-acct.png)

   >[!NOTE]
   >如果您的系統管理員尚未為您開啟此功能來重設自己的密碼，您將會看到 [請連絡您的系統管理員] 連結，而不是 [重新進入您的帳戶] 畫面。 此連結可讓您透過電子郵件或 Web 入口網站，連絡系統管理員來重設您的密碼。

3. 選擇下列其中一種方法，來驗證您的身分識別並變更密碼。 根據系統管理員設定組織的方式而定，您可能需要再次進行此程序，新增第二個驗證步驟的資訊。

    ![重新進入您的帳戶，驗證步驟 #1](media/security-info/security-info-back-into-acct2.png)

    >[!NOTE]
    >根據系統管理員設定組織的方式而定，這其中有一些驗證選項可能無法使用。 您先前必須已使用這其中至少一種方法，來設定行動裝置以進行驗證。<br><br>此外，您的新密碼可能需要符合特定強度需求。 強式密碼通常具有 8 到 16 個字元，包括大寫和小寫字元、至少一個數字，以及至少一個特殊字元。

- **使用電子郵件地址重設密碼。** 將電子郵件傳送到您先前在雙步驟驗證或安全性資訊中所設定的電子郵件地址。 如果您的系統管理員已開啟安全性資訊體驗，您就能在[將安全性資訊設定為使用電子郵件 (預覽)](security-info-setup-email.md) 一文中，找到設定電子郵件地址的詳細資訊。 如果您尚未使用安全性資訊，您可以在[對我的帳戶進行雙步驟驗證設定](multi-factor-authentication-end-user-first-time.md)一文中找到設定電子郵件地址的詳細資訊。 

    1. 選取 [寄電子郵件到我的備用電子郵件地址]，然後選取 [電子郵件]。

    2. 在方塊中輸入電子郵件內的驗證碼，然後選取 [下一步]。
    
    3. 輸入並確認您的新密碼，然後選取 [完成]。

- **使用簡訊重設密碼。** 將簡訊傳送到您先前在安全性資訊中設定的電話號碼。 如果您的系統管理員已開啟安全性資訊體驗，您就能在[將安全性資訊設定為使用簡訊 (預覽)](security-info-setup-text-msg.md) 一文中，找到設定簡訊的詳細資訊。 如果您尚未使用安全性資訊，您可以在[對我的帳戶進行雙步驟驗證設定](multi-factor-authentication-end-user-first-time.md)一文中找到設定簡訊的詳細資訊。

    1. 選取 [傳簡訊到我的行動電話]、輸入您的電話號碼，然後選取 [收發簡訊]。

    2. 在方塊中輸入簡訊內的驗證碼，然後選取 [下一步]。

    3. 輸入並確認您的新密碼，然後選取 [完成]。

- **使用電話號碼重設密碼。** 將簡訊傳送到您先前在安全性資訊中設定的電話號碼。 如果您的系統管理員已開啟安全性資訊體驗，您就能在[將安全性資訊設定為使用電話 (預覽)](security-info-setup-phone-number.md) 一文中，找到設定電話號碼的詳細資訊。 如果您尚未使用安全性資訊，您可以在[對我的帳戶進行雙步驟驗證設定](multi-factor-authentication-end-user-first-time.md)一文中找到設定電話號碼的詳細資訊。

    1. 選取 [撥打我的行動電話]、輸入您的電話號碼，然後選取 [撥打電話]。

    2. 接聽來電，並遵循指示來驗證您的身分識別，然後選取 [下一步]。

    3. 輸入並確認您的新密碼，然後選取 [完成]。

- **使用安全性問題重設密碼。** 顯示您在安全性資訊中設定的安全性問題清單。 如果您的系統管理員已開啟安全性資訊體驗，您就能在[將安全性資訊設定為使用預先定義的安全性問題 (預覽)](security-info-setup-questions.md) 一文中，找到設定安全性問題的詳細資訊。 如果您尚未使用安全性資訊，您可以在[對我的帳戶進行雙步驟驗證設定](multi-factor-authentication-end-user-first-time.md)一文中找到設定安全性問題的詳細資訊。

    1. 選取 [回答我的安全性問題]、回答問題，然後選取 [下一步]。

    2. 輸入並確認您的新密碼，然後選取 [完成]。

- **從驗證器應用程式使用通知來重設密碼。** 將核准通知傳送到驗證器應用程式。 如果您的系統管理員已開啟安全性資訊體驗，您就能在[將安全性資訊設定為使用驗證器應用程式 (預覽)](security-info-setup-auth-app.md) 一文中，找到設定驗證器應用程式以傳送通知的詳細資訊。 如果您尚未使用安全性資訊，您可以在[對我的帳戶進行雙步驟驗證設定](multi-factor-authentication-end-user-first-time.md)一文中找到設定驗證器應用程式以傳送通知的詳細資訊。

    1. 選取 [在驗證器應用程式上核准通知]，然後選取 [傳送通知]。

    2. 從您的驗證器應用程式核准登入。

    3. 輸入並確認您的新密碼，然後選取 [完成]。

- **從驗證器應用程式使用驗證碼來重設密碼。** 接受驗證應用程式所提供的隨機驗證碼。 如果您的系統管理員已開啟安全性資訊體驗，您就能在[將安全性資訊設定為使用驗證器應用程式 (預覽)](security-info-setup-auth-app.md) 一文中，找到設定驗證器應用程式以提供驗證碼的詳細資訊。 如果您尚未使用安全性資訊，您可以在[對我的帳戶進行雙步驟驗證設定](multi-factor-authentication-end-user-first-time.md)一文中找到設定驗證器應用程式以提供驗證碼的詳細資訊。

    1. 選取 [從驗證器應用程式輸入驗證碼]，然後選取 [傳送通知]。

    2. 開啟驗證器應用程式、在方塊中輸入帳戶的驗證碼，然後選取 [下一步]。

    3. 輸入並確認您的新密碼，然後選取 [完成]。

    4. 在您收到訊息指出密碼已重設之後，就可以使用新密碼來登入帳戶。
        
    如果您仍無法存取帳戶，您應該連絡貴組織的系統管理員以獲得更多協助。

重設密碼之後，您可能會收到確認電子郵件，其來源是「Microsoft 代表 \<*your_organization*>」之類的帳戶。 如果您收到類似的電子郵件，但您最近並未重設密碼，則您必須立即連絡貴組織的系統管理員。

## <a name="how-to-change-your-password"></a>如何變更密碼

如果您只想變更密碼，您可以透過 Office 365 入口網站、Azure 存取面板或 Windows 10 登入頁面來進行此動作。

### <a name="to-change-your-password-using-the-office-365-portal"></a>使用 Office 365 入口網站變更您的密碼

如果您通常會透過 Office 入口網站來存取應用程式，請使用此方法：

1. 使用現有的密碼登入 [Office 365 帳戶](https://www.office.com)。

2. 選取您位於右上方的設定檔，然後選取 [檢視帳戶]。

3. 選取 **[安全性與隱私權]** > **[密碼]**。

4. 輸入舊密碼、建立並確認新密碼，然後選取 [提交]。

### <a name="to-change-your-password-from-the-azure-access-panel"></a>從 Azure 存取面板變更您的密碼

如果您通常會從 Azure 存取面板 (MyApps) 存取應用程式，請使用此方法：

1. 使用您的現有密碼，登入 [Azure 存取面板](https://myapps.microsoft.com/)。

2. 選取您位於右上方的設定檔，然後選取 [設定檔]。

3. 選取 [變更密碼] 。

4. 輸入舊密碼、建立並確認新密碼，然後選取 [提交]。

### <a name="to-change-your-password-at-windows-sign-in"></a>在 Windows 登入畫面上變更您的密碼

如果您的系統管理員已開啟此功能，您就可以在 Windows 7、Windows 8、Windows 8.1 或 Windows 10 登入畫面上看見 [重設密碼] 連結。

1. 選取 [重設密碼] 連結以啟動密碼重設程序，而不需使用標準的 Web 體驗。

2. 確認您的使用者識別碼，然後選取 [下一步]。

3. 選取並確認驗證的連絡方法。 如有必要，請選擇與您前一個驗證選項不同的第二個驗證選項，並填入必要資訊。

4. 在 [建立新密碼] 頁面上輸入並確認您的新密碼，然後選取 [下一步]。

    強式密碼通常具有 8 到 16 個字元，包括大寫和小寫字元、至少一個數字，以及至少一個特殊字元。

5. 在您收到訊息指出密碼已重設之後，就可以選取 [完成]。

    如果您仍無法存取帳戶，您應該連絡貴組織的系統管理員以獲得更多協助。

## <a name="common-problems-and-their-solutions"></a>常見問題及其解決方案

以下提供一些常見的錯誤案例及其解決方案：

|問題|說明|解決方法|
| --- | --- | --- |
|當我嘗試變更密碼時，收到一個錯誤。 |您的密碼包含使密碼容易猜到的字組、片語或模式。| 請使用強度更高的密碼重試。|
|在輸入我的使用者識別碼之後，會移至一個畫面指出「請連絡您的系統管理員」。|Microsoft 已判定您的使用者帳戶密碼會由您內部部署環境中的系統管理員所管理。 因此，您無法從 [無法存取您的帳戶] 連結重設密碼。 |請連絡您的系統管理員以獲得更多協助。|
|在輸入我的使用者識別碼之後，出現了「您的帳戶未啟用密碼重設」錯誤。|您的系統管理員尚未設定您的帳戶，因此您可以重設自己的密碼。|您的系統管理員尚未針對組織開啟從 [無法存取您的帳戶] 連結重設密碼的功能，或尚未授權您使用該功能。<br><br> 若要重設密碼，您必須選取 [連絡系統管理員] 連結，以傳送電子郵件給公司的系統管理員，讓他們知道您想要重設密碼。|
|在輸入我的使用者識別碼之後，出現了「我們無法驗證您的帳戶」錯誤。|登入程序無法驗證您的帳戶資訊。|有兩個原因會讓您看到此訊息。<br><br>1.您的系統管理員已為組織開啟密碼重設，但您尚未註冊使用此服務。 若要註冊密碼重設，請根據您的驗證方法參閱下列其中一篇文章：[將安全性資訊設定為使用驗證器應用程式 (預覽)](security-info-setup-auth-app.md)、[將安全性資訊設定為使用電話 (預覽)](security-info-setup-phone-number.md)、[將安全性資訊設定為使用簡訊 (預覽)](security-info-setup-text-msg.md)、[將安全性資訊設定為使用電子郵件 (預覽)](security-info-setup-email.md) 或[將安全性資訊設定為使用安全性問題 (預覽)](security-info-setup-questions.md)。<br><br>2.您的系統管理員尚未為組織開啟密碼重設。 在此情況下，您必須選取 [連絡系統管理員] 連結，以傳送電子郵件給公司的系統管理員，要求重設您的密碼。|

## <a name="next-steps"></a>後續步驟

- 在[管理您的安全性資訊 (預覽)](security-info-manage-settings.md) 一文中，編輯您的安全性資訊以新增或變更您的驗證方法。 

- 了解[安全性資訊 (預覽) 概觀](user-help-security-info-overview.md)一文中的安全性資訊。

- 了解[雙步驟驗證概觀](user-help-two-step-verification-overview.md)一文中的雙步驟驗證。 

- 如果您遺失或忘記密碼，請從[密碼重設入口網站](https://passwordreset.microsoftonline.com/)重設密碼。

- 在[無法登入 Microsoft 帳戶](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)一文中取得登入問題的疑難排解秘訣和說明。