---
title: 如何管理 Azure Active Directory 中的應用程式密碼 | Microsoft Docs
description: 本頁面將協助使用者了解什麼是應用程式密碼，以及它們在雙步驟驗證方面的用途。
services: active-directory
author: eross-msft
manager: mtillman
ms.reviewer: richagi
ms.assetid: 345b757b-5a2b-48eb-953f-d363313be9e5
ms.workload: identity
ms.service: active-directory
ms.component: user-help
ms.topic: conceptual
ms.date: 07/30/2018
ms.author: lizross
ms.openlocfilehash: 836f426be950e33031ff74276218d1ba59f1f2f7
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43300908"
---
# <a name="manage-app-passwords-for-two-step-verification"></a>管理適用於雙步驟驗證的應用程式密碼

某些非瀏覽器應用程式 (例如 Outlook 2010) 不支援雙步驟驗證。 缺乏支援表示，如果您使用雙步驟驗證，應用程式將無法運作。 若要解決此問題，您可以建立自動產生的密碼來與每個非瀏覽器應用程式搭配使用，此密碼與您的一般密碼不同。

使用應用程式密碼時，請務必記住：

- 應用程式密碼會自動產生，而且每個應用程式只能輸入一次。

- 每位使用者的密碼以 40 組為限。 如果您在到達該限制之後嘗試建立一個密碼，系統就會提示您先刪除現有的密碼，然後才能建立新密碼。

