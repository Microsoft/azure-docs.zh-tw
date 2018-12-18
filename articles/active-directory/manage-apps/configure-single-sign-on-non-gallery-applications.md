---
title: 設定應用程式的 Azure AD SSO | Microsoft Docs
description: 了解如何使用 SAML 和密碼 SSO 以自助方式將應用程式連接到 Azure Active Directory
services: active-directory
author: barbkess
documentationcenter: na
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/11/2018
ms.author: barbkess
ms.reviewer: asmalser,luleon
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: fc7510fdc635de03ac4dd4f64118bc5be040e969
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44719404"
---
# <a name="configure-single-sign-on-to-applications-that-are-not-in-the-azure-active-directory-application-gallery"></a>為不在 Azure Active Directory 應用程式庫中的應用程式設定單一登入

本文是關於可讓系統管理員設定單一登入不存在於 Azure Active Directory 應用程式資源庫的應用程式，而「不需要撰寫程式碼」 的功能。 此功能已在 2015 年 11 月 18 日的技術預覽中發行，並且包含在 [Azure Active Directory Premium](../fundamentals/active-directory-whatis.md) 中。 如果您要改為尋找有關如何透過程式碼將自訂應用程式與 Azure AD 整合的開發人員指導方針，請參閱 [Azure AD 的驗證案例](../develop/authentication-scenarios.md)。

Azure Active Directory 應用程式資源庫提供一份已知能支援單一登入搭配 Azure Active Directory 的應用程式清單，如 [本文](what-is-single-sign-on.md)所說明。 在您 (假設您是 IT 專業人員或組織中的系統整合者) 找到所要連接的應用程式之後，就可以依照 Azure 入口網站中展示的逐步指示來啟用單一登入。

具有 [Azure Active Directory Premium](../fundamentals/active-directory-whatis.md) 授權的客戶還會獲得下列額外功能：

