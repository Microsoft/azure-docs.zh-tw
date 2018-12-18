---
title: Azure AD Xamarin 入門 | Microsoft Docs
description: 建置 Xamarin 應用程式來與 Azure AD 整合進行登入，並使用 OAuth 呼叫受 Azure AD 保護的 API。
services: active-directory
documentationcenter: xamarin
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 198cd2c3-f7c8-4ec2-b59d-dfdea9fe7d95
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: mobile-xamarin
ms.devlang: dotnet
ms.topic: article
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 8af6846da78d12460b7866297c9802c5dab20a69
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967518"
---
# <a name="quickstart-build-a-xamarin-app-that-integrates-microsoft-sign-in"></a>快速入門：建置可整合 Microsoft 登入的 Xamarin 應用程式

[!INCLUDE [active-directory-develop-applies-v1-adal](../../../includes/active-directory-develop-applies-v1-adal.md)]

Xamarin 可讓您使用 C# 撰寫可在 iOS、Android 和 Windows (行動裝置和電腦) 上執行的行動應用程式。 如果您正在建置使用 Xamarin 的應用程式，Azure Active Directory (Azure AD) 可讓您簡單地以其 Azure AD 帳戶驗證使用者。 應用程式也可以安全地使用任何受 Azure AD 保護的 Web API，例如 Office 365 API 或 Azure API。

對於需要存取受保護資源的 Xamarin 應用程式，Azure AD 提供 Active Directory 驗證程式庫 (ADAL)。 ADAL 的唯一目的是為了讓應用程式輕鬆取得存取權杖。 為了示範有多麼容易，本文說明如何建置 DirectorySearcher 應用程式︰

* 可在 iOS、Android、Windows 桌面、Windows Phone 和 Windows 市集上執行。
* 使用單一可攜式類別庫 (PCL) 來驗證使用者並取得 Azure AD Graph API 的權杖。
* 搜尋目錄以尋找具有指定 UPN 的使用者。

## <a name="prerequisites"></a>必要條件

* 下載[基本架構專案](https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-DotNet/archive/skeleton.zip)或下載[完整的範例](https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-DotNet/archive/complete.zip)。 每一個下載都是 Visual Studio 2013 解決方案。
* 您還需要一個可以建立使用者並註冊應用程式的 Azure AD 租用戶。 如果您還沒有租用戶， [了解如何取得租用戶](quickstart-create-new-tenant.md)。

當您準備就緒，請遵循接下來四個章節的程序。

## <a name="step-1-set-up-your-xamarin-development-environment"></a>步驟 1：設定您的 Xamarin 開發環境

