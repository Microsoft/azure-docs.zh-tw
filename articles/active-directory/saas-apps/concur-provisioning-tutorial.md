---
title: 教學課程︰以 Azure Active Directory 設定 Concur 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Concur 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: df47f55f-a894-4e01-a82e-0dbf55fc8af1
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: 5832444cd30d60f7b5fe7fe6108acd5604389474
ms.sourcegitcommit: 16ddc345abd6e10a7a3714f12780958f60d339b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36210100"
---
# <a name="tutorial-configure-concur-for-automatic-user-provisioning"></a>教學課程︰設定 Concur 來自動佈建使用者

本教學課程旨在說明您需要在 Concur 和 Azure AD 中執行的步驟，以將使用者帳戶從 Azure AD 自動佈建和取消佈建至 Concur。

## <a name="prerequisites"></a>先決條件

本教學課程中說明的案例假設您已經具有下列項目：

*   Azure Active Directory 租用戶。
*   已啟用 Concur 單一登入的訂用帳戶。
*   具有小組系統管理員權限的 Concur 使用者帳戶。

## <a name="assigning-users-to-concur"></a>將使用者指派給 Concur

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者帳戶佈建的內容中，只有「已指派」至 Azure AD 中的應用程式之使用者和群組會進行同步處理。

在設定並啟用佈建服務之前，您必須決定 Azure AD 中的哪些使用者及/或群組代表需要 Concur 應用程式存取權的使用者。 一旦決定後，您可以依照此處的指示，將這些使用者指派給 Concur 應用程式︰

[將使用者或群組指派給企業應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

### <a name="important-tips-for-assigning-users-to-concur"></a>將使用者指派給 Concur 的重要秘訣

*   建議將單一 Azure AD 使用者指派給 Concur，以測試佈建的組態。 其他使用者及/或群組可能會稍後再指派。

*   將使用者指派給 Concur 時，您必須選取有效的使用者角色。 「預設存取」角色不適用於佈建。

## <a name="enable-user-provisioning"></a>啟用使用者佈建

本節會引導您將 Azure AD 連接至 Concur 的使用者帳戶佈建 API，以及根據 Azure AD 中的使用者和群組指派，設定佈建服務以在 Concur 中建立、更新和停用指派的使用者帳戶。

> [!Tip] 
> 您也可以選擇啟用 Concur 的 SAML 型單一登入，請遵循 [Azure 入口網站](https://portal.azure.com)中提供的指示。 可以獨立設定自動佈建的單一登入，雖然這兩個功能彼此補充。

### <a name="to-configure-user-account-provisioning"></a>若要設定使用者帳戶佈建：

本節的目的是要說明如何對 Concur 啟用 Active Directory 使用者帳戶的佈建。

若要啟用 Expense Service 中的應用程式，您必須適當地設定及使用 Web 服務系統管理員設定檔， 而不要將 WS 系統管理員角色加入用於 T&E 系統管理功能的現有系統管理員設定檔。

Concur 顧問或用戶端系統管理員必須建立不同的 Web 服務系統管理員設定檔，而且用戶端系統管理員必須使用這個設定檔來執行 Web 服務系統管理員功能 (例如，啟用應用程式)。 這些設定檔必須與用戶端系統管理員的每日 T&E 系統管理員設定檔分開保存 (T&E 系統管理員設定檔不應該指派 WS 系統管理員角色)。

當您建立要用於啟用應用程式的設定檔時，請在使用者設定檔欄位中，輸入用戶端系統管理員的名稱。 這會對設定檔指派擁有權。 在建立一或多個設定檔之後，用戶端必須使用這個設定檔登入，並針對合作夥伴應用程式，按一下 [Web 服務] 功能表中的 [啟用] 按鈕。

基於下列原因，不應該透過用於一般 T&E 系統管理的設定檔完成這項動作。

* 用戶端必須在啟用應用程式之後所顯示的對話視窗中，按一下 [是]。 這個點選動作可確認用戶端是否願意讓合作夥伴應用程式存取其資料，因此您或合作夥伴無法按 [是] 按鈕。

* 如果使用 T&E 系統管理員設定檔啟用應用程式的用戶端系統管理員已離職 (導致設定檔被停用)，在使用另一個現用 WS 系統管理員設定檔啟用應用程式之前，使用該設定檔啟用的所有應用程式都無法正常運作。 因此，您必須建立不同的 WS 系統管理員設定檔。

* 如果系統管理員離職，與 WS 系統管理員設定檔關聯的名稱可視需要變更為續任的系統管理員，由於不需要停用該設定檔，因此不會影響已啟用的應用程式。

**若要設定使用者佈建，請執行下列步驟：**

1. 登入您的 **Concur** 租用戶。

2. 從 [管理] 功能表中，選取 [Web 服務]。
   
    ![Concur 租用戶](./media/concur-provisioning-tutorial/IC721729.png "Concur 租用戶")

3. 從左側的 [Web 服務] 窗格中，選取 [啟用合作夥伴應用程式]。
   
    ![啟用合作夥伴應用程式](./media/concur-provisioning-tutorial/ic721730.png "啟用合作夥伴應用程式")

4. 從 [啟用應用程式] 清單中選取 [Azure Active Directory]，然後按一下 [啟用]。
   
    ![Microsoft Azure Active Directory](./media/concur-provisioning-tutorial/ic721731.png "Microsoft Azure Active Directory")

5. 按一下 [是] 關閉 [確認動作] 對話方塊。
   
    ![確認動作](./media/concur-provisioning-tutorial/ic721732.png "確認動作")

6. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 [Azure Active Directory > 企業應用程式 > 所有應用程式] 區段。

7. 如果您已經設定單一登入的 Concur，使用 [搜尋] 欄位搜尋您的 Concur 執行個體。 否則，請選取 [新增]，並在應用程式庫中搜尋 [Concur]。 從搜尋結果中選取 Concur，並將它新增至您的應用程式清單。

8. 選取您的 Concur 執行個體，然後選取 [佈建] 索引標籤。

9. 將 [佈建模式] 設定為 [自動]。 
 
    ![佈建](./media/concur-provisioning-tutorial/provisioning.png)

10. 在 [管理員認證] 區段下，輸入 Concur 系統管理員的**使用者名稱**和**密碼**。

11. 在 Azure 入口網站中，按一下 [測試連接]以確保 Azure AD 可以連接到您的 Concur 應用程式。 如果連接失敗，請確定您的 Concur 帳戶具有小組系統管理員權限。

12. 在 [通知電子郵件] 欄位中輸入應收到佈建錯誤通知的個人或群組之電子郵件地址，然後勾選核取方塊。

13. 按一下 [儲存]。

14. 在 [對應] 區段中，選取 [同步處理 Azure Active Directory 使用者至 Concur]。

15. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 Concur 的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 Concur 中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可任何變更。

16. 若要啟用 Concur 的 Azure AD 佈建服務，在 [設定]區段中，將 [佈建狀態] 變更為 [開啟]

17. 按一下 [儲存]。

您現在可以建立測試帳戶。 請等候 20 分鐘以確認帳戶已同步至 Concur。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定單一登入](concur-tutorial.md)

