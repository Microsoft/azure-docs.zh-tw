---
title: 教學課程：Azure Active Directory 與 SharePoint 內部部署整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 SharePoint 內部部署之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 85b8d4d0-3f6a-4913-b9d3-8cc327d8280d
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/21/2018
ms.author: jeedes
ms.openlocfilehash: 100ea0147bf56242ba15bee39e4bda1b65af27da
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49390086"
---
# <a name="tutorial-azure-active-directory-integration-with-sharepoint-on-premises"></a>教學課程：Azure Active Directory 與 SharePoint 內部部署整合

在本教學課程中，您將了解如何整合 SharePoint 內部部署與 Azure Active Directory (Azure AD)。

將 SharePoint 內部部署與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 SharePoint 內部部署的人員。
- 您可以讓使用者使用其 Azure AD 帳戶來自動登入 SharePoint 內部部署 (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 SharePoint 內部部署整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 SharePoint 內部部署單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 SharePoint 內部部署
2. 設定並測試 Azure AD 單一登入

## <a name="adding-sharepoint-on-premises-from-the-gallery"></a>從資源庫新增 SharePoint 內部部署

若要設定將 SharePoint 內部部署整合到 Azure AD 中，您需要從資源庫將 SharePoint 內部部署新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 SharePoint 內部部署，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **SharePoint 內部部署**，從結果面板中選取 [SharePoint 內部部署]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 SharePoint 內部部署](./media\sharepoint-on-premises-tutorial/tutorial_sharepointonpremises_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 SharePoint 內部部署來設定和測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 SharePoint 內部部署與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 SharePoint 內部部署中的相關使用者之間建立連結關聯性。

若要設定及測試與 SharePoint 內部部署搭配運作的 Azure AD 單一登入，您需要完成下列基本工作：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[將存取權授與 SharePoint 內部部署測試使用者](#grant-access-to-sharePoint-on-premises-test-user)** - 讓 SharePoint 內部部署中的 Britta Simon 對應項目得以連結至 Azure AD 中代表該使用者的項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在 SharePoint 內部部署應用程式中設定單一登入。

**若要設定與 SharePoint 內部部署搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [SharePoint 內部部署] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media\sharepoint-on-premises-tutorial/tutorial_sharepointonpremises_samlbase.png)

3. 在 [SharePoint 內部部署網域及 URL] 區段上，執行下列步驟：

    ![[SharePoint 內部部署網域及 URL] 單一登入資訊](./media\sharepoint-on-premises-tutorial/tutorial_sharepointonpremises_url1.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<YourSharePointServerURL>/_trust/default.aspx`

    b. 在 [識別碼] 文字方塊中，輸入 URL：`urn:sharepoint:federation`

    c. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<YourSharePointServerURL>/_trust/default.aspx`

4. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media\sharepoint-on-premises-tutorial/tutorial_sharepointonpremises_certificate.png)

    > [!Note]
    > 請記下您已將憑證檔案下載到其中的檔案路徑，因為您稍後需要在 PowerShell 指令碼中使用它來進行設定。

5. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media\sharepoint-on-premises-tutorial/tutorial_general_400.png)

6. 在 [SharePoint 內部部署設定] 區段上，按一下 [設定 SharePoint 內部部署] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體識別碼]。 針對**單一登入服務 URL**，請使用下列模式的值：`https://login.microsoftonline.com/_my_directory_id_/wsfed` 

    > [!Note]
    > _my_directory_id_ 是 Azure AD 訂用帳戶的租用戶識別碼。

    ![SharePoint 內部部署設定](./media\sharepoint-on-premises-tutorial/tutorial_sharepointonpremises_configure.png)

    > [!NOTE]
    > SharePoint 內部部署應用程式使用 SAML 1.1 權杖，因此 Azure AD 會預期有來自 SharePoint 伺服器的 WS Fed 要求，然後會在驗證後發出 SAML 1.1 權杖。

7. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 SharePoint 內部部署公司網站。

