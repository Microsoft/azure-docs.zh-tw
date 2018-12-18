---
title: 在 Azure Active Directory B2C 中使用圖形 API | Microsoft Docs
description: 如何使用應用程式身分識別對 B2C 租用戶呼叫圖形 API，以將程序自動化。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/07/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 013fd019771fb9994c87e363c78eae36431f91de
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953220"
---
# <a name="azure-ad-b2c-use-the-azure-ad-graph-api"></a>Azure AD B2C：使用 Azure AD 圖形 API

>[!NOTE]
> 您必須使用 [Azure AD 圖形 API](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-operations-overview?f=255&MSPPError=-2147217396) 管理 Azure AD B2C 目錄中的使用者。 這點不同於 Microsoft Graph API。 [在此](https://blogs.msdn.microsoft.com/aadgraphteam/2016/07/08/microsoft-graph-or-azure-ad-graph/)深入了解。

Azure Active Directory (Azure AD) B2C 租用戶通常會很龐大。 這表示許多常見的租用戶管理工作需要以程式設計方式執行。 使用者管理是主要範例。 您可能需要將現有的使用者存放區移轉到 B2C 租用戶。 您希望在自己的頁面上裝載使用者註冊，並在幕後的 Azure AD B2C 目錄中建立使用者帳戶。 這類工作需要能夠建立、讀取、更新和刪除使用者帳戶。 這些工作都可以透過 Azure AD 圖形 API 達成

對於 B2C 租用戶，與圖形 API 通訊有兩種主要模式。

* 對於互動式、執行一次的工作，您應以 B2C 租用戶中的系統管理員帳戶來執行管理工作。 此模式需要系統管理員先使用認證登入，系統管理員才能對圖形 API進行任何呼叫。
* 對於自動化、持續的工作，您應使用您提供必要權限的某種服務帳戶來執行管理工作。 在 Azure AD 中，作法上您可以註冊應用程式並向 Azure AD 驗證。 使用採用 **OAuth 2.0 用戶端認證授與** 的 [應用程式識別碼](../active-directory/develop/service-to-service.md)即可完成。 在此情況下，應用程式會以本身 (而非使用者的身分) 呼叫圖形 API。

在本文中，您會了解如何執行自動化使用案例。 您將建置 .NET 4.5 `B2CGraphClient` 來執行使用者建立、讀取、更新和刪除 (CRUD) 作業。 用戶端會有 Windows 命令列介面讓您叫用各種方法。 不過，程式碼會撰寫成以非互動、自動化的方式運作。

## <a name="get-an-azure-ad-b2c-tenant"></a>取得 Azure AD B2C 租用戶
您需要有 Azure AD B2C 租用戶，才能建立應用程式或使用者。 如果您還沒有租用戶，請參閱 [開始使用 Azure AD B2C](active-directory-b2c-get-started.md)。

## <a name="register-your-application-in-your-tenant"></a>在您的租用戶中註冊您的應用程式
擁有 B2C 租用戶之後，您需要使用 [Azure 入口網站](https://portal.azure.com)註冊您的應用程式。

> [!IMPORTANT]
> 若要搭配 B2C 租用戶使用圖形 API，您必須使用 Azure 入口網站中的 [應用程式註冊] 服務 (**不是** Azure AD B2C 的 [應用程式] 功能表) 來註冊應用程式。 下列指示會引導您前往適當的功能表。 您無法重複使用您在 Azure AD B2C 的 [應用程式] 功能表中註冊的現有 B2C 應用程式。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在頁面右上角選取您的帳戶，以選擇您的 Azure AD B2C 租用戶。
3. 在左側導覽窗格中，選擇 [所有服務]、按一下 [應用程式註冊]，然後按一下 [新增]。
4. 遵照提示進行，並建立新的應用程式。 
    1. 選取 [Web 應用程式/API] 作為 [應用程式類型]。    
    2. 提供**任何登入 URL** (例如 https://B2CGraphAPI)，因為與本範例無關。  
5. 應用程式會立即顯示在應用程式清單中，按一下它以取得 [應用程式識別碼] (也稱為用戶端識別碼)。 將它複製下來，稍後一節需要用到。
6. 在 [設定] 功能表中，按一下 [金鑰]。
7. 在 [密碼] 區段中，輸入金鑰描述並選取持續時間，然後按一下 [儲存]。 複製金鑰值 (也稱為用戶端祕密)，以便在後面的章節中使用。

## <a name="configure-create-read-and-update-permissions-for-your-application"></a>設定應用程式的建立、讀取和更新權限
現在，您需要設定應用程式，以取得建立、讀取、更新和刪除使用者的所有必要權限。

1. 在 Azure 入口網站的 [應用程式註冊] 功能表中繼續，選取您的應用程式。
2. 在 [設定] 功能表中，按一下 [必要權限]。
3. 在 [必要權限] 功能表中，按一下 [Windows Azure Active Directory]。
4. 在 [啟用存取] 功能表中，從 [應用程式權限]中選取 [讀取和寫入目錄資料] 權限，然後按一下 [儲存]。
5. 最後，回到 [必要權限] 功能表，按一下 [授與權限] 按鈕。

現在，您的應用程式具有從 B2C 租用戶建立、讀取和更新使用者的權限。

> [!NOTE]
> 授與權限可能需要幾分鐘才能完全處理完成。
> 
> 

## <a name="configure-delete-permissions-for-your-application"></a>設定應用程式的刪除權限
目前，「讀取和寫入目錄資料」權限**不**包含執行任何刪除作業的能力，例如刪除使用者。 如果希望應用程式有能力刪除使用者，您必須執行下列額外步驟 (需要使用 PowerShell)，否則可以跳到下一節。

如果尚未安裝，請安裝 [Azure AD PowerShell v1 模組 (MSOnline)](https://docs.microsoft.com/powershell/azure/active-directory/install-msonlinev1?view=azureadps-1.0)：

```powershell
Install-Module MSOnline
```

安裝 PowerShell 模組之後，請連線到 Azure AD B2C 租用戶。

> [!IMPORTANT]
> 您必須使用 B2C 租用戶**本機**的 B2C 租用戶系統管理員帳戶。 這些帳戶看起來像這樣︰myusername@myb2ctenant.onmicrosoft.com。

```powershell
Connect-MsolService
```

現在，我們將於下列指令碼中使用**應用程式識別碼**，將使用者帳戶管理員角色指派給應用程式，讓它能夠刪除使用者。 這些角色具有已知的識別項，您只需要在下列指令碼中輸入您的**應用程式識別碼**即可。

```powershell
$applicationId = "<YOUR_APPLICATION_ID>"
$sp = Get-MsolServicePrincipal -AppPrincipalId $applicationId
Add-MsolRoleMember -RoleObjectId fe930be7-5e62-47db-91af-98c3a49a38b1 -RoleMemberObjectId $sp.ObjectId -RoleMemberType servicePrincipal
```

您的應用程式現在也具有從 B2C 租用戶刪除使用者的權限。

## <a name="download-configure-and-build-the-sample-code"></a>下載、設定和建置範例程式碼
首先，下載範例程式碼並開始執行。 然後我們會仔細查看。  您可以 [將範例程式碼下載為 .zip 檔案](https://github.com/AzureADQuickStarts/B2C-GraphAPI-DotNet/archive/master.zip)。 您可以將此檔案複製到您選擇的目錄中：

```cmd
git clone https://github.com/AzureADQuickStarts/B2C-GraphAPI-DotNet.git
```

在 Visual Studio 中開啟 `B2CGraphClient\B2CGraphClient.sln` Visual Studio 方案。 在 `B2CGraphClient` 專案中，開啟 `App.config` 檔案。 使用您自己的值取代三個應用程式設定：

```xml
<appSettings>
    <add key="b2c:Tenant" value="{Your Tenant Name}" />
    <add key="b2c:ClientId" value="{The ApplicationID from above}" />
    <add key="b2c:ClientSecret" value="{The Key from above}" />
</appSettings>
```

[!INCLUDE [active-directory-b2c-devquickstarts-tenant-name](../../includes/active-directory-b2c-devquickstarts-tenant-name.md)]

接下來，以滑鼠右鍵按一下 `B2CGraphClient` 方案並重建範例。 如果成功，您現在應該有一個 `B2C.exe` 可執行檔位於 `B2CGraphClient\bin\Debug`。

## <a name="build-user-crud-operations-by-using-the-graph-api"></a>使用圖形 API 來建置使用者 CRUD 作業
若要使用 B2CGraphClient，請開啟 `cmd` Windows 命令提示字元，將您的目錄變更為 `Debug` 目錄。 然後執行 `B2C Help` 命令。

```cmd
cd B2CGraphClient\bin\Debug
B2C Help
```

這會顯示每個命令的簡短描述。 您每次叫用上述其中一個命令時， `B2CGraphClient` 會對 Azure AD 圖形 API 發出要求。

### <a name="get-an-access-token"></a>取得存取權杖
對圖形 API 發出任何要求時，需要有存取權杖進行驗證。 `B2CGraphClient` 會使用開放原始碼 Active Directory 驗證程式庫 (ADAL) 來協助取得存取權杖。 ADAL 提供簡單的 API 並處理一些重要的細節，例如快取存取權杖，可讓您輕鬆取得權杖。 不過，您不必使用 ADAL 來取得權杖。 您也可以藉由製作 HTTP 要求來取得權杖。

> [!NOTE]
> 此程式碼範例會使用 ADAL v2，以便與圖形 API 通訊。  您必須使用 ADAL v2 或 v3，才能取得可與 Azure AD 圖形 API 搭配使用的存取權杖。
> 
> 

`B2CGraphClient` 執行時會建立 `B2CGraphClient` 類別的執行個體。 這個類別的建構函式會設定 ADAL 驗證架構：

```csharp
public B2CGraphClient(string clientId, string clientSecret, string tenant)
{
    // The client_id, client_secret, and tenant are provided in Program.cs, which pulls the values from App.config
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.tenant = tenant;

    // The AuthenticationContext is ADAL's primary class, in which you indicate the tenant to use.
    this.authContext = new AuthenticationContext("https://login.microsoftonline.com/" + tenant);

    // The ClientCredential is where you pass in your client_id and client_secret, which are
    // provided to Azure AD in order to receive an access_token by using the app's identity.
    this.credential = new ClientCredential(clientId, clientSecret);
}
```

我們將以 `B2C Get-User` 命令為例。 叫用 `B2C Get-User` 而沒有任何其他輸入時，CLI 會呼叫 `B2CGraphClient.GetAllUsers(...)` 方法。 這個方法會呼叫 `B2CGraphClient.SendGraphGetRequest(...)`，後者會送出 HTTP GET 要求給圖形 API。 在 `B2CGraphClient.SendGraphGetRequest(...)` 傳送 GET 要求之前，它會先使用 ADAL 取得存取權杖：

```csharp
public async Task<string> SendGraphGetRequest(string api, string query)
{
    // First, use ADAL to acquire a token by using the app's identity (the credential)
    // The first parameter is the resource we want an access_token for; in this case, the Graph API.
    AuthenticationResult result = authContext.AcquireToken("https://graph.windows.net", credential);

    ...

```

您可以呼叫 ADAL `AuthenticationContext.AcquireToken(...)` 方法，以取得圖形 API 的存取權杖。 然後 ADAL 會傳回 `access_token` 表示應用程式的身分識別。

### <a name="read-users"></a>讀取使用者
當您想要從圖形 API 取得使用者清單或取得特定的使用者時，您可以傳送 HTTP `GET` 要求給 `/users` 端點。 要求取得租用戶中所有使用者時，情況如下：

```
GET https://graph.windows.net/contosob2c.onmicrosoft.com/users?api-version=1.6
Authorization: Bearer eyJhbGciOiJSUzI1NiIsIng1dCI6IjdkRC1nZWNOZ1gxWmY3R0xrT3ZwT0IyZGNWQSIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJod...
```

若要查看此要求，請執行：

 ```cmd
 B2C Get-User
 ```

有兩個重點值得注意：

* 透過 ADAL 取得的存取權杖已利用 `Bearer` 配置加入至 `Authorization` 標頭。
* 對於 B2C 租用戶，您必須使用查詢參數 `api-version=1.6`。

這兩個細節都在 `B2CGraphClient.SendGraphGetRequest(...)` 方法中處理：

```csharp
public async Task<string> SendGraphGetRequest(string api, string query)
{
    ...

    // For B2C user management, be sure to use the 1.6 Graph API version.
    HttpClient http = new HttpClient();
    string url = "https://graph.windows.net/" + tenant + api + "?" + "api-version=1.6";
    if (!string.IsNullOrEmpty(query))
    {
        url += "&" + query;
    }

    // Append the access token for the Graph API to the Authorization header of the request by using the Bearer scheme.
    HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Get, url);
    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);
    HttpResponseMessage response = await http.SendAsync(request);

    ...
```

### <a name="create-consumer-user-accounts"></a>建立取用者的使用者帳戶
在 B2C 租用戶中建立使用者帳戶時，您可以傳送 HTTP `POST` 要求給 `/users` 端點：

```
POST https://graph.windows.net/contosob2c.onmicrosoft.com/users?api-version=1.6
Authorization: Bearer eyJhbGciOiJSUzI1NiIsIng1dCI6IjdkRC1nZWNOZ1gxWmY3R0xrT3ZwT0IyZGNWQSIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJod...
Content-Type: application/json
Content-Length: 338

{
    // All of these properties are required to create consumer users.

    "accountEnabled": true,
    "signInNames": [                            // controls which identifier the user uses to sign in to the account
        {
            "type": "emailAddress",             // can be 'emailAddress' or 'userName'
            "value": "joeconsumer@gmail.com"
        }
    ],
    "creationType": "LocalAccount",            // always set to 'LocalAccount'
    "displayName": "Joe Consumer",                // a value that can be used for displaying to the end user
    "mailNickname": "joec",                        // an email alias for the user
    "passwordProfile": {
        "password": "P@ssword!",
        "forceChangePasswordNextLogin": false   // always set to false
    },
    "passwordPolicies": "DisablePasswordExpiration"
}
```

此要求中的這些屬性，在建立取用者使用者時，大部分都是必要的屬性。 若要深入了解，請按一下[這裡](https://msdn.microsoft.com/library/azure/ad/graph/api/users-operations#CreateLocalAccountUser)。 請注意，加上 `//` 註解是為了講解， 請不要放入實際的要求中。

若要查看此要求，請執行下列其中一個命令：

```cmd
B2C Create-User ..\..\..\usertemplate-email.json
B2C Create-User ..\..\..\usertemplate-username.json
```

`Create-User` 命令會以 .json 檔案做為輸入參數。 這包含使用者物件的 JSON 表示法。 範例程式碼中有兩個範例 .json 檔案：`usertemplate-email.json` 和 `usertemplate-username.json`。 您可以修改這些檔案以符合您的需求。 除了上述必要欄位以外，這些檔案包含一些您可以使用的選擇性欄位。 [Azure AD 圖形 API 實體參考](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#user-entity)提供選擇性欄位的詳細資訊。

您可以在 `B2CGraphClient.SendGraphPostRequest(...)`中看到如何建構 POST 要求。

* 它會將存取權杖附加至要求的 `Authorization` 標頭。
* 它會設定 `api-version=1.6`。
* 它會將 JSON 使用者物件加入要求主體中。

> [!NOTE]
> 如果您想要從現有使用者存放區移轉的帳戶密碼強度比[由 Azure AD B2C 強制執行的強式密碼強度](https://msdn.microsoft.com/library/azure/jj943764.aspx)還低，您可以使用 `passwordPolicies` 屬性中的 `DisableStrongPassword` 值來停用強式密碼需求。 例如，您可以修改上述提供的建立使用者要求，如下所示︰ `"passwordPolicies": "DisablePasswordExpiration, DisableStrongPassword"`。
> 
> 

### <a name="update-consumer-user-accounts"></a>更新取用者的使用者帳戶
當您更新使用者物件時，此程序類似於您用來建立使用者物件的程序。 但此程序會使用 HTTP `PATCH` 方法：

```
PATCH https://graph.windows.net/contosob2c.onmicrosoft.com/users/<user-object-id>?api-version=1.6
Authorization: Bearer eyJhbGciOiJSUzI1NiIsIng1dCI6IjdkRC1nZWNOZ1gxWmY3R0xrT3ZwT0IyZGNWQSIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJod...
Content-Type: application/json
Content-Length: 37

{
    "displayName": "Joe Consumer",                // this request updates only the user's displayName
}
```

嘗試以新資料更新 JSON 檔案，進而更新使用者。 然後您可以使用 `B2CGraphClient` 來執行下列其中一個命令：

```cmd
B2C Update-User <user-object-id> ..\..\..\usertemplate-email.json
B2C Update-User <user-object-id> ..\..\..\usertemplate-username.json
```

如需有關如何傳送此要求的詳細資訊，請檢閱 `B2CGraphClient.SendGraphPatchRequest(...)` 方法。

### <a name="search-users"></a>搜尋使用者
您可以透過數種方式在 B2C 租用戶中搜尋使用者。 一個是利用使用者的物件識別碼，另一個則是使用者的登入識別碼 (亦即 `signInNames` 屬性)。

執行下列其中一個命令來搜尋特定的使用者︰

```cmd
B2C Get-User <user-object-id>
B2C Get-User <filter-query-expression>
```

以下是一些範例︰

```cmd
B2C Get-User 2bcf1067-90b6-4253-9991-7f16449c2d91
B2C Get-User $filter=signInNames/any(x:x/value%20eq%20%27joeconsumer@gmail.com%27)
```

### <a name="delete-users"></a>刪除使用者
刪除使用者的程序很簡單。 使用 HTTP `DELETE` 方法並以正確的物件識別碼建構 URL：

```
DELETE https://graph.windows.net/contosob2c.onmicrosoft.com/users/<user-object-id>?api-version=1.6
Authorization: Bearer eyJhbGciOiJSUzI1NiIsIng1dCI6IjdkRC1nZWNOZ1gxWmY3R0xrT3ZwT0IyZGNWQSIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJod...
```

若要查看範例，請輸入此命令並檢視主控台印出的 Delete 要求：

```cmd
B2C Delete-User <object-id-of-user>
```

如需有關如何傳送此要求的詳細資訊，請檢閱 `B2CGraphClient.SendGraphDeleteRequest(...)` 方法。

除了使用者管理，您還可以使用 Azure AD 圖形 API 執行其他許多動作。 [Azure AD 圖形 API 參考](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog) 會提供每個動作的詳細資訊以及範例要求。

## <a name="use-custom-attributes"></a>使用自訂屬性
大部分消費者應用程式都需要儲存某種自訂使用者設定檔資訊。 作法之一是在 B2C 租用戶中定義自訂屬性。 然後，您可將該屬性視同使用者物件的任何其他屬性一樣處理。 該屬性就像登入權杖中的宣告，您可以更新、刪除、查詢、傳送該屬性。

若要在 B2C 租用戶中定義自訂屬性，請參閱 [B2C 自訂屬性參考](active-directory-b2c-reference-custom-attr.md)。

您可以使用 `B2CGraphClient`來檢視 B2C 租用戶中定義的自訂屬性：

```cmd
B2C Get-B2C-Application
B2C Get-Extension-Attribute <object-id-in-the-output-of-the-above-command>
```

這些函式的輸出會顯示每個自訂屬性的詳細資料，例如：

```json
{
      "odata.type": "Microsoft.DirectoryServices.ExtensionProperty",
      "objectType": "ExtensionProperty",
      "objectId": "cec6391b-204d-42fe-8f7c-89c2b1964fca",
      "deletionTimestamp": null,
      "appDisplayName": "",
      "name": "extension_55dc0861f9a44eb999e0a8a872204adb_Jersey_Number",
      "dataType": "Integer",
      "isSyncedFromOnPremises": false,
      "targetObjects": [
        "User"
      ]
}
```

您可以使用完整名稱做為使用者物件的屬性，例如 `extension_55dc0861f9a44eb999e0a8a872204adb_Jersey_Number`。  以新的屬性和該屬性的值更新 .json 檔案，然後執行：

```cmd
B2C Update-User <object-id-of-user> <path-to-json-file>
```

使用 `B2CGraphClient`，您會有一個服務應用程式可利用程式設計方式來管理 B2C 租用戶使用者。 `B2CGraphClient` 會使用自己的應用程式身分識別，向 Azure AD 圖形 API 進行驗證。 它也會使用用戶端密碼來取得權杖。 將這項功能納入您的應用程式時，請記住 B2C 應用程式的幾個重點：

* 您需要將租用戶中的適當權限授與應用程式。
* 現在，您需要使用 ADAL (不是 MSAL) 取得存取權杖。 (也可以直接傳送通訊協定訊息，而不使用程式庫)。
* 當您呼叫圖形 API 時，請使用 `api-version=1.6`。
* 當建立和更新取用者使用者，有幾個必要的屬性，如上所述。

對於您想要使用圖形 API 在 B2C 租用戶上執行的動作，如有任何問題或要求，請在本文上留言，或在 GitHub 程式碼範例儲存機制中提出問題。

