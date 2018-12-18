---
title: 使用安全性資訊設定應用程式密碼 - Azure Active Directory | Microsoft Docs
description: 使用安全性資訊，設定自動產生的密碼 (應用程式密碼) 來與每個非瀏覽器應用程式搭配使用，此密碼與一般密碼不同。
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
ms.openlocfilehash: e527a0eaec433b96b5c37c5ec22f392a7166dfe8
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44162439"
---
# <a name="manage-app-passwords-using-security-info-preview"></a>使用安全性資訊管理應用程式密碼 (預覽)

[!INCLUDE [preview-notice](../../../includes/active-directory-end-user-preview-notice-security-info.md)]

某些非瀏覽器應用程式 (例如 Outlook 2010) 不支援雙步驟驗證。 缺乏支援表示，如果您使用雙步驟驗證，應用程式將無法運作。 若要解決此問題，您可以建立自動產生的密碼來與每個非瀏覽器應用程式搭配使用，此密碼與您的一般密碼不同。

使用應用程式密碼時，請務必記住：

- 應用程式密碼會自動產生，而且每個應用程式只能輸入一次。

- 每位使用者的密碼以 40 組為限。 如果您在到達該限制之後嘗試建立一個密碼，系統就會提示您先刪除現有的密碼，然後才能建立新密碼。

- 為每個裝置 (而非每個應用程式) 使用一個應用程式密碼。 例如，針對膝上型電腦上的所有應用程式建立單一密碼，然後針對桌上型電腦上的所有應用程式建立另一個單一密碼。

    >[!Note]
    >Office 2013 用戶端 (包括 Outlook) 支援新式驗證通訊協定，而且可搭配雙步驟驗證。 此支援表示在開啟雙步驟驗證之後，您將不再需要適用於 Office 2013 用戶端的應用程式密碼。 如需詳細資訊，請參閱 [Office 2013 和 Office 2016 用戶端應用程式的新式驗證運作方式](https://support.office.com/article/how-modern-authentication-works-for-office-2013-and-office-2016-client-apps-e4c45989-4b1a-462e-a81b-2a13191cf517)一文。

## <a name="create-and-delete-app-passwords-using-security-info"></a>使用安全性資訊建立和刪除應用程式密碼

如果您搭配公司或學校帳戶使用雙步驟驗證，而且您的系統管理員已開啟安全性資訊體驗，您就可以使用「我的應用程式」入口網站來建立和刪除應用程式密碼。

如果您的系統管理員尚未開啟安全性資訊體驗，您就必須遵循[管理應用程式密碼以進行雙步驟驗證](multi-factor-authentication-end-user-app-passwords.md)一節中的指示和資訊。

### <a name="to-create-app-passwords-using-the-my-apps-portal"></a>使用「我的應用程式」入口網站建立應用程式密碼

1. 登入您的公司或學校帳戶。

2. 移至 myapps.microsoft.com，從頁面右上角選取您的名稱，然後選取 [設定檔]。

3. 在 [管理帳戶] 區域中，選取 [編輯安全性資訊]。

    ![已醒目提示 [編輯安全性資訊] 連結的 [設定檔] 畫面](media/security-info/security-info-profile.png)

4. 在 [保護您的帳戶安全] 畫面中，選取 [新增安全性資訊]。

    ![包含現有可編輯資訊的安全性資訊畫面](media/security-info/security-info-edit-add-info.png)

5. 在 [新增安全性資訊] 畫面中，選取 [應用程式密碼]。

6. 在 [建立您的應用程式密碼] 畫面中，輸入您應用程式密碼的名稱，然後選取 [下一步]。

    ![您為應用程式密碼命名的畫面](media/security-info/security-info-name-app-password.png)

7. 選取 [複製] 以將密碼複製到剪貼簿，然後選取 [下一步]。

    ![複製應用程式密碼的畫面](media/security-info/security-info-create-app-password.png)
    
8. 確定應用程式密碼會出現在 [保護您的帳戶安全] 畫面上。

    ![保護安全畫面，使用應用程式密碼](media/security-info/security-info-keep-secure-app-password.png)

### <a name="to-delete-app-passwords-using-the-my-apps-portal"></a>使用「我的應用程式」入口網站刪除應用程式密碼

1. 在 [保護您的帳戶安全] 畫面上，選取要刪除之應用程式密碼旁的 **X**。

    ![保護安全畫面，刪除應用程式密碼](media/security-info/security-info-keep-secure-delete-app-password.png)

2. 在 [刪除應用程式密碼] 畫面中，選取 [刪除]。

    ![刪除應用程式密碼畫面](media/security-info/security-info-keep-secure-delete-app-password2.png)

## <a name="next-steps"></a>後續步驟

- 如果您需要更新安全性資訊，請遵循[管理安全性資訊](security-info-manage-settings.md)一文中的指示。

- 如需更多關於安全性資訊以及您可以做些什麼的一般資訊，請參閱[安全性資訊概觀](user-help-security-info-overview.md) 