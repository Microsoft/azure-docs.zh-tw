---
title: 快速入門：在 Azure 入口網站中新增來賓使用者
description: 使用此快速入門深入了解 Azure AD 系統管理員如何在 Azure 入口網站中新增 B2B 來賓使用者，並逐步說明 B2B 邀請工作流程。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: quickstart
ms.date: 07/02/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: ed49eed6dbfce9a71f13770e3ddcaec6557cf875
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45986540"
---
# <a name="quickstart-add-guest-users-to-your-directory-in-the-azure-portal"></a>快速入門：在 Azure 入口網站中將來賓使用者新增到您的目錄

您可以透過將任何人作為來賓使用者新增到您的目錄，來邀請他們與您的組織合作。 接著，您可以傳送包含兌換連結的邀請電子郵件，也可以傳送您要共用之應用程式的直接連結。 來賓使用者能以自己的公司、學校或社交身分識別登入。

在此快速入門中，您會將新的來賓使用者新增到 Azure AD、傳送邀請，並查看來賓使用者的邀請兌換程序。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程中的案例，您需要：

 - 允許您在租用戶目錄中建立使用者的角色，例如全域系統管理員角色，或任何受限的系統管理員目錄角色。
 - 您可以新增到租用戶目錄並用來接收測試邀請電子郵件的有效電子郵件帳戶。

## <a name="add-a-new-guest-user-in-azure-ad"></a>在 Azure AD 中新增來賓使用者

1. 以 Azure AD 系統管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在左窗格中，選取 [Azure Active Directory]。
3.  在 [管理] 底下選取 [使用者]。

    ![選取 Azure Active Directory](media/quickstart-add-users-portal/quickstart-users-portal-user.png)

4.  選取 [新增來賓使用者]。

    ![選取 Azure Active Directory](media/quickstart-add-users-portal/quickstart-users-portal-user-3.png)

5.  在 [使用者名稱] 下，輸入外部使用者的電子郵件地址。 在 [包含個人訊息與邀請] 下，輸入歡迎使用訊息。 

    ![選取 Azure Active Directory](media/quickstart-add-users-portal/quickstart-users-portal-user-4.png)

6. 選取 [邀請] 即可自動對來賓使用者傳送邀請。 右上角會顯示一則通知，其中顯示訊息 [已成功邀請使用者]。 
7.  在傳送邀請之後，系統就會自動將使用者帳戶以來賓身分新增至目錄。

## <a name="assign-an-app-to-the-guest-user"></a>將應用程式指派給來賓使用者
將 Salesforce 應用程式新增到您的測試租用戶，並將測試來賓使用者指派給應用程式。
1.  以 Azure AD 系統管理員身分登入 Azure 入口網站。
2.  在左側窗格中，選取 [企業應用程式]。
3.  選取 [新增應用程式]。
4. 在 [從資源庫新增] 底下，搜尋 **Salesforce**，然後選取它。

    ![選取 Azure Active Directory](media/quickstart-add-users-portal/quickstart-users-portal-select-salesforce.png)
5. 選取 [新增]。
6. 在 [管理] 下，選取 [單一登入]，然後在 [單一登入模式] 下，選取 [密碼登入]，然後按一下 [儲存]。
7. 在 [管理] 下選取 [使用者和群組] > [新增使用者] > [使用者和群組]。
8. 使用 [搜尋] 方塊來搜尋測試使用者 (如有必要)，並選取清單中的測試使用者。 接著，按一下 [選取]。
9. 選取 [指派]。 

## <a name="accept-the-invitation"></a>接受邀請
現在以來賓使用者的身分登入，以查看邀請。
1.  登入測試來賓使用者的電子郵件帳戶。
2.  在您的收件匣中，尋找「您已收到邀請」電子郵件。

    ![B2B 邀請電子郵件](media/quickstart-add-users-portal/quickstart-users-portal-email-small.png)

3.  在電子郵件內文中，選取 [開始使用]。 將在瀏覽器中開啟 [檢閱權限] 頁面。 

    ![B2B 接受邀請頁面](media/quickstart-add-users-portal/quickstart-users-portal-accept.png)

4. 選取 [接受]。 隨即開啟 [存取面板]，其中列出來賓使用者可以存取的應用程式。

## <a name="clean-up-resources"></a>清除資源
當您不再需要測試來賓使用者與測試應用程式時，可以將它們刪除。
1.  以 Azure AD 系統管理員身分登入 Azure 入口網站。
2.  在左窗格中，選取 [Azure Active Directory]。
3.  在 [管理] 下選取 [企業應用程式]。
4.  開啟應用程式 **Salesforce**，然後選取 [刪除]。
5.  在左窗格中，選取 [Azure Active Directory]。
6.  在 [管理] 底下選取 [使用者]。
7.  選取測試使用者，然後選取 [刪除使用者]。

## <a name="next-steps"></a>後續步驟
在此教學課程中，您可以在 Azure 入口網站中建立來賓使用者，並傳送一個共用應用程式的邀請。 接著，您從來賓使用者的觀點查看了兌換程序，並確認應用程式出現在來賓使用者的存取面板上。 若要深入了解如何新增共同作業的來賓使用者，請參閱[在 Azure 入口網站中新增 Azure Active Directory B2B 共同作業使用者](add-users-administrator.md)。