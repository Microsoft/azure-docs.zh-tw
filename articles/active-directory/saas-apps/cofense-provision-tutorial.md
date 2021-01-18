---
title: 教學課程：以 Azure Active Directory 設定 Cofense Recipient Sync 來自動佈建使用者 | Microsoft Docs
description: 了解如何從 Azure AD 對 Cofense Recipient Sync 自動佈建和取消佈建使用者帳戶。
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 84fe20ef-0de0-4f7c-9b42-6385f3d834db
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/11/2020
ms.author: Zhchia
ms.openlocfilehash: 084592a8db47a94a0fcd683105f749f514c05ea3
ms.sourcegitcommit: 08458f722d77b273fbb6b24a0a7476a5ac8b22e0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/15/2021
ms.locfileid: "98247568"
---
# <a name="tutorial-configure-cofense-recipient-sync-for-automatic-user-provisioning"></a>教學課程：設定 Cofense Recipient Sync 來自動佈建使用者

本教學課程說明您需要在 Cofense Recipient Sync 與 Azure Active Directory (Azure AD) 中執行哪些步驟，以設定自動使用者佈建。 設定後，Azure AD 就會使用 Azure AD 佈建服務，自動對 [Cofense Recipient Sync](https://cofense.com/) 佈建及取消佈建使用者。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](../app-provisioning/user-provisioning.md)。 


## <a name="capabilities-supported"></a>支援的功能
> [!div class="checklist"]
> * 在 Cofense Recipient Sync 中建立使用者
> * 當使用者不再需要存取時，將其從 Cofense Recipient Syncc 中移除
> * 讓 Azure AD 與 Cofense Recipient Sync 之間的使用者屬性保持同步

## <a name="prerequisites"></a>必要條件

本教學課程中概述的案例假設您已經具有下列必要條件：

* [Azure AD 租用戶](../develop/quickstart-create-new-tenant.md) 
* Azure AD 中具有設定佈建[權限](../roles/permissions-reference.md)的使用者帳戶 (例如，應用程式管理員、雲端應用程式管理員、應用程式擁有者或全域管理員)。 
* Cofense PhishMe 中的標準操作員帳戶。

## <a name="step-1-plan-your-provisioning-deployment"></a>步驟 1： 規劃您的佈建部署
1. 了解[佈建服務的運作方式](../app-provisioning/user-provisioning.md)。
2. 判斷誰會在[佈建範圍](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)內。
3. 判斷哪些資料要[在 Azure AD 與 Cofense Recipient Sync 之間對應](../app-provisioning/customize-application-attributes.md)。 

## <a name="step-2-configure-cofense-recipient-sync-to-support-provisioning-with-azure-ad"></a>步驟 2： 設定 Cofense Recipient Sync 以支援使用 Azure AD 進行佈建

1. 登入 Cofense PhishMe。 瀏覽至 [收件者] > [收件者同步]。 
2. 接受條款及條件，然後按一下 [開始使用]。

    ![收件者同步 tnc](media/cofense-provisioning-tutorial/recipient-sync-toc.png)

3. 複製 [URL] 和 [權杖] 欄位中的值。

    ![收件者同步](media/cofense-provisioning-tutorial/recipient-sync-getting-started.png)


## <a name="step-3-add-cofense-recipient-sync-from-the-azure-ad-application-gallery"></a>步驟 3： 從 Azure AD 應用程式庫新增 Cofense Recipient Sync

從 Azure AD 應用程式庫新增 Cofense Recipient Sync，以開始管理對 Cofense Recipient Sync 的佈建。如果您先前已針對 SSO 設定 Cofense Recipient Sync，則可使用相同的應用程式。 不過，建議您在一開始測試整合時，建立個別的應用程式。 [在此](../manage-apps/add-application-portal.md)深入了解從資源庫新增應用程式。 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>步驟 4： 定義將在佈建範圍內的人員 

Azure AD 佈建服務可讓您根據對應用程式的指派，或根據使用者/群組的屬性，界定將要佈建的人員。 如果您選擇根據指派來界定將佈建至應用程式的人員，您可以使用下列[步驟](../manage-apps/assign-user-or-group-access-portal.md)將使用者和群組指派給應用程式。 如果您選擇僅根據使用者或群組的屬性來界定將要佈建的人員，可以使用如[這裡](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)所述的範圍篩選條件。 

* 將使用者和群組指派給 Cofense Recipient Sync 時，您必須選取 [預設存取] 以外的角色。 具有預設存取角色的使用者會從佈建中排除，而且會在佈建記錄中被標示為沒有效率。 如果應用程式上唯一可用的角色是預設存取角色，您可[更新應用程式資訊清單](../develop/howto-add-app-roles-in-azure-ad-apps.md)以新增其他角色。 

* 從小規模開始。 在推出給所有人之前，先使用一小部分的使用者和群組進行測試。 當佈建範圍設為已指派的使用者和群組時，您可將一或兩個使用者或群組指派給應用程式來控制這點。 當範圍設為所有使用者和群組時，您可指定[以屬性為基礎的範圍篩選條件](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)。 


## <a name="step-5-configure-automatic-user-provisioning-to-cofense-recipient-sync"></a>步驟 5。 設定對 Cofense Recipient Sync 的自動使用者佈建 

本節將引導您逐步設定 Azure AD 佈建服務，以根據 Azure AD 中的使用者，在 Cofense Recipient Sync 中建立、更新及停用使用者。

### <a name="to-configure-automatic-user-provisioning-for-cofense-recipient-sync-in-azure-ad"></a>若要在 Azure AD 中進行將使用者自動佈建至 Cofense Recipient Sync 的設定：

1. 登入 [Azure 入口網站](https://portal.azure.com)。 選取 [企業應用程式]，然後選取 [所有應用程式]。

    ![企業應用程式刀鋒視窗](common/enterprise-applications.png)

2. 在應用程式清單中，選取 [Cofense Recipient Sync]。

    ![應用程式清單中的 Cofense 連結](common/all-applications.png)

3. 選取 [佈建] 索引標籤。

    ![佈建索引標籤](common/provisioning.png)

4. 將 [佈建模式] 設定為 [自動]。

    ![佈建索引標籤 [自動]](common/provisioning-automatic.png)

5. 在 [管理員認證] 區段底下，輸入先前從步驟 2 擷取的 **SCIM 2.0 基底 URL 和 SCIM 驗證權杖** 值。 按一下 [測試連線]，以確定 Azure AD 可連線至 Cofense Recipient Sync。如果連線失敗，請確定您的 Cofense Recipient Sync 帳戶具有管理員權限，然後再試一次。

    ![租用戶 URL 權杖](common/provisioning-testconnection-tenanturltoken.png)

6. 在 [通知電子郵件] 欄位中，輸入應該收到佈建錯誤通知的個人或群組電子郵件地址，然後選取 [發生失敗時傳送電子郵件通知] 核取方塊。

    ![通知電子郵件](common/provisioning-notification-email.png)

7. 選取 [儲存]。

8. 在 [對應] 區段底下，選取 [將 Azure Active Directory 使用者同步處理至 Cofense Recipient Sync]。

9. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步處理至 Cofense Recipient Sync 的使用者屬性。 選取為 [比對] 屬性 (Property) 的屬性 (Attribute)，會用來比對 Cofense Recipient Sync 中的使用者帳戶以進行更新作業。  選取 [儲存] 按鈕以認可所有變更。

   |屬性|類型|支援篩選|
   |---|---|---|
   |externalId|String|&check;|
   |userName|String|
   |作用中|Boolean|
   |displayName|String|
   |name.formatted|String|
   |name.givenName|String|
   |name.familyName|String|
   |name.honorificSuffix|字串|
   |phoneNumbers[type eq"work"].value|字串|
   |phoneNumbers[type eq"home"].value|字串|
   |phoneNumbers[type eq"other"].value|字串|
   |phoneNumbers[type eq"pager"].value|字串|
   |phoneNumbers[type eq"mobile"].value|字串|
   |phoneNumbers[type eq"fax"].value|字串|
   |addresses[type eq"other"].formatted|字串|
   |addresses[type eq"work"].formatted|字串|
   |addresses[type eq"work"].streetAddress|字串|
   |addresses[type eq"work"].locality|字串|
   |addresses[type eq"work"].region|字串|
   |addresses[type eq"work"].postalCode|字串|
   |addresses[type eq"work"].country|String|
   |title|String|
   |emails[type eq "work"].value|String|
   |emails[type eq "home"].value|字串|
   |emails[type eq "other"].value|String|
   |preferredLanguage|String|
   |nickName|String|
   |userType|String|
   |地區設定|String|
   |timezone|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|參考|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:organization|String|

10. 若要設定範圍篩選，請參閱[範圍篩選教學課程](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)中提供的下列指示。

11. 若要啟用 Cofense Recipient Sync 的 Azure AD 佈建服務，請在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]。

    ![佈建狀態已切換為開啟](common/provisioning-toggle-on.png)

12. 藉由在 [設定] 區段的 [範圍] 中選擇所需的值，定義要佈建至 Cofense Recipient Sync 的使用者和/或群組。

    ![佈建範圍](common/provisioning-scope.png)

13. 當您準備好要佈建時，按一下 [儲存]。

    ![儲存雲端佈建設定](common/provisioning-configuration-save.png)

此作業會對在 [設定] 區段的 [範圍] 中定義的所有使用者和群組，啟動首次同步處理週期。 初始週期會比後續週期花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。 

## <a name="step-6-monitor-your-deployment"></a>步驟 6. 監視您的部署
設定佈建後，請使用下列資源來監視您的部署：

* 使用[佈建記錄](../reports-monitoring/concept-provisioning-logs.md)來判斷哪些使用者已佈建成功或失敗
* 檢查[進度列](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md)來查看佈建週期的狀態，以及其接近完成的程度
* 如果佈建設定似乎處於狀況不良的狀態，應用程式將會進入隔離狀態。 [在此](../app-provisioning/application-provisioning-quarantine-status.md)深入了解隔離狀態。 

## <a name="change-log"></a>變更記錄

* 2020/01/15 - 對於 objectId -> externalId 對應，從「僅限物件建立期間」變更為「一律」。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>後續步驟

* [瞭解如何針對佈建活動檢閱記錄和取得報告](../app-provisioning/check-status-user-account-provisioning.md)