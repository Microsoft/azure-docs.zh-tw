---
title: 教學課程︰以 Azure Active Directory 設定 Slack 來自動佈建使用者 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 來自動佈建並取消佈建使用者帳戶至 Slack。
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
ms.reviewer: asmalser
ms.openlocfilehash: 9763c7a9e79f4c9e9d6296efb79e944205e8a99c
ms.sourcegitcommit: a5eb246d79a462519775a9705ebf562f0444e4ec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39264142"
---
# <a name="tutorial-configure-slack-for-automatic-user-provisioning"></a>教學課程︰設定 Slack 來自動佈建使用者


本教學課程旨在說明您需要在 Slack 和 Azure AD 中執行的步驟，以將使用者帳戶從 Azure AD 自動佈建和取消佈建至 Slack。 

## <a name="prerequisites"></a>必要條件

本教學課程中說明的案例假設您已經具有下列項目：

*   Azure Active Directory 租用戶
*   已啟用具有 [Plus 方案](https://aadsyncfabric.slack.com/pricing)或更高方案的 Slack 
*   具有小組系統管理員權限的 Slack 使用者帳戶 

注意︰Azure AD 佈建整合仰賴 [Slack SCIM API](https://api.slack.com/scim)，其可在 Plus 方案或更高方案中提供 Slack 小組。

## <a name="assigning-users-to-slack"></a>將使用者指派給 Slack

Azure Active Directory 會使用稱為「指派」的概念，來判斷哪些使用者應接收對指定應用程式的存取權。 在自動使用者帳戶佈建的內容中，只有已「指派」至 Azure AD 中的應用程式之使用者和群組會進行同步處理。 

在設定並啟用佈建服務之前，您必須決定 Azure AD 中的哪些使用者及/或群組代表需要 Slack 應用程式存取權的使用者。 一旦決定後，您可以依照此處的指示，將這些使用者指派給 Slack 應用程式︰

[將使用者或群組指派給企業應用程式](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-slack"></a>將使用者指派給 Slack 的重要秘訣

*   建議將單一 Azure AD 使用者指派給 Slack，以測試佈建的組態。 其他使用者及/或群組可能會稍後再指派。

*   將使用者指派給 Slack 時，您必須在指派對話方塊中選取**使用者**或「群組」角色。 「預設存取」角色不適用於佈建。


## <a name="configuring-user-provisioning-to-slack"></a>設定使用者佈建至 Slack 

本節會引導您將 Azure AD 連接至 Slack 的使用者帳戶佈建 API，以及根據 Azure AD 中的使用者和群組指派，設定佈建服務以在 Slack 中建立、更新和停用指派的使用者帳戶。

**秘訣：** 您也可以選擇啟用 Slack 的 SAML 型單一登入，請遵循 (Azure 入口網站)[https://portal.azure.com] 中提供的指示。 可以獨立設定自動佈建的單一登入，雖然這兩個功能彼此補充。


### <a name="to-configure-automatic-user-account-provisioning-to-slack-in-azure-ad"></a>若要在 Azure AD 中設定自動使用者帳戶佈建至 Slack︰


1)  在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 [Azure Active Directory > 企業應用程式 > 所有應用程式] 區段。

2) 如果您已經設定單一登入的 Slack，使用 [搜尋] 欄位搜尋您的 Slack 執行個體。 否則，請選取 [新增]，並在應用程式庫中搜尋 [Slack]。 從搜尋結果中選取 Slack，並將它新增至您的應用程式清單。

3)  選取您的 Slack 執行個體，然後選取 [佈建] 索引標籤。

4)  將 [佈建模式] 設定為 [自動]。

![Slack 佈建](./media/slack-provisioning-tutorial/Slack1.PNG)

5)  在 [系統管理員認證] 區段下，按一下 [授權]。 這會在新的瀏覽器視窗中開啟 Slack 授權對話方塊。 

6) 在新視窗中，使用您的小組系統管理帳戶登入 Slack。 在產生的授權對話方塊中，選取您想要啟用佈建的 Slack 小組，然後選取 [授權]。 一旦完成後，回到 Azure 入口網站以完成佈建組態。

![授權對話方塊](./media/slack-provisioning-tutorial/Slack3.PNG)

7) 在 Azure 入口網站中，按一下 [測試連接]以確保 Azure AD 可以連接到您的 Slack 應用程式。 如果連接失敗，請確定您的 Slack 帳戶具有小組系統管理員權限，並再試一次「授權」步驟。

8) 在 [通知電子郵件] 欄位中輸入應收到佈建錯誤通知的個人或群組之電子郵件地址，然後勾選下列核取方塊。

9) 按一下 [檔案] 。 

10) 在 [對應] 區段中，選取 [同步處理 Azure Active Directory 使用者至 Slack]。

11) 在 [屬性對應] 區段中，檢閱將從 Azure AD 同步至 Slack 的使用者屬性。 請注意，選取為 [比對] 屬性的屬性會用來比對 Slack 中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可任何變更。

12) 若要啟用 Slack 的 Azure AD 佈建服務，在 [設定]區段中，將 [佈建狀態] 變更為 [開啟]

13) 按一下 [檔案] 。 

這會啟動在 [使用者和群組] 區段中指派給 Slack 的任何使用者和/或群組之初始同步處理。 請注意，初始同步處理會比後續的同步處理花費較多時間執行，只要服務正在執行，這大約每 10 分鐘便會發生一次。 您可以使用 [同步處理詳細資料] 區段來監視進度，並遵循連結來佈建活動報告，其會描述您 Slack 應用程式上的佈建服務所執行之所有動作。

## <a name="optional-configuring-group-object-provisioning-to-slack"></a>[選用] 設定群組物件佈建至 Slack 

您可以選擇性地啟用從 Azure AD 佈建群組物件至 Slack。 這不同於「指派使用者群組」，因為除了其成員之外，實際群組物件會從 Azure AD 複寫至 Slack。 例如，如果您在 Azure AD 中有名為「我的群組」的群組，則會在 Slack 內建立名為「我的群組」的完全相同群組。

### <a name="to-enable-provisioning-of-group-objects"></a>若要啟用群組物件的佈建︰

1) 在 [對應] 區段中，選取 [同步處理 Azure Active Directory 群組至 Slack]。

2) 在 [屬性對應] 刀鋒視窗中，將 [已啟用] 設定為 [是]。

3) 在 [屬性對應] 區段中，檢閱將從 Azure AD 同步至 Slack 的群組屬性。 請注意，選取為 [比對] 屬性的屬性會用來比對 Slack 中的群組以進行更新作業。 

4) 按一下 [檔案] 。

這會造成在 [使用者和群組] 區段中指派至 Slack 的任何群組物件完全從 Azure AD 同步至 Slack。 您可以使用 [同步處理詳細資料] 區段來監視進度，並依循連結前往佈建活動記錄，此記錄會描述您 Slack 應用程式上佈建服務所執行的所有動作。

如需如何讀取 Azure AD 佈建記錄的詳細資訊，請參閱[關於使用者帳戶自動佈建的報告](../active-directory-saas-provisioning-reporting.md)。


## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
