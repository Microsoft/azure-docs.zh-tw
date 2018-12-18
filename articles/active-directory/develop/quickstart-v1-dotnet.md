---
title: 從 .NET 桌面 (WPF) 應用程式登入使用者並呼叫 Microsoft Graph API | Microsoft Docs
description: 深入了解如何建置 .NET Windows 桌面應用程式來與 Azure AD 整合進行登入，並使用 OAuth 2.0 呼叫受 Azure AD 保護的 API。
services: active-directory
documentationcenter: .net
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: ed33574f-6fa3-402c-b030-fae76fba84e1
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: jmprieur
ms.custom: aaddev
ms.openlocfilehash: f9389a7c0e80f075c01f2236fa1bdf9dc9544ac6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987436"
---
# <a name="quickstart-sign-in-users-and-call-the-microsoft-graph-api-from-a-net-desktop-wpf-app"></a>快速入門：從 .NET 桌面 (WPF) 應用程式登入使用者並呼叫 Microsoft Graph API

[!INCLUDE [active-directory-develop-applies-v1-adal](../../../includes/active-directory-develop-applies-v1-adal.md)]

針對需要存取受保護資源的 .NET 原生用戶端，Azure Active Directory (Azure AD) 提供 Active Directory 驗證程式庫 (ADAL)。 ADAL 可讓您的應用程式輕鬆取得存取權杖。 

在本快速入門中，您將了解如何建置 .NET WPF 待辦事項清單應用程式：

* 使用 OAuth 2.0 驗證通訊協定取得呼叫 Azure AD Graph API 的存取權杖。
* 在目錄中搜尋具有指定別名的使用者。
* 將使用者登出。

若要建置可完整運作的應用程式，您必須：

1. 向 Azure AD 註冊您的應用程式。
2. 安裝及設定 ADAL。
3. 使用 ADAL 來取得 Azure AD 的權杖。

## <a name="prerequisites"></a>必要條件

若要開始，請完成以下先決條件：

