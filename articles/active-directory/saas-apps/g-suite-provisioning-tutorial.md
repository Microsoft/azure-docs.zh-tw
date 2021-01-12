---
title: 教學課程︰以 Azure Active Directory 設定 G Suite 來自動佈建使用者 | Microsoft Docs
description: 了解如何將使用者帳戶從 Azure AD 針對 G Suite 進行自動佈建和取消佈建。
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/06/2020
ms.author: Zhchia
ms.openlocfilehash: c3f61c3fe688a0b7533902fb0caa19b67f883482
ms.sourcegitcommit: 5e762a9d26e179d14eb19a28872fb673bf306fa7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/05/2021
ms.locfileid: "97901584"
---
# <a name="tutorial-configure-g-suite-for-automatic-user-provisioning"></a>教學課程︰設定 G Suite 來自動佈建使用者

此教學課程說明您需要在 G Suite 與 Azure Active Directory (Azure AD) 中執行的步驟，以設定自動使用者佈建。 設定後，Azure AD 就會使用 Azure AD 佈建服務，自動對 [G Suite](https://gsuite.google.com/) \(英文\) 佈建及取消佈建使用者和群組。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](../app-provisioning/user-provisioning.md)。 

> [!NOTE]
> 本教學課程會說明建置在 Azure AD 使用者佈建服務之上的連接器。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](../app-provisioning/user-provisioning.md)。

> [!NOTE]
> 本文包含字詞「*允許清單*」的參考 (Microsoft 已不再使用該字詞)。 從軟體中移除該字詞時，我們也會將其從本文中移除。

## <a name="capabilities-supported"></a>支援的功能
> [!div class="checklist"]
> * 在 G Suite 中建立使用者
> * 當使用者不再需要存取權時，將其從 G Suite 中移除
> * 讓 Azure AD 與 G Suite 之間的使用者屬性保持同步
> * 在 G Suite 中佈建群組和群組成員資格
> * [單一登入](./google-apps-tutorial.md)至 G Suite (建議)

## <a name="prerequisites"></a>必要條件

本教學課程中概述的案例假設您已經具有下列必要條件：

