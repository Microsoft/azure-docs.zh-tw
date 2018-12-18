---
title: 教學課程︰以 Azure Active Directory 設定 Cornerstone OnDemand 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 以對 Cornerstone OnDemand 自動佈建及取消佈建使用者帳戶。
services: active-directory
documentationcenter: ''
author: zhchia
writer: zhchia
manager: beatrizd
ms.assetid: d4ca2365-6729-48f7-bb7f-c0f5ffe740a3
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/31/2018
ms.author: v-ant
ms.openlocfilehash: 6a6cfb2cb1fd6b70be0437c8b6fa62f50e76e53b
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44345408"
---
# <a name="tutorial-configure-cornerstone-ondemand-for-automatic-user-provisioning"></a>教學課程︰設定 Cornerstone OnDemand 來自動佈建使用者


本教學課程旨在示範將 Azure AD 設定為可對 Cornerstone OnDemand 自動佈建及取消佈建使用者和/或群組時，Cornerstone OnDemand 與 Azure Active Directory (Azure AD) 中須執行的步驟。


> [!NOTE]
> 本教學課程會說明建置在 Azure AD 使用者佈建服務之上的連接器。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](../manage-apps/user-provisioning.md)。

## <a name="prerequisites"></a>必要條件

本教學課程中概述的案例假設您已經具有下列必要條件：

*   Azure AD 租用戶
*   Cornerstone OnDemand 租用戶
*   Cornerstone OnDemand 中具有管理員權限的使用者帳戶


> [!NOTE]
> Azure AD 佈建整合依賴 [Cornerstone OnDemand Webservice](https://help.csod.com/help/csod_0/Content/Resources/Documents/WebServices/CSOD_-_Summary_of_Web_Services_v20151106.pdf) (僅提供給 Cornerstone OnDemand 小組使用)。

## <a name="adding-cornerstone-ondemand-from-the-gallery"></a>從資源庫新增 Cornerstone OnDemand
將 Cornerstone OnDemand 設定為可使用 Azure AD 自動佈建使用者之前，您必須將 Cornerstone OnDemand 從 Azure AD 應用程式庫新增至您的受控 SaaS 應用程式清單。

**若要從 Azure AD 應用程式庫新增 Cornerstone OnDemand，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式] > [所有應用程式]。

    ![企業應用程式區段][2]
    
3. 若要新增 Cornerstone OnDemand，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Cornerstone OnDemand**。

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/AppSearch.png)

5. 在結果窗格中，選取 [Cornerstone OnDemand]，然後按一下 [新增] 按鈕來將 Cornerstone OnDemand 新增至您的 SaaS 應用程式清單。

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/AppSearchResults.png)

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/AppCreation.png)

## <a name="assigning-users-to-cornerstone-ondemand"></a>將使用者指派給 Cornerstone OnDemand

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者佈建的內容中，只有「已指派」至 Azure AD 中的應用程式之使用者和/或群組會進行同步處理。 

設定並啟用自動使用者佈建之前，您應先決定 Azure AD 中的哪些使用者和/或群組需要存取 Cornerstone OnDemand。 一旦決定後，您可以依照此處的指示，將這些使用者和/或群組指派給 Cornerstone OnDemand：

*   [將使用者或群組指派給企業應用程式](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-cornerstone-ondemand"></a>將使用者指派給 Cornerstone OnDemand 的重要秘訣

*   建議將單一 Azure AD 使用者指派給 Cornerstone OnDemand，以測試自動使用者佈建的組態。 其他使用者及/或群組可能會稍後再指派。

*   將使用者指派給 Cornerstone OnDemand 時，您必須在 [指派] 對話方塊中選取任何有效的應用程式特有角色 (如果有的話)。 具有**預設存取**角色的使用者會從佈建中排除。

## <a name="configuring-automatic-user-provisioning-to-cornerstone-ondemand"></a>對 Cornerstone OnDemand 設定自動使用者佈建

本節將引導您逐步設定 Azure AD 佈建服務，以根據 Azure AD 中的使用者和/或群組指派，在 Cornerstone OnDemand 中建立、更新和停用使用者和/或群組。


### <a name="to-configure-automatic-user-provisioning-for-cornerstone-ondemand-in-azure-ad"></a>在 Azure AD 中為 Cornerstone OnDemand 設定自動使用者佈建：


1. 登入 [Azure 入口網站](https://portal.azure.com)，然後瀏覽至 [Azure Active Directory] > [企業應用程式] > [所有應用程式]。

2. 從您的 SaaS 應用程式清單中選取 Cornerstone OnDemand。
 
    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/Successcenter2.png)

3. 選取 [佈建] 索引標籤。

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningTab.png)

