---
title: 教學課程︰以 Azure Active Directory 設定 Tableau Online 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 來自動佈建並取消佈建使用者帳戶至 Tableau Online。
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd-msft
ms.assetid: na
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/30/2018
ms.author: v-wingf-msft
ms.openlocfilehash: bbee7f0e6022e2945563c102a851819734d52e77
ms.sourcegitcommit: 7b845d3b9a5a4487d5df89906cc5d5bbdb0507c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "40107018"
---
# <a name="tutorial-configure-tableau-online-for-automatic-user-provisioning"></a>教學課程︰設定 Tableau Online 來自動佈建使用者

本教學課程旨在示範將 Azure AD 設定為可對 Tableau Online 自動佈建及取消佈建使用者和/或群組時，應在 Tableau Online 與 Azure Active Directory (Azure AD) 中執行的步驟。

> [!NOTE]
> 本教學課程會說明建置在 Azure AD 使用者佈建服務之上的連接器。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](../active-directory-saas-app-provisioning.md)。

## <a name="prerequisites"></a>必要條件

本教學課程中說明的案例假設您已經具有下列項目：

*   Azure AD 租用戶
*   [Tableau Online 租用戶](https://www.tableau.com/)
*   Tableau Online 中具有管理員權限的使用者帳戶

> [!NOTE]
> Azure AD 佈建整合需仰賴可供 Tableau Online 開發人員使用的 [Tableau Online Rest API](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm)。

## <a name="adding-tableau-online-from-the-gallery"></a>從資源庫新增 Tableau Online
將 Tableau Online 設定為可使用 Azure AD 自動佈建使用者之前，您必須從 Azure AD 應用程式庫將 Tableau Online 新增至您的受控 SaaS 應用程式清單。

**若要從 Azure AD 應用程式庫新增 Tableau Online，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式] > [所有應用程式]。

    ![企業應用程式區段][2]

3. 若要新增 Tableau Online，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Tableau Online**。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/AppSearch.png)

5. 在結果窗格中，選取 [Tableau Online]，然後按一下 [新增] 按鈕，將 Tableau Online 新增至您的 SaaS 應用程式清單。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/AppSearchResults.png)

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/AppCreation.png)

## <a name="assigning-users-to-tableau-online"></a>將使用者指派給 Tableau Online

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者佈建的內容中，只有「已指派」至 Azure AD 中的應用程式之使用者和/或群組會進行同步處理。

設定並啟用自動使用者佈建之前，您應先決定 Azure AD 中的哪些使用者和/或群組需要存取 Tableau Online。 一旦決定後，您可以依照此處的指示，將這些使用者和/或群組指派給 Tableau Online：

*   [將使用者或群組指派給企業應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

### <a name="important-tips-for-assigning-users-to-tableau-online"></a>將使用者指派給 Tableau Online 的重要秘訣

*   建議將單一 Azure AD 使用者指派給 Tableau Online，以測試自動使用者佈建的設定。 其他使用者及/或群組可能會稍後再指派。

*   將使用者指派給 Tableau Online 時，您必須在 [指派] 對話方塊中選取任何有效應用程式特有的角色 (如果有的話)。 具有**預設存取**角色的使用者會從佈建中排除。

## <a name="configuring-automatic-user-provisioning-to-tableau-online"></a>對 Tableau Online 設定自動使用者佈建

本節將引導您逐步設定 Azure AD 佈建服務，以根據 Azure AD 中的使用者和/或群組指派，在 Tableau Online 中建立、更新和停用使用者和/或群組。

> [!TIP]
> 您也可以選擇為 Tableau Online 啟用 SAML 型單一登入，請遵循 [Tableau Online 單一登入教學課程](tableauonline-tutorial.md)中提供的指示。 雖然自動使用者佈建和單一登入這兩個功能互相補充，您還是可以將它們分開設定。

### <a name="to-configure-automatic-user-provisioning-for-tableau-online-in-azure-ad"></a>若要在 Azure AD 中為 Tableau Online 設定自動使用者佈建：

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後瀏覽至 [Azure Active Directory] > [企業應用程式] > [所有應用程式]。

2. 從您的 SaaS 應用程式清單中選取 Tableau Online。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/AppInstanceSearch.png)