* [Azure AD 租用戶](../develop/quickstart-create-new-tenant.md) 
* Azure AD 中具有設定佈建[權限](../roles/permissions-reference.md)的使用者帳戶 (例如，應用程式管理員、雲端應用程式管理員、應用程式擁有者或全域管理員)。 
* [G Suite 租用戶](https://gsuite.google.com/pricing.html)
* G Suite 上具有管理員權限的使用者帳戶。

## <a name="step-1-plan-your-provisioning-deployment"></a>步驟 1： 規劃您的佈建部署
1. 了解[佈建服務的運作方式](../app-provisioning/user-provisioning.md)。
2. 判斷誰會在[佈建範圍](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)內。
3. 判斷哪些資料要[在 Azure AD 與 G Suite 之間對應](../app-provisioning/customize-application-attributes.md)。 

## <a name="step-2-configure-g-suite-to-support-provisioning-with-azure-ad"></a>步驟 2： 設定 G Suite 以支援使用 Azure AD 進行佈建

以 Azure AD 設定 G Suite 來自動佈建使用者之前，您必須在 G Suite 上啟用 SCIM 佈建。

1. 使用您的系統管理員帳戶登入 [G Suite 管理主控台](https://admin.google.com/)，然後選取 [安全性]。 如果您沒有看到連結，它可能隱藏在畫面底部的 [其他控制項] 功能表之下。

    ![G Suite 安全性](./media/g-suite-provisioning-tutorial/gapps-security.png)

2. 在 [安全性] 頁面上，選取 [API 參考]。

    ![G Suite API](./media/g-suite-provisioning-tutorial/gapps-api.png)

3. 選取 [啟用 API 存取]。

    ![G Suite API 已啟用](./media/g-suite-provisioning-tutorial/gapps-api-enabled.png)

    > [!IMPORTANT]
   > 針對您想要佈建至 G Suite 的每個使用者，其使用者名稱在 Azure AD 中 **必須** 繫結至自訂網域。 例如，G Suite 不會接受類似 bob@contoso.onmicrosoft.com 的使用者名稱。 另一方面，則接受 bob@contoso.com。 您可以遵循[此處](../fundamentals/add-custom-domain.md)的指示來變更現有使用者的網域。

4. 當您使用 Azure AD 新增並驗證所需的自訂網域之後，您必須使用 G Suite 再次進行驗證。 若要驗證 G Suite 中的網域，請參閱下列步驟：

    a. 在 [G Suite 管理主控台](https://admin.google.com/)中，選取 [網域]。

    ![G Suite 網域](./media/g-suite-provisioning-tutorial/gapps-domains.png)

    b. 選取 [新增網域或網域別名]。

    ![G Suite 新增網域](./media/g-suite-provisioning-tutorial/gapps-add-domain.png)

    c. 選取 [新增另一個網域]，然後輸入您想要新增的網域名稱。

    ![G Suite 新增另一個](./media/g-suite-provisioning-tutorial/gapps-add-another.png)

    d. 選取 [繼續並驗證網域擁有權]。 然後依照步驟以驗證您擁有網域名稱。 如需如何向 Google 驗證您網域的完整指示，請參閱[驗證網站擁有權](https://support.google.com/webmasters/answer/35179)。

    e. 對於您想要新增至 G Suite 的任何其他網域，重複上述步驟。

5. 接下來，決定您想要用來在 G Suite 中管理使用者佈建的管理帳戶。 瀏覽至 [管理員角色]。

    ![G Suite 管理員](./media/g-suite-provisioning-tutorial/gapps-admin.png)

6. 對於該帳戶的 [管理員角色]，編輯該角色的 [權限]。 務必啟用所有 [管理 API 權限]，以便讓此帳戶可以用來佈建。

    ![G Suite 管理員權限](./media/g-suite-provisioning-tutorial/gapps-admin-privileges.png)

## <a name="step-3-add-g-suite-from-the-azure-ad-application-gallery"></a>步驟 3： 從 Azure AD 應用程式庫新增 G Suite

從 Azure AD 應用程式庫新增 G Suite，以開始管理對 G Suite 的佈建。 如果您先前已針對 SSO 設定 G Suite，則可使用相同的應用程式。 不過，建議您在一開始測試整合時，建立個別的應用程式。 [在此](../manage-apps/add-application-portal.md)深入了解從資源庫新增應用程式。 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>步驟 4： 定義將在佈建範圍內的人員 

Azure AD 佈建服務可讓您根據對應用程式的指派，或根據使用者/群組的屬性，界定將要佈建的人員。 如果您選擇根據指派來界定將佈建至應用程式的人員，您可以使用下列[步驟](../manage-apps/assign-user-or-group-access-portal.md)將使用者和群組指派給應用程式。 如果您選擇僅根據使用者或群組的屬性來界定將要佈建的人員，可以使用如[這裡](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)所述的範圍篩選條件。 

* 將使用者和群組指派給 G Suite 時，您必須選取 [預設存取] 以外的角色。 具有預設存取角色的使用者會從佈建中排除，而且會在佈建記錄中被標示為沒有效率。 如果應用程式上唯一可用的角色是預設存取角色，您可[更新應用程式資訊清單](../develop/howto-add-app-roles-in-azure-ad-apps.md)以新增其他角色。 

* 從小規模開始。 在推出給所有人之前，先使用一小部分的使用者和群組進行測試。 當佈建範圍設為已指派的使用者和群組時，您可將一或兩個使用者或群組指派給應用程式來控制這點。 當範圍設為所有使用者和群組時，您可指定[以屬性為基礎的範圍篩選條件](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)。 


## <a name="step-5-configure-automatic-user-provisioning-to-g-suite"></a>步驟 5。 設定將使用者自動佈建至 G Suite 

此節將引導您逐步設定 Azure AD 佈建服務，以根據 Azure AD 中的使用者和/或群組指派，在 TestApp 中建立、更新和停用使用者和/或群組。

> [!NOTE]
> 若要深入了解 G Suite 的目錄 API 端點，請參閱[目錄 API](https://developers.google.com/admin-sdk/directory)。

### <a name="to-configure-automatic-user-provisioning-for-g-suite-in-azure-ad"></a>若要在 Azure AD 中為 G Suite 設定自動使用者佈建：

1. 登入 [Azure 入口網站](https://portal.azure.com)。 選取 [企業應用程式]，然後選取 [所有應用程式]。 使用者必須登入 portal.azure.com，而且將無法使用 aad.portal.azure.com

    ![企業應用程式刀鋒視窗](./media/g-suite-provisioning-tutorial/enterprise-applications.png)

    ![所有應用程式刀鋒視窗](./media/g-suite-provisioning-tutorial/all-applications.png)

2. 在應用程式清單中，選取 [G Suite]。

    ![應用程式清單中的 G Suite 連結](common/all-applications.png)

3. 選取 [佈建] 索引標籤。按一下 [開始使用]。

    ![[管理] 選項的螢幕擷取畫面，並已指出 [佈建] 選項。](common/provisioning.png)

      ![[開始使用] 刀鋒視窗](./media/g-suite-provisioning-tutorial/get-started.png)

4. 將 [佈建模式] 設定為 [自動]。

    ![[佈建模式] 下拉式清單的螢幕擷取畫面，並已指出 [自動] 選項。](common/provisioning-automatic.png)

5. 在 [管理員認證] 區段下，按一下 [授權]。 系統會將您重新導向至新瀏覽器視窗中的 [Google 授權] 對話方塊。

      ![G Suite 授權](./media/g-suite-provisioning-tutorial/authorize-1.png)

6. 確認您想要授與 Azure AD 權限來變更您的 G Suite 租用戶。 選取 [接受]。

     ![G Suite 租用戶驗證](./media/g-suite-provisioning-tutorial/gapps-auth.png)

7. 在 Azure 入口網站中，按一下 [測試連線] 以確保 Azure AD 可以連線到 G Suite。 如果連線失敗，請確定您的 G Suite 帳戶具有管理員權限，然後再試一次。 然後再試一次 **授權** 步驟。

6. 在 [通知電子郵件] 欄位中，輸入應該收到佈建錯誤通知的個人或群組電子郵件地址，然後選取 [發生失敗時傳送電子郵件通知] 核取方塊。

    ![通知電子郵件](common/provisioning-notification-email.png)

7. 選取 [儲存]。

8. 在 [對應] 區段底下，選取 [佈建 Azure Active Directory 使用者]。

9. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步處理至 G Suite 的使用者屬性。 選取為 [比對] 屬性 (Property) 的屬性 (Attribute) 會用來比對 G Suite 中的使用者帳戶，以進行更新作業。 如果您選擇變更[比對目標屬性](../app-provisioning/customize-application-attributes.md)，則必須確保 G Suite API 支援根據該屬性來篩選使用者。 選取 [儲存] 按鈕以認可所有變更。

   |屬性|類型|
   |---|---|
   |primaryEmail|字串|
   |relations.[type eq "manager"].value|String|
   |name.familyName|String|
   |name.givenName|String|
   |暫止|字串|
   |externalIds.[type eq "custom"].value|字串|
   |externalIds.[type eq "organization"].value|字串|
   |addresses.[type eq "work"].country|字串|
   |addresses.[type eq "work"].streetAddress|字串|
   |addresses.[type eq "work"].region|字串|
   |addresses.[type eq "work"].locality|字串|
   |addresses.[type eq "work"].postalCode|字串|
   |emails.[type eq "work"].address|字串|
   |organizations.[type eq "work"].department|字串|
   |organizations.[type eq "work"].title|字串|
   |phoneNumbers.[type eq "work"].value|字串|
   |phoneNumbers.[type eq "mobile"].value|字串|
   |phoneNumbers.[type eq "work_fax"].value|字串|
   |emails.[type eq "work"].address|字串|
   |organizations.[type eq "work"].department|字串|
   |organizations.[type eq "work"].title|字串|
   |phoneNumbers.[type eq "work"].value|字串|
   |phoneNumbers.[type eq "mobile"].value|字串|
   |phoneNumbers.[type eq "work_fax"].value|字串|
   |addresses.[type eq "home"].country|字串|
   |addresses.[type eq "home"].formatted|字串|
   |addresses.[type eq "home"].locality|字串|
   |addresses.[type eq "home"].postalCode|字串|
   |addresses.[type eq "home"].region|字串|
   |addresses.[type eq "home"].streetAddress|字串|
   |addresses.[type eq "other"].country|字串|
   |addresses.[type eq "other"].formatted|字串|
   |addresses.[type eq "other"].locality|字串|
   |addresses.[type eq "other"].postalCode|字串|
   |addresses.[type eq "other"].region|字串|
   |addresses.[type eq "other"].streetAddress|字串|
   |addresses.[type eq "work"].formatted|字串|
   |changePasswordAtNextLogin|字串|
   |emails.[type eq "home"].address|字串|
   |emails.[type eq "other"].address|字串|
   |externalIds.[type eq "account"].value|字串|
   |externalIds.[type eq "custom"].customType|字串|
   |externalIds.[type eq "customer"].value|字串|
   |externalIds.[type eq "login_id"].value|字串|
   |externalIds.[type eq "network"].value|字串|
   |gender.type|字串|
   |GeneratedImmutableId|字串|
   |識別碼|字串|
   |ims.[type eq "home"].protocol|字串|
   |ims.[type eq "other"].protocol|字串|
   |ims.[type eq "work"].protocol|字串|
   |includeInGlobalAddressList|字串|
   |ipWhitelisted|字串|
   |organizations.[type eq "school"].costCenter|字串|
   |organizations.[type eq "school"].department|字串|
   |organizations.[type eq "school"].domain|字串|
   |organizations.[type eq "school"].fullTimeEquivalent|字串|
   |organizations.[type eq "school"].location|字串|
   |organizations.[type eq "school"].name|字串|
   |organizations.[type eq "school"].symbol|字串|
   |organizations.[type eq "school"].title|字串|
   |organizations.[type eq "work"].costCenter|字串|
   |organizations.[type eq "work"].domain|字串|
   |organizations.[type eq "work"].fullTimeEquivalent|字串|
   |organizations.[type eq "work"].location|字串|
   |organizations.[type eq "work"].name|字串|
   |organizations.[type eq "work"].symbol|字串|
   |OrgUnitPath|字串|
   |phoneNumbers.[type eq "home"].value|字串|
   |phoneNumbers.[type eq "other"].value|字串|
   |websites.[type eq "home"].value|字串|
   |websites.[type eq "other"].value|字串|
   |websites.[type eq "work"].value|字串|
   

10. 在 [對應] 區段底下，選取 [佈建 Azure Active Directory 群組]。

11. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 G Suite 的群組屬性。 選取為 [比對] 屬性的屬性會用來比對 G Suite 中的群組以進行更新作業。 選取 [儲存] 按鈕以認可所有變更。

      |屬性|類型|
      |---|---|
      |電子郵件|字串|
      |成員|字串|
      |NAME|String|
      |description|String|

12. 若要設定範圍篩選，請參閱[範圍篩選教學課程](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)中提供的下列指示。

13. 若要對 G Suite 啟用 Azure AD 佈建服務，請在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]。

    ![佈建狀態已切換為開啟](common/provisioning-toggle-on.png)

14. 透過在 [設定] 區段的 [範圍] 中選擇需要的值，可定義要佈建到 G Suite 的使用者和/或群組。

    ![佈建範圍](common/provisioning-scope.png)

15. 當您準備好要佈建時，按一下 [儲存]。

    ![儲存雲端佈建設定](common/provisioning-configuration-save.png)

此作業會對在 [設定] 區段的 [範圍] 中定義的所有使用者和群組，啟動首次同步處理週期。 初始週期會比後續週期花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。

> [!NOTE]
> 如果使用者已經有使用 Azure AD 使用者電子郵件地址的現有個人/消費者帳戶，則可能會造成一些問題，但您可以在執行目錄同步之前，使用 Google Transfer Tool 來加以解決。

## <a name="step-6-monitor-your-deployment"></a>步驟 6. 監視您的部署
設定佈建後，請使用下列資源來監視您的部署：

1. 使用[佈建記錄](../reports-monitoring/concept-provisioning-logs.md)來判斷哪些使用者已佈建成功或失敗
2. 檢查[進度列](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md)來查看佈建週期的狀態，以及其接近完成的程度
3. 如果佈建設定似乎處於狀況不良的狀態，應用程式將會進入隔離狀態。 [在此](../app-provisioning/application-provisioning-quarantine-status.md)深入了解隔離狀態。  

## <a name="change-log"></a>變更記錄

* 2020/10/17 - 已新增對其他 G Suite 使用者和群組屬性的支援。
* 2020/10/17 - 已更新 G Suite 目標屬性名稱，以符合[這裡](https://developers.google.com/admin-sdk/directory)的定義。
* 2020/10/17 - 已更新預設屬性對應。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>後續步驟

* [瞭解如何針對佈建活動檢閱記錄和取得報告](../app-provisioning/check-status-user-account-provisioning.md)