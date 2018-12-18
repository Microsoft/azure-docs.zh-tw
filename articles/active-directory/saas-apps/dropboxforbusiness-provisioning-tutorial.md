---
title: 教學課程︰以 Azure Active Directory 設定 Dropbox 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Dropbox for Business 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 0f3a42e4-6897-4234-af84-b47c148ec3e1
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: 8e61fe4af83ee72df74027b2e52a3e81db486798
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44347302"
---
# <a name="tutorial-configure-dropbox-for-business-for-automatic-user-provisioning"></a>教學課程︰設定 Dropbox for Business 來自動佈建使用者

本教學課程旨在說明您需要在 Dropbox for Business 和 Azure AD 中執行的步驟，以將使用者帳戶從 Azure AD 自動佈建和取消佈建至 Dropbox for Business。

## <a name="prerequisites"></a>必要條件

本教學課程中說明的案例假設您已經具有下列項目：

*   Azure Active Directory 租用戶。
*   已啟用 Dropbox for Business 單一登入的訂用帳戶。
*   具有小組系統管理員權限的 Dropbox for Business 使用者帳戶。

## <a name="assigning-users-to-dropbox-for-business"></a>將使用者指派至 Dropbox for Business

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者帳戶佈建的內容中，只有「已指派」至 Azure AD 中的應用程式之使用者和群組會進行同步處理。

在設定並啟用佈建服務之前，您必須決定 Azure AD 中的哪些使用者及/或群組代表需要 Dropbox for Business 應用程式存取權的使用者。 一旦決定後，您可以依照此處的指示，將這些使用者指派給 Dropbox for Business 應用程式︰

[將使用者或群組指派給企業應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

### <a name="important-tips-for-assigning-users-to-dropbox-for-business"></a>將使用者指派給 Dropbox for Business 的重要秘訣

*   建議將單一 Azure AD 使用者指派給 Dropbox for Business，以測試佈建設定。 其他使用者及/或群組可能會稍後再指派。

*   將使用者指派給 Dropbox for Business 時，必須選取有效的使用者角色。 「預設存取」角色不適用於佈建.

## <a name="enable-automated-user-provisioning"></a>啟用自動使用者佈建

本節會引導您將 Azure AD 連線至 Dropbox for Business 的使用者帳戶佈建 API，以及根據 Azure AD 中的使用者和群組指派，設定佈建服務以在 Dropbox for Business 中建立、更新和停用指派的使用者帳戶。

>[!Tip]
>您也可以選擇啟用 Dropbox for Business 的 SAML 型單一登入，請遵循 [Azure 入口網站](https://portal.azure.com)中提供的指示。 可以獨立設定自動佈建的單一登入，雖然這兩個功能彼此補充。

### <a name="to-configure-automatic-user-account-provisioning"></a>若要設定自動使用者帳戶佈建：

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 [Azure Active Directory > 企業應用程式 > 所有應用程式] 區段。

2. 如果您已經設定單一登入的 Dropbox for Business，請使用 [搜尋] 欄位搜尋您的 Dropbox for Business 執行個體。 否則，請選取 [新增]，並在應用程式庫中搜尋 [Dropbox for Business]。 從搜尋結果中選取 Dropbox for Business，並將它新增至您的應用程式清單。

3. 選取您的 Dropbox for Business 執行個體，然後選取 [佈建] 索引標籤。

4. 將 [佈建模式] 設定為 [自動]。 

    ![佈建](./media/dropboxforbusiness-provisioning-tutorial/provisioning.png)

5. 在 [系統管理員認證] 區段下，按一下 [授權]。 會在新的瀏覽器視窗中開啟 Dropbox for Business 對話方塊。

6. 在 [登入 Dropbox 來與 Azure AD 連結]  對話方塊上，登入您的 Dropbox for Business 租用戶。

     ![使用者佈建](./media/dropboxforbusiness-provisioning-tutorial/ic769518.png "使用者佈建")

7. 確認您需要授與 Azure Active Directory 權限來變更您的 Dropbox for Business 租用戶。 按一下 [允許]。
    
      ![使用者佈建](./media/dropboxforbusiness-provisioning-tutorial/ic769519.png "使用者佈建")

8. 在 Azure 入口網站中，按一下 [測試連線]以確保 Azure AD 可以連線到您的 Dropbox for Business 應用程式。 如果連線失敗，請確定您的 Dropbox for Business 帳戶具有小組系統管理員權限，並再試一次 **「授權」** 步驟。

9. 在 [通知電子郵件] 欄位中，輸入應收到佈建錯誤通知的個人或群組之電子郵件地址，然後勾選核取方塊。

10. 按一下 [儲存]。

11. 在 [對應] 區段中，選取 [同步處理 Azure Active Directory 使用者至 Dropbox for Business]。

12. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 Dropbox for Business 的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 Dropbox for Business 中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可任何變更。

13. 若要啟用 Dropbox for Business 的 Azure AD 佈建服務，在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]

14. 按一下 [儲存]。

這會啟動在 [使用者和群組] 區段中指派給 Dropbox for Business 的任何使用者和/或群組之首次同步處理。 初始同步處理會比後續同步處理花費更多時間執行，只要服務正在執行，這大約每 40 分鐘便會發生一次。 您可以使用 [同步處理詳細資料] 區段來監視進度，並依循連結前往佈建活動記錄，此記錄會描述您 Dropbox for Business 應用程式上佈建服務所執行的所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../manage-apps/check-status-user-account-provisioning.md)。


## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定單一登入](dropboxforbusiness-tutorial.md)