- 為每個裝置 (而非每個應用程式) 使用一個應用程式密碼。 例如，針對膝上型電腦上的所有應用程式建立單一密碼，然後針對桌上型電腦上的所有應用程式建立另一個單一密碼。

    >[!Note]
    >Office 2013 用戶端 (包括 Outlook) 支援新式驗證通訊協定，而且可搭配雙步驟驗證。 此支援表示在開啟雙步驟驗證之後，您將不再需要適用於 Office 2013 用戶端的應用程式密碼。 如需詳細資訊，請參閱 [Office 2013 和 Office 2016 用戶端應用程式的新式驗證運作方式](https://support.office.com/article/how-modern-authentication-works-for-office-2013-and-office-2016-client-apps-e4c45989-4b1a-462e-a81b-2a13191cf517)一文。

## <a name="where-to-create-and-delete-your-app-passwords"></a>在哪裡建立和刪除應用程式密碼

您會在最初雙步驟驗證註冊期間獲得應用程式密碼。 如果需要一個以上的密碼，您可以根據使用雙步驟驗證的方式建立其他密碼：

- **搭配您的公司或學校帳戶和 MyApps 入口網站使用雙步驟驗證。** 使用本文內[使用 MyApps 入口網站建立和刪除應用程式密碼](#create-and-delete-app-passwords-using-the-myapps-portal)一節中的指示，來建立和刪除您的應用程式密碼。 如需 MyApps 入口網站及其使用方式的詳細資訊，請參閱[什麼是 Azure Active Directory 中的 MyApps 入口網站？](active-directory-saas-access-panel-introduction.md)。

- **搭配您的公司或學校帳戶和 Office 365 入口網站使用雙步驟驗證。** 使用本文內[使用 Office 365 入口網站建立和刪除應用程式密碼](#create-and-delete-app-passwords-using-the-office-365-portal)一節中的指示，來建立和刪除您的應用程式密碼。

- **搭配您的個人 Microsoft 帳戶使用雙步驟驗證。** 透過您個人的 Microsoft 帳戶，使用[安全性基本概念](https://account.microsoft.com/account/)頁面來建立和刪除應用程式密碼。 如需詳細資訊，請參閱[ 應用程式密碼和雙步驟驗證](https://support.microsoft.com/help/12409/microsoft-account-app-passwords-two-step-verification)一文。

## <a name="create-and-delete-app-passwords-using-the-myapps-portal"></a>使用 MyApps 入口網站建立和刪除應用程式密碼
您可以透過 MyApps 入口網站建立和刪除應用程式密碼。

### <a name="to-create-an-app-password-using-the-myapps-portal"></a>使用 MyApps 入口網站建立應用程式密碼

1. 登入 [https://myapps.microsoft.com](https://myapps.microsoft.com)。

2. 在右上方選取您的名稱，然後選擇 [設定檔]。

3. 選取 [其他安全性驗證]。

   ![選取 [其他安全性驗證] - 螢幕擷取畫面](./media/multi-factor-authentication-end-user-app-passwords/myapps1.png)

4. 選取 [應用程式密碼]。

   ![選取 [應用程式密碼] - 螢幕擷取畫面](./media/multi-factor-authentication-end-user-app-passwords/apppass2.png)

5. 按一下頁面底部的 [新增] 。

6. 輸入應用程式密碼的名稱，然後選取 [下一步]。

7. 將應用程式密碼複製到剪貼簿，並將它貼到您的應用程式。
   
    ![建立應用程式密碼](./media/multi-factor-authentication-end-user-app-passwords/create2.png)

### <a name="to-delete-an-app-password-using-the-myapps-portal"></a>使用 MyApps 入口網站刪除應用程式密碼

1. 移至您的設定檔，然後選取 [其他安全性驗證]。

2. 選取 [應用程式密碼]，然後選取您想要刪除之應用程式密碼旁的 [刪除]。

   ![刪除應用程式密碼](./media/multi-factor-authentication-end-user-app-passwords/delete1.png)

3. 選取 [是] 以確認您想要刪除密碼，然後選取 [關閉]。

## <a name="create-and-delete-app-passwords-using-the-office-365-portal"></a>使用 Office 365 入口網站建立和刪除應用程式密碼

如果您搭配公司或學校帳戶和 Office 365 應用程式使用雙步驟驗證，您就可以使用 Office 365 入口網站來建立和刪除應用程式密碼。 您在任何時候最多可擁有 40 組應用程式密碼。 如果您在到達該限制之後需要另一個應用程式密碼，將必須刪除現有的其中一個應用程式密碼。

### <a name="to-create-app-passwords-using-the-office-365-portal"></a>使用 Office 365 入口網站建立應用程式密碼

1. 登入您的公司或學校帳戶。

2. 前往 https://portal.office.com、選取 [Office 365 入口網站] 頁面右上角的 [設定] 圖示，然後展開 [其他安全性驗證]。

    ![顯示已展開 [其他安全性驗證] 區域的 Office 入口網站](media/security-info/security-info-o365password.png)

3. 選取**建立和管理應用程式密碼**的文字來開啟 [應用程式密碼] 頁面。

4. 選取 [建立]、輸入需要應用程式密碼的應用程式易記名稱，然後選取 [下一步]。

5. 選取 [將密碼複製到剪貼簿]，然後選取 [關閉]。

6. 使用複製的應用程式密碼來登入非瀏覽器應用程式。 您只需輸入此密碼一次，系統就會記住它以供日後使用。

### <a name="to-delete-app-passwords-using-the-office-365-portal"></a>使用 Office 365 入口網站刪除應用程式密碼

1. 登入您的公司或學校帳戶。

2. 前往 https://portal.office.com、選取 [Office 365 入口網站] 頁面右上角的 [設定] 圖示，然後選取 [其他安全性驗證]。

3. 選取**建立和管理應用程式密碼**的文字來開啟 [應用程式密碼] 頁面。

4. 針對要刪除的應用程式密碼選取 [刪除]、在確認方塊中選取 [是]，然後選取 [關閉]。

    已成功刪除應用程式密碼。

5. 遵循建立應用程式密碼的步驟來建立新的應用程式密碼。

## <a name="if-your-app-passwords-arent-working-properly"></a>如果您的應用程式密碼無法正確運作

請確定您已正確輸入密碼。 如果您確定已正確輸入密碼，您可以再次嘗試登入，並建立新的應用程式密碼。 如果那些選項都無法修正您的問題，請連絡公司支援人員，如此一來，他們就能刪除您的現有應用程式密碼，讓您能夠建立全新的密碼。 

## <a name="next-steps"></a>後續步驟

- [管理雙步驟驗證設定](multi-factor-authentication-end-user-manage-settings.md)

- 試用 [Microsoft Authenticator 應用程式](microsoft-authenticator-app-how-to.md)，透過應用程式通知來驗證您的登入，而不是透過文字訊息或電話。
