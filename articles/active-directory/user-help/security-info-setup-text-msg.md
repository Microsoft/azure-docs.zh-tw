---
title: 將安全性資訊設定為使用簡訊 - Azure Active Directory | Microsoft Docs
description: 設定您的安全性資訊，以使用簡訊 (SMS) 來驗證您的身分識別。
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
ms.openlocfilehash: f20bcb4245a2870d8eaa442c6f747ee533b362db
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39347704"
---
# <a name="set-up-security-info-to-use-text-messaging-preview"></a>將安全性資訊設定為使用簡訊 (預覽)

[!INCLUDE[preview-notice](../../../includes/active-directory-end-user-preview-notice-security-info.md)]

設定您的安全性資訊，需要您登入公司或學校帳戶，然後完成註冊程序。 如果您從未設定安全性資訊，系統將要求您立即進行設定。

## <a name="set-up-text-messaging"></a>安裝文字傳訊

根據貴組織的設定，系統可能會在您登入時提示您將簡訊新增到安全性資訊。 否則，若要開始在安全性資訊中設定簡訊，請遵循[管理安全性資訊](security-info-manage-settings.md)中的步驟。

簡訊選項是電話選項的一部分，因此，將以您針對電話號碼所使用的相同方式來設定每個項目，但您將選擇使用簡訊，而不是讓 Microsoft 致電給您。 如果您未看到電話選項，很可能您的組織不允許您使用電話號碼進行驗證。 如果發生這種情況，您必須選擇其他方法，或連絡您的系統管理員以取得更多協助。

### <a name="to-use-a-text-message"></a>使用簡訊

1. 選取 [電話] 選項。

    [設定您的電話] 精靈隨即出現。

    ![設定您的國家或地區碼和電話號碼](media/security-info/security-info-keep-secure-setup-text.png)

2. 從下拉式清單方塊中挑選您的**國家或地區**、在 [電話號碼] 方塊中輸入您的電話號碼 (包括區碼，如果適用)、選取 [以簡訊傳送代碼給我] 選項，然後按一下 [下一步]。

    您將收到一則簡訊，其中包含您需要輸入到驗證頁面的代碼。

    ![輸入簡訊代碼的驗證頁面](media/security-info/security-info-keep-secure-verify-text-msg.png)

    您的安全性資訊會更新以傳送簡訊給您，在使用雙步驟驗證或自助式密碼重設時，驗證您的身分識別。

    >[!Note]
    >如果您想要接到來電，而非簡訊，請遵循[將安全性資訊設定為使用電話](security-info-setup-phone-number.md)一文中的步驟。

## <a name="additional-security-info-options"></a>其他安全性資訊選項

根據您要嘗試執行的動作，您會有組織如何連絡您來驗證您身分識別的選項。 選項包括：

- **驗證器應用程式。** 下載並使用驗證器應用程式，以取得核准通知或隨機產生的核准代碼來進行雙步驟驗證或密碼重設。 如需如何設定和使用 Microsoft Authenticator 應用程式的逐步指示，請參閱[將安全性資訊設定為使用驗證器應用程式](security-info-setup-auth-app.md)。

- **行動裝置或公司電話。** 輸入您的行動裝置號碼，然後等候來電以進行雙步驟驗證或密碼重設。 如需如何透過電話號碼驗證您身分識別的逐步指示，請參閱[將安全性資訊設定為使用電話](security-info-setup-phone-number.md)。

- **電子郵件地址。** 輸入您的公司或學校電子郵件地址以接收用於密碼重設的電子郵件。 此選項不適用雙步驟驗證。 如需如何設定電子郵件的逐步指示，請參閱[將安全性資訊設定為使用電子郵件](security-info-setup-email.md)。
   
    >[!Note]
    >如果這其中一些選項遺失，很可能是因為您的組織不允許那些方法。 如果發生這種情況，您將必須選擇可用的方法，或連絡您的系統管理員以取得更多協助。

- **安全性問題。** 回答由您的系統管理員為組織所建立的一些安全性問題。 此選項僅適用於密碼重設，不適用雙步驟驗證。 如需如何設定安全性問題的逐步指示，請參閱[將安全性資訊設定為使用安全性問題](security-info-setup-questions.md)一文。

## <a name="next-steps"></a>後續步驟

- 如果您需要更新安全性資訊，請遵循[管理安全性資訊](security-info-manage-settings.md)一文中的指示。

- 如果您遺失或忘記密碼，請從[密碼重設入口網站](https://passwordreset.microsoftonline.com/)或遵循[重設公司或學校密碼](user-help-reset-password.md)一文中的步驟來重設密碼。

- 在[無法登入 Microsoft 帳戶](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)一文中取得登入問題的疑難排解秘訣和說明。