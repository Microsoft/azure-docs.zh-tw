---
title: 教學課程︰以 Azure Active Directory 設定 Cerner Central 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 以自動佈建使用者至 Cerner Central 中的名冊。
services: active-directory
documentationcenter: ''
author: asmalser-msft
writer: asmalser-msft
manager: mtillman
ms.assetid: d4ca2365-6729-48f7-bb7f-c0f5ffe740a3
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/26/2018
ms.author: asmalser-msft
ms.openlocfilehash: 694274abb9a762bc6ca6309cbd56b0eb65bb5a5e
ms.sourcegitcommit: 16ddc345abd6e10a7a3714f12780958f60d339b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36228776"
---
# <a name="tutorial-configure-cerner-central-for-automatic-user-provisioning"></a>教學課程︰設定 Cerner Central 來自動佈建使用者

本教學課程旨在說明您需要在 Cerner Central 和 Azure AD 中執行的步驟，以將使用者帳戶從 Azure AD 自動佈建和取消佈建至 Cerner Central 中的使用者名冊。 


## <a name="prerequisites"></a>先決條件

本教學課程中說明的案例假設您已經具有下列項目：

*   Azure Active Directory 租用戶
*   Cerner Central 租用戶 

> [!NOTE]
> Azure Active Directory 使用 [SCIM](http://www.simplecloud.info/) 通訊協定與 Cerner Central 整合。

## <a name="assigning-users-to-cerner-central"></a>將使用者指派給 Cerner Central

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者帳戶佈建的內容中，只有「已指派」至 Azure AD 中的應用程式之使用者和群組會進行同步處理。 

在設定並啟用佈建服務之前，您應該決定 Azure AD 中的哪些使用者及/或群組代表需要 Cerner Central 存取權的使用者。 一旦決定後，您可以依照此處的指示，將這些使用者指派給 Cerner Central：

[將使用者或群組指派給企業應用程式](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-cerner-central"></a>將使用者指派給 Cerner Central 的重要秘訣

*   建議將單一 Azure AD 使用者指派給 Cerner Central，以測試佈建設定。 其他使用者及/或群組可能會稍後再指派。

* 對單一使用者完成初始測試後，Cerner Central 建議指派嘗試存取要佈建至 Cerner 使用者名冊之任何 Cerner 解決方案 (不只是 Cerner Central) 的完整使用者清單。  其他 Cerner 解決方案會利用這份使用者名冊中的使用者清單。

*   將使用者指派給 Cerner Central 時，您必須在 [指派] 對話方塊中選取 [使用者] 角色。 具有「預設存取」角色的使用者會從佈建中排除。


## <a name="configuring-user-provisioning-to-cerner-central"></a>設定使用者佈建至 Cerner Central

本節會引導您使用 Cerner 的 SCIM 使用者帳戶佈建 API，將 Azure AD 連線至 Cerner Central 的使用者名冊，以及根據 Azure AD 中的使用者和群組指派，設定佈建服務以在 Cerner Central 中建立、更新和停用已指派的使用者帳戶。

> [!TIP]
> 您也可以選擇啟用 Cerner Central 的 SAML 型單一登入，請遵循 [Azure 入口網站 (https://portal.azure.com) 中提供的指示。 可以獨立設定自動佈建的單一登入，雖然這兩個功能彼此補充。 如需詳細資訊，請參閱 [Cerner Central 單一登入教學課程](cernercentral-tutorial.md)。


### <a name="to-configure-automatic-user-account-provisioning-to-cerner-central-in-azure-ad"></a>若要在 Azure AD 中設定自動使用者帳戶佈建至 Cerner Central：


為了將使用者帳戶佈建至 Cerner Central，您必須從 Cerner 要求 Cerner Central 系統帳戶，並產生 Azure AD 可用來連線到 Cerner 之 SCIM 端點的 OAuth 持有人權杖。 此外，建議在 Cerner 沙箱環境中執行整合，再部署至生產環境。

1.  第一個步驟是確定管理 Cerner 與 Azure AD 整合的人員擁有 CernerCare 帳戶，必須有此帳戶才能存取完成指示所需的文件。 如有必要，請使用下列 URL 在每個適用的環境中建立 CernerCare 帳戶。

   * 沙箱：https://sandboxcernercare.com/accounts/create

   * 生產環境：https://cernercare.com/accounts/create  

2.  接下來，必須建立 Azure AD 的系統帳戶。 您可以使用下列指示，要求沙箱和生產環境的系統帳戶。

   * 指示：https://wiki.ucern.com/display/CernerCentral/Requesting+A+System+Account

   * 沙箱：https://sandboxcernercentral.com/system-accounts/

   * 生產環境：https://cernercentral.com/system-accounts/

3.  接下來，產生每個系統帳戶的 OAuth 持有人權杖。 若要這樣做，請遵循下列指示。

   * 指示：https://wiki.ucern.com/display/public/reference/Accessing+Cerner%27s+Web+Services+Using+A+System+Account+Bearer+Token

   * 沙箱：https://sandboxcernercentral.com/system-accounts/

   * 生產環境：https://cernercentral.com/system-accounts/

4. 最後，您需要針對 Cerner 中的沙箱和生產環境取得「使用者名冊領域識別碼」，以完成設定。 如需如何取得此識別碼的相關資訊，請參閱：https://wiki.ucern.com/display/public/reference/Publishing+Identity+Data+Using+SCIM。 

5. 現在您可以設定 Azure AD 將使用者帳戶佈建至 Cerner。 登入 [Azure 入口網站](https://portal.azure.com)，然後瀏覽至 [Azure Active Directory] > [企業應用程式] > [所有應用程式] 區段。

6. 如果您已經設定 Cerner Central 單一登入，使用 [搜尋] 欄位搜尋您的 Cerner Central 執行個體。 否則，請選取 [新增]，並在應用程式庫中搜尋 [Cerner Central]。 從搜尋結果中選取 Cerner Central，並將它新增至您的應用程式清單。

7.  選取您的 Cerner Central 執行個體，然後選取 [佈建] 索引標籤。

8.  將 [佈建模式] 設定為 [自動]。

   ![Cerner Central 佈建](./media/cernercentral-provisioning-tutorial/Cerner.PNG)

9.  填寫 [系統管理員認證] 底下的下列欄位：

   * 在 [租用戶 URL] 欄位中，以下列格式輸入 URL，並將 "User-Roster-Realm-ID" 取代為您在步驟 4 中取得的領域識別碼。

> 沙箱：https://user-roster-api.sandboxcernercentral.com/scim/v1/Realms/User-Roster-Realm-ID/ 

> 生產環境：https://user-roster-api.cernercentral.com/scim/v1/Realms/User-Roster-Realm-ID/ 

   * 在 [祕密權杖] 欄位中，輸入您在步驟 3 中產生的 OAuth 持有人權杖，然後按一下 [測試連接]。

   * 您應該會在入口網站的右上方看到成功通知。

10. 在 [通知電子郵件] 欄位中輸入應收到佈建錯誤通知的個人或群組之電子郵件地址，然後勾選下列核取方塊。

11. 按一下 [檔案] 。 

12. 在 [屬性對應] 區段中，檢閱將從 Azure AD 同步處理至 Cerner Central 的使用者和群組屬性。 選取為 [比對] 屬性的屬性會用來比對 Cerner Central 中的使用者帳戶和群組以進行更新作業。 選取 [儲存] 按鈕以認可任何變更。

13. 若要啟用 Cerner Central 的 Azure AD 佈建服務，在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]

14. 按一下 [檔案] 。 

這會啟動在 [使用者和群組] 區段中指派給 Cerner Central 的任何使用者和/或群組之首次同步處理。 初始同步處理會比後續同步處理花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。 您可以使用 [同步處理詳細資料] 區段來監視進度，並依循連結前往佈建活動記錄，此記錄會表描述您 Cerner Central 應用程式上佈建服務所執行的所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../active-directory-saas-provisioning-reporting.md)。

## <a name="additional-resources"></a>其他資源

* [Cerner Central：使用 Azure AD 發佈身分識別資料](https://wiki.ucern.com/display/public/reference/Publishing+Identity+Data+Using+Azure+AD)
* [教學課程：設定 Cerner Central 搭配 Azure Active Directory 進行單一登入](cernercentral-tutorial.md)
* [管理企業應用程式的使用者帳戶佈建](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>後續步驟
* [了解如何針對佈建活動檢閱記錄和取得報告](https://docs.microsoft.com/azure/active-directory/active-directory-saas-provisioning-reporting)。
