---
title: 教學課程︰以 Azure Active Directory 設定 Zendesk 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 來對 Zendesk 自動佈建及取消佈建使用者帳戶。
services: active-directory
documentationcenter: ''
author: zhchia
writer: zhchia
manager: beatrizd-msft
ms.assetid: na
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/31/2018
ms.author: v-ant
ms.openlocfilehash: 2dc965547511d27ed43a88c1f45b50593b30a937
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44347931"
---
# <a name="tutorial-configure-zendesk-for-automatic-user-provisioning"></a>教學課程︰設定 Zendesk 來自動佈建使用者

本教學課程旨在示範將 Azure AD 設定為可對 Zendesk 自動佈建及取消佈建使用者和/或群組時，Zendesk 與 Azure Active Directory (Azure AD) 中須執行的步驟。 

> [!NOTE]
> 本教學課程會說明建置在 Azure AD 使用者佈建服務之上的連接器。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](../manage-apps/user-provisioning.md)。

## <a name="prerequisites"></a>必要條件

本教學課程中概述的案例假設您已經具有下列必要條件：

*   Azure AD 租用戶
*   已啟用 [Enterprise 方案](https://www.zendesk.com/product/pricing/)或更好方案的 Zendesk 租用戶 
*   Zendesk 中具有系統管理員權限的使用者帳戶 

> [!NOTE]
> Azure AD 佈建整合仰賴 [Zendesk REST API](https://developer.zendesk.com/rest_api/docs/core/introduction)，其可供 Zendesk 小組在 Enterprise 方案或更好方案上使用。

## <a name="adding-zendesk-from-the-gallery"></a>從資源庫新增 Zendesk
將 Zendesk 設定為可使用 Azure AD 自動佈建使用者之前，您必須將 Zendesk 從 Azure AD 應用程式庫新增至您的受控 SaaS 應用程式清單。

**若要從 Azure AD 應用程式庫新增 Zendesk，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式] > [所有應用程式]。

    ![企業應用程式區段][2]
    
3. 若要新增 Zendesk，按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Zendesk**。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk6.png)

5. 在結果窗格中，選取 [Zendesk]，然後按一下 [新增] 按鈕將 Zendesk 新增至您的 SaaS 應用程式清單。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk7.png)

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk20.png)

## <a name="assigning-users-to-zendesk"></a>將使用者指派給 Zendesk

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者佈建的內容中，只有「已指派」至 Azure AD 中的應用程式之使用者和/或群組會進行同步處理。 

設定並啟用自動使用者佈建之前，您應先決定 Azure AD 中的哪些使用者和/或群組需要存取 Zendesk。 一旦決定後，您可以依照此處的指示，將這些使用者和/或群組指派給 Zendesk：

*   [將使用者或群組指派給企業應用程式](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-zendesk"></a>將使用者指派給 Zendesk 的重要秘訣

*   建議將單一 Azure AD 使用者指派給 Zendesk，以測試自動使用者佈建的設定。 其他使用者及/或群組可能會稍後再指派。

*   將使用者指派給 Zendesk 時，您必須在 [指派] 對話方塊中選取任何有效的應用程式特有角色 (如果有的話)。 具有**預設存取**角色的使用者會從佈建中排除。

## <a name="configuring-automatic-user-provisioning-to-zendesk"></a>對 Zendesk 設定自動使用者佈建 

本節將引導您逐步設定 Azure AD 佈建服務，以根據 Azure AD 中的使用者和/或群組指派，在 Zendesk 中建立、更新和停用使用者和/或群組。

> [!TIP]
> 您也可以選擇為 Zendesk 啟用 SAML 型單一登入，請遵循 [Zendesk 單一登入教學課程](zendesk-tutorial.md)中提供的指示。 雖然自動使用者佈建和單一登入這兩個功能互相補充，您還是可以將它們分開設定。

### <a name="to-configure-automatic-user-provisioning-for-zendesk-in-azure-ad"></a>在 Azure AD 中為 Zendesk 設定自動使用者佈建：

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後瀏覽至 [Azure Active Directory] > [企業應用程式] > [所有應用程式]。

2. 從您的 SaaS 應用程式清單中選取 Zendesk。
 
    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk3.png)

