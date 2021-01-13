---
title: 如何管理應用程式密碼 - Azure Active Directory | Microsoft Docs
description: 了解應用程式密碼，以及其在雙步驟驗證方面的用途。
services: active-directory
author: curtand
manager: daveba
ms.reviewer: richagi
ms.assetid: 345b757b-5a2b-48eb-953f-d363313be9e5
ms.workload: identity
ms.service: active-directory
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 05/28/2020
ms.author: curtand
ms.custom: user-help, seo-update-azuread-jan
ms.openlocfilehash: 07303a0b0b3007ade9adb90af7397855a5014cc0
ms.sourcegitcommit: c136985b3733640892fee4d7c557d40665a660af
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/13/2021
ms.locfileid: "98179417"
---
# <a name="manage-app-passwords-for-two-step-verification"></a>管理適用於雙步驟驗證的應用程式密碼

> [!Important]
>您的系統管理員可能不允許您使用應用程式密碼。 如果沒有看到 [應用程式密碼] 的選項，即是無法在貴組織使用此功能。

使用應用程式密碼時，請務必記住：

- 應用程式密碼會自動產生，而且每個應用程式都應該建立並輸入一次。

- 每位使用者的密碼以 40 組為限。 如果您在到達該限制之後嘗試建立一個密碼，系統就會提示您先刪除現有的密碼，然後才能建立新密碼。

    >[!Note]
    >Office 2013 用戶端 (包括 Outlook) 支援新式驗證通訊協定，而且可搭配雙步驟驗證。 此支援表示在開啟雙步驟驗證之後，您將不再需要適用於 Office 2013 用戶端的應用程式密碼。 如需詳細資訊，請參閱 [Office 2013 和 Office 2016 用戶端應用程式的新式驗證運作方式](https://support.office.com/article/how-modern-authentication-works-for-office-2013-and-office-2016-client-apps-e4c45989-4b1a-462e-a81b-2a13191cf517)一文。

## <a name="create-new-app-passwords"></a>建立新的應用程式密碼

在進行初始雙因素驗證註冊程序期間，系統會提供您單一應用程式密碼。 如果您需要多個應用程式密碼，則必須自行建立。 您可以根據組織中設定雙因素驗證的方式，從多個區域建立應用程式密碼。 如需註冊以搭配您的公司或學校帳戶使用雙因素驗證的詳細資訊，請參閱[雙因素驗證和公司或學校帳戶的概觀](multi-factor-authentication-end-user-first-time.md)及其相關文章。

### <a name="where-to-create-and-delete-your-app-passwords"></a>在哪裡建立和刪除應用程式密碼

您可以根據您使用雙因素驗證的方式來建立和刪除應用程式密碼：

- **您的組織使用雙重要素驗證和 [其他安全性驗證] 頁面。** 如果您在組織中使用公司或學校帳戶 (例如，alain@contoso.com) 搭配雙因素驗證，則可以從 [[其他安全性驗證] 頁面](https://account.activedirectory.windowsazure.com/Proofup.aspx)管理您的應用程式密碼。 如需詳細指示，請參閱本文中的[使用 [其他安全性驗證] 頁面來建立和刪除應用程式密碼](#create-and-delete-app-passwords-from-the-additional-security-verification-page)。

- **您的組織使用雙因素驗證和 Office 365 入口網站。** 如果您使用公司或學校帳戶 (例如 alain@contoso.com) 、雙因素驗證，以及組織中的 Microsoft 365 應用程式，則可以從 [Office 365 入口網站頁面](https://www.office.com)管理您的應用程式密碼。 如需詳細指示，請參閱本文中的[使用 Office 365 入口網站來建立和刪除應用程式密碼](#create-and-delete-app-passwords-using-the-office-365-portal)。

- **您使用雙因素驗證搭配個人 Microsoft 帳戶。** 如果您使用個人 Microsoft 帳戶 (例如，alain@outlook.com) 搭配雙因素驗證，則可以從 [[安全性基本概念] 頁面](https://account.microsoft.com/security/)管理您的應用程式密碼。 如需詳細指示，請參閱[使用應用程式密碼搭配不支援雙步驟驗證的應用程式](https://support.microsoft.com/help/12409/microsoft-account-app-passwords-and-two-step-verification)。

## <a name="create-and-delete-app-passwords-from-the-additional-security-verification-page"></a>從 [其他安全性驗證] 頁面中建立和刪除應用程式密碼

您可以從工作或學校帳戶的 [其他安全性驗證] 頁面中建立和刪除應用程式密碼。

1. 登入 [[其他安全性驗證]](https://account.activedirectory.windowsazure.com/Proofup.aspx) 頁面，然後選取 [應用程式密碼]。

    ![[應用程式密碼] 頁面，其中反白顯示 [應用程式密碼] 索引標籤](media/multi-factor-authentication-end-user-app-passwords/mfa-app-passwords-page.png)

2. 選取 [建立]、輸入需要應用程式密碼的應用程式名稱，然後選取 [下一步]。

    ![[建立應用程式密碼] 頁面，隨附需要密碼的應用程式名稱](media/multi-factor-authentication-end-user-app-passwords/mfa-create-app-password-page.png)

3. 從 [您的應用程式密碼] 頁面複製密碼，然後選取 [關閉]。

    ![您的應用程式密碼頁面，隨附所指定應用程式的密碼](media/multi-factor-authentication-end-user-app-passwords/mfa-your-app-password-page.png)

4. 在 [ **應用程式密碼** ] 頁面上，確認已列出您的應用程式。

    ![[應用程式密碼] 頁面，其中新的應用程式顯示在清單中](media/multi-factor-authentication-end-user-app-passwords/mfa-app-passwords-page-with-new-password.png)  

5. 開啟您為其建立應用程式密碼的應用程式 (例如，Outlook 2010)，然後在要求應用程式密碼時將其貼上。 您應該只需要針對每個應用程式執行此動作一次。

### <a name="to-delete-an-app-password-using-the-app-passwords-page"></a>使用 [應用程式密碼] 頁面刪除應用程式密碼

1. 在 [ **應用程式密碼** ] 頁面上，選取您要刪除之應用程式密碼旁的 [ **刪除** ]。

   ![顯示在 [應用程式密碼] 頁面上刪除應用程式密碼的螢幕擷取畫面](media/multi-factor-authentication-end-user-app-passwords/mfa-app-passwords-page-delete.png)

2. 選取 [是] 以確認您想要刪除密碼，然後選取 [關閉]。

    已成功刪除應用程式密碼。

## <a name="create-and-delete-app-passwords-using-the-office-365-portal"></a>使用 Office 365 入口網站建立和刪除應用程式密碼

如果您搭配公司或學校帳戶和您的 Microsoft 365 apps 使用雙步驟驗證，則可以使用 Office 365 入口網站來建立和刪除應用程式密碼。

### <a name="to-create-app-passwords-using-the-office-365-portal"></a>使用 Office 365 入口網站建立應用程式密碼

1. 登入您的公司或學校帳戶，移至 [ [我的帳戶] 頁面](https://myaccount.microsoft.com)，然後選取 [ **安全性資訊**]。

    ![顯示 [安全性資訊] 索引標籤的 Office 入口網站](media/multi-factor-authentication-end-user-app-passwords/mfa-security-info.png)

2. 選取 [ **新增方法**]，從下拉式清單中選擇 [ **應用程式密碼** ]，然後按一下 [ **新增**]。

    ![安全性資訊頁面，使用 [新增方法 drowpdown] 清單](media/multi-factor-authentication-end-user-app-passwords/mfa-add-method.png)

3. 輸入應用程式密碼的名稱，然後選取 **[下一步]**。

    ![使用應用程式密碼的名稱來建立應用程式密碼頁面](media/multi-factor-authentication-end-user-app-passwords/mfa-enter-app-password-name.png)

4. 從 [ **應用程式密碼** ] 頁面複製密碼，然後選取 [ **完成**]。

    ![應用程式密碼頁面，其中包含您建立的新應用程式密碼](media/multi-factor-authentication-end-user-app-passwords/mfa-copy-app-password.png)

5. 在 [ **安全性資訊** ] 頁面上，確認已列出您的應用程式密碼。

    ![安全性資訊頁面，清單中顯示新的應用程式密碼](media/multi-factor-authentication-end-user-app-passwords/mfa-verify-app-password.png)  

6. 開啟您為 (建立應用程式密碼的應用程式，例如 Outlook 2016) ，然後在要求時貼上應用程式密碼。 您應該只需要針對每個應用程式執行此動作一次。

### <a name="to-delete-app-passwords-using-the-security-info-page"></a>使用安全性資訊頁面刪除應用程式密碼

1. 在 [ **安全性資訊** ] 頁面上，選取您要刪除之應用程式密碼旁的 [ **刪除** ]。

   ![顯示在 [安全性資訊] 頁面上刪除應用程式密碼的螢幕擷取畫面](media/multi-factor-authentication-end-user-app-passwords/mfa-delete-app-password.png)

2. 在確認方塊中選取 **[確定]** 。

    已成功刪除應用程式密碼。

## <a name="if-your-app-passwords-arent-working-properly"></a>如果您的應用程式密碼無法正確運作

請確定您已正確輸入密碼。 如果您確定已正確輸入密碼，您可以再次嘗試登入，並建立新的應用程式密碼。 如果那些選項都無法修正您的問題，請連絡貴組織的技術支援中心，如此一來，他們就能刪除您的現有應用程式密碼，讓您能夠建立全新的密碼。

## <a name="next-steps"></a>後續步驟

- [管理雙步驟驗證設定](multi-factor-authentication-end-user-manage-settings.md)

- 試用 [Microsoft Authenticator 應用程式](user-help-auth-app-download-install.md)，透過應用程式通知來驗證您的登入，而不是透過文字訊息或電話。
