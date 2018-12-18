---
title: 教學課程︰以 Azure Active Directory 設定 Asana 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 將使用者帳戶自動佈建及取消佈建至 Asana。
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
ms.date: 01/26/2018
ms.author: asmalser
ms.reviewer: asmalser
ms.openlocfilehash: 26642fefbb86b2709e110b13d782286fd18d5e60
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44348050"
---
# <a name="tutorial-configure-asana-for-automatic-user-provisioning"></a>教學課程︰設定 Asana 來自動佈建使用者

本教學課程旨在說明您需要在 Asana 和 Azure Active Directory (Azure AD) 中執行的步驟，以將使用者帳戶從 Azure AD 自動佈建和取消佈建至 Asana。

## <a name="prerequisites"></a>必要條件

本教學課程中說明的案例假設您已經具有下列項目：

*   Azure AD 租用戶
*   已啟用 [Enterprise 方案](https://www.asana.com/pricing)或更好方案的 Asana 租用戶 
*   Asana 中具有管理員權限的使用者帳戶 

> [!NOTE] 
> Azure AD 佈建整合需仰賴可供 Asana 使用的 [Asana API](https://asana.com/developers/api-reference/users)。

## <a name="assign-users-to-asana"></a>將使用者指派給 Asana

Azure AD 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者帳戶佈建的內容中，只有已指派至 Azure AD 中應用程式的使用者會進行同步處理。 

在您設定並啟用佈建服務之前，必須決定 Azure AD 中的哪些使用者需要存取 Asana 應用程式。 然後，您就可以遵循此處的指示，將這些使用者指派給 Asana 應用程式︰

[將使用者指派給企業應用程式](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-asana"></a>將使用者指派給 Asana 的重要秘訣

建議您將單一 Azure AD 使用者指派給 Asana 來測試佈建組態。 稍後可以指派其他使用者。

## <a name="configure-user-provisioning-to-asana"></a>設定將使用者佈建至 Asana 

本節將引導您將 Azure AD 連線至 Asana 使用者帳戶佈建 API。 您也會將佈建服務設定為根據 Azure AD 中的使用者指派，建立、更新和停用 Asana 中已指派的使用者帳戶。

> [!TIP]
> 若要啟用 Asana 的 SAML 型單一登入，請遵循 [Azure 入口網站](https://portal.azure.com)中提供的指示。 可以獨立設定自動佈建的單一登入，雖然這兩個功能彼此補充。

### <a name="to-configure-automatic-user-account-provisioning-to-asana-in-azure-ad"></a>若要在 Azure AD 中設定自動將使用者帳戶佈建至 Asana

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 [Azure Active Directory] > [企業應用程式] > [所有應用程式] 區段。

1. 如果您已經設定單一登入的 Asana，請使用 [搜尋] 欄位搜尋您的 Asana 執行個體。 否則，請選取 [新增]，並在應用程式庫中搜尋 [Asana]。 從搜尋結果中選取 [Asana]，並將它新增至您的應用程式清單。

1. 選取您的 Asana 執行個體，然後選取 [佈建] 索引標籤。

1. 將 [佈建模式] 設定為 [自動]。

    ![Asana 佈建](./media/asana-provisioning-tutorial/asanaazureprovisioning.png)

1. 在 [系統管理員認證] 區段下，請遵循以下指示來產生權杖，並在 [祕密權杖] 中輸入該權杖：

    a. 使用系統管理員帳戶登入 [Asana](https://app.asana.com)。

    b. 選取頂端列的設定檔照片，然後選取目前的組織名稱設定。

    c. 移至 [服務帳戶] 索引標籤。

    d. 選取 [新增服務帳戶]。

    e. 視需要更新 [名稱] 和 [關於] 及設定檔照片。 複製 [權杖] 中的權杖，然後在 [儲存變更] 中選取它。

1. 在 Azure 入口網站中，選取 [測試連線]，以確保 Azure AD 可以連線至您的 Asana 應用程式。 如果連線失敗，請確定您的 Asana 帳戶具有系統管理員權限，並再試一次**測試連線**步驟。

1. 在 [通知電子郵件] 中，輸入應收到佈建錯誤通知之個人或群組的電子郵件地址。 選取下方的核取方塊。

1. 選取 [ **儲存**]。 

1. 在 [對應] 區段中，選取 [將 Azure Active Directory 使用者同步至 Asana]。

1. 在 [屬性對應] 區段中，檢閱要從 Azure AD 同步至 Asana 的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 Asana 中的使用者帳戶以進行更新作業。 選取 [儲存] 認可任何變更。 如需詳細資訊，請參閱[自訂使用者佈建屬性對應](../manage-apps/customize-application-attributes.md)。

1. 若要啟用 Asana 的 Azure AD 佈建服務，請在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]。

1. 選取 [ **儲存**]。 

現在首次同步處理會針對在 [使用者] 區段中指派給 Asana 的任何使用者啟動。 初始同步處理會比後續同步處理花費更多時間執行，只要服務正在執行，這大約每 40 分鐘便會發生一次。 請使用 [同步處理詳細資料] 區段來監視進度，並依循連結來佈建活動記錄。 稽核記錄會描述在 Asana 應用程式上佈建服務執行的所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../manage-apps/check-status-user-account-provisioning.md)。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定單一登入](asana-tutorial.md)