3. 選取 [佈建] 索引標籤。
    
    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk16.png)

4. 將 [佈建模式] 設定為 [自動]。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk1.png)

5. 在 [管理員認證] 區段下，輸入 Zendesk 帳戶的 [管理員使用者名稱]、[祕密權杖] 和 [網域]。 這些值的範例如下：

    *   在 [管理員使用者名稱] 欄位中，填入 Zendesk 租用戶中管理帳戶的使用者名稱。 範例： admin@contoso.com.

    *   在 [秘密權杖] 欄位中，填入秘密權杖，如同步驟 6 所述。

    *   在 [網域] 欄位中，填入您的 Zendesk 租用戶子網域。
    範例：針對租用戶 URL 為 https://my-tenant.zendesk.com 的帳戶，您的子網域會是 **my-tenant**。

6. 您的 Zendesk 帳戶的 [秘密權杖] 位於 [網域 > API > 設定]。 

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk4.png) ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk2.png)

7. 如步驟 5 所示填寫欄位後，按一下 [測試連線] 以確認 Azure AD 可以連線到 Zendesk。 如果連線失敗，請確定您的 Zendesk 帳戶具有系統管理員權限並再試一次。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk19.png)
    
8. 在 [通知電子郵件] 欄位中，輸入應該收到佈建錯誤通知的個人或群組電子郵件地址，然後選取 [發生失敗時傳送電子郵件通知] 核取方塊。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk9.png)

9. 按一下 [檔案] 。

10. 在 [對應] 區段中，選取 [同步處理 Azure Active Directory 使用者至 Zendesk]。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk10.png)

11. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 Zendesk 的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 Zendesk 中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可所有變更。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk11.png)

12. 在 [對應] 區段中，選取 [同步處理 Azure Active Directory 群組至 Zendesk]。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk12.png)

13. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 Zendesk 的群組屬性。 選取為 [比對] 屬性的屬性會用來比對 Zendesk 中的群組以進行更新作業。 選取 [儲存] 按鈕以認可所有變更。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk13.png)

14. 若要設定範圍篩選，請參閱[範圍篩選教學課程](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md)中提供的下列指示。

15. 若要啟用 Zendesk 的 Azure AD 佈建服務，在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk14.png)

16. 透過在 [設定] 區段的 [範圍] 中選擇需要的值，可定義要佈建到 Zendesk 的使用者和/或群組。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk15.png)

17. 當您準備好要佈建時，按一下 [儲存]。

    ![Zendesk 佈建](./media/zendesk-provisioning-tutorial/ZenDesk18.png)


此作業會對在 [設定] 區段的 [範圍] 中定義的所有使用者和/或群組，啟動首次同步處理。 初始同步處理會比後續同步處理花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。 您可以使用 [同步處理詳細資料] 區段來監視進度，並遵循連結來佈建活動報告，當中會描述 Zendesk 上的 Azure AD 佈建服務所執行之所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../manage-apps/check-status-user-account-provisioning.md)。

## <a name="connector-limitations"></a>連接器限制
* Zendesk 僅支援具備代理程式角色的使用者群組使用。 如需詳細資訊，請參閱 [Zendesk 文件](https://support.zendesk.com/hc/en-us/articles/203661966-Creating-managing-and-using-groups) \(英文\)。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>後續步驟

* [瞭解如何針對佈建活動檢閱記錄和取得報告](../manage-apps/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/zendesk-tutorial/tutorial_general_01.png
[2]: ./media/zendesk-tutorial/tutorial_general_02.png
[3]: ./media/zendesk-tutorial/tutorial_general_03.png
