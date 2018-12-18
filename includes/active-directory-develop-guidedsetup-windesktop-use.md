---
title: 包含檔案
description: 包含檔案
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/18/2018
ms.author: andret
ms.custom: include file
ms.openlocfilehash: d4ba15e4ad46044c04c242c8805af9f320e95150
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46368439"
---
## <a name="use-msal-to-get-a-token-for-the-microsoft-graph-api"></a>使用 MSAL 取得 Microsoft Graph API 的權杖

在本節中，使用 MSAL 取得 Microsoft Graph API 的權杖。

1.  在 MainWindow.xaml.cs 檔案中，將 MSAL 的參考新增至類別：

    ```csharp
    using Microsoft.Identity.Client;
    ```

2. 使用下列程式碼來取代 `MainWindow` 類別程式碼：

    ```csharp
    public partial class MainWindow : Window
    {
        //Set the API Endpoint to Graph 'me' endpoint
        string _graphAPIEndpoint = "https://graph.microsoft.com/v1.0/me";

        //Set the scope for API call to user.read
        string[] _scopes = new string[] { "user.read" };

        public MainWindow()
        {
            InitializeComponent();
        }

        /// <summary>
        /// Call AcquireTokenAsync - to acquire a token requiring user to sign-in
        /// </summary>
        private async void CallGraphButton_Click(object sender, RoutedEventArgs e)
        {
            AuthenticationResult authResult = null;

            var app = App.PublicClientApp;
            ResultText.Text = string.Empty;
            TokenInfoText.Text = string.Empty;

            var accounts = await app.GetAccountsAsync();

            try
            {
                authResult = await app.AcquireTokenSilentAsync(_scopes, accounts.FirstOrDefault());
            }
            catch (MsalUiRequiredException ex)
            {
                // A MsalUiRequiredException happened on AcquireTokenSilentAsync. This indicates you need to call AcquireTokenAsync to acquire a token
                System.Diagnostics.Debug.WriteLine($"MsalUiRequiredException: {ex.Message}");

                try
                {
                    authResult = await App.PublicClientApp.AcquireTokenAsync(_scopes);
                }
                catch (MsalException msalex)
                {
                    ResultText.Text = $"Error Acquiring Token:{System.Environment.NewLine}{msalex}";
                }
            }
            catch (Exception ex)
            {
                ResultText.Text = $"Error Acquiring Token Silently:{System.Environment.NewLine}{ex}";
                return;
            }

            if (authResult != null)
            {
                ResultText.Text = await GetHttpContentWithToken(_graphAPIEndpoint, authResult.AccessToken);
                DisplayBasicTokenInfo(authResult);
                this.SignOutButton.Visibility = Visibility.Visible;
            }
        }
    }
    ```

<!--start-collapse-->
### <a name="more-information"></a>詳細資訊

#### <a name="get-a-user-token-interactively"></a>以互動方式取得使用者權杖

呼叫 `AcquireTokenAsync` 方法時會顯示一個視窗，提示使用者登入。 當使用者第一次需要存取受保護的資源時，應用程式通常會要求使用者以互動方式登入。 當取得權杖的無訊息作業失敗 (例如，使用者的密碼過期) 時，使用者也可能需要登入。

#### <a name="get-a-user-token-silently"></a>以無訊息方式取得使用者權杖

`AcquireTokenSilentAsync` 方法會處理權杖取得和更新作業，不需要與使用者進行任何互動。 在 `AcquireTokenAsync` 第一次執行之後，`AcquireTokenSilentAsync` 就會成為用來取得權杖的常用方法，以在後續呼叫中存取受保護的資源，因為系統會以無訊息方式進行呼叫來要求或更新權杖。

最後，`AcquireTokenSilentAsync` 方法會失敗。 失敗的原因可能是使用者已經登出，或已經在其他裝置上變更其密碼。 當 MSAL 偵測到可透過要求執行互動式動作來解決問題時，就會發出一個 `MsalUiRequiredException` 例外狀況。 您的應用程式可以透過兩種方式處理此例外狀況：

* 它可以立即對 `AcquireTokenAsync` 進行呼叫。 此呼叫會促使系統提示使用者登入。 此模式通常用於沒有離線內容可供使用者使用的線上應用程式。 此引導式設定所產生的範例會遵循此模式，您可以在第一次執行範例時看到它運作。 

* 因為沒有任何使用者用過該應用程式，所以 `PublicClientApp.Users.FirstOrDefault()` 會包含一個 null 值，而且會擲回 `MsalUiRequiredException` 例外狀況。 

* 然後範例中的程式碼會透過呼叫 `AcquireTokenAsync` 來處理例外狀況，進而提示使用者登入。

