---
title: 登入針對密碼單一登入設定之 Azure AD 資源庫應用程式的問題 | Microsoft Docs
description: 如何為針對密碼單一登入設定的 Azure AD 資源庫應用程式問題疑難排解
documentationcenter: ''
author: barbkess
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: barbkess
ms.reviewer: asteen
ms.openlocfilehash: 6af369c9ba1621dcedc2adf74f0c25321422f3c4
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355133"
---
# <a name="problems-signing-in-to-an-azure-ad-gallery-application-configured-for-password-single-sign-on"></a>登入針對密碼單一登入設定之 Azure AD 資源庫應用程式的問題

存取面板是網頁型入口網站，可讓在 Azure Active Directory (Azure AD) 中具有公司或學校帳戶的使用者，檢視和啟動 Azure AD 系統管理員已授權他們存取的雲端式應用程式。 具有 Azure AD 版本的使用者也可以透過存取面板使用自助群組和應用程式管理功能。 存取面板與 Azure 入口網站分開，使用者不需要具備 Azure 訂用帳戶也能使用。

若要在存取面板中使用密碼單一登入 (SSO)，必須在使用者的瀏覽器中安裝存取面板延伸模組。 當使用者選取已設定密碼 SSO 的應用程式時，就會自動下載此存取面板延伸模組。

## <a name="meeting-browser-requirements-for-the-access-panel"></a>符合存取面板的瀏覽器需求

存取面板需要支援 JavaScript 且已啟用 CSS 的瀏覽器。 若要在存取面板中使用密碼單一登入 (SSO)，必須在使用者的瀏覽器中安裝存取面板延伸模組。 當使用者選取已設定密碼 SSO 的應用程式時，就會自動下載此存取面板延伸模組。

若是密碼 SSO，則使用者的瀏覽器可以是：

-   Internet Explorer 8、9、10、11 - 在 Windows 7 或更新版本上

-   Chrome - 在 Windows 7 或更新版本，和在 MacOS X 或更新版本上

-   Firefox 26.0 或更新版本 - 在 Windows XP SP2 或更新版本，和在 Mac OS X 10.6 或更新版本上

>[!NOTE]
>當瀏覽器擴充開始支援 Edge 時，Windows 10 中的 Edge 就能使用密碼 SSO 擴充。
>
>

## <a name="how-to-install-the-access-panel-browser-extension"></a>如何安裝存取面板的瀏覽器延伸模組

若要安裝存取面板的瀏覽器擴充功能，請遵循下列步驟：

