---
title: 使用自訂原則動態地自訂 Azure Active Directory B2C 使用者介面 (UI) | Microsoft Docs
description: 透過會在執行階段動態變更的 HTML5/CSS 內容支援多個品牌體驗。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/20/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 4e7cc47bddf3663cbc1c8bb5c4470020a84073e4
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37441646"
---
# <a name="azure-active-directory-b2c-configure-the-ui-with-dynamic-content-by-using-custom-policies"></a>Azure Active Directory B2C：使用自訂原則設定具有動態內容的 UI

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

使用 Azure Active Directory B2C (Azure AD B2C) 自訂原則，您可以在查詢字串中傳送參數。 將參數傳遞至您的 HTML 端點，即可動態變更網頁內容。 例如，視您從 Web 或行動裝置應用程式傳遞的參數而定，您可以變更 Azure AD B2C 註冊或登入背景影像。 

## <a name="prerequisites"></a>先決條件
本文著重於如何使用自訂原則，自訂具有「動態內容」的 Azure AD B2C 使用者介面。 一開始，請參閱[自訂原則中的 UI 自訂](active-directory-b2c-ui-customization-custom.md)。 

>[!NOTE]
>Azure AD B2C 的[設定自訂原則中的 UI 自訂](active-directory-b2c-ui-customization-custom.md)一文討論下列基本概念：
> * 頁面使用者介面 (UI) 自訂功能。
> * 使用我們的範例內容測試頁面 UI 自訂功能的基本工具。
> * 每種頁面類型的核心 UI 元素。
> * 套用這項功能的最佳做法。

## <a name="add-a-link-to-html5css-templates-to-your-user-journey"></a>將 HTML5/CSS 範本的連結新增至使用者旅程圖

在自訂原則中，內容定義會定義用於指定之 UI 步驟的 HTML5 頁面 URI (例如，登入或註冊頁面)。 基礎原則會藉由指向 HTML5 檔案的 URI (採用 CSS) 來定義預設的外觀及操作。 在擴充原則中，您可以藉由覆寫該 HTML5 檔案的 LoadUri 來修改外觀及操作。 在適當情況下，內容定義包含透過製作 HTML5/CSS 檔案定義的外部內容的 URL。 

`ContentDefinitions` 區段包含一系列的 `ContentDefinition` XML 元素。 `ContentDefinition` 元素的識別碼屬性會指定與內容定義相關的頁面類型。 也就是，此元素會定義自訂 HTML5/CSS 範本即將套用的內容。 下表說明 IEF 引擎所能辨識之內容定義識別碼的集合，以及這些識別碼的相關頁面類型。

