---
title: 將安全性資訊設定為使用驗證器應用程式 - Azure Active Directory | Microsoft Docs
description: 設定您的安全性資訊，以使用 Microsoft Authenticator 應用程式來驗證您的身分識別。
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
ms.openlocfilehash: 776f98269e3e0c31766246ca96636528f60c3e09
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44160621"
---
# <a name="set-up-security-info-to-use-an-authenticator-app-preview"></a>將安全性資訊設定為使用驗證器應用程式 (預覽)

[!INCLUDE [preview-notice](../../../includes/active-directory-end-user-preview-notice-security-info.md)]

設定您的安全性資訊，需要您登入公司或學校帳戶，然後完成註冊程序。 如果您從未設定安全性資訊，系統將要求您立即進行設定。

## <a name="set-up-the-microsoft-authenticator-app"></a>設定 Microsoft Authenticator 應用程式

根據貴組織的設定，系統可能會在您登入時提示您設定 Microsoft Authenticator 應用程式。 否則，若要開始在安全性資訊中設定 Microsoft Authenticator 應用程式，請遵循[管理安全性資訊](security-info-manage-settings.md)中的步驟。

若要下載並深入了解 Microsoft Authenticator 應用程式，請參閱[開始使用 Microsoft Authenticator 應用程式](microsoft-authenticator-app-how-to.md)。

>[!Note]
>如果您不想使用 Microsoft Authenticator 應用程式，可以在設定期間選擇不同的應用程式。 本文使用 Microsoft Authenticator 應用程式。 如果您沒有看到驗證器應用程式選項，很可能您的組織不允許您使用驗證器應用程式進行驗證。 如果發生這種情況，您將必須選擇其他方法，或連絡您的系統管理員以取得更多協助。

### <a name="to-use-the-microsoft-authenticator-app"></a>使用 Microsoft Authenticator 應用程式

1. 選取 [驗證器應用程式] 選項。

    [取得應用程式] 精靈隨即出現。

    ![[取得應用程式] 精靈，第一個畫面](media/security-info/security-info-auth-app-wizard.png)

    如果您不想使用 Microsoft Authenticator 應用程式，可以從 [取得應用程式] 畫面中按一下 [我想要使用其他驗證器應用程式] 連結。

2. 安裝 Microsoft Authenticator 應用程式之後，請選取 [下一步]。

    如果您看見提示，請允許通知、新增新的帳戶，然後選取 [公司或學校帳戶]。

3. 選取 [下一步] 。

    [掃描 QR 代碼] 畫面隨即出現。

    ![使用驗證器應用程式的 [掃描 QR 代碼]](media/security-info/security-info-scan-qr.png)

4. 開啟 Microsoft Authenticator 應用程式、從右上角的 [自訂和控制] 圖示中選取 [新增帳戶]，然後選取 [公司或學校帳戶]。 

5. 如果您擁有 QR 代碼讀取器應用程式，請掃描所提供的代碼。 如果您沒有代碼讀取器應用程式，可以選取 [無法掃描 QR 代碼] 連結，然後以手動方式將代碼和 URL 輸入到 Microsoft Authenticator 應用程式。

6. 使用 Microsoft Authenticator 應用程式來核准通知以啟動用程式。

    您的安全性資訊會更新來使用 Microsoft Authenticator 應用程式，在使用雙步驟驗證或自助式密碼重設時，驗證您的身分識別。

    >[!Note]
    >如果您的組織允許它，您也會獲得驗證碼以及 Microsoft Authenticator 應用程式通知。 如果您想要使該驗證碼成為您的預設方法，請遵循[管理您的安全性資訊](security-info-setup-auth-app.md)中的指示。

## <a name="additional-security-info-options"></a>其他安全性資訊選項

根據您要嘗試執行的動作，您會有組織如何連絡您來驗證您身分識別的其他選項。 選項包括：

- **行動裝置簡訊。** 輸入您的行動裝置號碼，然後取得將用來進行雙步驟驗證或密碼重設的代碼簡訊。 如需如何透過簡訊 (SMS) 驗證您身分識別的逐步指示，請參閱[將安全性資訊設定為使用簡訊 (SMS)](security-info-setup-text-msg.md)。

- **行動裝置或公司電話。** 輸入您的行動裝置號碼，然後等候來電以進行雙步驟驗證或密碼重設。 如需如何透過電話號碼驗證您身分識別的逐步指示，請參閱[將安全性資訊設定為使用電話](security-info-setup-phone-number.md)。

- **電子郵件地址。** 輸入您的公司或學校電子郵件地址以接收用於密碼重設的電子郵件。 此選項不適用於雙步驟驗證。 如需如何設定電子郵件的逐步指示，請參閱[將安全性資訊設定為使用電子郵件](security-info-setup-email.md)。

- **安全性問題。** 回答由您的系統管理員為組織所建立的一些安全性問題。 此選項僅適用於密碼重設，不適用雙步驟驗證。 如需如何設定安全性問題的逐步指示，請參閱[將安全性資訊設定為使用安全性問題](security-info-setup-questions.md)一文。
    
    >[!Note]
    >如果這其中一些選項遺失，很可能是因為您的組織不允許那些方法。 如果發生這種情況，您將必須選擇可用的方法，或連絡您的系統管理員以取得更多協助。

## <a name="next-steps"></a>後續步驟

- 如果您需要更新安全性資訊，請遵循[管理安全性資訊](security-info-manage-settings.md)一文中的指示。

- 如果您遺失或忘記密碼，請從[密碼重設入口網站](https://passwordreset.microsoftonline.com/)或遵循[重設公司或學校密碼](user-help-reset-password.md)一文中的步驟來重設密碼。

- 在[無法登入 Microsoft 帳戶](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)一文中，取得登入問題的疑難排解秘訣和說明。