8. **在 SharePoint Server 2016 中設定新的受信任身分識別提供者**

    登入 SharePoint Server 2016 伺服器，然後開啟 SharePoint 2016 管理命令介面。 從 Azure 入口網站填入 $realm (Azure 入口網站中，來自 SharePoint 內部部署網域及 URL 區段的識別碼值)、$wsfedurl (單一登入服務 URL) 及 $filepath (您已將憑證檔案下載到其中的檔案路徑) 的值，然後執行下列命令來設定信任的新身分識別提供者。

    > [!TIP]
    > 如果您不熟悉 PowerShell 的使用方式，或想要深入了解 PowerShell 的運作方式，請參閱 [SharePoint PowerShell](https://docs.microsoft.com/powershell/sharepoint/overview?view=sharepoint-ps)。 

    ```
    $realm = "<Identifier value from the SharePoint on-premises Domain and URLs section in the Azure portal>"
    $wsfedurl="<SAML single sign-on service URL value which you have copied from the Azure portal>"
    $filepath="<Full path to SAML signing certificate file which you have downloaded from the Azure portal>"
    $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($filepath)
    New-SPTrustedRootAuthority -Name "AzureAD" -Certificate $cert
    $map = New-SPClaimTypeMapping -IncomingClaimType "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name" -IncomingClaimTypeDisplayName "name" -LocalClaimType "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"
    $map2 = New-SPClaimTypeMapping -IncomingClaimType "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname" -IncomingClaimTypeDisplayName "GivenName" -SameAsIncoming
    $map3 = New-SPClaimTypeMapping -IncomingClaimType "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname" -IncomingClaimTypeDisplayName "SurName" -SameAsIncoming
    $map4 = New-SPClaimTypeMapping -IncomingClaimType "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress" -IncomingClaimTypeDisplayName "Email" -SameAsIncoming
    $ap = New-SPTrustedIdentityTokenIssuer -Name "AzureAD" -Description "SharePoint secured by Azure AD" -realm $realm -ImportTrustCertificate $cert -ClaimsMappings $map,$map2,$map3,$map4 -SignInUrl $wsfedurl -IdentifierClaim "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"
    ```

    接著，依照下列步驟為您的應用程式啟用受信任的身分識別提供者：

    a. 在 [管理中心] 中，瀏覽至 [管理 Web 應用程式]，然後選取您想要以 Azure AD 保護的 Web 應用程式。

    b. 在功能區中，按一下 [驗證提供者]，然後選擇您想要使用的區域。

    c. 選取 [信任的身分識別提供者]，然後選取您剛才註冊名為 *AzureAD* 的身分識別提供者。

    d. 在 [登入頁面 URL] 設定中，選取 [自訂登入頁面] 並提供值 “/_trust/”。

    e. 按一下 [確定]。

    ![設定您的驗證提供者](./media\sharepoint-on-premises-tutorial/fig10-configauthprovider.png)

    > [!NOTE]
    > 某些外部使用者將無法使用這個單一登入整合，因為他們的 UPN 將含有類似 `MYEMAIL_outlook.com#ext#@TENANT.onmicrosoft.com` 的損害值。 我們很快地將會為客戶提供如何根據使用者類型來處理 UPN 的應用程式設定。 在那之後，您的所有來賓使用者應該都能以組織員工的身分順暢地使用 SSO。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media\sharepoint-on-premises-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media\sharepoint-on-premises-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media\sharepoint-on-premises-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media\sharepoint-on-premises-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="grant-access-to-sharepoint-on-premises-test-user"></a>將存取權授與 SharePoint 內部部署測試使用者

要登入 Azure AD 並存取 SharePoint 的使用者，必須獲得授與應用程式的存取權。使用下列步驟來設定用於存取 Web 應用程式的權限。

1. 在 [管理中心] 中，按一下 [應用程式管理]。

2. 在 [應用程式管理] 頁面的 [Web 應用程式] 區段中，按一下 [管理 Web 應用程式]。

3. 按一下適當的 Web 應用程式，然後按一下 [使用者原則]。

4. 在 [Web 應用程式的原則] 中，按一下 [新增使用者]。

    ![依使用者的名稱宣告來搜尋使用者](./media\sharepoint-on-premises-tutorial/fig11-searchbynameclaim.png)

5. 在 [新增使用者] 對話方塊中，按一下 [區域] 中的適當區域，然後按 [下一步]。

6. 在 [Web 應用程式的原則] 對話方塊的 [選擇使用者] 區段中，按一下 [瀏覽] 圖示。

7. 在 [尋找] 文字方塊中，輸入**使用者主體名稱 (UPN)** 值 (您已針對該值在 Azure AD 中設定 SharePoint 內部部署應用程式)，然後按一下 [搜尋]。 </br>範例：*brittasimon@contoso.com*。

8. 在清單檢視中的 AzureAD 標題下，選取 [name] 屬性並按一下 [新增]，然後按一下 [確定] 以關閉對話方塊。

9. 在 [權限] 中，按一下 [完全控制]。

    ![對宣告使用者授與完全控制](./media\sharepoint-on-premises-tutorial/fig12-grantfullcontrol.png)

10. 按一下 [完成]，然後按一下 [確定]。

### <a name="configuring-one-trusted-identity-provider-for-multiple-web-applications"></a>對多個 Web 應用程式設定一個受信任的識別提供者

此組態適用於單一 Web 應用程式，但是如果您想要對多個 Web 應用程式使用相同受信任的識別提供者，則需要額外的組態。 例如，假設我們已將 Web 應用程式擴充為使用 URL `https://portal.contoso.local`，而現在也想要向 `https://sales.contoso.local` 驗證使用者。 若要完成這項操作，我們必須更新識別提供者以接受 WReply 參數，並且在 Azure AD 中更新應用程式註冊以新增回覆 URL。

1. 在 Azure 入口網站中，開啟 Azure AD 目錄。 按一下 [應用程式註冊]，然後按一下 [檢視所有應用程式]。 按一下您先前建立的應用程式 (SharePoint SAML 整合)。

2. 按一下 [設定] 。

3. 在 [設定] 刀鋒視窗中，按一下 [回覆 URL]。 

4. 為額外的 Web 應用程式新增 URL，將 `/_trust/default.aspx` 附加到 URL (例如 `https://sales.contoso.local/_trust/default.aspx`)，然後按一下 [儲存]。

5. 在 SharePoint 伺服器上，開啟 [SharePoint 2016 管理命令介面]，然後使用您先前使用的受信任身分識別權杖簽發者名稱，來執行下列命令。

    ```
    $t = Get-SPTrustedIdentityTokenIssuer "AzureAD"
    $t.UseWReplyParameter=$true
    $t.Update()
    ```
6. 在 [管理中心] 內，移至 Web 應用程式，並且啟用現有受信任的識別提供者。 請記得也要將登入頁面 URL 設定為自訂登入頁面 `/_trust/`。

7. 在 [管理中心] 內，按一下 Web 應用程式，然後選擇 [使用者原則]。 如本文中先前的示範，新增具有適當權限的使用者。

### <a name="fixing-people-picker"></a>修正人員選擇器

使用者現在可以使用來自 Azure AD 的身分識別登入 SharePoint 2016，但此使用者體驗還有改善的機會。 例如，搜尋使用者會在人員選擇器中呈現多個搜尋結果。 在宣告對應中建立之 3 種宣告類型的每種類型都有一個搜尋結果。 若要使用人員選擇器來選擇使用者，您必須精確輸入使用者名稱，然後選擇 [name] 宣告結果。

![宣告搜尋結果](./media\sharepoint-on-premises-tutorial/fig16-claimssearchresults.png)

由於不會驗證您搜尋的值，因此可能會導致拼字錯誤或使用者不小心選擇錯誤的宣告類型 (例如 **SurName** 宣告) 來進行指派。 這可能會導致使用者無法成功存取資源。

為了協助應付這種情況，有一個稱為 [AzureCP](https://yvand.github.io/AzureCP/) 的開放原始碼解決方案，可為 SharePoint 2016 提供自訂宣告提供者。 它將使用 Azure AD Graph 來解析使用者的輸入內容並執行驗證。 深入了解 [AzureCP](https://yvand.github.io/AzureCP/)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 SharePoint 內部部署的存取權授與 Britta Simon，使其能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 SharePoint 內部部署，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

2. 在應用程式清單中，選取 [SharePoint 內部部署]。

    ![應用程式清單中的 SharePoint 連結](./media\sharepoint-on-premises-tutorial/tutorial_sharepointonpremises_app.png)

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 [SharePoint 內部部署] 圖格時，應該就會自動登入 SharePoint 內部部署應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [使用 Azure AD 進行 SharePoint Server 驗證](https://docs.microsoft.com/office365/enterprise/using-azure-ad-for-sharepoint-server-authentication)

<!--Image references-->

[1]: ./media\sharepoint-on-premises-tutorial/tutorial_general_01.png
[2]: ./media\sharepoint-on-premises-tutorial/tutorial_general_02.png
[3]: ./media\sharepoint-on-premises-tutorial/tutorial_general_03.png
[4]: ./media\sharepoint-on-premises-tutorial/tutorial_general_04.png

[100]: ./media\sharepoint-on-premises-tutorial/tutorial_general_100.png

[200]: ./media\sharepoint-on-premises-tutorial/tutorial_general_200.png
[201]: ./media\sharepoint-on-premises-tutorial/tutorial_general_201.png
[202]: ./media\sharepoint-on-premises-tutorial/tutorial_general_202.png
[203]: ./media\sharepoint-on-premises-tutorial/tutorial_general_203.png