3. 選取 [佈建] 索引標籤。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/ProvisioningTab.png)

4. 將 [佈建模式] 設定為 [自動]。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/ProvisioningCredentials.png)

5. 在 [管理員認證] 區段下，輸入 Tableau Online 帳戶的 [網域]、[管理員使用者名稱]、[管理員密碼]和 [內容 URL]：

    *   在 [網域] 欄位中，根據步驟 6 填入子網域。

    *   在 [管理員使用者名稱] 欄位中，填入 Clarizen 租用戶中系統管理員帳戶的使用者名稱。 範例： admin@contoso.com.

    *   在 [管理員密碼] 欄位中，填入與管理員使用者名稱相對應的系統管理員帳戶密碼。

    *   在 [內容 URL] 欄位中，根據步驟 6 填入子網域。

6. 登入 Tableau Online 的系統管理帳戶之後，即可從系統管理頁面的 URL 中擷取 [網域] 和 [內容 URL] 的值。

    *   Tableau Online 帳戶的**網域**可以從 URL 的此部分複製：![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/DomainUrlPart.png)

    *   Tableau Online 帳戶的**內容 URL**可從此區段複製，這是在帳戶設定期間所定義的值。 在此範例中，此值為 "contoso"：![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/ContentUrlPart.png)

        > [!NOTE]
        > 您的**網域**可能會與此處顯示的不一樣。 


7. 如步驟 5 所示填寫欄位後，按一下 [測試連線] 以確認 Azure AD 可以連線到 Tableau Online。 如果連線失敗，請確定您的 Tableau Online 帳戶具有管理員權限，然後再試一次。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/TestConnection.png)

8. 在 [通知電子郵件] 欄位中，輸入應收到佈建錯誤通知的個人或群組之電子郵件地址，然後勾選 [發生失敗時傳送電子郵件通知] 核取方塊。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/EmailNotification.png)

10. 按一下 [檔案] 。

11. 在 [對應] 區段中，選取 [將 Azure Active Directory 使用者同步至 Tableau]。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/UserMappings.png)

12. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步處理至 Tableau Online 的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 Tableau Online 中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可所有變更。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/UserAttributeMapping.png)

13. 在 [對應] 區段中，選取 [同步處理 Azure Active Directory 群組至 Tableau]。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/GroupMappings.png)

14. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步處理至 Tableau Online 的群組屬性。 選取為 [比對] 屬性的屬性會用來比對 Tableau Online 中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可所有變更。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/GroupAttributeMapping.png)

15. 若要設定範圍篩選，請參閱[範圍篩選教學課程](../active-directory-saas-scoping-filters.md)中提供的下列指示。

16. 若要啟用 Tableau Online 的 Azure AD 佈建服務，在 [設定]區段中，將 [佈建狀態] 變更為 [開啟]。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/ProvisioningStatus.png)

17. 透過在 [設定] 區段的 [範圍] 中選擇需要的值，可定義要佈建到 Tableau Online 的使用者和/或群組。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/ScopeSync.png)

18. 當您準備好要佈建時，按一下 [儲存]。

    ![Tableau Online 佈建](./media/tableau-online-provisioning-tutorial/SaveProvisioning.png)

此作業會對在 [設定] 區段的 [範圍] 中定義的所有使用者和/或群組，啟動首次同步處理。 初始同步處理會比後續同步處理花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。 您可以使用 [同步處理詳細資料] 區段來監視進度，並遵循連結來佈建活動報告；當中會描述 Tableau Online 上 Azure AD 佈建服務所執行的所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../active-directory-saas-provisioning-reporting.md)。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)


## <a name="next-steps"></a>後續步驟

* [瞭解如何針對佈建活動檢閱記錄和取得報告](../active-directory-saas-provisioning-reporting.md)

<!--Image references-->
[1]: ./media/tableau-online-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/tableau-online-provisioning-tutorial/tutorial_general_02.png
[3]: ./media/tableau-online-provisioning-tutorial/tutorial_general_03.png
