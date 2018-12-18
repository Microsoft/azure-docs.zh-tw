---
title: 從 Azure AD App 資源庫設定 OpenID/OAuth 應用程式的步驟 | Microsoft Docs
description: 從 Azure AD App 資源庫設定 OpenID/OAuth 應用程式的步驟。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.assetid: eedebb76-e78c-428f-9cf0-5891852e79fb
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/25/2018
ms.author: jeedes
ms.openlocfilehash: 69e9d66458409bbc744416a58ceb508349418a76
ms.sourcegitcommit: 0fa8b4622322b3d3003e760f364992f7f7e5d6a9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37019548"
---
# <a name="steps-to-configure-an-openidoauth-application-from-azure-ad-app-gallery"></a>從 Azure AD App 資源庫設定 OpenID/OAuth 應用程式的步驟

## <a name="process-of-open-id-application-addition-from-gallery"></a>從資源庫新增 Open ID 應用程式的流程

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕](./media/openidoauth-tutorial/tutorial_general_01.png)

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗](./media/openidoauth-tutorial/tutorial_general_02.png)

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕](./media/openidoauth-tutorial/tutorial_general_03.png)

4. 在搜尋方塊中，輸入**應用程式名稱**，從結果面板中選取**所需的應用程式**，然後向應用程式註冊。

    ![新增應用程式](./media/openidoauth-tutorial/addfromgallery.png)

    > [!NOTE]
    > 對於 Open ID Connect 和 OAuth 應用程式，預設會停用 [新增] 按鈕。 租用戶系統管理員應該在這裡按一下 [註冊] 按鈕並為該應用程式提供同意。 這會將該應用程式新增到客戶租用戶，不需明確地新增並進行設定。

    ![[新增] 按鈕](./media/openidoauth-tutorial/addbutton.png)

5. 當您按一下註冊連結時，便會重新導向到 Azure AD 頁面以提供登入認證。

6. 成功驗證之後，使用者必須從同意頁面接受同意，在那之後，系統將會顯示應用程式首頁。

    > [!NOTE]
    > 客戶只能新增應用程式的一個執行個體。 如果您已經新增一個並嘗試再次提供同意，系統將不會再次將它新增至租用戶。 因此，邏輯上他們只能在租用戶中使用一個應用程式執行個體。

## <a name="authentication-flow-using-openid-connect"></a>使用 OpenID Connect 驗證流程

最基本的登入流程包含下列步驟 - 以下將詳細說明每一個步驟。

![使用 OpenID Connect 驗證流程](./media/openidoauth-tutorial/authenticationflow.png)

