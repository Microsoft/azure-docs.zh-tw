---
title: 教學課程：以 Azure Active Directory 設定 Peakon 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 將使用者帳戶自動佈建及取消佈建至 Peakon。
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/28/2019
ms.author: zhchia
ms.openlocfilehash: c04bbd5459690262b484582e807569b965a0439b
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349883"
---
# <a name="tutorial-configure-peakon-for-automatic-user-provisioning"></a>教學課程：設定 Peakon 來自動佈建使用者

本教學課程旨在示範將 Azure AD 設定為可對 Peakon 自動佈建及取消佈建使用者和/或群組時，Peakon 與 Azure Active Directory (Azure AD) 中須執行的步驟。

> [!NOTE]
>  本教學課程會說明建置在 Azure AD 使用者佈建服務之上的連接器。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](../app-provisioning/user-provisioning.md)。
>
> 此連接器目前為預覽版。 如需有關預覽功能的一般 Microsoft Azure 使用規定詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用規定](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。
## <a name="prerequisites"></a>Prerequisites

本教學課程中概述的案例假設您已經具有下列必要條件

* Azure AD 租用戶。
* [Peakon 租用戶](https://peakon.com/us/pricing/)。
* Peakon 中具有管理員權限的使用者帳戶。

## <a name="assigning-users-to-peakon"></a>將使用者指派給 Peakon

Azure Active Directory 使用所謂「指派」的概念，決定應該授權哪些使用者存取選取的應用程式。 在自動使用者佈建的內容中，只有已指派至 Azure AD 中應用程式的使用者和/或群組會進行同步處理。

設定並啟用自動使用者佈建之前，您應先決定 Azure AD 中的哪些使用者和/或群組需要存取 Peakon。 一旦決定後，您可以依照此處的指示，將這些使用者和/或群組指派給 Peakon：

* [將使用者或群組指派給企業應用程式](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-peakon"></a>將使用者指派給 Peakon 的重要秘訣 

* 建議將單一 Azure AD 使用者指派給 Peakon，以測試自動使用者佈建的設定。 其他使用者及/或群組可能會稍後再指派。

* 將使用者指派給 Peakon 時，您必須在 [指派] 對話方塊中選取任何有效的應用程式特有角色 (如果有的話)。 具有 **預設存取** 角色的使用者會從佈建中排除。

## <a name="set-up-peakon-for-provisioning"></a>設定 Peakon 以進行佈建

1.  登入您的 [Peakon 管理主控台](https://app.Peakon.com/login)。 按一下 [組態]。 

    ![Peakon 管理主控台](media/Peakon-provisioning-tutorial/Peakon-admin-configuration.png)

2.  選取 [整合]。
    
    ![[設定] 選項的螢幕擷取畫面，其中已指出 [整合] 選項。](media/Peakon-provisioning-tutorial/Peakon-select-integration.png)

3.  啟用 [員工佈建]。

    ![[員工佈建] 區段的螢幕擷取畫面，其中已指出 [啟用] 選項。](media/Peakon-provisioning-tutorial/peakon05.png)

4.  複製 [SCIM 2.0 URL] 和 [OAuth 持有人權杖] 的值。 在 Azure 入口網站中，這些值會輸入至 Peakon 應用程式 [佈建] 索引標籤中的 [租用戶 URL] 和 [祕密權杖] 欄位。

    ![Peakon 建立權杖](media/Peakon-provisioning-tutorial/peakon04.png)

## <a name="add-peakon-from-the-gallery"></a>從資源庫新增 Peakon

若要以 Azure AD 設定 Peakon 來自動佈建使用者，您必須將 Peakon 從 Azure AD 應用程式庫新增至您的受控 SaaS 應用程式清單。

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，選取 [Azure Active Directory]。

    ![Azure Active Directory 按鈕](common/select-azuread.png)

2. 移至 [企業應用程式]，然後選取 [所有應用程式]。

    ![企業應用程式刀鋒視窗](common/enterprise-applications.png)

3. 若要新增新的應用程式，請選取窗格頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕](common/add-new-app.png)

4. 在搜尋方塊中輸入 **Peakon**，並從結果面板中選取 [Peakon]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Peakon](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-peakon"></a>設定將使用者自動佈建至 Peakon 

本節將引導您逐步設定 Azure AD 佈建服務，以根據 Azure AD 中的使用者和/或群組指派，在 Peakon 中建立、更新和停用使用者和/或群組。

> [!TIP]
> 建議您選擇為 Peakon 啟用 SAML 型單一登入，方法是遵循 [Peakon 單一登入教學課程](peakon-tutorial.md)中提供的指示。 雖然自動使用者佈建和單一登入這兩個功能互相補充，您還是可以將它們分開設定。

### <a name="to-configure-automatic-user-provisioning-for-peakon--in-azure-ad"></a>若要在 Azure AD 中為 Peakon 設定自動使用者佈建：

1. 登入 [Azure 入口網站](https://portal.azure.com)。 選取 [企業應用程式]，然後選取 [所有應用程式]。

    ![企業應用程式刀鋒視窗](common/enterprise-applications.png)

2. 在應用程式清單中，選取[Peakon]。

    ![應用程式清單中的 Peakon 連結](common/all-applications.png)

3. 選取 [佈建]  索引標籤。

    ![[管理] 選項的螢幕擷取畫面，並已指出 [佈建] 選項。](common/provisioning.png)

4. 將 [佈建模式]  設定為 [自動]  。

    ![[佈建模式] 下拉式清單的螢幕擷取畫面，並已指出 [自動] 選項。](common/provisioning-automatic.png)

5. 在 [管理員認證] 區段下，分別在 [租用戶 URL] 和 [秘密權杖] 中輸入稍早擷取到的 [SCIM 2.0 URL] 和 [OAuth 持有人權杖] 值。 按一下 [測試連線]，以確保 Azure AD 可以連線至 Peakon。 如果連線失敗，請確定您的 Peakon 帳戶具有管理員權限並再試一次。

    ![租用戶 URL + 權杖](common/provisioning-testconnection-tenanturltoken.png)

7. 在 [通知電子郵件]  欄位中，輸入應該收到佈建錯誤通知的個人或群組電子郵件地址，然後選取 [發生失敗時傳送電子郵件通知]  核取方塊。

    ![通知電子郵件](common/provisioning-notification-email.png)

8. 按一下 [檔案]  。

9. 在 [對應] 區段中，選取 [將 Azure Active Directory 使用者同步至 Peakon]。

    ![Peakon 使用者對應](media/Peakon-provisioning-tutorial/Peakon-user-mappings.png)

10. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 Peakon 的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 Peakon 中的使用者帳戶以進行更新作業。 選取 [儲存]  按鈕以認可所有變更。

    ![Peakon 使用者屬性](media/Peakon-provisioning-tutorial/Peakon-user-attributes.png)

12. 若要設定範圍篩選，請參閱[範圍篩選教學課程](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)中提供的下列指示。
    
    ![佈建範圍](common/provisioning-scope.png)

15. 當您準備好要佈建時，按一下 [儲存]  。

    ![儲存雲端佈建設定](common/provisioning-configuration-save.png)

此作業會對在 [設定]  區段的 [範圍]  中定義的所有使用者和/或群組，啟動首次同步處理。 初始同步處理會比後續同步處理花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。 您可以使用 [同步處理詳細資料] 區段來監視進度，並遵循連結來佈建活動報告，當中會描述 Peakon 上的 Azure AD 佈建服務所執行之所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../app-provisioning/check-status-user-account-provisioning.md)。

## <a name="connector-limitations"></a>連接器限制

* Peakon 中的所有自訂使用者屬性都必須延伸自 Peakon 的自訂 SCIM 使用者延伸 `urn:ietf:params:scim:schemas:extension:peakon:2.0:User`。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
## <a name="next-steps"></a>後續步驟

* [瞭解如何針對佈建活動檢閱記錄和取得報告](../app-provisioning/check-status-user-account-provisioning.md)