4. 將 [佈建模式] 設定為 [自動]。

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningCredentials.png)

5. 在 [管理員認證] 區段下，輸入 Cornerstone OnDemand 帳戶的**管理員使用者名稱**、**管理員密碼**和**網域**。

    *   在 [管理員使用者名稱] 欄位中，填入 Cornerstone OnDemand 租用戶中系統管理員帳戶的網域\使用者名稱。 範例：contoso\admin。

    *   在 [管理員密碼] 欄位中，填入與管理員使用者名稱相對應的密碼。

    *   在 [網域] 欄位中，填入 Cornerstone OnDemand 租用戶的 Webservice URL。 範例：服務位於 `https://ws-[corpname].csod.com/feed30/clientdataservice.asmx`，因此 Contoso 的網域是 `https://ws-contoso.csod.com/feed30/clientdataservice.asmx`。 如需有關如何擷取 Webservice URL 的詳細資訊，請參閱[這裡](https://help.csod.com/help/csod_0/Content/Resources/Documents/WebServices/CSOD_Web_Services_-_User-OU_Technical_Specification_v20160222.pdf)。

6. 如步驟 5 所示填寫欄位後，按一下 [測試連線]，以確保 Azure AD 可以連線到 Cornerstone OnDemand。 如果連線失敗，請確定您的 Cornerstone OnDemand 帳戶具有管理員權限，然後再試一次。

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/TestConnection.png)

7. 在 [通知電子郵件] 欄位中，輸入應該收到佈建錯誤通知的個人或群組電子郵件地址，然後選取 [發生失敗時傳送電子郵件通知] 核取方塊。

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/EmailNotification.png)

8. 按一下 [檔案] 。

9. 在 [對應] 區段中，選取 [同步處理 Azure Active Directory 使用者至 Cornerstone OnDemand]。

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/UserMapping.png)

10. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 Cornerstone OnDemand 的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 Cornerstone OnDemand 中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可所有變更。

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/UserMappingAttributes.png)

11. 若要設定範圍篩選，請參閱[範圍篩選教學課程](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md)中提供的下列指示。

12. 若要啟用 Cornerstone OnDemand 的 Azure AD 佈建服務，在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningStatus.png)

13. 透過在 [設定] 區段的 [範圍] 中選擇需要的值，可定義要佈建到 Cornerstone OnDemand 的使用者和/或群組。

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/SyncScope.png)

14. 當您準備好要佈建時，按一下 [儲存]。

    ![Cornerstone OnDemand 佈建](./media/cornerstone-ondemand-provisioning-tutorial/Save.png)


此作業會對在 [設定] 區段的 [範圍] 中定義的所有使用者和/或群組，啟動首次同步處理。 初始同步處理會比後續同步處理花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。 您可以使用 [同步處理詳細資料] 區段來監視進度，並遵循連結來佈建活動報告，當中會描述 Cornerstone OnDemand 上的 Azure AD 佈建服務所執行之所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../manage-apps/check-status-user-account-provisioning.md)。
## <a name="connector-limitations"></a>連接器限制

* Cornerstone OnDemand **Position** 屬性預期的值是對應至 Cornerstone OnDemand 入口網站上的角色。 有效 **Position** 值的清單可以藉由瀏覽至 Cornerstone OnDemand 入口網站的 [編輯使用者記錄 > 組織結構 > 位置] 來取得。
    ![Cornerstone OnDemand 佈建編輯使用者](./media/cornerstone-ondemand-provisioning-tutorial/UserEdit.png) ![Cornerstone OnDemand 佈建位置](./media/cornerstone-ondemand-provisioning-tutorial/UserPosition.png) ![Cornerstone OnDemand 佈建位置清單](./media/cornerstone-ondemand-provisioning-tutorial/PostionId.png)
    
## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



## <a name="next-steps"></a>後續步驟

* [瞭解如何針對佈建活動檢閱記錄和取得報告](../manage-apps/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_02.png
[3]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_03.png
