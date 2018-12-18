---
title: 教學課程︰以 Azure Active Directory 設定 BlueJeans 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 將使用者帳戶自動佈建和取消佈建至BlueJeans。
services: active-directory
documentationcenter: ''
author: zhchia
writer: zhchia
manager: beatrizd-msft
ms.assetid: d4ca2365-6729-48f7-bb7f-c0f5ffe740a3
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/01/2018
ms.author: v-ant
ms.openlocfilehash: d4fe2d14485207aae89c5f82d6c8eaf3ada7f28d
ms.sourcegitcommit: 16ddc345abd6e10a7a3714f12780958f60d339b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36226525"
---
# <a name="tutorial-configure-bluejeans-for-automatic-user-provisioning"></a>教學課程︰設定 BlueJeans 來自動佈建使用者

本教學課程旨在示範將 Azure AD 設定為可對 BlueJeans 自動佈建及取消佈建使用者和/或群組時，BlueJeans 與 Azure Active Directory (Azure AD) 中須執行的步驟。

> [!NOTE]
> 本教學課程會說明建置在 Azure AD 使用者佈建服務之上的連接器。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](./../active-directory-saas-app-provisioning.md)。

## <a name="prerequisites"></a>先決條件

本教學課程中說明的案例假設您已經具有下列項目：

*   Azure AD 租用戶
*   已啟用 [My Company](https://www.BlueJeans.com/pricing) 方案或更高方案的 BlueJeans 租用戶
*   BlueJeans 中具有管理員權限的使用者帳戶

> [!NOTE]
> Azure AD 佈建整合仰賴 [BlueJeans API](https://BlueJeans.github.io/developer)，此 API 僅提供給標準方案或更高方案中的 BlueJeans 小組使用。

## <a name="adding-bluejeans-from-the-gallery"></a>從資源庫新增 BlueJeans
將 BlueJeans 設定為可使用 Azure AD 自動佈建使用者之前，您必須將 BlueJeans 從 Azure AD 應用程式庫新增至您的受控 SaaS 應用程式清單。

**若要從 Azure AD 應用程式庫新增 BlueJeans，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式] > [所有應用程式]。

    ![企業應用程式區段][2]
    
3. 若要新增 BlueJeans，按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **BlueJeans**。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/BluejeansAppSearch.png)

5. 在結果窗格中，選取 [BlueJeans]，然後按一下 [新增] 按鈕將 BlueJeans 新增至您的 SaaS 應用程式清單。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/BluejeansAppSearchResults.png)

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/BluejeansAppCreate.png)
    
## <a name="assigning-users-to-bluejeans"></a>將使用者指派給 BlueJeans

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者佈建的內容中，只有「已指派」至 Azure AD 中的應用程式之使用者和/或群組會進行同步處理。

設定並啟用自動使用者佈建之前，您應先決定 Azure AD 中的哪些使用者和/或群組需要存取 BlueJeans。 一旦決定後，您可以依照此處的指示，將這些使用者和/或群組指派給 BlueJeans：

*   [將使用者或群組指派給企業應用程式](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-bluejeans"></a>將使用者指派給 BlueJeans 的重要秘訣

*   建議將單一 Azure AD 使用者指派給 BlueJeans，以測試自動使用者佈建的設定。 其他使用者及/或群組可能會稍後再指派。

*   將使用者指派給 BlueJeans 時，您必須在 [指派] 對話方塊中選取任何有效的應用程式特有角色 (如果有的話)。 具有**預設存取**角色的使用者會從佈建中排除。

## <a name="configuring-automatic-user-provisioning-to-bluejeans"></a>對 BlueJeans 設定自動使用者佈建

本節將引導您逐步設定 Azure AD 佈建服務，以根據 Azure AD 中的使用者和/或群組指派，在 BlueJeans 中建立、更新和停用使用者和/或群組。

> [!TIP]
> 您也可以選擇為 BlueJeans 啟用 SAML 型單一登入，請遵循 [BlueJeans 單一登入教學課程](bluejeans-tutorial.md)中提供的指示。 雖然自動使用者佈建和單一登入這兩個功能彼此補充，您可以分開設定它們。

### <a name="to-configure-automatic-user-provisioning-for-bluejeans-in-azure-ad"></a>在 Azure AD 中為 BlueJeans 設定自動使用者佈建：

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後瀏覽至 [Azure Active Directory] > [企業應用程式] > [所有應用程式]。

2. 從您的 SaaS 應用程式清單中選取 BlueJeans。
 
    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/Bluejeans2.png)

3. 選取 [佈建] 索引標籤。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/BluejeansProvisioningTab.png)

4. 將 [佈建模式] 設定為 [自動]。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/Bluejeans1.png)

5. 在 [管理員認證] 區段下，輸入 BlueJeans 帳戶的 [管理員使用者名稱] 和 [管理員密碼]。 這些值的範例如下：

    *   在 [管理員使用者名稱] 欄位中，填入 BlueJeans 租用戶中管理帳戶的使用者名稱。 範例： admin@contoso.com.

    *   在 [管理員密碼] 欄位中，填入與管理員使用者名稱相對應的密碼。

6. 如步驟 5 所示填寫欄位後，按一下 [測試連線] 以確認 Azure AD 可以連線到 BlueJeans。 如果連線失敗，請確定您的 BlueJeans 帳戶具有系統管理員權限並再試一次。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/BluejeansTestConnection.png)

7. 在 [通知電子郵件] 欄位中，輸入應該收到佈建錯誤通知的個人或群組電子郵件地址，然後選取 [發生失敗時傳送電子郵件通知] 核取方塊。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/BluejeansNotificationEmail.png)

8. 按一下 [檔案] 。

9. 在 [對應] 區段中，選取 [同步處理 Azure Active Directory 使用者至 BlueJeans]。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/BluejeansMapping.png)

10. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 BlueJeans 的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 BlueJeans 中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可所有變更。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/BluejeansUserMappingAtrributes.png)

11. 若要設定範圍篩選，請參閱[範圍篩選教學課程](./../active-directory-saas-scoping-filters.md)中提供的下列指示。

12. 若要啟用 BlueJeans 的 Azure AD 佈建服務，在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/BluejeansProvisioningStatus.png)

13. 透過在 [設定] 區段的 [範圍] 中選擇需要的值，可定義要佈建到 BlueJeans 的使用者和/或群組。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/UserGroupSelection.png)

14. 當您準備好要佈建時，按一下 [儲存]。

    ![BlueJeans 佈建](./media/bluejeans-provisioning-tutorial/SaveProvisioning.png)

此作業會對在 [設定] 區段的 [範圍] 中定義的所有使用者和/或群組，啟動首次同步處理。 初始同步處理會比後續同步處理花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。 您可以使用 [同步處理詳細資料] 區段來監視進度，並遵循連結來佈建活動報告，當中會描述 BlueJeans 上的 Azure AD 佈建服務所執行之所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../active-directory-saas-provisioning-reporting.md)。

## <a name="connector-limitations"></a>連接器限制

* Bluejeans 不允許超過 30 個字元的使用者名稱。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>後續步驟

* [瞭解如何針對佈建活動檢閱記錄和取得報告](../active-directory-saas-provisioning-reporting.md)

<!--Image references-->
[1]: ./media/bluejeans-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/bluejeans-tutorial/tutorial_general_02.png
[3]: ./media/bluejeans-tutorial/tutorial_general_03.png