* 任何支援 SAML 2.0 身分識別提供者的應用程式皆可進行自助式整合 (SP 起始或 IdP 起始)
* Web 應用程式可在使用 [密碼型 SSO](what-is-single-sign-on.md#password-based-single-sign-on)
* 應用程式可使用 SCIM 通訊協定進行自助式連線，以執行使用者佈建 ([說明請見此處](use-scim-to-provision-users-and-groups.md))
* 能夠在 [Office 365 應用程式啟動器](https://blogs.office.com/2014/10/16/organize-office-365-new-app-launcher-2/)或 [Azure AD 存取面板](what-is-single-sign-on.md#deploying-azure-ad-integrated-applications-to-users)中新增任何應用程式的連結

這不僅包括您所使用、但尚未在 Azure AD 應用程式庫中上線的 SaaS 應用程式，也包括您的組織已部署至您所控制的伺服器 (在雲端或內部部署中) 的第三方 Web 應用程式。

這些功能 (也稱為「應用程式整合範本」 )，為支援 SAML、SCIM 或表單型驗證的應用程式提供標準型連接點，包括有彈性的選項和與廣泛應用程式相容性的設定。 

## <a name="adding-an-unlisted-application"></a>新增未列出的應用程式
若要使用應用程式整合範本連線應用程式，請以您的 Azure Active Directory 系統管理員帳戶登入 Azure 入口網站。 依序瀏覽至 [Active Directory] > [企業應用程式] > [新增應用程式] > [不在資源庫內的應用程式] 區段，選取 [新增]，然後選取 [從資源庫新增應用程式]。

  ![新增應用程式](./media/configure-single-sign-on-non-gallery-applications/customapp1.png)

在應用程式庫中，如果找不到您想要的應用程式，您可以選取顯示在搜尋結果中的 [不在資源庫內的應用程式] 圖格，以新增未列出的應用程式。 輸入應用程式的名稱之後，您可以設定單一登入選項和行為。 

**快速提示**：最佳作法是使用搜尋函式來查看應用程式是否已存在於應用程式庫中。 如果找到應用程式，且其描述提及「單一登入」，即表示應用程式已支援同盟單一登入。

  ![Search](./media/configure-single-sign-on-non-gallery-applications/customapp2.png)

以這樣的方式新增應用程式所提供的體驗，非常類似於預先整合的應用程式所提供的體驗。 若要開始，請選取 [設定單一登入] 或按一下應用程式左側導覽功能表中的 [單一登入]。 下個畫面會列出單一登入的設定選項。 本文後續章節將說明這些選項。
  
![組態選項](./media/configure-single-sign-on-non-gallery-applications/customapp3.png)

## <a name="saml-based-single-sign-on"></a>SAML 型單一登入
選取此選項，可為應用程式設定 SAML 型驗證。 應用程式必須支援 SAML 2.0 才能使用此選項。 您應先收集如何使用應用程式的 SAML 功能相關資訊，再繼續執行。 完成下列區段以設定應用程式和 Azure AD 之間的單一登入。

### <a name="enter-basic-saml-configuration"></a>輸入基本 SAML 組態

若要設定 Azure AD，請輸入基本的 SAML 組態。 您可手動輸入值或上傳中繼資料檔案，以擷取欄位的值。

  ![Litware 網域和 URL](./media/configure-single-sign-on-non-gallery-applications/customapp4.png)

- **登入 URL (僅限 SP 起始)** - 使用者在此登入此應用程式。 如果已將應用程式設定為執行服務提供者所起始的單一登入，則當使用者瀏覽至此 URL 時，服務提供者就會執行必要的重新導向來連至 Azure AD，以進行使用者驗證和登入。 如果已填入此欄位，Azure AD 將使用此 URL 從 Office 365 和 Azure AD 存取面板中啟動應用程式。 如果略過這個欄位，則從 Office 365、Azure AD 存取面板或 Azure AD 單一登入 URL (可從 [儀表板] 索引標籤複製) 啟動應用程式時，Azure AD 會改為執行識別提供者所起始的登入。
- **識別碼** - 可唯一識別要設定單一登入的應用程式。 您可在應用程式傳送的 AuthRequest (SAML 要求) 中的簽發者元素找到這個值。 在應用程式中提供的任何 SAML 中繼資料中，這個值也會顯示為實體識別碼  。 請查看應用程式的 SAML 文件，以了解其「實體識別碼」或「對象」值的詳細資訊。 

    以下範例說明應用程式傳送至 Azure AD 的 SAML 要求中如何顯示識別碼或簽發者：

    ```
    <samlp:AuthnRequest
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
    ID="id6c1c178c166d486687be4aaf5e482730"
    Version="2.0" IssueInstant="2013-03-18T03:28:54.1839884Z"
    xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
    <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
    </samlp:AuthnRequest>
    ```

- **回覆 URL** -回覆 URL 是應用程式預期接收 SAML 權杖的位置。 這也稱為判斷提示取用者服務 (ACS) URL。 查看應用程式的 SAML 文件，了解 SAML 權杖回覆 URL 或 ACS URL 的詳細資訊。 

    若要設定多個 replyURL，您可使用下列 PowerShell 指令碼。

    ```PowerShell
    $sp = Get-AzureADServicePrincipal -SearchString "<Exact App  name>"
    $app = Get-AzureADApplication -SearchString "<Exact app name>"
    Set-AzureADApplication -ObjectId $app.ObjectId -ReplyUrls "<ReplyURLs>"
    Set-AzureADServicePrincipal -ObjectId $sp.ObjectId -ReplyUrls "<ReplyURLs>"
    ```

如需詳細資訊，請參閱 [Azure Active Directory (Azure AD) 支援的 SAML 2.0 驗證要求和回應](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference?/?WT.mc_id=DOC_AAD_How_to_Debug_SAML)


### <a name="review-or-customize-the-claims-issued-in-the-saml-token"></a>檢閱或自訂 SAML 權杖簽發的宣告

當使用者對應用程式進行驗證時，Azure AD 會將 SAML 權杖發出給應用程式，其中包含可唯一識別使用者的使用者相關資訊 (或宣告)。 根據預設，其中包含使用者的使用者名稱、電子郵件地址、名字和姓氏。 

您可以在 [屬性]  索引標籤下檢視或編輯在 SAML 權杖中傳送給應用程式的宣告。

  ![屬性](./media/configure-single-sign-on-non-gallery-applications/customapp7.png)

您可能需要編輯 SAML 權杖中簽發的宣告有以下兩個原因：

- 應用程式是設計為要求不同的宣告 URI 組或宣告值。
- 您的應用程式已部署為要求 NameIdentifier 宣告必須是 Azure Active Directory 中儲存之 username (也稱為使用者主體名稱) 以外的項目。 

如需詳細資訊，請參閱[針對企業應用程式自訂 SAML 權杖中發出的宣告](./../develop/../develop/active-directory-saml-claims-customization.md)。 



### <a name="review-certificate-expiration-data-status-and-email-notification"></a>檢閱憑證到期資料、狀態和電子郵件通知

建立資源庫或不在資源庫內的應用程式時，Azure AD 會建立應用程式特定的憑證，到期日為自建立日期算起 3 年。 您需要此憑證才能設定 Azure AD 和應用程式之間的信任。 如需憑證格式的詳細資料，請參閱應用程式的 SAML 文件。 

您可從 Azure AD 下載 Base64 或原始格式的憑證。 此外，您可藉由下載應用程式中繼資料 XML 檔案或使用應用程式同盟中繼資料 URL 取得憑證。

  ![憑證](./media/configure-single-sign-on-non-gallery-applications/certificate.png)

請確認憑證具有：

- 所需到期日。 您可以設定最多 3 年的到期日。
- 作用中狀態。 如果狀態處於非作用中，請將狀態變更為作用中。 若要變更狀態，請勾選 [作用中] 並儲存設定。 
- 正確的通知電子郵件。 作用中的憑證接近到期日時，Azure AD 會傳送通知到此欄位設定的電子郵件地址。  

如需詳細資訊，請參閱[在 Azure Active Directory 中管理同盟單一登入的憑證](manage-certificates-for-federated-single-sign-on.md)。

### <a name="set-up-target-application"></a>設定目標應用程式

若要為應用程式設定單一登入，請找出應用程式的說明文件。 將 SAML 型單一登入組態頁面捲動至結尾處，然後按一下 [設定 ]<application name>，即可找到需要的文件。 

必要的值會根據應用程式而有所不同。 如需詳細資訊，請參閱應用程式的 SAML 文件。 登入和登出服務 URL 都會解析至相同的端點，該端點即是 Azure AD 執行個體的 SAML 要求處理端點。 SAML 實體 ID 值在核發給應用程式的 SAML 權杖中會顯示為簽發者。


### <a name="assign-users-and-groups-to-your-saml-application"></a>將使用者和群組指派給您的 SAML 應用程式

您的應用程式在設定為使用 Azure AD 作為 SAML 型身分識別提供者之後，就幾乎可以進行測試了。 為了控制安全性，Azure AD 不會核發允許使用者登入應用程式的權杖，除非 Azure AD 已授與存取權給該使用者。 使用者可直接獲得存取權，或透過群組成員資格取得。 

若要將使用者或群組指派給您的應用程式，請按一下 [指派使用者]  按鈕。 選取您要指派的使用者或群組，然後選取 [指派]  按鈕。

  ![指派使用者](./media/configure-single-sign-on-non-gallery-applications/customapp6.png)

指派使用者會允許 Azure AD 簽發該使用者的權杖， 也會導致該應用程式磚出現在使用者的存取面板中。 如果使用者使用 Office 365，則也會有應用程式圖格出現在 Office 365 應用程式啟動器中。 

> [!NOTE] 
> 您可以在應用程式的 [設定] 索引標籤上使用 [上傳標誌] 按鈕，來上傳應用程式的圖格標誌。 


### <a name="test-the-saml-application"></a>測試 SAML 應用程式

測試 SAML 應用程式前，您必須透過 Azure AD 設定應用程式，並將使用者或群組指派至該應用程式。 若要測試 SAML 應用程式，請參閱[如何偵錯 SAML 型單一登入 Azure Active Directory 中的應用程式](../develop/howto-v1-debug-saml-sso-issues.md)。

## <a name="password-single-sign-on"></a>密碼單一登入

選取此選項，可為具有 HTML 登入頁面的 Web 應用程式設定 [密碼單一登入](what-is-single-sign-on.md) 。 密碼 SSO 也稱為密碼儲存庫存，可讓您管理使用者對不支援身分識別同盟之 Web 應用程式的存取和密碼。 如果有數個使用者需要共用單一帳戶 (例如共用組織的社交媒體應用程式帳戶)，這也很有用處。 

選取 [下一步] 之後，系統會提示您輸入應用程式的 Web 型登入頁面的 URL。 請注意，這必須是包含使用者名稱和密碼輸入欄位的頁面。 輸入之後，Azure AD 會啟動程序來剖析使用者名稱輸入和密碼輸入的登入頁面。 如果此程序不成功，系統會引導您執行安裝瀏覽器延伸模組 (需要 Internet Explorer、Chrome 或 Firefox) 的替代程序，以讓您手動擷取欄位。

在擷取登入頁面後，即可指派使用者和群組，並且可像 [密碼 SSO 應用程式](what-is-single-sign-on.md)一般設定認證原則。

> [!NOTE] 
> 您可以在應用程式的 [設定] 索引標籤上使用 [上傳標誌] 按鈕，來上傳應用程式的圖格標誌。 
>

## <a name="existing-single-sign-on"></a>現有單一登入
選取此選項，可將應用程式的連結新增至組織的 Azure AD 存取面板或 Office 365 入口網站。 使用此選項，可讓您新增目前使用 Azure Active Directory 同盟服務 (或其他同盟服務)、而不是使用 Azure AD 的自訂 Web 應用程式的連結，以進行驗證。 或者，您可以新增特定 SharePoint 網頁或其他只要出現在使用者存取面板上的網頁的深層連結。 

選取 [下一步] 之後，系統會提示您輸入要連結到的應用程式的 URL。 完成之後，使用者和群組即可指派給應用程式，而使應用程式出現在這些使用者的 [Office 365 應用程式啟動器](https://blogs.office.com/2014/10/16/organize-office-365-new-app-launcher-2/)或 [Azure AD 存取面板](what-is-single-sign-on.md#deploying-azure-ad-integrated-applications-to-users)中。

> [!NOTE] 
> 您可以在應用程式的 [設定] 索引標籤上使用 [上傳標誌] 按鈕，來上傳應用程式的圖格標誌。 
>

## <a name="related-articles"></a>相關文章
- [如何為預先整合的應用程式自訂在 SAML 權杖中發出的宣告](../develop/active-directory-saml-claims-customization.md)
- [對 SAML 型單一登入進行疑難排解](../develop/howto-v1-debug-saml-sso-issues.md)

