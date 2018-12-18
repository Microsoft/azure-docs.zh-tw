---
title: 教學課程︰以 Azure Active Directory 設定 Salesforce 沙箱來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Salesforce Sandbox 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: bab73fda-6754-411d-9288-f73ecdaa486d
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: b1ad53c4ba1b79a1918177ce73862b603d157153
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44346495"
---
# <a name="tutorial-configure-salesforce-sandbox-for-automatic-user-provisioning"></a>教學課程︰設定 Salesforce 沙箱來自動佈建使用者

本教學課程旨在說明您需要在 Salesforce 沙箱和 Azure AD 中執行的步驟，以將使用者帳戶從 Azure AD 自動佈建和取消佈建至 Salesforce 沙箱。

## <a name="prerequisites"></a>必要條件

本教學課程中說明的案例假設您已經具有下列項目：

*   Azure Active Directory 租用戶。
*   Salesforce Sandbox for Work 或 Salesforce Sandbox for Education 的有效租用戶。 您可以使用免費試用帳戶來使用任何服務。
*   具有小組系統管理員權限的 Salesforce 沙箱使用者帳戶。

## <a name="assigning-users-to-salesforce-sandbox"></a>將使用者指派給 Salesforce 沙箱

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者帳戶佈建的內容中，只有「已指派」至 Azure AD 中的應用程式之使用者和群組會進行同步處理。

在設定並啟用佈建服務之前，您必須決定 Azure AD 中的哪些使用者或群組需要 Salesforce 沙箱應用程式的存取權。 完成決定後，可以遵循[將使用者或群組指派給企業應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)中的指示，將這些使用者指派給 Salesforce 沙箱應用程式

### <a name="important-tips-for-assigning-users-to-salesforce-sandbox"></a>將使用者指派給 Salesforce 沙箱的重要秘訣

* 建議將單一 Azure AD 使用者指派給 Salesforce 沙箱，以測試佈建設定。 其他使用者及/或群組可能會稍後再指派。

* 將使用者指派給 Salesforce 沙箱時，必須選取有效的使用者角色。 「預設存取」角色不適用於佈建。

> [!NOTE]
> 此應用程式在佈建流程中，會從 Salesforce 沙箱匯入自訂角色，客戶在指派使用者時也可以選取該角色。

## <a name="enable-automated-user-provisioning"></a>啟用自動使用者佈建

本節會引導您將 Azure AD 連接至 Salesforce 沙箱的使用者帳戶佈建 API，以及根據 Azure AD 中的使用者和群組指派，設定佈建服務以在 Salesforce 沙箱中建立、更新和停用指派的使用者帳戶。

>[!Tip]
>您也可以選擇啟用 Salesforce 沙箱的 SAML 型單一登入，請遵循 [Azure 入口網站](https://portal.azure.com)中提供的指示。 可以獨立設定自動佈建的單一登入，雖然這兩個功能彼此補充。

### <a name="configure-automatic-user-account-provisioning"></a>設定使用者帳戶自動佈建

本節的目的是要說明如何對 Salesforce 沙箱啟用 Active Directory 使用者帳戶的使用者佈建。

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 [Azure Active Directory > 企業應用程式 > 所有應用程式] 區段。

1. 如果您已經設定單一登入的 Salesforce 沙箱，請使用 [搜尋] 欄位搜尋您的 Salesforce 沙箱執行個體。 否則，請選取 [新增]，並在應用程式庫中搜尋 [Salesforce 沙箱]。 從搜尋結果中選取 Salesforce 沙箱，並將它新增至您的應用程式清單。

1. 選取您的 Salesforce 沙箱執行個體，然後選取 [佈建] 索引標籤。

1. 將 [佈建模式] 設定為 [自動]。

    ![佈建](./media/salesforce-sandbox-provisioning-tutorial/provisioning.png)

1. 在 [管理員認證] 區段下，提供下列組態設定：
   
    a. 在 [管理使用者名稱] 文字方塊中，輸入已在 Salesforce.com 中指派**系統管理員**設定檔的 Salesforce 沙箱帳戶名稱。
   
    b. 在 [管理員密碼] 文字方塊中，輸入這個帳戶的密碼。

1. 若要取得您的 Salesforce 沙箱安全性權杖，請開啟新索引標籤並登入相同的 Salesforce 沙箱管理帳戶。 在頁面右上角，按一下您的名稱，然後按一下 [設定]。

     ![啟用自動使用者佈建](./media/salesforce-sandbox-provisioning-tutorial/sf-my-settings.png "啟用自動使用者佈建")

1. 在左方導覽窗格上，按一下 [我的個人資訊] 以展開相關的區段，然後按一下 [重設我的安全性權杖]。
  
    ![啟用自動使用者佈建](./media/salesforce-sandbox-provisioning-tutorial/sf-personal-reset.png "啟用自動使用者佈建")

1. 在 [重設安全性權杖] 頁面上，按一下 [重設安全性權杖] 按鈕。

    ![啟用自動使用者佈建](./media/salesforce-sandbox-provisioning-tutorial/sf-reset-token.png "啟用自動使用者佈建")

1. 檢查與此系統管理員帳戶相關聯的電子郵件收件匣。 尋找來自 Salesforce Sandbox.com，包含新安全性權杖的電子郵件。

1. 複製該權杖，移至您的 Azure AD 視窗，然後將它貼到 [祕密權杖] 欄位。

1. 在 Azure 入口網站中，按一下 [測試連接]，以確保 Azure AD 可以連接到您的 Salesforce 沙箱應用程式。

1. 在 [通知電子郵件] 欄位中輸入應收到佈建錯誤通知的個人或群組之電子郵件地址，然後勾選核取方塊。

1. 按一下 [儲存]。  
    
1.  在 [對應] 區段中，選取 [同步處理 Azure Active Directory 使用者至 Salesforce 沙箱]。

1. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 Salesforce 沙箱的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 Salesforce 沙箱中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可任何變更。

1. 若要啟用 Salesforce 沙箱的 Azure AD 佈建服務，請在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]

1. 按一下 [儲存]。

這會啟動在 [使用者和群組] 區段中指派給 Salesforce 沙箱的任何使用者和/或群組之首次同步處理。 初始同步處理會比後續同步處理花費更多時間執行，只要服務正在執行，這大約每 40 分鐘便會發生一次。 您可以使用 [同步處理詳細資料] 區段來監視進度，並依循連結前往佈建活動記錄，此記錄會描述 Salesforce Sandbox 應用程式上佈建服務所執行的所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../manage-apps/check-status-user-account-provisioning.md)。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定單一登入](https://docs.microsoft.com/azure/active-directory/active-directory-saas-salesforce-sandbox-tutorial)
