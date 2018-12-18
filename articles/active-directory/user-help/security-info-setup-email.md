---
title: 將安全性資訊設定為使用電子郵件 - Azure Active Directory | Microsoft Docs
description: 設定您的安全性資訊，以使用公司或學校電子郵件地址來驗證您的身分識別。
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
ms.openlocfilehash: 3f98b4e34c4c76efc33395530ef016b26e1ad523
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44163034"
---
# <a name="set-up-security-info-to-use-email-preview"></a>將安全性資訊設定為使用電子郵件 (預覽)

[!INCLUDE [preview-notice](../../../includes/active-directory-end-user-preview-notice-security-info.md)]

設定您的安全性資訊，需要您登入公司或學校帳戶，然後完成註冊程序。 如果您從未設定安全性資訊，系統將要求您立即進行設定。

## <a name="set-up-email"></a>設定電子郵件

根據您組織的設定，系統可能會在您登入時提示您將電子郵件地址新增到安全性資訊。 否則，若要開始在安全性資訊中設定電子郵件，請遵循[管理安全性資訊](security-info-manage-settings.md)中的步驟。

>[!Note]
>建議使用不需要您的網路密碼即可存取的電子郵件帳戶。<br>如果您沒有看到電子郵件選項，很可能是您的組織不允許您使用電子郵件進行驗證。 如果發生這種情況，您必須選擇其他方法，或連絡您的系統管理員以取得更多協助。

### <a name="to-use-your-email-address"></a>使用您的電子郵件地址

1. 選取 [電子郵件] 選項，然後在方塊中輸入您的電子郵件地址。 這個電子郵件地址不能是您的公司或學校電子郵件。

     ![安全性資訊頁面，內含電子郵件輸入方塊](media/security-info/security-info-keep-secure-setup-email.png)

2. 檢查 Microsoft 傳送給您組織的電子郵件，將內含的驗證碼輸入到 [驗證您的電子郵件] 方塊中，然後選取 [完成]。

     ![安全性資訊頁面，內含電子郵件驗證碼輸入方塊](media/security-info/security-info-verify-email.png)

    >[!Note]
    >如果您沒有看到 Microsoft 代表您的組織傳送的電子郵件，請確定您輸入的電子郵件地址正確無誤，然後檢查您的垃圾郵件資料夾。

3. 在 [保護您的帳戶安全] 頁面中，選取 [完成]。

    您的安全性資訊會更新為使用電子郵件地址，以在使用雙步驟驗證時驗證您的身分識別。

## <a name="additional-security-info-options"></a>其他安全性資訊選項

根據您要嘗試執行的動作，您會有組織如何連絡您來驗證您身分識別的選項。 選項包括：

- **驗證器應用程式。** 下載並使用驗證器應用程式，以取得核准通知或隨機產生的核准代碼來進行雙步驟驗證或密碼重設。 如需如何設定和使用 Microsoft Authenticator 應用程式的逐步指示，請參閱[將安全性資訊設定為使用驗證器應用程式](security-info-setup-auth-app.md)。

- **行動裝置簡訊。** 輸入您的行動裝置號碼，然後取得將用來進行雙步驟驗證或密碼重設的代碼簡訊。 如需如何透過簡訊 (SMS) 驗證您身分識別的逐步指示，請參閱[將安全性資訊設定為使用簡訊 (SMS)](security-info-setup-text-msg.md)。

- **行動裝置或公司電話。** 輸入您的行動裝置號碼，然後等候來電以進行雙步驟驗證或密碼重設。 如需如何透過電話號碼驗證您身分識別的逐步指示，請參閱[將安全性資訊設定為使用電話](security-info-setup-phone-number.md)。

- **安全性問題。** 回答由您的系統管理員為組織所建立的一些安全性問題。 此選項僅適用於密碼重設，不適用雙步驟驗證。 如需如何設定安全性問題的逐步指示，請參閱[將安全性資訊設定為使用安全性問題](security-info-setup-questions.md)一文。
    
    >[!Note]
    >如果這其中一些選項遺失，很可能是因為您的組織不允許那些方法。 如果發生這種情況，您將必須選擇可用的方法，或連絡您的系統管理員以取得更多協助。

## <a name="next-steps"></a>後續步驟

- 如果您需要更新安全性資訊，請遵循[管理安全性資訊](security-info-manage-settings.md)一文中的指示。

- 如果您遺失或忘記密碼，請從[密碼重設入口網站](https://passwordreset.microsoftonline.com/)或遵循[重設公司或學校密碼](user-help-reset-password.md)一文中的步驟來重設密碼。

- 在[無法登入 Microsoft 帳戶](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)一文中，取得登入問題的疑難排解秘訣和說明。