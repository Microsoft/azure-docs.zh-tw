---
title: 教學課程：以 Azure Active Directory 設定 Zscaler Three 來自動佈建使用者 | Microsoft Docs
description: 在本教學課程中，您將了解如何透過設定，來使 Azure Active Directory 自動對 Zscaler Three 進行或取消使用者帳戶的佈建。
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: c16c02a870edb1a777b63da6fea57fc02136d643
ms.sourcegitcommit: 2aa52d30e7b733616d6d92633436e499fbe8b069
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/06/2021
ms.locfileid: "97936595"
---
# <a name="tutorial-configure-zscaler-three-for-automatic-user-provisioning"></a>教學課程：設定 Zscaler Three 來自動佈建使用者

在本教學課程中，您將了解如何透過設定，來使 Azure Active Directory (Azure AD) 自動對 Zscaler Three 進行和取消使用者和/或群組的佈建。

> [!NOTE]
> 此教學課程會描述建置在 Azure AD 使用者佈建服務之上的連接器。 如需此服務的用途、運作方式和常見問題答案等重要詳細資訊，請參閱[使用 Azure Active Directory 自動對 SaaS 應用程式進行和取消使用者的佈建](../app-provisioning/user-provisioning.md)。

## <a name="prerequisites"></a>Prerequisites

若要完成此教學課程中概述的步驟，您需要下列項目：

* Azure AD 租用戶。
* Zscaler Three 租用戶。
* Zscaler Three 中具有系統管理員權限的使用者帳戶。

> [!NOTE]
> Azure AD 佈建整合需仰賴可供 Enterprise 帳戶使用的 Zscaler ZSCloud SCIM API。

## <a name="adding-zscaler-three-from-the-gallery"></a>從資源庫新增 Zscaler Three

將 Zscaler Three 設定為可使用 Azure AD 自動佈建使用者之前，您必須將 Zscaler Three 從 Azure AD 應用程式庫新增至您的受控 SaaS 應用程式清單。