* 其可改為對使用者呈現視覺指示，讓使用者知道需要透過互動方式登入，以便選取正確的登入時機。 或者，應用程式可以稍後重試 `AcquireTokenSilentAsync`。 此方式通常用於使用者可以使用其他應用程式功能，不需要因此中斷作業的情況，例如，應用程式中有離線內容可供使用時。 在此情況下，使用者可以決定何時登入以存取受保護的資源，或重新整理過期的資訊。 此外，當網路暫時無法使用而後還原時，應用程式可以決定是否重試 `AcquireTokenSilentAsync`。
<!--end-collapse-->

## <a name="call-the-microsoft-graph-api-by-using-the-token-you-just-obtained"></a>使用您剛剛取得的權杖呼叫 Microsoft Graph API

將下列新方法新增至 `MainWindow.xaml.cs`。 此方法是用來使用授權標頭針對圖形 API 提出 `GET` 要求：

```csharp
/// <summary>
/// Perform an HTTP GET request to a URL using an HTTP Authorization header
/// </summary>
/// <param name="url">The URL</param>
/// <param name="token">The token</param>
/// <returns>String containing the results of the GET operation</returns>
public async Task<string> GetHttpContentWithToken(string url, string token)
{
    var httpClient = new System.Net.Http.HttpClient();
    System.Net.Http.HttpResponseMessage response;
    try
    {
        var request = new System.Net.Http.HttpRequestMessage(System.Net.Http.HttpMethod.Get, url);
        //Add the token in Authorization header
        request.Headers.Authorization = new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token);
        response = await httpClient.SendAsync(request);
        var content = await response.Content.ReadAsStringAsync();
        return content;
    }
    catch (Exception ex)
    {
        return ex.ToString();
    }
}
```

<!--start-collapse-->
### <a name="more-information-about-making-a-rest-call-against-a-protected-api"></a>針對受保護 API 進行 REST 呼叫的相關詳細資訊

在這個範例應用程式中，使用 `GetHttpContentWithToken` 方法來針對受保護資源提出 HTTP `GET` 要求，那些受保護資源需要權杖才能存取，然後再將內容傳回給使用者。 此方法會在「HTTP 授權標頭」中新增取得的權杖。 對於此範例，資源為 Microsoft Graph API *me* 端點，它會顯示使用者的設定檔資訊。
<!--end-collapse-->

## <a name="add-a-method-to-sign-out-a-user"></a>新增方法來將使用者登出

若要登出使用者，將下列方法新增到您的 `MainWindow.xaml.cs` 檔案：

```csharp
/// <summary>
/// Sign out the current user
/// </summary>
private async void SignOutButton_Click(object sender, RoutedEventArgs e)
{
    var accounts = await App.PublicClientApp.GetAccountsAsync(); 

    if (accounts.Any())
    {
        try
        {
            await App.PublicClientApp.RemoveAsync(accounts.FirstOrDefault()); 
            this.ResultText.Text = "User has signed-out";
            this.CallGraphButton.Visibility = Visibility.Visible;
            this.SignOutButton.Visibility = Visibility.Collapsed;
        }
        catch (MsalException ex)
        {
            ResultText.Text = $"Error signing-out user: {ex.Message}";
        }
    }
}
```

<!--start-collapse-->
### <a name="more-information-about-user-sign-out"></a>使用者登出的詳細資訊

`SignOutButton_Click` 方法會從 MSAL 使用者快取中移除使用者，這可有效告知 MSAL 忘記目前的使用者，只有將此作業設為互動式作業，未來取得權杖的要求才能成功。

雖然此範例中的應用程式支援單一使用者，MSAL 也支援可同時登入多個帳戶的案例。 例如，使用者擁有多個帳戶的電子郵件應用程式。
<!--end-collapse-->

## <a name="display-basic-token-information"></a>顯示基本權杖資訊

若要顯示和權杖有關的基本資訊，將下列方法新增到您的 MainWindow.xaml.cs 檔案：

```csharp
/// <summary>
/// Display basic information contained in the token
/// </summary>
private void DisplayBasicTokenInfo(AuthenticationResult authResult)
{
    TokenInfoText.Text = "";
    if (authResult != null)
    {
        TokenInfoText.Text += $"Username: {authResult.Account.Username}" + Environment.NewLine;
        TokenInfoText.Text += $"Token Expires: {authResult.ExpiresOn.ToLocalTime()}" + Environment.NewLine;
        TokenInfoText.Text += $"Access Token: {authResult.AccessToken}" + Environment.NewLine;
    }
}
```

<!--start-collapse-->
### <a name="more-information"></a>詳細資訊

除了用來呼叫 Microsoft Graph API 的存取權杖之外，使用者登入之後，MSAL 也會取得 ID 權杖。 此權杖包含與使用者相關的一小部分資訊。 `DisplayBasicTokenInfo` 方法會顯示權杖中包含的基本資訊。 例如，它會顯示使用者的顯示名稱和 ID，以及權杖到期日期和代表存取權杖本身的字串。 您可以選取 [呼叫 Microsoft Graph API] 按鈕多次，以了解相同的權杖如何重複用於多個後續要求。 您也可以在 MSAL 決定應該要更新權杖時，看到延長的到期日期。
<!--end-collapse-->