因為本教學課程包含 iOS、Android 和 Windows 專案，所以您需要 Visual Studio 和 Xamarin。 若要建立必要的環境，請完成 MSDN 上[設定和安裝 Visual Studio 和 Xamarin](https://msdn.microsoft.com/library/mt613162.aspx) 的處理。 這些指示包含的資料，可供您於等待安裝完成時檢閱，以深入了解 Xamarin。

完成設定之後，您可以在 Visual Studio 中開啟解決方案。 您會在其中看到六個專案：五個平台特定專案和一個 PCL，DirectorySearcher.cs，可跨所有平台共用。

## <a name="step-2-register-the-directorysearcher-app"></a>步驟 2：註冊 DirectorySearcher 應用程式

若要讓應用程式取得權杖，您必須先在 Azure AD 租用戶中註冊這個應用程式，並授權它存取 Azure AD Graph API。 方式如下：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在頂端列中，按一下您的帳戶。 然後，在 [目錄] 清單下，選取您要註冊應用程式的 Active Directory 租用戶。
3. 按一下左窗格中的 [所有服務]，然後選取 [Azure Active Directory]。
4. 按一下 [應用程式註冊]，然後選取 [新增]。
5. 若要建立新的**原生用戶端應用程式**，請遵照提示進行。
  * [名稱] 向使用者說明該應用程式。
  * [重新導向 URI] 是配置與字串的組合，Azure AD 會用它來傳回權杖回應。 輸入一個值 (例如， http://DirectorySearcher) 。
6. 完成註冊之後，Azure AD 會為應用程式指派唯一的應用程式識別碼。 從 [應用程式] 索引標籤複製值，因為稍後將會需要它。
7. 在 [設定] 頁面中，選取 [必要的權限]，然後選取 [新增]。
8. 選取 [Microsoft Graph] 做為 API。 在 [委派的權限] 下，新增 [讀取目錄資料] 權限。 這個動作可讓應用程式查詢使用者的圖形 API。

## <a name="step-3-install-and-configure-adal"></a>步驟 3：安裝及設定 ADAL

既然您在 Azure AD 中已經擁有應用程式，您可以安裝 ADAL，並撰寫身分識別相關程式碼。 若要讓 ADAL 與 Azure AD 進行通訊，請提供它一些應用程式註冊相關資訊。

1. 使用 Package Manager Console 將 ADAL 新增到 DirectorySearcher 專案中。

    `
    PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirectorySearcherLib
    `

    `
    PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-Android
    `

    `
    PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-Desktop
    `

    `
    PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-iOS
    `

    `
    PM> Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -ProjectName DirSearchClient-Universal
    `

    請注意，每個專案會新增兩個程式庫參考：ADAL 的 PCL 部分和平台特定部分。
2. 在 DirectorySearcherLib 專案中，開啟 DirectorySearcher.cs。
3. 將類別成員值取代為您在 Azure 入口網站中輸入的值。 每當使用 ADAL 時，您的程式碼便會參考這些值。

  * *tenant* 是您 Azure AD 租用戶的網域 (例如 contoso.onmicrosoft.com)。
  * *clientId* 是您從入口網站複製的應用程式用戶端識別碼。
  * returnUri 是您在入口網站中輸入的重新導向 URI (例如， http://DirectorySearcher) 。

## <a name="step-4-use-adal-to-get-tokens-from-azure-ad"></a>步驟 4：使用 ADAL 來取得 Azure AD 的權杖

幾乎所有應用程式的驗證邏輯採用 `DirectorySearcher.SearchByAlias(...)`。 平台特定專案所需的只是將內容參數傳遞至 `DirectorySearcher` PCL。

1. 開啟DirectorySearcher.cs，然後將新的參數新增至 `SearchByAlias(...)` 方法。 `IPlatformParameters` 是個內容參數，其中包含 ADAL 必須執行驗證的平台特定物件。

    ```csharp
    public static async Task<List<User>> SearchByAlias(string alias, IPlatformParameters parent)
    {
    ```

2. 初始化 `AuthenticationContext`，這是 ADAL 的主要類別。 這個動作會傳遞它與 Azure AD 通訊所需的 ADAL 座標。
3. 呼叫 `AcquireTokenAsync(...)`，它可接受 `IPlatformParameters` 物件，並叫用將權杖傳回應用程式所需的驗證流程。

    ```csharp
    ...
        AuthenticationResult authResult = null;
        try
        {
            AuthenticationContext authContext = new AuthenticationContext(authority);
            authResult = await authContext.AcquireTokenAsync(graphResourceUri, clientId, returnUri, parent);
        }
        catch (Exception ee)
        {
            results.Add(new User { error = ee.Message });
            return results;
        }
    ...
    ```

    `AcquireTokenAsync(...)` 會先嘗試為要求的資源 (在此範例中會是圖形 API) 傳回權杖，而不提示使用者輸入其認證 (透過快取或重新整理舊的權杖)。 必要時，它才會在取得要求的權杖之前，對使用者顯示 Azure AD 登入頁面。
4. 將存取權杖附加至**授權**標頭中的圖形 API 要求：

    ```csharp
    ...
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", authResult.AccessToken);
    ...
    ```

`DirectorySearcher` PCL 和應用程式的身分識別相關程式碼到此告一段落。 剩下的工作只是在每個平台檢視中呼叫 `SearchByAlias(...)` 方法，並在必要的地方新增程式碼以便正確地處理 UI 生命週期。

### <a name="android"></a>Android
1. 在 MainActivity.cs 中，在按鈕點擊處理常式中新增 `SearchByAlias(...)` 的呼叫：

    ```csharp
    List<User> results = await DirectorySearcher.SearchByAlias(searchTermText.Text, new PlatformParameters(this));
    ```
2. 覆寫 `OnActivityResult` 生命週期方法，將任何驗證重新導向轉送回到適當的方法。 ADAL 針對 Android 中的此項作業提供協助程式方法：

    ```csharp
    ...
    protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
    {
        base.OnActivityResult(requestCode, resultCode, data);
        AuthenticationAgentContinuationHelper.SetAuthenticationAgentContinuationEventArgs(requestCode, resultCode, data);
    }
    ...
    ```

### <a name="windows-desktop"></a>Windows 桌面

在 MainWindow.xaml.cs 中，只需進行 `SearchByAlias(...)` 呼叫，即可在桌面的 `PlatformParameters` 物件中傳遞 `WindowInteropHelper`：

```csharp
List<User> results = await DirectorySearcher.SearchByAlias(
  SearchTermText.Text,
  new PlatformParameters(PromptBehavior.Auto, this.Handle));
```

#### <a name="ios"></a>iOS
在 DirSearchClient_iOSViewController.cs 中，iOS `PlatformParameters` 物件會參考檢視控制器︰

```csharp
List<User> results = await DirectorySearcher.SearchByAlias(
  SearchTermText.Text,
  new PlatformParameters(PromptBehavior.Auto, this.Handle));
```

### <a name="windows-universal"></a>Windows Universal
在 Windows 通用中，開啟 MainPage.xaml.cs，然後實作 `Search` 方法。 此方法會使用共用專案中的協助程式方法，視需要更新 UI。

```csharp
...
List<User> results = await DirectorySearcherLib.DirectorySearcher.SearchByAlias(SearchTermText.Text, new PlatformParameters(PromptBehavior.Auto, false));
...
```

您現在擁有一個能夠在五個不同平台之間驗證使用者並使用 OAuth 2.0 安全地呼叫 Web API 的運作中 Xamarin 應用程式。

## <a name="step-5-populate-your-tenant"></a>步驟 5︰填入租用戶 

如果您尚未在租用戶中填入使用者，現在便可以這麼做。

1. 執行 DirectorySearcher 應用程式，然後使用其中一個使用者來登入。
2. 根據 UPN 搜尋其他使用者。

## <a name="next-steps"></a>後續步驟

ADAL 可讓您輕鬆地將常見的身分識別功能納入應用程式。 它會為您處理一切麻煩的事，包括快取管理、OAuth 通訊協定支援、向使用者顯示登入 UI，以及重新整理過期權杖。 您只需要知道單一 API 呼叫 `authContext.AcquireToken*(…)`。

* 請下載[完整的範例](https://github.com/AzureADQuickStarts/NativeClient-MultiTarget-DotNet/archive/complete.zip) (不含設定值)。
* 了解如何[使用 Azure AD 保護 .NET Web API](quickstart-v1-dotnet-webapi.md)。
