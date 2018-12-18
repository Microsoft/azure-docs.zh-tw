---
title: 在 Azure Active Directory B2C 中驗證、註冊、編輯設定檔 | Microsoft Docs
description: 如何使用 Azure Active Directory B2C 來建置包含登入、註冊及設定檔管理功能的 Windows 桌面應用程式。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 01/07/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: bd504beabbb126db2cd90ac010dbc2757e571185
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37441894"
---
# <a name="azure-ad-b2c-build-a-windows-desktop-app"></a>Azure AD B2C：建置 Windows 桌面應用程式
如果您利用 Azure Active Directory (Azure AD) B2C，只要幾個簡短的步驟，就在您的桌面應用程式中新增功能強大的自助式身分識別管理功能。 本文章說明如何建立 .NET Windows Presentation Foundation (WPF)「待辦事項清單」應用程式，其中包含使用者註冊、登入和設定檔管理的功能。 該應用程式將支援以使用者名稱或電子郵件來註冊及登入的功能。 它也會支援以社交帳戶 (例如 Facebook 和 Google) 來註冊及登入。

## <a name="get-an-azure-ad-b2c-directory"></a>取得 Azure AD B2C 目錄
您必須先建立目錄或租用戶，才可使用 Azure AD B2C。  目錄是適用於所有使用者、app、群組等項目的容器。 如果您還沒有此資源，請先 [建立 B2C 目錄](active-directory-b2c-get-started.md) ，再繼續進行本指南。

## <a name="create-an-application"></a>建立應用程式
接著，您必須在 B2C 目錄中建立應用程式。 這會提供必要資訊給 Azure AD，讓它與應用程式安全地通訊。 若要建立應用程式，請遵循 [這些指示](active-directory-b2c-app-registration.md)。  請務必：

* 在應用程式中加入 **原生用戶端** 。
* 複製**重新導向 URI** `urn:ietf:wg:oauth:2.0:oob`。 這是此程式碼範例的預設 URL。
* 複製指派給您的應用程式的 **應用程式識別碼** 。 稍後您將會用到此資訊。

