---
title: 將安全性資訊設定為使用電話 - Azure Active Directory | Microsoft Docs
description: 設定您的安全性資訊，以使用行動裝置或公司電話號碼來驗證您的身分識別。
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
ms.openlocfilehash: 701a47a3907c77dbf5d51692e5d1ddc0c49ab985
ms.sourcegitcommit: e3d5de6d784eb6a8268bd6d51f10b265e0619e47
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39390097"
---
# <a name="set-up-security-info-to-use-phone-calls-preview"></a>將安全性資訊設定為使用電話 (預覽)

[!INCLUDE[preview-notice](../../../includes/active-directory-end-user-preview-notice-security-info.md)]

要設定安全性資訊，您必須登入公司或學校帳戶，然後完成註冊程序。 如果您從未設定安全性資訊，系統將要求您立即進行設定。

## <a name="set-up-phone-calls"></a>設定撥打電話

根據您的組織設定，系統可能會在您登入時提示您將電話號碼新增至安全性資訊。 否則，若要開始在安全性資訊中設定電話，請遵循[管理安全性資訊](security-info-manage-settings.md)中的步驟。

>[!Note]
>安全性資訊不支援使用電話分機。 即使您新增適當格式 +1 4255551234X12345，在撥號之前仍會移除該分機號碼。<br>如果您未看到電話選項，可能是因為您的組織不允許您使用電話進行驗證。 如果發生這種情況，您將必須選擇其他方法，或連絡您的系統管理員以取得更多協助。

### <a name="to-use-your-phone-number"></a>使用您的電話號碼

1. 選取 [電話] 選項。

    [設定您的電話] 精靈隨即出現。

    ![設定您的國家或地區碼和電話號碼](media/security-info/security-info-keep-secure-setup-phone.png)

2. 從下拉式清單方塊中挑選您的 [國家或地區]、在 [電話號碼] 方塊中輸入您的電話號碼 (包括適用的區碼)、選取 [打電話給我] 選項，然後選取 [下一步]。

    您會收到電話以確認您輸入的電話號碼正確無誤。 屆時，系統會要求您按井字 (#) 鍵以確認並完成設定。

    ![確認您的手機螢幕顯示已成功接聽來電](media/security-info/security-info-keep-secure-verify-phone-call.png)

    您的安全性資訊會更新成使用電話號碼，在使用雙步驟驗證或自助式密碼重設時驗證您的身分識別。

    >[!Note]
    >如果您想要接收傳至行動裝置的簡訊，而非來電，請遵循[將安全性資訊設定為使用簡訊 (SMS)](security-info-setup-text-msg.md) 一文中的步驟。

## <a name="additional-security-info-options"></a>其他安全性資訊選項

根據您要嘗試執行的動作，您會有組織如何連絡您來驗證您身分識別的其他選項。 選項包括：

- **驗證器應用程式。** 下載並使用驗證器應用程式，以取得核准通知或隨機產生的核准代碼來進行雙步驟驗證或密碼重設。 如需如何設定和使用 Microsoft Authenticator 應用程式的逐步指示，請參閱[將安全性資訊設定為使用驗證器應用程式](security-info-setup-auth-app.md)。

- **行動裝置簡訊。** 輸入您的行動裝置號碼，然後取得您將用來進行雙步驟驗證或密碼重設的代碼簡訊。 如需如何透過簡訊 (SMS) 驗證您身分識別的逐步指示，請參閱[將安全性資訊設定為使用簡訊 (SMS)](security-info-setup-text-msg.md)。

- **電子郵件地址。** 輸入您的公司或學校電子郵件地址以接收用於密碼重設的電子郵件。 此選項不適用於雙步驟驗證。 如需如何設定電子郵件的逐步指示，請參閱[將安全性資訊設定為使用電子郵件](security-info-setup-email.md)。

- **安全性問題。** 回答由您的系統管理員為組織所建立的一些安全性問題。 此選項僅適用於密碼重設，不適用雙步驟驗證。 如需如何設定安全性問題的逐步指示，請參閱[將安全性資訊設定為使用安全性問題](security-info-setup-questions.md)一文。
    
    >[!Note]
    >如果這其中一些選項遺失，很可能是因為您的組織不允許那些方法。 如果發生這種情況，您將必須選擇可用的方法，或連絡您的系統管理員以取得更多協助。

## <a name="next-steps"></a>後續步驟

- 如果您需要更新安全性資訊，請遵循[管理安全性資訊](security-info-manage-settings.md)一文中的指示。

- 如果您遺失或忘記密碼，請從[密碼重設入口網站](https://passwordreset.microsoftonline.com/)或遵循[重設公司或學校密碼](user-help-reset-password.md)一文中的步驟來重設密碼。

- 在[無法登入 Microsoft 帳戶](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)一文中，取得登入問題的疑難排解秘訣和說明。