在 [Azure 入口網站](https://portal.azure.com)的左窗格中，選取 [Azure Active Directory]  ：

![選取 Azure Active Directory](common/select-azuread.png)

移至 [企業應用程式]  ，然後選取 [所有應用程式]  ：

![企業應用程式](common/enterprise-applications.png)

若要新增應用程式，請選取視窗頂端的 [新增應用程式]  ：

![選取 [新增應用程式]](common/add-new-app.png)

在搜尋方塊中，輸入 **Zscaler Three**。 在結果中選取 [Zscaler Three]  ，然後選取 [新增]  。

![結果清單](common/search-new-app.png)

## <a name="assign-users-to-zscaler-three"></a>將使用者指派給 Zscaler Three

Azure AD 使用者必須先獲得所選應用程式的存取權，才能使用它們。 在自動使用者佈建的內容中，只會同步處理已指派至 Azure AD 中應用程式的使用者或群組。

設定並啟用自動使用者佈建之前，您應該先決定 Azure AD 中的哪些使用者和/或群組需要存取 Zscaler Three。 做好決定之後，您可依照[將使用者或群組指派給企業應用程式](../manage-apps/assign-user-or-group-access-portal.md)中的指示，將這些使用者和群組指派至 Zscaler Three。

### <a name="important-tips-for-assigning-users-to-zscaler-three"></a>將使用者指派給 Zscaler Three 的重要秘訣

* 建議先將單一 Azure AD 使用者指派至 Zscaler Three，以測試自動使用者佈建組態。 您可以稍後指派更多的使用者和群組。

* 將使用者指派至 Zscaler Three 時，您必須在 [指派] 對話方塊中選取任何有效的應用程式特有角色 (如果有的話)。 具有 **預設存取** 角色的使用者會從佈建中排除。

## <a name="set-up-automatic-user-provisioning"></a>設定自動使用者佈建

本節將引導您逐步設定 Azure AD 佈建服務，以根據 Azure AD 中的使用者和群組指派，在 Zscaler Three 中建立、更新和停用使用者和群組。

> [!TIP]
> 您也可以為 Zscaler Three 啟用SAML 型單一登入。 如果您這麼做，請依照 [Zscaler Three 單一登入教學課程](zscaler-three-tutorial.md)中的指示操作。 單一登入可以與自動使用者佈建分開設定，但這兩個功能彼此補充。

> [!NOTE]
> 在佈建或取消佈建使用者和群組時，建議您定期重新啟動佈建，以確保群組成員資格會正確更新。 重新啟動會強制讓我們的服務重新評估所有群組，並更新成員資格。 

1. 登入 [Azure 入口網站](https://portal.azure.com)設定，然後選取 [企業應用程式]   > [所有應用程式]   > [Zscaler Three]  ：

    ![企業應用程式](common/enterprise-applications.png)

2. 在應用程式清單中，選取 [Zscaler Three]  ：

    ![應用程式清單](common/all-applications.png)

3. 選取 [佈建]  索引標籤：

    ![Zscaler Three 佈建](./media/zscaler-three-provisioning-tutorial/provisioning-tab.png)

4. 將 [佈建模式]  設定為 [自動]  ：

    ![設定佈建模式](./media/zscaler-three-provisioning-tutorial/provisioning-credentials.png)

5. 在 [管理員認證]  區段中，輸入 Zscaler Three 帳戶的 [租用戶 URL]  和 [祕密權杖]  ，如下一個步驟所述。

6. 若要取得 [租用戶 URL]  和 [祕密權杖]  ，請移至 Zscaler Three 中入口網站中的 [管理]   > [驗證設定]  ，然後選取 [驗證類型]  下方的 [SAML]  ：

    ![Zscaler Three 驗證設定](./media/zscaler-three-provisioning-tutorial/secret-token-1.png)

    選取 [設定 SAML]  來開啟 [設定 SAML]  視窗：

    ![[設定 SAML] 視窗](./media/zscaler-three-provisioning-tutorial/secret-token-2.png)

    選取 [啟用 SCIM 型佈建]  並複製 [基底 URL]  和 [持有人權杖]  ，然後儲存設定。 在 Azure 入口網站中，將 [基底 URL]  貼上到 [租用戶 URL]  方塊中，以及將 [持有人權杖]  貼到 [祕密權杖]  方塊中。

7. 在 [租用戶 URL]  和 [祕密權杖]  方塊中輸入值之後，請選取 [測試連線]  以確定 Azure AD 可以連線到 Zscaler Three。 如果連線失敗，請確定您的 Zscaler Three 帳戶具有系統管理員權限並再試一次。

    ![測試連線](./media/zscaler-three-provisioning-tutorial/test-connection.png)

8. 在 [通知電子郵件]  方塊中，輸入應收到佈建錯誤通知的個人或群組電子郵件地址。 選取 [發生失敗時傳送電子郵件通知]  ：

    ![設定通知電子郵件](./media/zscaler-three-provisioning-tutorial/notification.png)

9. 選取 [儲存]  。

10. 在 [對應]  區段中，選取 [將 Azure Active Directory 使用者同步至 ZscalerThree]  ：

    ![同步處理 Azure AD 使用者](./media/zscaler-three-provisioning-tutorial/user-mappings.png)

11. 在 [屬性對應]  區段中，檢閱從 Azure AD 同步至 Zscaler Three 的使用者屬性。 選取為 [比對]  屬性的屬性會用來比對 Zscaler Three 中的使用者帳戶以進行更新作業。 選取 [儲存]  認可任何變更。

    ![[屬性對應] 區段的螢幕擷取畫面，其中顯示七個對應。](./media/zscaler-three-provisioning-tutorial/user-attribute-mappings.png)

12. 在 [對應]  區段中，選取 [將 Azure Active Directory 群組同步至 ZscalerThree]  ：

    ![同步處理 Azure AD 群組](./media/zscaler-three-provisioning-tutorial/group-mappings.png)

13. 在 [屬性對應]  區段中，檢閱從 Azure AD 同步至 Zscaler Three 的群組屬性。 選取為 [比對]  屬性的屬性會用來比對 Zscaler Three 中的群組以進行更新作業。 選取 [儲存]  認可任何變更。

    ![[屬性對應] 區段的螢幕擷取畫面，其中顯示三個對應。](./media/zscaler-three-provisioning-tutorial/group-attribute-mappings.png)

14. 若要設定範圍篩選條件，請參閱[範圍篩選條件教學課程](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)中的指示。

15. 若要啟用 Zscaler Three 的 Azure AD 佈建服務，在 [設定]  區段中，將 [佈建狀態]  變更為 [開啟]  ：

    ![佈建狀態](./media/zscaler-three-provisioning-tutorial/provisioning-status.png)

16. 藉由在 [設定]  區段的 [範圍]  中選擇您要的值，定義要佈建到 Zscaler Three 的使用者和/或群組：

    ![範圍值](./media/zscaler-three-provisioning-tutorial/scoping.png)

17. 當您準備好要佈建時，請選取 [儲存]  ：

    ![選取 [儲存]](./media/zscaler-three-provisioning-tutorial/save-provisioning.png)

此作業會對在 [設定]  區段的 [範圍]  下定義的所有使用者和群組，啟動首次同步處理。 初始同步處理會比後續同步處理花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。 您可以在 [同步處理詳細資料]  區段中監視進度。 您也可以遵循連結來佈建活動報告，該報告會說明 Zscaler Three 上的 Azure AD 佈建服務所執行的所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../app-provisioning/check-status-user-account-provisioning.md)。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>後續步驟

* [瞭解如何針對佈建活動檢閱記錄和取得報告](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/zscaler-three-provisioning-tutorial/tutorial-general-01.png
[2]: ./media/zscaler-three-provisioning-tutorial/tutorial-general-02.png
[3]: ./media/zscaler-three-provisioning-tutorial/tutorial-general-03.png