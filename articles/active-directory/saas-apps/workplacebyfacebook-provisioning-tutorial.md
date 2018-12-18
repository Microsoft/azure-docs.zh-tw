---
title: 教學課程︰以 Azure Active Directory 設定 Workplace by Facebook 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Workplace by Facebook 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 6341e67e-8ce6-42dc-a4ea-7295904a53ef
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: 2b085d8b825055b5cb318da200eafb6d9000bf4b
ms.sourcegitcommit: 16ddc345abd6e10a7a3714f12780958f60d339b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36213098"
---
# <a name="tutorial-configure-workplace-by-facebook-for-automatic-user-provisioning"></a>教學課程：設定 Workplace by Facebook 來自動佈建使用者

本教學課程旨在說明您需要在 Workplace by Facebook 和 Azure AD 中執行的步驟，以將使用者帳戶從 Azure AD 自動佈建和取消佈建至 Workplace by Facebook。

## <a name="prerequisites"></a>先決條件

若要設定 Azure AD 與 Workplace by Facebook 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Workplace by Facebook 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="assigning-users-to-workplace-by-facebook"></a>將使用者指派給 Workplace by Facebook

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者帳戶佈建的內容中，只有「已指派」至 Azure AD 中的應用程式之使用者和群組會進行同步處理。

在設定並啟用佈建服務之前，您必須決定 Azure AD 中的哪些使用者及/或群組代表需要 Workplace by Facebook 應用程式存取權的使用者。 一旦決定後，您可以依照此處的指示，將這些使用者指派給 Workplace by Facebook 應用程式︰

[將使用者或群組指派給企業應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

### <a name="important-tips-for-assigning-users-to-workplace-by-facebook"></a>將使用者指派給 Workplace by Facebook 的重要秘訣

*   建議將單一 Azure AD 使用者指派給 Workplace by Facebook，以測試佈建設定。 其他使用者及/或群組可能會稍後再指派。

*   將使用者指派給 Workplace by Facebook 時，必須選取有效的使用者角色。 「預設存取」角色不適用於佈建。

## <a name="enable-user-provisioning"></a>啟用使用者佈建

本節會引導您將 Azure AD 連線至 Workplace by Facebook 的使用者帳戶佈建 API，以及根據 Azure AD 中的使用者和群組指派，設定佈建服務以在 Workplace by Facebook 中建立、更新和停用指派的使用者帳戶。

>[!Tip]
>您也可以選擇啟用 Workplace by Facebook 的 SAML 型單一登入，並遵循 [Azure 入口網站](https://portal.azure.com)中提供的指示。 可以獨立設定自動佈建的單一登入，雖然這兩個功能彼此補充。

### <a name="to-configure-user-account-provisioning-to-workplace-by-facebook-in-azure-ad"></a>若要在 Azure AD 中設定要佈建到 Workplace by Facebook 的使用者帳戶：

本節旨在說明如何對 Workplace by Facebook 啟用 Active Directory 使用者帳戶的佈建。

Azure AD 支援將指派之使用者的帳戶詳細資料自動同步處理至 Workplace by Facebook 的功能。 這個自動同步處理可讓 Workplace by Facebook 在使用者嘗試第一次登入之前，取得授權使用者存取所需的資料。 當存取在 Azure AD 中被撤銷時，它也會從 Workplace by Facebook 取消佈建使用者。

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 [Azure Active Directory] > [企業應用程式] > [所有應用程式] 區段。

2. 如果您已經設定單一登入的 Workplace by Facebook，使用 [搜尋] 欄位搜尋您的 Workplace by Facebook 執行個體。 否則，請選取 [新增]，並在應用程式庫中搜尋 [Workplace by Facebook]。 從搜尋結果中選取 Workplace by Facebook，並將它新增至您的應用程式清單。

3. 選取您的 Workplace by Facebook 執行個體，然後選取 [佈建] 索引標籤。

4. 將 [佈建模式] 設定為 [自動]。 

    ![佈建](./media/workplacebyfacebook-provisioning-tutorial/provisioning.png)

5. 在 [管理員認證] 區段下，輸入 Workplace by Facebook 系統管理員的 [存取權杖]，並且將 [租用戶 URL] 設定為 `https://www.facebook.com/scim/v1/`。 請參閱這些[指示](https://developers.facebook.com/docs/workplace/integrations/custom-integrations/apps)，以建立工作場所的存取權杖。 

6. 在 Azure 入口網站中，按一下 [測試連線]以確保 Azure AD 可以連線到您的 Workplace by Facebook 應用程式。 如果連線失敗，請確定您的 Workplace by Facebook 帳戶具有小組系統管理員權限。

7. 在 [通知電子郵件] 欄位中，輸入應收到佈建錯誤通知的個人或群組之電子郵件地址，然後勾選核取方塊。

8. 按一下 [儲存]。

9. 在 [對應] 區段中，選取 [同步處理 Azure Active Directory 使用者至 Workplace by Facebook]。

10. 在 [屬性對應] 區段中，檢閱從 Azure AD 同步至 Workplace by Facebook 的使用者屬性。 選取為 [比對] 屬性的屬性會用來比對 Workplace by Facebook 中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可任何變更。

11. 若要啟用 Workplace by Facebook 的 Azure AD 佈建服務，在 [設定] 區段中，將 [佈建狀態] 變更為 [開啟]

12. 按一下 [儲存]。

如需有關如何設定自動佈建的詳細資訊，請參閱 [https://developers.facebook.com/docs/facebook-at-work/provisioning/cloud-providers](https://developers.facebook.com/docs/facebook-at-work/provisioning/cloud-providers)。

您現在可以建立測試帳戶了。 請等候 20 分鐘以驗證帳戶已同步至 Workplace by Facebook。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定單一登入](workplacebyfacebook-tutorial.md)
