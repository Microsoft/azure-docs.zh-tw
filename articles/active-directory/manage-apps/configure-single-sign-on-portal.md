---
title: 設定單一登入 - Azure Active Directory | Microsoft Docs
description: 本教學課程會使用 Azure 入口網站為應用程式設定透過 Azure Active Directory (Azure AD) 進行的 SAML 型單一登入。
services: active-directory
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.topic: tutorial
ms.workload: identity
ms.date: 08/09/2018
ms.author: barbkess
ms.reviewer: arvinh,luleon
ms.openlocfilehash: b0180f162996c5fc4647071feaf02d42320b7c9a
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40036283"
---
# <a name="tutorial-configure-saml-based-single-sign-on-for-an-application-with-azure-active-directory"></a>教學課程：為應用程式設定透過 Azure Active Directory 進行的 SAML 型單一登入

本教學課程會使用 [Azure 入口網站](https://portal.azure.com)為應用程式設定透過 Azure Active Directory (Azure AD) 進行的 SAML 型單一登入。 請使用本教學課程來設定沒有[應用程式專用教學課程](../saas-apps/tutorial-list.md)的應用程式。 


本教學課程會使用 Azure 入口網站執行下列動作：

> [!div class="checklist"]
> * 選取 SAML 型單一登入模式
> * 設定應用程式特定網域和 URL
> * 設定使用者屬性
> * 建立 SAML 簽署憑證
> * 將使用者指派給應用程式
> * 設定應用程式使用 SAML 型單一登入
> * 測試 SAML 設定

## <a name="before-you-begin"></a>開始之前

1. 如果應用程式未新增至您的 Azure AD 租用戶，請參閱[快速入門：將應用程式新增至 Azure AD 租用戶](add-application-portal.md)。

2. 要求您的應用程式廠商提供[設定網域和 URL](#configure-domain-and-urls)中所說明的資訊。

3. 若要測試本教學課程中的步驟，我們建議您使用非生產環境。 如果您沒有 Azure AD 的非生產環境，您可以[取得為期一個月的試用版](https://azure.microsoft.com/pricing/free-trial/)。

4. 以 Azure AD 租用戶全域管理員、雲端應用程式系統管理員或應用程式系統管理員的身分登入 [Azure 入口網站](https://portal.azure.com)。

## <a name="select-a-single-sign-on-mode"></a>選取單一登入模式

將應用程式新增至 Azure AD 租用戶之後，您即可為應用程式設定單一登入。

若要開啟單一登入設定：

1. 在 [Azure 入口網站](https://portal.azure.com)的左方瀏覽窗格中，按一下 [Azure Active Directory]。 

2. 在 [Azure Active Directory] 刀鋒視窗中，按一下 [企業應用程式]。 [所有應用程式] 刀鋒視窗會隨即開啟，並顯示 Azure AD 租用戶中應用程式的隨機樣本。 

3. 在 [應用程式類型] 功能表中，選取 [所有應用程式]，然後按一下 [套用]。

4. 輸入要設定單一登入的應用程式名稱。 您可以選擇自己的應用程式，或使用在[新增應用程式](add-application-portal.md)快速入門中新增的 GitHub 測試應用程式。

5. 按一下 [單一登入]。 在 [單一登入模式] 下方，[SAML 型登入] 會顯示為預設選項。 

    ![組態選項](media/configure-single-sign-on-portal/config-options.png)

6. 按一下刀鋒視窗頂端的 [儲存] 。 

## <a name="configure-domain-and-urls"></a>設定網域和 URL

若要設定網域和 URL：

1. 與應用程式廠商連絡，以取得下列設定的正確資訊：

    | 組態設定 | SP 起始 | idP 起始 | 說明 |
    |:--|:--|:--|:--|
    | 登入 URL | 必要 | 請勿指定 | 當使用者開啟此 URL 時，服務提供者會重新導向至 Azure AD 以進行驗證，並將使用者登入。 Azure AD 會使用此 URL 從 Office 365 和 Azure AD 存取面板中啟動應用程式。 如果空白，則在使用者從 Office 365、Azure AD 存取面板或 Azure AD 單一登入 URL 啟動應用程式時，Azure AD 會執行 idP 起始的單一登入。|
    | 識別碼 (實體識別碼) | 某些應用程式需要 | 某些應用程式需要 | 可唯一識別要設定單一登入的應用程式。 Azure AD 會將識別碼傳回至應用程式作為 SAML 權杖的 Audience 參數，且應用程式應會加以驗證。 在應用程式中提供的任何 SAML 中繼資料中，此值也會顯示為實體識別碼。|
    | 回覆 URL | 選用 | 必要 | 指定應用程式預期要接收 SAML 權杖的位置。 此回覆 URL 也稱為判斷提示取用者服務 (ACS) URL。 |
    | 轉送狀態 | 選用 | 選用 | 對應用程式指定在驗證完成後應將使用者重新導向到的位置。 此值通常是用程式有效的 URL，但有些應用程式會以不同方式使用此欄位。 如需詳細資訊，請詢問應用程式廠商。

2. 輸入資訊。 若要查看所有設定，請按一下 [顯示進階 URL 設定]。

    ![組態選項](media/configure-single-sign-on-portal/config-urls.png)

3. 在刀鋒視窗頂端，按一下 [儲存]。

4. 此區段中會出現 [測試 SAML 設定] 按鈕。 請在本教學課程稍後[測試單一登入](#test-single-sign-on)一節中執行此測試。

## <a name="configure-user-attributes"></a>設定使用者屬性

使用者屬性可讓您控制 Azure AD 會將哪些資訊傳送至應用程式。 例如，Azure AD 可將使用者的名稱、電子郵件和員工識別碼傳送至應用程式。 Azure AD 會在每次使用者登入時，在 SAML 權杖中將使用者屬性傳送至應用程式。 

這些屬性可能是單一登入正常運作所需的必要或選擇性屬性。 如需詳細資訊，請參閱[應用程式專用教學課程](../saas-apps/tutorial-list.md)，或洽詢應用程式廠商。

1. 若要查看所有選項，請按一下 [檢視及編輯所有其他使用者屬性]。

    ![設定使用者屬性](media/configure-single-sign-on-portal/config-user-attributes.png)

2. 輸入 [使用者識別碼]。

    使用者識別碼可唯一識別應用程式內的每個使用者。 例如，如果電子郵件地址同時是使用者名稱和唯一識別碼，請將此值設定為 *user.mail*。

3. 如需更多 SAML 權杖屬性，請按一下 [檢視及編輯所有其他使用者屬性]。

4. 若要將屬性新增至 [SAML 權杖屬性]，請按一下 [新增屬性]。 輸入 [名稱]，然後從功能表中選取 [值]。

5. 按一下 [檔案] 。 您會在資料表中看到新屬性。
 
## <a name="create-a-saml-signing-certificate"></a>建立 SAML 簽署憑證

Azure AD 會使用憑證來簽署它傳送至應用程式的 SAML 權杖。 

1. 若要查看所有選項，請按一下 [顯示進階憑證簽署選項]。

    ![設定憑證](media/configure-single-sign-on-portal/config-certificate.png)

2. 若要設定憑證，請按一下 [建立新憑證]。

3. 在 [建立新憑證] 刀鋒視窗中設定到期日，然後按一下 [儲存]。

4. 按一下 [使新憑證成為使用中]。

5. 若要深入了解，請查看 [進階憑證簽署設定](certificate-signing-options.md)。

6. 若要保留您到目前為止所做的變更，請務必按一下 [單一登入] 刀鋒視窗頂端的 [儲存]。 

## <a name="assign-users-to-the-application"></a>將使用者指派給應用程式

Microsoft 建議您先對數個使用者或群組測試單一登入，再對您的組織推出應用程式。

若要將使用者或群組指派給應用程式：

1. 在入口網站中開啟應用程式 (如果尚未開啟)。
2. 在左側的應用程式刀鋒視窗中，按一下 [使用者和群組]。
3. 按一下 [新增使用者] 。
4. 在 [新增指派] 刀鋒視窗中，按一下 [使用者和群組]。
5. 若要尋找特定的使用者，請在 [選取] 中輸入使用者名稱，按一下 使用者設定檔相片或標誌旁的核取方塊，然後按一下 [選取]。 
6. 尋找您目前的使用者名稱，並加以選取。 您可以選擇性地選取更多使用者。
7. 在 [新增指派] 刀鋒視窗中，按一下 [指派]。 作業完成後，選取的使用者即會出現在 [使用者和群組] 清單中。

## <a name="configure-the-application-to-use-azure-ad"></a>將應用程式設定為使用 Azure AD

就快要完成了。  在最後一個步驟中，您必須將應用程式設定成使用 Azure AD 作為 SAML 身分識別提供者。 

1. 向下捲動至應用程式的 [單一登入] 刀鋒視窗結尾處。 

    ![設定應用程式](media/configure-single-sign-on-portal/configure-app.png)

2. 在入口網站中按一下 [設定應用程式]，並依照指示操作。
3. 在應用程式中手動建立要用來測試單一登入的使用者帳戶。 建立您在[上一節](#assign-users-to-the-application)中指派給應用程式的使用者帳戶。   當您準備好要對組織推出應用程式時，建議您使用自動使用者佈建，自動在應用程式中建立使用者帳戶。

## <a name="test-single-sign-on"></a>測試單一登入

您已準備就緒，可開始測試設定。  

1. 開啟應用程式的單一登入設定。 
2. 捲動至 [設定網域和 URL] 區段。
2. 按一下 [測試 SAML 設定]。 測試選項隨即出現。

    ![測試單一登入選項](media/configure-single-sign-on-portal/test-single-sign-on.png) 

3. 按一下 [以目前使用者身分登入]。 這可讓您先以管理員身分確認單一登入是否正常運作。
4. 如果發生錯誤，將會出現錯誤訊息。 請將訊息內容複製並貼到 [錯誤的描述為何?] 方塊中。

    ![取得解決方案指引](media/configure-single-sign-on-portal/error-guidance.png)

5. 按一下 [取得解決方案指引]。 此時會出現根本原因和解決方案指引。  在此範例中，使用者未指派給應用程式。

    ![修正錯誤](media/configure-single-sign-on-portal/fix-error.png)

6. 閱讀解決方案指引，然後如果適當的話，按一下 [修正此問題]。

7. 再次執行測試，直到順利完成。



## <a name="next-steps"></a>後續步驟
在本教學課程中，您使用 Azure 入口網站為應用程式設定了透過 Azure AD 進行的單一登入。 您找到單一登入組態設定頁面，並設定了單一登入設定。 完成設定後，您將使用者指派給應用程式，並將應用程式設定為使用 SAML 型單一登入。 當這項工作全部完成後，您驗證了 SAML 登入可正常運作。

您進行了下列事項：
> [!div class="checklist"]
> * 選取 SAML 作為單一登入模式
> * 連絡應用程式廠商以設定網域和 URL
> * 設定使用者屬性
> * 建立 SAML 簽署憑證
> * 手動將使用者或群組指派給應用程式
> * 設定應用程式使用單一登入
> * 測試 SAML 型單一登入

若要對組織中的更多使用者推出應用程式，建議使用自動佈建。

> [!div class="nextstepaction"]
>[了解如何使用自動佈建指派使用者](configure-automatic-user-provisioning-portal.md)


