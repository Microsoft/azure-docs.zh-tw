---
title: 註冊自助式密碼重設 - Azure Active Directory | Microsoft Docs
description: 註冊 Azure AD 自助式密碼重設的驗證資料
services: active-directory
author: eross-msft
manager: mtillman
ms.reviewer: sahenry
ms.assetid: ''
ms.service: active-directory
ms.component: user-help
ms.workload: identity
ms.topic: conceptual
ms.date: 01/11/2018
ms.author: lizross
ms.openlocfilehash: f8b4fb8861760d2b5b10f61b3ab6a8c718f849b6
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47106474"
---
# <a name="register-for-self-service-password-reset"></a>註冊自助式密碼重設

> [!IMPORTANT]
> 您是否因無法登入而在這裡？ 若是如此，請參閱[重設您的工作密碼或學校密碼](active-directory-passwords-update-your-own-password.md)。

身為終端使用者，如果您是使用 Azure Active Directory (Azure AD) 自助式密碼重設 (SSPR)，就可以自行重設密碼或解除鎖定您的帳戶。 在您可以使用這項功能之前，必須先註冊驗證方法，或確認您系統管理員已填入的預先定義驗證方法。

## <a name="register-or-confirm-authentication-data-with-sspr"></a>使用 SSPR 註冊或確認驗證資料

1. 開啟您裝置的 web 瀏覽器並移至[密碼重設註冊頁面](https://aka.ms/ssprsetup)。
2. 輸入您的使用者名稱和您系統管理員所提供的密碼。
3. 視您的 IT 人員如何設定而定，下列一或多個選項可供您進行設定及驗證。 如果您的系統管理員具有權限可使用您的資訊，他們可以填入一些您的資訊。
    * **辦公室電話**：只有您的系統管理員可以設定此選項。
    * **驗證電話**：將這個選項設為另一個您可以存取的電話號碼。 例如，可以接收簡訊或電話的行動電話。
    * **驗證電子郵件**：將此選項設定為替代電子郵件地址，您可加以存取而不需要使用所需重設的密碼。
    * **安全性問題**：您的系統管理員已核准這份問題清單以供您回答。 您不可以使用相同的問題或回答一次以上。
4. 提供並確認您系統管理員所需的資訊。 如果可以使用多個選項，建議您註冊多個方法。 當其中一種方法無法使用時，這可為您提供彈性。 例如，當您在旅行時，無法接聽您的辦公室電話。

    ![註冊驗證方法，並選取 [完成]][Register]

5. 選取 [完成]。 現在當您日後需要 SSPR 時就可使用。

如果您輸入**驗證電話**或**驗證電子郵件**的資料，在全域目錄中會看不見。 您和您的系統管理員是唯一可以看到此資料的人員。 只有您可以查看您安全性問題的答案。

系統管理員在一段時間後可能會要求您確認驗證方法，確定您仍有適當的已註冊方法。

## <a name="common-problems-and-their-solutions"></a>常見問題及其解決方案

 以下提供一些常見的錯誤案例及其解決方案：

| 錯誤案例| 您看到什麼錯誤訊息？| 解決方法 |
| --- | --- | --- |
| 在輸入我的使用者 ID 後，出現了「請連絡您的系統管理員」頁面 | 請連絡您的系統管理員。 <br> <br> 我們偵測到您的使用者帳戶密碼未受到 Microsoft 管理。 因此，我們無法自動重設您的密碼。 <br> <br> 請連絡 IT 人員以尋求進一步的協助。 | 您之所以看到此訊息，是因為 IT 人員在內部部署環境中管理您的密碼，而不允許您從 [無法存取您的帳戶] 連結重設您的密碼。 <br> <br> 若要重設密碼，請直接連絡您的 IT 人員以取得協助。 讓它們知道您需要重設密碼，以便它們可以為您啟用此功能。|
| 在輸入我的使用者 ID 後，出現了「您的帳戶未啟用密碼重設」錯誤 | 您的帳戶未啟用密碼重設功能。 <br> <br> 很抱歉，IT 人員還沒將您的帳戶設定用於此服務。 <br> <br> 如果您願意，我們可以連絡貴組織的系統管理員來為您重設密碼。 | 您之所以看到此訊息，是因為 IT 人員尚未針對貴組織啟用從 [無法存取您的帳戶] 連結重設密碼的功能，或尚未授權您使用該功能。 <br> <br> 若要重設您的密碼，請選取 [連絡系統管理員] 連結。 系統會傳送電子郵件至貴公司的 IT 人員。 電子郵件會讓它們知道您需要重設密碼，以便它們可以為您啟用此功能。 |
| 在輸入我的使用者 ID 後，出現了「我們無法驗證您的帳戶」錯誤 | 我們無法驗證您的帳戶。 <br> <br> 如果您願意，我們可以連絡貴組織的系統管理員來為您重設密碼。 | 您之所以看到此訊息，是因為您已啟用密碼重設，但並未註冊使用此服務。 若要註冊密碼重設，請在重新取得帳戶存取權後移至[密碼重設註冊頁面](https://aka.ms/ssprsetup)。 <br> <br> 若要重設密碼，請選取 [連絡系統管理員] 連結，以傳送電子郵件給公司的 IT 人員。 |

## <a name="next-steps"></a>後續步驟

* [使用自助式密碼重設來變更您的密碼](active-directory-passwords-update-your-own-password.md)
* [密碼重設註冊頁面](https://aka.ms/ssprsetup)
* [密碼重設入口網站](https://passwordreset.microsoftonline.com/)
* [當您無法登入 Microsoft 帳戶時](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)

[Register]: ./media/active-directory-passwords-reset-register/register-2-methods.png "顯示已註冊方法及完成按鈕的密碼重設註冊頁面"