1.  在其中一種支援的瀏覽器中開啟[存取面板](https://myapps.microsoft.com)，然後在您的 Azure AD 中以**使用者**身分登入。

2.  按一下存取面板中的 [密碼-SSO 應用程式]。

3.  在要求安裝軟體的提示中，選取 [立即安裝]。

4.  系統會根據您的瀏覽器將您導向至下載連結。 將延伸模組**新增**到瀏覽器中。

5.  如果您的瀏覽器要求，請選取 [啟用] 或 [允許] 該延伸模組。

6.  安裝之後，**重新啟動**瀏覽器工作階段。

7.  登入存取面板，並查看您是否可以**啟動**密碼-SSO 應用程式

您可能也會從下列直接連結中下載適用於 Chrome 和 Firefox 的延伸模組：

-   [Chrome 存取面板延伸模組](https://chrome.google.com/webstore/detail/access-panel-extension/ggjhpefgjjfobnfoldnjipclpcfbgbhl)

-   [Firefox 存取面板延伸模組](https://addons.mozilla.org/firefox/addon/access-panel-extension/)

## <a name="setting-up-a-group-policy-for-internet-explorer"></a>設定適用於 Internet Explorer 的群組原則

您可以設定一個群組原則，以允許您在使用者電腦上遠端安裝適用於 Internet Explorer 的存取面板延伸模組。

必要條件包括：

-   您已設定了 [Active Directory 網域服務](https://msdn.microsoft.com/library/aa362244%28v=vs.85%29.aspx)，並且已將使用者的電腦加入網域。

-   您必須擁有「編輯設定」權限，才能編輯群組原則物件 (GPO)。 根據預設，下列安全性群組的成員擁有此權限：網域系統管理員、企業系統管理員及群組原則建立者擁有者。 [深入了解](https://technet.microsoft.com/library/cc781991%28v=ws.10%29.aspx)。

如需如何設定群組原則並將它部署到使用者的逐步指示，請遵循[如何使用群組原則部署 Internet Explorer 的存取面板延伸模組](https://docs.microsoft.com/azure/active-directory/active-directory-saas-ie-group-policy)教學課程。

## <a name="troubleshoot-the-access-panel-in-internet-explorer"></a>進行 Internet Explorer 中存取面板的疑難排解

如需存取診斷工具和設定 IE 延伸模組的逐步指示，請遵循[疑難排解 Internet Explorer 的存取面板延伸模組](https://docs.microsoft.com/azure/active-directory/active-directory-saas-ie-troubleshooting)指南。

## <a name="how-to-configure-password-single-sign-on-for-an-azure-ad-gallery-application"></a>如何為 Azure AD 資源庫應用程式設定密碼單一登入

若要設定 Azure AD 資源庫中的應用程式，您必須：

-   [從 Azure AD 資源庫新增應用程式](#_Add_an_application)

-   [設定應用程式使用密碼單一登入](#configure-the-application-for-password-single-sign-on)

-   [將使用者指派至應用程式](#assign-users-to-the-application)

### <a name="add-an-application-from-the-azure-ad-gallery"></a>從 Azure AD 資源庫新增應用程式

若要從 Azure AD 資源庫新增應用程式，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [企業應用程式] 窗格右上角的 [新增] 按鈕。

6.  在 [從資源庫新增] 區段的 [輸入名稱] 文字方塊中，輸入應用程式名稱。

7.  選取您要設為單一登入的應用程式。

8.  新增應用程式之前，您可以從 [名稱] 文字方塊變更其名稱。

9.  按一下 [新增] 按鈕新增應用程式。

稍候片刻，您便能看見應用程式的設定窗格。

### <a name="configure-the-application-for-password-single-sign-on"></a>設定應用程式使用密碼單一登入

若要設定應用程式使用單一登入，請遵循下列步驟：

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，然後以**全域管理員**或**共同管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

   * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  選取您要設定單一登入的應用程式

7.  應用程式載入後，按一下應用程式的左側導覽功能表中的 [單一登入]。

8.  選取 [以密碼為基礎的登入] 模式。

9.  [將使用者指派至應用程式](#_How_to_assign)。

10. 此外，您也可以選取使用者資料列，按一下 [更新認證]，然後代表使用者輸入使用者名稱和密碼，以代表使用者提供認證。 否則，系統會提示使用者在啟動時自行輸入認證。

### <a name="assign-users-to-the-application"></a>將使用者指派給應用程式

若要直接將一或多個使用者指派給應用程式，請遵循下列步驟︰

1.  開啟 [Azure 入口網站](https://portal.azure.com/)，以**全域管理員**身分登入。

2.  按一下左側主導覽功能表底部的 [所有服務]，以開啟 [Azure Active Directory 延伸模組]。

3.  在篩選搜尋方塊中輸入 **“Azure Active Directory**”，然後選取 [Azure Active Directory] 項目。

4.  在 Azure Active Directory 左側導覽功能表中，按一下 [企業應用程式]。

5.  按一下 [所有應用程式]，以檢視所有應用程式的清單。

   * 若在這裡沒看到您要顯示的應用程式，請使用 [所有應用程式清單] 頂端的 [篩選] 控制項，並將 [顯示] 選項設定為 [所有應用程式]。

6.  從清單中選取您想要指派使用者的應用程式。

7.  應用程式載入之後，按一下應用程式左側導覽功能表中的 [使用者和群組]。

8.  按一下 [使用者和群組] 清單頂端的 [新增] 按鈕，以開啟 [新增指派] 窗格。

9.  按一下 [新增指派] 窗格中的 [使用者和群組] 選取器。

10. 在 [依姓名或電子郵件地址搜尋] 搜尋方塊中，輸入您有興趣指派之使用者的**全名**或**電子郵件地址**。

11. 將滑鼠停留在清單中的**使用者**上方，以顯示**核取方塊**。 按一下使用者設定檔照片或標誌旁邊的核取方塊，將使用者新增至 [已選取] 清單。

12. **選擇性︰** 如果您想要**新增多位使用者**，請在 [依姓名或電子郵件地址搜尋] 搜尋方塊中，輸入另一個**全名**或**電子郵件地址**，然後按一下核取方塊，將此使用者新增至 [已選取] 清單。

13. 當您完成選取使用者時，按一下 [選取] 按鈕，將他們新增到要指派至應用程式的使用者和群組清單。

14. **選擇性︰** 按一下 [新增指派] 窗格中的 [選取角色] 選取器，以選取要指派給您已選取之使用者的角色。

15. 按一下 [指派] 按鈕，將應用程式指派給選取的使用者。

稍待片刻，您已選取的使用者便能在存取面板中啟動這些應用程式。

## <a name="if-these-troubleshoot-steps-dont-resolve-the-issue"></a>如果這些疑難排解步驟無法解決問題 
使用下列資訊 (若有的話) 開啟支援票證︰

-   相互關聯錯誤 ID

-   UPN (使用者電子郵件地址)

-   TenantID

-   瀏覽器類型

-   錯誤發生期間的時區和時間/時間範圍

-   Fiddler 追蹤

## <a name="next-steps"></a>後續步驟
[使用應用程式 Proxy 提供單一登入應用程式](application-proxy-configure-single-sign-on-with-kcd.md)
