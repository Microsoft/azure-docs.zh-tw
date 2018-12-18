---
title: 將安全性資訊設定為使用安全性問題 - Azure Active Directory | Microsoft Docs
description: 設定您的安全性資訊，以使用預先定義的安全性問題來驗證您的身分識別。
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
ms.openlocfilehash: b4913d55ee3d254f197512d9251ae750d8896f53
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44160433"
---
# <a name="set-up-security-info-to-use-pre-defined-security-questions-preview"></a>將安全性資訊設定為使用預先定義的安全性問題 (預覽)

[!INCLUDE [preview-notice](../../../includes/active-directory-end-user-preview-notice-security-info.md)]

要設定安全性資訊，您必須登入公司或學校帳戶，然後完成註冊程序。 如果您從未設定安全性資訊，系統將要求您立即進行設定。

## <a name="set-up-security-questions"></a>設定安全性問題

根據您組織的設定，系統可能會在您登入時提示您將安全性問題新增至安全性資訊。 否則，若要開始在安全性資訊中設定安全性問題，請遵循[管理安全性資訊](security-info-manage-settings.md)中的步驟。

如果您使用安全性問題，建議結合另一個方法共同使用。 安全性問題可能會比其他方法更不安全，因為有些人可能會知道他人問題的答案。

>[!Note]
>安全性問題會私密且安全地儲存在目錄中的使用者物件上，只能在註冊期間由您回答。 系統管理員無法讀取或修改您的問題或答案。<br>如果您未看到安全性問題選項，可能是因為您的組織不允許您使用安全性問題進行驗證。 如果發生這種情況，您將必須選擇其他方法，或連絡您的系統管理員以取得更多協助。

### <a name="to-choose-and-answer-your-security-questions"></a>選擇並回答安全性問題

1. 選取 [安全性問題]，然後選擇您要回答的安全性問題。 

    您必須挑選的安全性問題數目由系統管理員決定。

    ![安全性資訊頁面，選擇您的安全性問題](media/security-info/security-info-keep-secure-setup-pick-questions.png)

2. 提供您所選問題的答案，然後選取 [完成]。

## <a name="additional-security-info-options"></a>其他安全性資訊選項

根據您要嘗試執行的動作，您會有組織如何連絡您來驗證您身分識別的選項。 選項包括：

- **驗證器應用程式。** 下載並使用驗證器應用程式，以取得核准通知或隨機產生的核准代碼來進行雙步驟驗證或密碼重設。 如需如何設定和使用 Microsoft Authenticator 應用程式的逐步指示，請參閱[將安全性資訊設定為使用驗證器應用程式](security-info-setup-auth-app.md)。

- **行動裝置簡訊。** 輸入您的行動裝置號碼，然後取得將用來進行雙步驟驗證或密碼重設的代碼簡訊。 如需如何透過簡訊 (SMS) 驗證您身分識別的逐步指示，請參閱[將安全性資訊設定為使用簡訊 (SMS)](security-info-setup-text-msg.md)。

- **行動裝置或公司電話。** 輸入您的行動裝置號碼，然後等候來電以進行雙步驟驗證或密碼重設。 如需如何透過電話號碼驗證您身分識別的逐步指示，請參閱[將安全性資訊設定為使用電話](security-info-setup-phone-number.md)。

- **電子郵件地址。** 輸入您的公司或學校電子郵件地址以接收用於密碼重設的電子郵件。 此選項不適用於雙步驟驗證。 如需如何設定電子郵件的逐步指示，請參閱[將安全性資訊設定為使用電子郵件](security-info-setup-email.md)。
   
    >[!Note]
    >如果這其中一些選項遺失，很可能是因為您的組織不允許那些方法。 如果發生這種情況，您將必須選擇可用的方法，或連絡您的系統管理員以取得更多協助。

## <a name="next-steps"></a>後續步驟

- 如果您需要更新安全性資訊，請遵循[管理安全性資訊](security-info-manage-settings.md)一文中的指示。

- 如果您遺失或忘記密碼，請從[密碼重設入口網站](https://passwordreset.microsoftonline.com/)或遵循[重設公司或學校密碼](user-help-reset-password.md)一文中的步驟來重設密碼。

- 在[無法登入 Microsoft 帳戶](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)一文中，取得登入問題的疑難排解秘訣和說明。