## <a name="create-your-policies"></a>建立您的原則
在 Azure AD B2C 中，每個使用者體驗皆是由某個 [原則](active-directory-b2c-reference-policies.md)所定義。 此程式碼範例包含三種身分識別體驗：註冊、登入和編輯設定檔。 您必須為每個類型建立一個原則，如 [原則參考文章](active-directory-b2c-reference-policies.md#create-a-sign-up-policy)中所述。 建立這三個原則時，請務必：

* 在識別提供者刀鋒視窗中，選擇 [使用者識別碼註冊] 或 [電子郵件註冊]。
* 在註冊原則中，選擇 [顯示名稱]  和其他註冊屬性。
* 針對每個原則選擇 [顯示名稱] 和 [物件識別碼] 宣告做為應用程式宣告。 您也可以選擇其他宣告。
* 在您建立每個原則之後，請複製原則的 [名稱]  。 其前置詞應該為 `b2c_1_`。  稍後您將需要這些原則名稱。

[!INCLUDE [active-directory-b2c-devquickstarts-policy](../../includes/active-directory-b2c-devquickstarts-policy.md)]

當您成功建立這三個原則之後，就可以開始建置您的 app。

## <a name="download-the-code"></a>下載程式碼
本教學課程的程式碼保留在 [GitHub](https://github.com/AzureADQuickStarts/B2C-NativeClient-DotNet)。 如要依照指示建置範例，請 [下載 .zip 檔案格式的基本架構專案](https://github.com/AzureADQuickStarts/B2C-NativeClient-DotNet/archive/skeleton.zip)。 您也可以複製基本架構：

```
git clone --branch skeleton https://github.com/AzureADQuickStarts/B2C-NativeClient-DotNet.git
```

完整的 App 也[提供 .zip 檔案格式](https://github.com/AzureADQuickStarts/B2C-NativeClient-DotNet/archive/complete.zip)，或放在相同儲存機制的 `complete` 分支中。

下載範例程式碼後，請開啟 Visual Studio .sln 檔案開始進行。 `TaskClient` 專案是使用者與之互動的 WPF 桌面應用程式。 基於本教學課程的目的，它會呼叫後端工作 Web API (裝載在 Azure 中) 以儲存每個使用者的待辦事項清單。  您不需要建置 Web API，我們已經為您執行它。

如要了解 Web API 如何利用 Azure AD B2C 來安全地驗證要求，請查看 [Web API 開始使用文章](active-directory-b2c-devquickstarts-api-dotnet.md)。

## <a name="execute-policies"></a>執行原則
您的應用程式與 Azure AD B2C 通訊時會傳送驗證訊息，其中指定它們想要在 HTTP 要求中執行的原則。 對於 .NET 桌面應用程式，您可以使用預覽 Microsoft Authentication Library (MSAL) 來傳送 OAuth 2.0 驗證訊息、執行原則，並取得會呼叫 Web API 的權杖。

### <a name="install-msal"></a>安裝 MSAL
使用 Visual Studio Package Manager Console 來把 MSAL 新增到 `TaskClient` 專案中。

```
PM> Install-Package Microsoft.Identity.Client -IncludePrerelease
```

### <a name="enter-your-b2c-details"></a>輸入 B2C 詳細資料
開啟檔案 `Globals.cs` ，然後用您自己的值來取代每個屬性值。 整個 `TaskClient` 都會使用這個類別來參考常用的值。

```csharp
public static class Globals
{
    ...

    // TODO: Replace these five default with your own configuration values
    public static string tenant = "fabrikamb2c.onmicrosoft.com";
    public static string clientId = "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6";
    public static string signInPolicy = "b2c_1_sign_in";
    public static string signUpPolicy = "b2c_1_sign_up";
    public static string editProfilePolicy = "b2c_1_edit_profile";

    ...
}
```

[!INCLUDE [active-directory-b2c-devquickstarts-tenant-name](../../includes/active-directory-b2c-devquickstarts-tenant-name.md)]

### <a name="create-the-publicclientapplication"></a>建立 PublicClientApplication
MSAL 的主要類別是 `PublicClientApplication`。 此類別代表您在 Azure AD B2C 系統中的應用程式。 當應用程式初始化時，請在 `MainWindow.xaml.cs` 中建立 `PublicClientApplication` 的執行個體。 這可用在整個視窗上。

```csharp
protected async override void OnInitialized(EventArgs e)
{
    base.OnInitialized(e);

    pca = new PublicClientApplication(Globals.clientId)
    {
        // MSAL implements an in-memory cache by default.  Since we want tokens to persist when the user closes the app,
        // we've extended the MSAL TokenCache and created a simple FileCache in this app.
        UserTokenCache = new FileCache(),
    };

    ...
```

### <a name="initiate-a-sign-up-flow"></a>起始註冊流程
當使用者選擇要註冊時，您會想要起始利用您所建立註冊原則的註冊流程。 藉由使用 MSAL，您只要呼叫 `pca.AcquireTokenAsync(...)`即可。 您傳遞給 `AcquireTokenAsync(...)` 的參數決定您會收到哪個權杖、用於驗證要求中的原則等等。

```csharp
private async void SignUp(object sender, RoutedEventArgs e)
{
    AuthenticationResult result = null;
    try
    {
        // Use the app's clientId here as the scope parameter, indicating that
        // you want a token to the your app's backend web API (represented by
        // the cloud hosted task API).  Use the UiOptions.ForceLogin flag to
        // indicate to MSAL that it should show a sign-up UI no matter what.
        result = await pca.AcquireTokenAsync(new string[] { Globals.clientId },
                string.Empty, UiOptions.ForceLogin, null, null, Globals.authority,
                Globals.signUpPolicy);

        // Upon success, indicate in the app that the user is signed in.
        SignInButton.Visibility = Visibility.Collapsed;
        SignUpButton.Visibility = Visibility.Collapsed;
        EditProfileButton.Visibility = Visibility.Visible;
        SignOutButton.Visibility = Visibility.Visible;

        // When the request completes successfully, you can get user
        // information from the AuthenticationResult
        UsernameLabel.Content = result.User.Name;

        // After the sign up successfully completes, display the user's To-Do List
        GetTodoList();
    }

    // Handle any exeptions that occurred during execution of the policy.
    catch (MsalException ex)
    {
        if (ex.ErrorCode != "authentication_canceled")
        {
            // An unexpected error occurred.
            string message = ex.Message;
            if (ex.InnerException != null)
            {
                message += "Inner Exception : " + ex.InnerException.Message;
            }

            MessageBox.Show(message);
        }

        return;
    }
}
```

### <a name="initiate-a-sign-in-flow"></a>起始登入流程
您可以用您起始註冊流程的相同方式來起始登入流程。 當使用者登入時，請用同樣的方式呼叫 MSAL，但這次是利用您的登入原則：

```csharp
private async void SignIn(object sender = null, RoutedEventArgs args = null)
{
    AuthenticationResult result = null;
    try
    {
        result = await pca.AcquireTokenAsync(new string[] { Globals.clientId },
                    string.Empty, UiOptions.ForceLogin, null, null, Globals.authority,
                    Globals.signInPolicy);
        ...
```

### <a name="initiate-an-edit-profile-flow"></a>起始編輯設定檔流程
同樣地，您可以用相同的方式執行編輯設定檔原則：

```csharp
private async void EditProfile(object sender, RoutedEventArgs e)
{
    AuthenticationResult result = null;
    try
    {
        result = await pca.AcquireTokenAsync(new string[] { Globals.clientId },
                    string.Empty, UiOptions.ForceLogin, null, null, Globals.authority,
                    Globals.editProfilePolicy);
```

在所有這些案例中，MSAL 會傳回 `AuthenticationResult` 中的權杖，或是擲回例外狀況。 每次從 MSAL 取得權杖時，您都可以使用 `AuthenticationResult.User` 物件來更新應用程式中的使用者資料，例如 UI。 ADAL 也會快取權杖，以供應用程式的其他部分使用。

### <a name="check-for-tokens-on-app-start"></a>在應用程式啟動時檢查權杖
您也可以使用 MSAL 來追蹤使用者的登入狀態。  在此應用程式中，我們想要讓使用者在關閉應用程式再予以重新開啟後仍保持登入狀態。  回到 `OnInitialized` 覆寫，使用 MSAL 的 `AcquireTokenSilent` 方法來檢查已快取的權杖︰

```csharp
AuthenticationResult result = null;
try
{
    // If the user has has a token cached with any policy, we'll display them as signed-in.
    TokenCacheItem tci = pca.UserTokenCache.ReadItems(Globals.clientId).Where(i => i.Scope.Contains(Globals.clientId) && !string.IsNullOrEmpty(i.Policy)).FirstOrDefault();
    string existingPolicy = tci == null ? null : tci.Policy;
    result = await pca.AcquireTokenSilentAsync(new string[] { Globals.clientId }, string.Empty, Globals.authority, existingPolicy, false);

    SignInButton.Visibility = Visibility.Collapsed;
    SignUpButton.Visibility = Visibility.Collapsed;
    EditProfileButton.Visibility = Visibility.Visible;
    SignOutButton.Visibility = Visibility.Visible;
    UsernameLabel.Content = result.User.Name;
    GetTodoList();
}
catch (MsalException ex)
{
    if (ex.ErrorCode == "failed_to_acquire_token_silently")
    {
        // There are no tokens in the cache.  Proceed without calling the To Do list service.
    }
    else
    {
        // An unexpected error occurred.
        string message = ex.Message;
        if (ex.InnerException != null)
        {
            message += "Inner Exception : " + ex.InnerException.Message;
        }
        MessageBox.Show(message);
    }
    return;
}
```

## <a name="call-the-task-api"></a>呼叫工作 API
您已經使用 MSAL 來執行原則並取得權杖。  當您想要使用這些權杖的其中一個來呼叫工作 API 時，您可以再次使用 MSAL 的 `AcquireTokenSilent` 方法來檢查已快取的權杖︰

```csharp
private async void GetTodoList()
{
    AuthenticationResult result = null;
    try
    {
        // Here we want to check for a cached token, independent of whatever policy was used to acquire it.
        TokenCacheItem tci = pca.UserTokenCache.ReadItems(Globals.clientId).Where(i => i.Scope.Contains(Globals.clientId) && !string.IsNullOrEmpty(i.Policy)).FirstOrDefault();
        string existingPolicy = tci == null ? null : tci.Policy;

        // Use AcquireTokenSilent to indicate that MSAL should throw an exception if a token cannot be acquired
        result = await pca.AcquireTokenSilentAsync(new string[] { Globals.clientId }, string.Empty, Globals.authority, existingPolicy, false);

    }
    // If a token could not be acquired silently, we'll catch the exception and show the user a message.
    catch (MsalException ex)
    {
        // There is no access token in the cache, so prompt the user to sign-in.
        if (ex.ErrorCode == "failed_to_acquire_token_silently")
        {
            MessageBox.Show("Please sign up or sign in first");
            SignInButton.Visibility = Visibility.Visible;
            SignUpButton.Visibility = Visibility.Visible;
            EditProfileButton.Visibility = Visibility.Collapsed;
            SignOutButton.Visibility = Visibility.Collapsed;
            UsernameLabel.Content = string.Empty;
        }
        else
        {
            // An unexpected error occurred.
            string message = ex.Message;
            if (ex.InnerException != null)
            {
                message += "Inner Exception : " + ex.InnerException.Message;
            }
            MessageBox.Show(message);
        }

        return;
    }
    ...
```

當您成功呼叫 `AcquireTokenSilentAsync(...)`，且在快取中找到權杖之後，便可以將該權杖新增至 HTTP 要求的 `Authorization` 標頭中。 工作 Web API 會使用此標頭來驗證讀取使用者待辦事項清單的要求︰

```csharp
    ...
    // Once the token has been returned by MSAL, add it to the http authorization header, before making the call to access the To Do list service.
    httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.Token);

    // Call the To Do list service.
    HttpResponseMessage response = await httpClient.GetAsync(Globals.taskServiceUrl + "/api/tasks");
    ...
```

## <a name="sign-the-user-out"></a>登出使用者
最後，您可以在使用者選取 [登出] 時，使用 MSAL 來結束使用者的應用程式工作階段。在使用 MSAL 時，您只要清除權杖快取中的所有權杖即可：

```csharp
private void SignOut(object sender, RoutedEventArgs e)
{
    // Clear any remnants of the user's session.
    pca.UserTokenCache.Clear(Globals.clientId);

    // This is a helper method that clears browser cookies in the browser control that MSAL uses, it is not part of MSAL.
    ClearCookies();

    // Update the UI to show the user as signed out.
    TaskList.ItemsSource = string.Empty;
    SignInButton.Visibility = Visibility.Visible;
    SignUpButton.Visibility = Visibility.Visible;
    EditProfileButton.Visibility = Visibility.Collapsed;
    SignOutButton.Visibility = Visibility.Collapsed;
    return;
}
```

## <a name="run-the-sample-app"></a>執行範例應用程式
最後，建置並執行範例。  使用電子郵件地址或使用者名稱來註冊應用程式。 登出，再以相同的使用者身分重新登入。 編輯該使用者的設定檔。 請登出，然後以不同的使用者身分登入。

## <a name="add-social-idps"></a>新增社交 IDP
目前，應用程式僅支援使用者以 **本機帳戶**來註冊及登入。 這些是儲存在 B2C 目錄中使用使用者名稱和密碼的帳戶。 藉由使用 Azure AD B2C，您可以新增對其他身分識別提供者 (IDP) 的支援，但不需變更任何程式碼。

若要將社交 IDP 加入至應用程式，請依照下列文章中的詳細指示開始。 針對您想要支援的每個 IDP，您必須在該系統中註冊應用程式並取得用戶端識別碼。

* [將 Facebook 設定為 IDP](active-directory-b2c-setup-fb-app.md)
* [將 Google 設定為 IDP](active-directory-b2c-setup-goog-app.md)
* [將 Amazon 設定為 IDP](active-directory-b2c-setup-amzn-app.md)
* [將 LinkedIn 設定為 IDP](active-directory-b2c-setup-li-app.md)

當您將身分識別提供者新增到 B2C 目錄之後，就必須分別編輯您的三個原則來包含新的 IDP，如 [原則參考文章](active-directory-b2c-reference-policies.md)中所述。 儲存您的原則之後，重新執行應用程式。 在每一次的身分識別體驗中，您應該會看到新的 IDP 已加入成為登入和註冊選項。

您可以在範例應用程式上對原則進行試驗，並觀察試驗結果。 新增或移除 IDP、管理應用程式宣告，或變更註冊屬性。 試驗到您能夠了解原則、驗證要求和 MSAL 如何結合在一起為止。

為供您參考，我們提供 [.zip 檔案格式](https://github.com/AzureADQuickStarts/B2C-NativeClient-DotNet/archive/complete.zip)的完整範例。 您也可以從 Github 複製它：

```git clone --branch complete https://github.com/AzureADQuickStarts/B2C-NativeClient-DotNet.git```