* **多租用戶應用程式** - 多租用戶應用程式可供許多組織使用，而不限單一組織。 這些通常是由獨立軟體廠商 (ISV) 撰寫的軟體即服務 (SaaS) 應用程式。 多租用戶應用程式需要佈建在將會用到它們的每個目錄中，而這需要使用者或系統管理員同意才能註冊。 當應用程式目錄中已在目錄中註冊並獲權存取 Graph API 或其他 Web API 時，此同意程序會啟動。 當不同組織的使用者或系統管理員註冊來使用應用程式時，他們會看到一個對話方塊顯示該應用程式需要的權限。 使用者或系統管理員可以同意應用程式，讓應用程式存取所述的資料，最後將應用程式註冊在他們的目錄中。

    > [!NOTE]
    > 如果要讓您的應用程式可供多個目錄中的使用者使用，則需要有機制來判斷使用者所在的租用戶。 單一租用戶應用程式只需要在它自己的目錄中查看使用者，但多租用戶應用程式需要從 Azure AD 的所有目錄中識別特定的使用者。 為了完成這項工作，Azure AD 提供一個共同驗證端點，供任何多租用戶應用程式引導登入要求，而非提供租用戶特定的端點。 針對 Azure AD 中的所有目錄，這個端點是 [https://login.microsoftonline.com/common](https://login.microsoftonline.com/common)，而租用戶特定的端點可能是 [https://login.microsoftonline.com/contoso.onmicrosoft.com](https://login.microsoftonline.com/contoso.onmicrosoft.com)。 開發您的應用程式時尤其必須考量共同端點，因為在登入、登出和權杖驗證期間，您需要必要的邏輯來處理多個租用戶。

Azure AD 小組預設會宣導使用多租用戶應用程式，因為它們可以輕鬆地跨各種組織進行存取，且在接受同意之後很容易使用。

## <a name="what-is-consent-framework"></a>什麼是同意架構？

Azure AD 的同意架構可讓您輕鬆地開發多租用戶的 Web 應用程式和原生用戶端應用程式。 這些應用程式允許與應用程式註冊所在的租用戶不同之 Azure AD 租用戶的使用者帳戶登入。 這些應用程式除了存取自己的 Web API 之外，可能也需要存取 Microsoft Graph API 之類的 Web API (用以存取 Azure Active Directory、Intune 及 Office 365 服務) 以及其他 Microsoft 服務的 API。 此架構以使用者或系統管理員為根據，他們可同意讓應用程式在他們的目錄中註冊，並可能包括存取目錄資料。 取得同意之後，用戶端應用程式將能代表使用者呼叫 Microsoft Graph API，並視需要使用資訊。

[Microsoft Graph API (英文)](https://graph.microsoft.io/) 可供存取 Office 365 中的資料 (例如 Exchange 中的行事曆和訊息、SharePoint 中的網站和清單、OneDrive 中的文件、OneNote 中的筆記本、Planner 中的工作、Excel 中的活頁簿等等)，以及 Azure AD 中的使用者和群組和多項 Microsoft 雲端服務中的其他資料物件。

下列步驟會示範如何將同意體驗用於應用程式開發人員和使用者。

1. 假設您擁有一個 Web 用戶端應用程式，它需要要求特定權限以存取資源/API。 Azure 入口網站會在設定期間用來宣告權限要求。 就和其他組態設定一樣，它們會成為應用程式之 Azure AD 註冊的一部分：

    ![圖形 API](./media/openidoauth-tutorial/graphapi.png)

2. 請考量您的應用程式權限是否已更新、是否正在執行應用程式，以及使用者是否即將第一次使用它。 首先，應用程式必須從 Azure AD 的 /authorize 端點取得授權碼。 授權碼接著可用來取得新的存取權和重新整理權杖。

3. 如果使用者尚未經過驗證，Azure AD 的 /authorize 端點就會提示使用者登入。

    ![驗證](./media/openidoauth-tutorial/authentication.png)

4. 使用者登入之後，Azure AD 會判斷是否需要向使用者顯示同意頁面。 此判斷根據使用者 (或其組織的系統管理員) 是否已經同意應用程式。 如果尚未授與同意，Azure AD 會提示使用者取得同意，並顯示其運作所需的必要權限。 同意對話方塊中顯示的權限集，會和在 Azure 入口網站上 [委派的權限] 中選取的權限相符。

    ![同意頁面](./media/openidoauth-tutorial/consentpage.png)

有些權限可以由一般使用者同意，有些則需要租用戶系統管理員的同意。

## <a name="whats-the-difference-between-admin-consent-and-user-consent"></a>系統管理員同意和使用者同意之間的差異為何？

身為系統管理員，您也可以代表租用戶中的所有使用者，同意應用程式的委派權限。 系統管理員同意可避免對租用戶中的每個使用者顯示同意對話方塊，並且可由具有系統管理員角色的使用者在 Azure 入口網站中完成。 在應用程式的 [設定] 頁面中，按一下 [必要權限]，然後按一下 [授與權限] 按鈕。

![授與權限](./media/openidoauth-tutorial/grantpermission.png)

> [!NOTE]
> 使用 ADAL.js 的單一頁面應用程式 (SPA) 目前必須使用 [授與權限] 按鈕來授與明確的同意。 否則，應用程式會在要求存取權杖時失敗。

僅限應用程式的權限一律需要租用戶系統管理員的同意。 如果您的應用程式要求僅限應用程式的權限，當使用者嘗試登入應用程式時，將會出現錯誤訊息，指出該使用者無法同意。

如果您的應用程式使用需要系統管理員同意的權限，您就必須要有相關的表示，例如可供系統管理員起始動作的按鈕或連結。 您的應用程式針對此動作傳送的要求是一般的 OAuth2/OpenID Connect 授權要求，其中也包含 prompt=admin_consent 查詢字串參數。 在系統管理員同意且系統在客戶的租用戶中建立服務主體之後，後續的登入要求就不需要 prompt=admin_consent 參數。 由於系統管理員已決定可接受要求的權限，因此從該時間點之後，就不會再提示租用戶中的任何其他使用者行使同意權。 租用戶系統管理員可以停用一般使用者對應用程式行使同意權的能力。 如果停用這項功能，就一律需要系統管理員同意，才能在租用戶中使用應用程式。 如果您想要在停用使用者同意的情況下測試應用程式，則可以在 [Azure 入口網站](https://portal.azure.com/)中 [企業應用程式] 下方的[使用者設定](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/UserSettings/menuId/)區段中找到設定參數。

要求的權限不需要系統管理員同意的應用程式，也可以使用 prompt=admin_consent 參數。 這項作業的使用時機範例如下：如果應用程式需要租用戶系統管理員「註冊」一次，之後就不會再提示其他使用者表示同意的情況。

如果應用程式需要系統管理員同意，但系統管理員是在未傳送 prompt=admin_consent 參數的情況下登入，則當該系統管理員順利同意該應用程式時，該同意只會套用至其使用者帳戶。 一般使用者將仍然無法登入此應用程式或對其行使同意權。 當您想要先讓租用戶系統管理員能夠瀏覽您的應用程式，然後才允許其他使用者存取時，這個功能相當有用。