| 內容定義識別碼 | 預設 HTML5 範本| 說明 | 
|-----------------------|--------|-------------|
| api.error | [exception.cshtml](https://login.microsoftonline.com/static/tenant/default/exception.cshtml) | **錯誤頁面**。 在發生例外狀況或錯誤時，系統會顯示此頁面。 |
| api.idpselections | [idpSelector.cshtml](https://login.microsoftonline.com/static/tenant/default/idpSelector.cshtml) | **識別提供者選取頁面**。 此頁面列出使用者可以在登入期間選擇的識別提供者清單。 這些選項通常是企業識別提供者、社交識別提供者 (如 Facebook 和 Google+) 或本機帳戶。 |
| api.idpselections.signup | [idpSelector.cshtml](https://login.microsoftonline.com/static/tenant/default/idpSelector.cshtml) | **用於註冊的識別提供者選取**。 此頁面列出使用者可以在註冊期間選擇的識別提供者清單。 這些選項是企業識別提供者、社交識別提供者 (如 Facebook 和 Google+) 或本機帳戶。 |
| api.localaccountpasswordreset | [selfasserted.html](https://login.microsoftonline.com/static/tenant/default/selfAsserted.cshtml) | **忘記密碼頁面**。 此頁面包含一份表單，使用者必須填妥此表單才能開始重設密碼。  |
| *api.localaccountsignin* | [selfasserted.html](https://login.microsoftonline.com/static/tenant/default/selfAsserted.cshtml) | **本機帳戶登入頁面**。 此頁面包含一份表單，可供使用以電子郵件地址或使用者名稱為基礎的本機帳戶進行登入。 此表單可以包含文字輸入方塊和密碼輸入方塊。 |
| *api.localaccountsignup* | [selfasserted.html](https://login.microsoftonline.com/static/tenant/default/selfAsserted.cshtml) | **本機帳戶註冊頁面**。 此頁面包含一份表單，可供使用以電子郵件地址或使用者名稱為基礎的本機帳戶進行登入。 此表單可以包含各種輸入控制項，例如文字輸入方塊、密碼輸入方塊、選項按鈕、單選下拉式清單方塊和多選核取方塊。 |
| *api.phonefactor* | [multifactor-1.0.0.cshtml](https://login.microsoftonline.com/static/tenant/default/multifactor-1.0.0.cshtml) | **Multi-Factor Authentication 頁面**。 在此頁面上，使用者可以在註冊或登入期間驗證其電話號碼 (藉由使用文字或語音)。 |
| *api.selfasserted* | [selfasserted.html](https://login.microsoftonline.com/static/tenant/default/selfAsserted.cshtml) | **社交帳戶註冊頁面**。 此頁面包含一份表單，使用者在使用社交識別提供者的現有帳戶註冊時必須填妥此表單。 此頁面類似於上述的社交帳戶註冊頁面，但密碼輸入欄位除外。 |
| *api.selfasserted.profileupdate* | [updateprofile.html](https://login.microsoftonline.com/static/tenant/default/updateProfile.cshtml) | **設定檔更新頁面**。 此頁面包含一份表單，使用者可用來更新其設定檔。 此頁面類似於社交帳戶註冊頁面，但密碼輸入欄位除外。 |
| *api.signuporsignin* | [unified.html](https://login.microsoftonline.com/static/tenant/default/unified.cshtml) | **統一的註冊或登入頁面**。 此頁面可處理使用者的註冊和登入程序。 使用者可以使用企業識別提供者、社交識別提供者 (如 Facebook 或 Google+) 或本機帳戶。  |

## <a name="serving-dynamic-content"></a>提供動態內容
在[在自訂原則中設定 UI 自訂](active-directory-b2c-ui-customization-custom.md)一文中，您會將 HTML5 檔案上傳至 Azure Blob 儲存體。 這些 HTML5 檔案都是靜態，並且對每個要求呈現相同的 HTML 內容。 

在本文中，我們會使用 ASP.Net Web 應用程式，它可以接受查詢字串參數並據以回應。 

在本逐步解說中，您將：
* 建立會主控 HTML5 範本的 ASP.NET Core Web 應用程式。 
* 新增自訂 HTML5 範本 _unified.cshtml_。 
* 將您的 Web 應用程式發佈至 Azure App Service。 
* 為您的 Web 應用程式設定跨原始來源資源分享 (CORS)。
* 覆寫 `LoadUri` 元素以指向 HTML5 檔案。

## <a name="step-1-create-an-aspnet-web-app"></a>步驟 1：建立 ASP.NET Web 應用程式

1. 在 Visual Studio 中，選取 [檔案] > [新增] > [專案] 以建立專案。

2. 在 [新增專案] 視窗中，選取 [Visual C#] > [Web] > [ASP.NET Core Web 應用程式 (.NET Core)]。

3. 為應用程式命名 (例如 Contoso.AADB2C.UI)，然後選取 [確定]。

    ![建立新的 Visual Studio 專案](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-create-project1.png)

4. 選取 [Web 應用程式] 範本。

5. 將驗證設為 [沒有驗證]。

    ![選取 Web 應用程式範本](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-create-project2.png)

6. 選取 [確定] 以建立專案。

## <a name="step-2-create-mvc-view"></a>步驟 2：建立 MVC 檢視
### <a name="step-21-download-the-b2c-built-in-html5-template"></a>步驟 2.1：下載 B2C 內建 HTML5 範本
您的自訂 HTML5 範本是以 Azure AD B2C 內建 HTML5 範本為基礎。 您可以下載 [unified.html 檔案](https://login.microsoftonline.com/static/tenant/default/unified.cshtml)，或透過[入門套件](https://github.com/AzureADQuickStarts/B2C-AzureBlobStorage-Client/tree/master/sample_templates/wingtip)下載範本。 您可以使用 HTML5 檔案來建立統一註冊或登入頁面。

### <a name="step-22-add-the-mvc-view"></a>步驟 2.2：新增 MVC 檢視
1. 以滑鼠右鍵按一下 [檢視]/[首頁] 資料夾，然後按一下 [新增] > [新增項目]。

    ![新增 MVC 項目](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-add-view1.png)

2. 在 [新增項目 - Contoso.AADB2C.UI] 視窗中，選取 [Web] > [ASP.NET]。

3. 選取 [MVC 檢視頁面]。

4. 在 [名稱] 方塊中，將名稱變更為 **unified.cshtml**。

5. 選取 [新增] 。

    ![新增 MVC 檢視](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-add-view2.png)

6. 如果 unified.cshtml 檔案尚未開啟，請按兩下此檔案將它開啟，然後清除檔案內容。

7. 在此逐步解說中，我們會移除對版面配置頁面的參考。 將下列程式碼片段新增至 unified.cshtml：

    ```csharp
    @{
        Layout = null;
    }
    ```

8. 開啟您從 Azure AD B2C 內建 HTML5 範本下載的 unified.cshtml 檔案。

9. 以兩個 @@ 符號取代單一 @ 符號。

10. 複製檔案的內容，並將它在版面配置定義下貼上。 您的程式碼看起來應該類似：

    ![新增 HTML5 之後的 unified.cshtml 檔案](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-edit-view1.png)

### <a name="step-23-change-the-background-image"></a>步驟 2.3：變更背景影像

找出包含 `ID` 值 *background_background_image* 的 `<img>` 元素，然後以 **https://kbdevstorage1.blob.core.windows.net/asset-blobs/19889_en_1** 或您想要使用的任何其他背景影像取代 `src` 值。

![變更頁面背景](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-add-static-background.png)

### <a name="step-24-add-your-view-to-the-mvc-controller"></a>步驟 2.4：將檢視新增至 MVC 控制器

1. 開啟 **Controllers\HomeController.cs**，然後新增下列方法： 

    ```C
    public IActionResult unified()
    {
        return View();
    }
    ```
    此程式碼會指定方法應使用 View 範本檔案來向瀏覽器呈現回應。 因為我們未明確指定 View 範本檔案的名稱，MVC 預設會使用 /Views/Home 資料夾中的 unified.cshtml View 檔案。
    
    新增 unified 方法之後，您的程式碼看起來應該類似：
    
    ![變更控制器來呈現檢視](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-controller-view.png)

2. 偵錯您的 Web 應用程式，然後確定 unified 頁面可供存取 (例如，`http://localhost:<Port number>/Home/unified`)。

### <a name="step-25-publish-to-azure"></a>步驟 2.5：發佈至 Azure
1. 在 [方案總管] 中，以滑鼠右鍵按一下 **Contoso.AADB2C.UI** 專案，然後選取 [發佈]。

    ![發佈至 Microsoft Azure App Service](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-publish1.png)

2. 選取 [Microsoft Azure App Service] 圖格，然後選取 [發佈]。

    ![建立新的 Microsoft Azure App Service](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-publish2.png)

    [建立 App Service] 視窗隨即開啟。 您可以在此開始建立所有必要 Azure 資源，以在 Azure 中執行 ASP.NET Web 應用程式。

    > [!NOTE]
    > 如需有關發佈的詳細資訊，請參閱[在 Azure 中建立 ASP.NET Web 應用程式](https://docs.microsoft.com/azure/app-service-web/app-service-web-get-started-dotnet#publish-to-azure)。

3. 在 [Web 應用程式名稱] 方塊中，輸入唯一的應用程式名稱 (有效字元為 a-z、A-Z、0-9 及連字號 (-))。 Web 應用程式的 URL 是 `http://<app_name>.azurewebsites.NET`，其中 `<app_name>` 是您的 Web 應用程式名稱。 您可以接受自動產生的名稱 (這是唯一的)。

4. 選取 [建立] 開始建立 Azure 資源。

    ![提供 App Service 屬性](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-publish3.png)

    完成建立程序之後，精靈會將 ASP.NET Web 應用程式發佈至 Azure，然後在預設瀏覽器中啟動該應用程式。

5. 複製 unified 頁面的 URL (例如，https://<app_name>.azurewebsites.net/home/unified)。

## <a name="step-3-configure-cors-in-azure-app-service"></a>步驟 3：在 Azure App Service 中設定 CORS
1. 從 [Azure 入口網站](https://portal.azure.com/)中，選取 [應用程式服務]，然後選取 API 應用程式的名稱。

    ![在 Azure 入口網站中選取 API 應用程式](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-CORS1.png)

2. 在 [設定] 區段的 [API] 區段之下，選取 [CORS]。

    ![選取 CORS 設定](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-CORS2.png)

3. 在 [CORS] 視窗的 [允許的來源] 方塊中，執行下列其中一項：

    * 輸入您想要允許的一或多個 JavaScript 呼叫來源 URL。
    * 輸入星號 (*) 來指定接受所有的原始網域。

4. 選取 [ **儲存**]。

    ![CORS 視窗](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-CORS3.png)

    選取 [儲存] 之後，API 應用程式會接受來自指定 URL 的 JavaScript 呼叫。 

## <a name="step-4-html5-template-validation"></a>步驟 4：HTML5 範本驗證
您的 HTML5 範本已可供使用。 不過，它無法在 `ContentDefinition` 程式碼中使用。 請先確認下列事項，才可將 `ContentDefinition` 新增至自訂原則：
* 您的內容符合 HTML5 規範且可供存取。
* 您的內容伺服器已啟用 CORS。

    >[!NOTE]
    >若要確認您裝載內容的網站已啟用 CORS 並可測試 CORS 要求，請移至 [test-cors.org](http://test-cors.org/) 網站。 

* 您提供的內容是透過 **HTTPS** 保護。
* 您會對所有連結、CSS 內容與影像使用「絕對 URL」，例如 https://yourdomain/content。

## <a name="step-5-configure-your-content-definition"></a>步驟 5：設定您的內容定義
若要設定 `ContentDefinition`，請執行下列動作：
1. 開啟原則的基本檔案 (例如，TrustFrameworkBase.xml)。

2. 尋找 `<ContentDefinitions>` 元素，然後複製 `<ContentDefinitions>` 節點的完整內容。

3. 開啟擴充檔案 (例如，TrustFrameworkExtensions.xml)，然後搜尋 `<BuildingBlocks>` 元素。 如果此元素不存在，請加以新增。

4. 貼上您複製作為 `<BuildingBlocks>` 元素之子元素的 `<ContentDefinitions>` 節點的完整內容。 

5. 在您複製的 XML 中，搜尋包含 `Id="api.signuporsignin"` 的 `<ContentDefinition>` 節點。

6. 將 `LoadUri` 的值從 ~/tenant/default/unified 變更為 https://<app_name>.azurewebsites.net/home/unified。  
    自訂原則看起來應該如下所示：
    
    ![您的內容定義](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-content-definition.png)

## <a name="step-6-upload-the-policy-to-your-tenant"></a>步驟 6：將原則上傳至您的租用戶
1. 在 [Azure 入口網站](https://portal.azure.com)中，切換至[您的 Azure AD B2C 租用戶環境](active-directory-b2c-navigate-to-b2c-context.md)，然後選取 [Azure AD B2C]。

2. 選取 [識別體驗架構]。

3. 選取 [所有原則]。

4. 選取 [上傳原則]。

5. 選取 [覆寫已存在的原則] 核取方塊。

6. 上傳 TrustFrameworkExtensions.xml 檔案，確定它通過驗證。

## <a name="step-7-test-the-custom-policy-by-using-run-now"></a>步驟 7：使用 [立即執行] 測試自訂原則
1. 選取 [Azure AD B2C 設定]，然後選取 [識別體驗架構]。

    >[!NOTE]
    >[立即執行] 需要在租用戶上至少預先註冊一個應用程式。 若要了解如何註冊應用程式，請參閱 Azure AD B2C [開始使用](active-directory-b2c-get-started.md)一文或[應用程式註冊](active-directory-b2c-app-registration.md)一文。

2. 開啟 **B2C_1A_signup_signin** (此為您上傳的信賴憑證者 (RP) 自訂原則)，然後選取 [立即執行]。  
    您應該能夠看到您的自訂 HTML5 具有您稍早建立的背景。

    ![您的註冊或登入原則](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-demo1.png)

## <a name="step-8-add-dynamic-content"></a>步驟 8：新增動態內容
我們會根據名為 campaignId 的查詢字串參數來變更背景。 RP 應用程式 (Web 和行動裝置應用程式) 會將參數傳送至 Azure AD B2C。 您的原則會讀取參數，並將其值傳送至您的 HTML5 範本。 

### <a name="step-81-add-a-content-definition-parameter"></a>步驟 8.1：新增內容定義參數

執行下列動作來新增 `ContentDefinitionParameters` 元素：
1. 開啟您的原則的 SignUpOrSignin 檔案 (例如，SignUpOrSignin.xml)。

2. 搜尋 `<DefaultUserJourney>` 節點。 

3. 在 `<DefaultUserJourney>` 節點中，新增下列 XML 程式碼片段：  

    ```XML
    <UserJourneyBehaviors>
        <ContentDefinitionParameters>
            <Parameter Name="campaignId">{OAUTH-KV:campaignId}</Parameter>
        </ContentDefinitionParameters>
    </UserJourneyBehaviors>
    ```

### <a name="step-82-change-your-code-to-accept-a-query-string-parameter-and-replace-the-background-image"></a>步驟 8.2：將您的程式碼變更為接受查詢字串參數，並取代背景影像
修改 HomeController `unified` 方法，以接受 campaignId 參數。 此方法會接著檢查此參數的值，並據以設定 `ViewData["background"]` 變數。

1. 開啟 Controllers\HomeController.cs，然後藉由新增下列程式碼片段來變更 `unified` 方法：

    ```csharp
    public IActionResult unified(string campaignId)
    {
        // If campaign ID is Hawaii, show Hawaii background
        if (campaignId != null && campaignId.ToLower() == "hawaii")
        {
            ViewData["background"] = "https://kbdevstorage1.blob.core.windows.net/asset-blobs/19889_en_1";
        }
        // If campaign ID is Tokyo, show Tokyo background
        else if (campaignId != null && campaignId.ToLower() == "tokyo")
        {
            ViewData["background"] = "https://kbdevstorage1.blob.core.windows.net/asset-blobs/19666_en_1";
        }
        // Default background
        else
        {
            ViewData["background"] = "https://kbdevstorage1.blob.core.windows.net/asset-blobs/18983_en_1";
        }

        return View();
    }

    ```

2. 找出具有識別碼 `background_background_image` 的 `<img>` 元素，並以 `@ViewData["background"]` 取代 `src` 值。

    ![變更頁面背景](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-add-dynamic-background.png)

### <a name="83-upload-the-changes-and-publish-your-policy"></a>8.3：上傳變更並發佈您的原則
1. 將您的 Visual Studio 專案發佈至 Azure App Service。

2. 將 SignUpOrSignin.xml 原則上傳至 Azure AD B2C。

3. 開啟 **B2C_1A_signup_signin** (此為您上傳的 RP 自訂原則)，然後選取 [立即執行]。  
    您應會看到之前顯示的相同背景影像。

4. 從瀏覽器的網址列複製 URL。

5. 將 campaignId 查詢字串參數新增至此 URI。 例如，新增 `&campaignId=hawaii`，如下圖所示：

    ![變更頁面背景](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-campaignId-param.png)

6. 選取 **Enter** 以顯示夏威夷背景影像。

    ![變更頁面背景](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-demo2.png)

7. 將值變更為 Tokyo，然後選取 **Enter**。  
    瀏覽器會顯示東京背景。

    ![變更頁面背景](media/active-directory-b2c-ui-customization-custom-dynamic/aadb2c-ief-ui-customization-demo3.png)

## <a name="step-9-change-the-rest-of-the-user-journey"></a>步驟 9：變更使用者旅程圖的其餘部分
如果您選取登入頁面上的 [立即註冊] 連結，瀏覽器會顯示預設背景影像，而非您定義的影像。 因為您只變更註冊或登入頁面，所以會發生這種行為。 若要變更自行決定內容定義的其餘部分：
1. 返回「步驟 2」，並執行下列作業：

    a. 下載 *selfasserted* 檔案。

    b. 複製檔案內容。

    c. 建立新的檢視 selfasserted。

    d. 將 selfasserted 新增至 **Home** 控制器。

2. 返回「步驟 4」，並執行下列作業： 

    a. 在您的擴充原則中，尋找包含 `Id="api.selfasserted"`、`Id="api.localaccountsignup"` 和 `Id="api.localaccountpasswordreset"` 的 `<ContentDefinition>` 節點。

    b. 將 `LoadUri` 屬性設定為您的 selfasserted URI。

3. 返回「步驟 8.2」，並變更您的程式碼以接受查詢字串參數，但這次變更為 selfasserted 函式。 

4. 上傳 TrustFrameworkExtensions.xml 原則，確定它通過驗證。

5. 執行原則測試，然後選取 [立即註冊] 來查看結果。

## <a name="optional-download-the-complete-policy-files-and-code"></a>(選用) 下載完整的原則檔案和程式碼
* 在完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)逐步解說之後，建議您使用自己的自訂原則檔案來建置您的情節。 我們已提供[範例原則檔案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-ui-customization)，供您參考。
* 您可以從[供參考的範例 Visual Studio 解決方案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-ui-customization)中下載完整程式碼。