* [下載應用程式基本架構](https://github.com/AzureADQuickStarts/NativeClient-DotNet/archive/skeleton.zip)或[下載完整的範例](https://github.com/AzureADQuickStarts/NativeClient-DotNet/archive/complete.zip)
* 擁有 Azure AD 租用戶，供您建立使用者並註冊應用程式。 如果您還沒有租用戶， [了解如何取得租用戶](quickstart-create-new-tenant.md)。

## <a name="step-1-register-the-directorysearcher-application"></a>步驟 1：註冊 DirectorySearcher 應用程式

若要啟用您的應用程式以取得權杖，在 Azure AD 租用戶中註冊您的應用程式，並對其授與存取 Azure AD Graph API 的權限：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在頂端列上選取您的帳戶，然後在 [目錄] 清單底下，選擇您要註冊應用程式的 Active Directory 租用戶。
3. 選取左側導覽區中的 [所有服務]，然後選擇 [Azure Active Directory]。
4. 在 [應用程式註冊] 上，選擇 [新增]。
5. 遵照提示進行，並建立新的**原生**用戶端應用程式。
    * 應用程式的**名稱**將對使用者說明您的應用程式
    * 「 **重新導向 Uri** 」是配置和字串的組合，Azure AD 可用它來傳回權杖回應。 輸入應用程式特定的值，例如 `http://DirectorySearcher`。

6. 完成註冊後，AAD 會為您的應用程式指派唯一的應用程式識別碼。 您會在後續章節中用到這個值，所以請從應用程式頁面中複製此值。
7. 從 [設定] 頁面，選擇 [必要權限]，然後選擇 [新增]。 選取 [Microsoft Graph] 作為 API，然後在 [委派權限] 底下新增 [讀取目錄資料] 權限。 設定此權限可讓您的應用程式查詢 Graph API 的使用者。

## <a name="step-2-install-and-configure-adal"></a>步驟 2：安裝及設定 ADAL

既然您在 Azure AD 中已經擁有應用程式，您可以安裝 ADAL，並撰寫身分識別相關程式碼。 為了讓 ADAL 能與 Azure AD 通訊，您需要提供一些應用程式註冊的相關資訊。

1. 首先，使用 Package Manager Console，將 ADAL 新增至 `DirectorySearcher` 專案。

    ```
    PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

1. 在 `DirectorySearcher` 專案中，開啟 `app.config`。
1. 取代 `<appSettings>` 區段中的元素值，以反映您在 Azure 入口網站中所輸入的值。 每當使用 ADAL 時，您的程式碼便會參考這些值。
  * `ida:Tenant` 是指您的 Azure AD 租用戶網域，例如 contoso.onmicrosoft.com
  * `ida:ClientId` 是指您從入口網站複製的應用程式用戶端識別碼。
  * `ida:RedirectUri` 是您在入口網站中註冊的重新導向 URL。

## <a name="step-3-use-adal-to-get-tokens-from-azure-ad"></a>步驟 3：使用 ADAL 從 Azure AD 取得權杖

ADAL 的基本原則是每當您的應用程式需要存取權杖時，您的應用程式只需呼叫 `authContext.AcquireTokenAsync(...)`，ADAL 就會進行其餘工作。

1. 在 `DirectorySearcher` 專案中，開啟 `MainWindow.xaml.cs`。
1. 尋找 `MainWindow()` 方法。 
1. 初始化應用程式的 `AuthenticationContext` - ADAL 的主要類別。 您在 `AuthenticationContext` 將 ADAL 與 Azure AD 通訊所需的座標傳給 ADAL，並告訴它如何快取權杖。

    ```csharp
    public MainWindow()
    {
        InitializeComponent();

        authContext = new AuthenticationContext(authority, new FileCache());

        CheckForCachedToken();
    }
    ```

1. 找到 `Search(...)` 方法，這是在使用者選取應用程式 UI 的 [搜尋] 按鈕時所呼叫的方法。 這個方法會對 Azure AD Graph API 提出 GET 要求，以查詢 UPN 開頭為指定搜尋詞彙的使用者。
1. 若要查詢 Graph API，要求的 `Authorization` 標頭必須包含 access_token - ADAL 可以提供這方面的協助。

    ```csharp
    private async void Search(object sender, RoutedEventArgs e)
    {
        // Validate the Input String
        if (string.IsNullOrEmpty(SearchText.Text))
        {
            MessageBox.Show("Please enter a value for the To Do item name");
            return;
        }

        // Get an Access Token for the Graph API
        AuthenticationResult result = null;
        try
        {
            result = await authContext.AcquireTokenAsync(graphResourceId, clientId, redirectUri, new PlatformParameters(PromptBehavior.Auto));
            UserNameLabel.Content = result.UserInfo.DisplayableId;
            SignOutButton.Visibility = Visibility.Visible;
        }
        catch (AdalException ex)
        {
            // An unexpected error occurred, or user canceled the sign in.
            if (ex.ErrorCode != "access_denied")
                MessageBox.Show(ex.Message);

            return;
        }

        ...
    }
    ```

    當您的應用程式透過呼叫 `AcquireTokenAsync(...)`要求權杖時，ADAL 會嘗試在不要求使用者認證的情況下傳回權杖。
    * 如果 ADAL 決定使用者需要登入才能取得權杖，它會顯示登入對話方塊、收集使用者的認證，並在成功驗證後傳回權杖。 
    * 如果基於任何原因 ADAL 無法傳回權杖，則會擲回 `AdalException`。

1. 請注意，`AuthenticationResult` 物件包含 `UserInfo` 物件，可用來收集您的應用程式可能需要的資訊。 在 DirectorySearcher 中，`UserInfo` 用來以使用者識別碼自訂應用程式的 UI。
1. 當使用者選取 [登出] 按鈕時，確保下次呼叫 `AcquireTokenAsync(...)` 時會要求使用者登入。 您可以使用 ADAL 清除權杖快取，輕鬆地完成這項操作：

    ```csharp
    private void SignOut(object sender = null, RoutedEventArgs args = null)
    {
        // Clear the token cache
        authContext.TokenCache.Clear();

        ...
    }
    ```

    如果使用者未按一下 [登出] 按鈕，您需要維護使用者的工作階段，供他們下一次執行 DirectorySearcher 繼續使用。 當應用程式啟動時，您可以在 ADAL 的權杖快取中檢查現有的權杖，並據此更新 UI。

1. 在 `CheckForCachedToken()` 方法中，再次呼叫 `AcquireTokenAsync(...)`，這次傳入 `PromptBehavior.Never` 參數。 `PromptBehavior.Never` 告訴 ADAL 不要提示使用者進行登入，而如果 ADAL 無法傳回權杖，則應該改為擲回例外狀況。

    ```csharp
    public async void CheckForCachedToken() 
    {
        // As the application starts, try to get an access token without prompting the user. If one exists, show the user as signed in.
        AuthenticationResult result = null;
        try
        {
            result = await authContext.AcquireTokenAsync(graphResourceId, clientId, redirectUri, new PlatformParameters(PromptBehavior.Never));
        }
        catch (AdalException ex)
        {
            if (ex.ErrorCode != "user_interaction_required")
            {
                // An unexpected error occurred.
                MessageBox.Show(ex.Message);
            }

            // If user interaction is required, proceed to main page without singing the user in.
            return;
        }

        // A valid token is in the cache
        SignOutButton.Visibility = Visibility.Visible;
        UserNameLabel.Content = result.UserInfo.DisplayableId;
    }
    ```

恭喜！ 您現在有一個運作中的 .NET WPF 應用程式，可以驗證使用者、使用 OAuth 2.0 安全地呼叫 Web API，以及取得使用者的基本資訊。 如果您還沒有這麼做，現在是將一些使用者植入租用戶的時候。 執行 DirectorySearcher 應用程式，並使用其中一個使用者登入。 根據 UPN 搜尋其他使用者。 關閉應用程式，然後重新執行它。 請注意，使用者工作階段會維持不變。 登出，再以另一個使用者身分重新登入。

ADAL 可讓您輕鬆地將這些常見的身分識別功能納入您的應用程式。 它會為您處理麻煩的事，包括快取管理、OAuth 通訊協定支援、向使用者顯示登入 UI、重新整理過期權杖等等。 您唯一需要知道的就是單一 API 呼叫， `authContext.AcquireTokenAsync(...)`。

如需參考資料，請參閱 [GitHub](https://github.com/AzureADQuickStarts/NativeClient-DotNet/archive/complete.zip) 上的完整的範例 (不含您的組態值)。

## <a name="next-steps"></a>後續步驟

深入了解如何使用 OAuth 2.0 持有人存取權杖來保護 Web API。
> [!div class="nextstepaction"]
> [使用 Azure AD 保護 .NET Web API >>](quickstart-v1-dotnet-webapi.md)
