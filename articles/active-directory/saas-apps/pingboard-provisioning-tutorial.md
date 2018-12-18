---
title: 教學課程︰以 Azure Active Directory 設定 Pingboard 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 將使用者帳戶自動佈建和取消佈建至 Pingboard。
services: active-directory
documentationcenter: ''
author: asmalser-msft
writer: asmalser-msft
manager: sakula
ms.assetid: 0b38ee73-168b-42cb-bd8b-9c5e5126d648
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/19/2017
ms.author: asmalser
ms.reviewer: asmalser
ms.openlocfilehash: e96ea7d212f1a34bb6d10f8c49a15e1b34bfc469
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44348951"
---
# <a name="tutorial-configure-pingboard-for-automatic-user-provisioning"></a>教學課程︰設定 Pingboard 來自動佈建使用者

本教學課程旨在說明您需要遵循的步驟，以將使用者帳戶從 Azure Active Directory (Azure AD) 自動佈建和取消佈建至 Pingboard。

## <a name="prerequisites"></a>必要條件

本教學課程中說明的案例假設您已經具有下列項目：

*   Azure AD 租用戶
*   Pingboard 租用戶 [Pro 帳戶](https://pingboard.com/pricing) 
*   Pingboard 中具有管理員權限的使用者帳戶 

> [!NOTE] 
> Azure AD 佈建整合依存於可供您的帳戶使用的 [Pingboard API](https://pingboard.docs.apiary.io/#)。

## <a name="assign-users-to-pingboard"></a>將使用者指派給 Pingboard

Azure AD 會使用稱為「指派」的概念，來判斷哪些使用者應收到對所選應用程式的存取權。 在自動使用者帳戶佈建的內容中，只有已指派至 Azure AD 中應用程式的使用者會進行同步處理。 

在您設定並啟用佈建服務之前，必須決定 Azure AD 中的哪些使用者需要存取 Pingboard 應用程式。 然後，您就可以遵循此處的指示，將這些使用者指派給 Pingboard 應用程式︰

[將使用者指派給企業應用程式](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-pingboard"></a>將使用者指派給 Pingboard 的重要秘訣

建議您將單一 Azure AD 使用者指派給 Pingboard 來測試佈建組態。 稍後可以指派其他使用者。

## <a name="configure-user-provisioning-to-pingboard"></a>設定將使用者佈建至 Pingboard 

本節將引導您將 Azure AD 連線至 Pingboard 使用者帳戶佈建 API。 您也會將佈建服務設定為根據 Azure AD 中的使用者指派，建立、更新和停用 Pingboard 中已指派的使用者帳戶。

> [!TIP]
> 若要啟用 Pingboard 的 SAML 型單一登入，請遵循 [Azure 入口網站](https://portal.azure.com)中提供的指示。 可以獨立設定自動佈建的單一登入，雖然這兩個功能彼此補充。

### <a name="to-configure-automatic-user-account-provisioning-to-pingboard-in-azure-ad"></a>若要在 Azure AD 中設定將使用者帳戶自動佈建至 Pingboard：

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 [Azure Active Directory] > [企業應用程式] > [所有應用程式] 區段。

1. 如果您已設定單一登入的 Pingboard，請使用搜尋欄位來搜尋您的 Pingboard 執行個體。 否則，請選取 [新增]，並在應用程式資源庫中搜尋 [Pingboard]。 從搜尋結果中選取 [Pingboard]，並將它新增至您的應用程式清單。

1. 選取您的 Pingboard 執行個體，然後選取 [佈建] 索引標籤。

1. 將 [佈建模式] 設定為 [自動]。

    ![Pingboard 佈建](./media/pingboard-provisioning-tutorial/pingboardazureprovisioning.png)
    
1. 在 [管理員認證] 區段底下，使用下列步驟：

    a. 在 [租用戶 URL] 中輸入 `https://your_domain.pingboard.com/scim/v2`，並將 "your_domain" 取代為您實際的網域。

    b. 使用系統管理員帳戶登入 [Pingboard](https://pingboard.com/)。

    c. 選取 [附加元件] > [整合] > [Azure Active Directory]。

    d. 移至 [設定] 索引標籤，然後選取 [啟用從 Azure 進行使用者佈建]。

    e. 複製 [OAuth 持有人權杖] 中的權杖，並在 [祕密權杖] 中輸入此權杖。

1. 在 Azure 入口網站中，選取 [測試連線] 以測試 Azure AD 是否可以連線到您的 Pingboard 應用程式。 如果連線失敗，請測試您的 Pingboard 帳戶是否具有系統管理員權限，並再試一次**測試連線**步驟。

1. 在 [通知電子郵件] 中，輸入應收到佈建錯誤通知之個人或群組的電子郵件地址。 選取下方的核取方塊。

1. 選取 [ **儲存**]。 

1. 在 [對應] 區段之下，選取 [將 Azure Active Directory 使用者同步至 Pingboard]。

1. 在 [屬性對應] 區段中，檢閱要從 Azure AD 同步至 Pingboard 的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 Pingboard 中的使用者帳戶，以進行更新作業。 選取 [儲存] 認可任何變更。 如需詳細資訊，請參閱[自訂使用者佈建屬性對應](../manage-apps/customize-application-attributes.md)。

1. 若要啟用 Pingboard 的 Azure AD 佈建服務，請在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]。

1. 選取 [儲存] 以啟動對指派給 Pingboard 的使用者進行首次同步處理。

初始同步處理會比後續同步處理花費更多時間執行，只要服務正在執行，這大約每 40 分鐘便會發生一次。 請使用 [同步處理詳細資料] 區段來監視進度，並依循連結來佈建活動記錄。 這些記錄會描述在 Pingboard 應用程式上佈建服務採取的所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../manage-apps/check-status-user-account-provisioning.md)。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定單一登入](pingboard-tutorial.md)
