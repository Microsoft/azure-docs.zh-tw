---
title: 教學課程：使用 Azure Active Directory 設定 Gtmhub 來自動布建使用者 |Microsoft Docs
description: 瞭解如何從 Azure AD 將使用者帳戶自動布建和取消布建至 Gtmhub。
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 10b68d00-a544-480b-9bd6-f6ac291a90d0
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/03/2020
ms.author: Zhchia
ms.openlocfilehash: 0e160def31a43bc94e4f6151b46efe72e585e953
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98735702"
---
# <a name="tutorial-configure-gtmhub-for-automatic-user-provisioning"></a>教學課程：設定 Gtmhub 來自動布建使用者

本教學課程說明在 Gtmhub 和 Azure Active Directory (Azure AD) 設定自動使用者布建時所需執行的步驟。 當設定時，Azure AD 會使用 Azure AD 布建服務，自動將使用者和群組布建和取消布建至 [Gtmhub](https://www.gtmhub.com/) 。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](../app-provisioning/user-provisioning.md)。 

>[!NOTE]
>目前，當設定自動使用者布建時，Azure AD 只會自動將使用者和群組布建到 Gtmhub，以及使用 Azure AD 布建服務將使用者對應至其各自的小組。但在2021中，使用 Gtmhub 啟用 SSO 之後，使用者將會在透過 SSO 登入時自動布建，並將其指派給其各自的小組。


## <a name="capabilities-supported"></a>支援的功能
> [!div class="checklist"]
> * 當使用者不再需要存取權時，請移除 Gtmhub 中的使用者。
> * 在 Azure AD 和 Gtmhub 之間保持使用者屬性同步。
> * 自動將使用者對應到其小組，並加以對齊。

## <a name="prerequisites"></a>Prerequisites

本教學課程中概述的案例假設您已經具有下列必要條件：

* [Azure AD 租用戶](../develop/quickstart-create-new-tenant.md)。
* Azure AD 中具有設定佈建[權限](../roles/permissions-reference.md)的使用者帳戶 (例如，應用程式管理員、雲端應用程式管理員、應用程式擁有者或全域管理員)。 
* 企業 Gtmhub 帳戶。

## <a name="step-1-plan-your-provisioning-deployment"></a>步驟 1： 規劃您的佈建部署
1. 了解[佈建服務的運作方式](../app-provisioning/user-provisioning.md)。
2. 判斷誰會在[佈建範圍](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)內。
3. 判斷要 [在 Azure AD 與 Gtmhub 之間對應](../app-provisioning/customize-application-attributes.md)的資料。 

## <a name="step-2-configure-gtmhub-to-support-team-mapping-and-user-de-provisioning-with-azure-ad"></a>步驟 2： 使用 Azure AD 設定 Gtmhub 以支援小組對應和使用者解除布建

若要將布建應用程式連線到您的 Gtmhub 帳戶，您必須發出 SCIM 權杖，並編譯租使用者 URL。

### <a name="to-issue-a-new-scim-token"></a>若要發出新的 SCIM token：

1. 登入您的 **Gtmhub 帳戶**。 流覽至 [ **設定] > 設定 > API 權杖**。

    ![API 權杖索引標籤](media/gtmhub-provisioning-tutorial/api-tokens.png)
2. 按一下 [ **問題權杖** ]，然後選取 [ **SCIM**]。 輸入權杖的名稱，然後按一下 [ **產生 API 權杖** ] 按鈕。

    ![[產生權杖] 索引標籤](media/gtmhub-provisioning-tutorial/generate-token.png)
3. 產生權杖後，您就可以在 Azure AD 布建應用程式中複製和使用權杖。

    ![複製權杖](media/gtmhub-provisioning-tutorial/token.png)

### <a name="to-compile-the-tenant-url"></a>若要編譯租使用者 url：

1. 您的租使用者 url 必須採用下列格式：

    `https://app.gtmhub.com/api/v1/scim/azure/{account_id}`

2. 如果您的 Gtmhub 帳戶位於美國資料中心，您也必須將資料中心新增至 URL：
    
     `https://app.us.gtmhub.com/api/v1/scim/azure/{account_id}`

3. 若要取得帳戶識別碼，請移至 [ **設定** ]，然後選取 [ **API 權杖** ] 索引標籤，並複製帳戶識別碼：  ![ 帳戶識別碼](media/gtmhub-provisioning-tutorial/account-id.png)

## <a name="step-3-add-gtmhub-from-the-azure-ad-application-gallery"></a>步驟 3： 從 Azure AD 應用程式資源庫新增 Gtmhub

從 Azure AD 應用程式資源庫新增 Gtmhub，以開始管理布建至 Gtmhub。 如果您先前已設定 SSO 的 Gtmhub，您可以使用相同的應用程式。 不過，建議您在一開始測試整合時，建立個別的應用程式。 [在此](../manage-apps/add-application-portal.md)深入了解從資源庫新增應用程式。 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>步驟 4： 定義將在佈建範圍內的人員 

Azure AD 佈建服務可讓您根據對應用程式的指派，或根據使用者/群組的屬性，界定將要佈建的人員。 如果您選擇根據指派來界定將佈建至應用程式的人員，您可以使用下列[步驟](../manage-apps/assign-user-or-group-access-portal.md)將使用者和群組指派給應用程式。 如果您選擇僅根據使用者或群組的屬性來界定將要佈建的人員，可以使用如[這裡](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)所述的範圍篩選條件。 

* 將使用者和群組指派給 Gtmhub 時，您必須選取 **預設存取** 以外的角色。 具有預設存取角色的使用者會從佈建中排除，而且會在佈建記錄中被標示為沒有效率。 如果應用程式上唯一可用的角色是預設存取角色，您可[更新應用程式資訊清單](../develop/howto-add-app-roles-in-azure-ad-apps.md)以新增其他角色。 

* 從小規模開始。 在推出給所有人之前，先使用一小部分的使用者和群組進行測試。 當佈建範圍設為已指派的使用者和群組時，您可將一或兩個使用者或群組指派給應用程式來控制這點。 當範圍設為所有使用者和群組時，您可指定[以屬性為基礎的範圍篩選條件](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)。 


## <a name="step-5-configure-automatic-user-provisioning-to-gtmhub"></a>步驟 5。 設定自動使用者布建至 Gtmhub 

此節將引導您逐步設定 Azure AD 佈建服務，以根據 Azure AD 中的使用者和/或群組指派，在 TestApp 中建立、更新和停用使用者和/或群組。

### <a name="to-configure-automatic-user-provisioning-for-gtmhub-in-azure-ad"></a>若要在 Azure AD 中為 Gtmhub 設定自動使用者布建：

1. 登入 [Azure 入口網站](https://portal.azure.com)。 選取 [企業應用程式]，然後選取 [所有應用程式]。

    ![企業應用程式刀鋒視窗](common/enterprise-applications.png)

2. 在應用程式清單中，選取 [ **Gtmhub**]。

    ![應用程式清單中的 Gtmhub 連結](common/all-applications.png)

3. 選取 [佈建] 索引標籤。

    ![佈建索引標籤](common/provisioning.png)

4. 將 [佈建模式] 設定為 [自動]。

    ![佈建索引標籤 [自動]](common/provisioning-automatic.png)

5. 在 [系統 **管理員認證** ] 區段下，輸入您的 Gtmhub 租使用者 URL 和秘密權杖。 按一下 [ **測試連接** ] 以確保 Azure AD 可以連線至 Gtmhub。 如果連接失敗，請確定您的 Gtmhub 帳戶具有系統管理員許可權，然後再試一次。

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

6. 在 [通知電子郵件] 欄位中，輸入應該收到佈建錯誤通知的個人或群組電子郵件地址，然後選取 [發生失敗時傳送電子郵件通知] 核取方塊。

    ![通知電子郵件](common/provisioning-notification-email.png)

7. 選取 [儲存]。

8. **在 [對應**] 區段下，選取 [**同步處理 Azure Active Directory 使用者至 Gtmhub**]。

9. 在 [ **屬性對應** ] 區段中，檢查從 Azure AD 同步處理到 Gtmhub 的使用者屬性。 選取為 [比對 **] 屬性的屬性會** 用來比對 Gtmhub 中的使用者帳戶以進行更新作業。 如果您選擇變更相符的 [目標屬性](../app-provisioning/customize-application-attributes.md)，您將必須確定 Gtmhub API 支援根據該屬性篩選使用者。 選取 [儲存] 按鈕以認可所有變更。

   |屬性|類型|支援篩選|
   |---|---|---|
   |userName|String|&check;|
   |externalId|String|&check;|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|參考|

10. 若要設定範圍篩選，請參閱[範圍篩選教學課程](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)中提供的下列指示。

13. 若要啟用 Gtmhub Azure AD 的布建服務，請在 [**設定**] 區段中，將 [布建 **狀態**] 變更為 [**開啟**]。

    ![佈建狀態已切換為開啟](common/provisioning-toggle-on.png)

14. 在 [**設定**] 區段的 [**範圍**] 中選擇所需的值，以定義您想要布建到 Gtmhub 的使用者和/或群組。

    ![佈建範圍](common/provisioning-scope.png)

15. 當您準備好要佈建時，按一下 [儲存]。

    ![儲存雲端佈建設定](common/provisioning-configuration-save.png)

此作業會對在 [設定] 區段的 [範圍] 中定義的所有使用者和群組，啟動首次同步處理週期。 初始週期會比後續週期花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。 

## <a name="step-6-monitor-your-deployment"></a>步驟 6. 監視您的部署
設定佈建後，請使用下列資源來監視您的部署：

* 使用[佈建記錄](../reports-monitoring/concept-provisioning-logs.md)來判斷哪些使用者已佈建成功或失敗
* 檢查[進度列](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md)來查看佈建週期的狀態，以及其接近完成的程度
* 如果佈建設定似乎處於狀況不良的狀態，應用程式將會進入隔離狀態。 [在此](../app-provisioning/application-provisioning-quarantine-status.md)深入了解隔離狀態。  

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>後續步驟

* [瞭解如何針對佈建活動檢閱記錄和取得報告](../app-provisioning/check-status-user-account-provisioning.md)