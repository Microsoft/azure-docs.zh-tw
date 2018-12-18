---
title: Azure AD v2 JavaScript 快速入門 | Microsoft Docs
description: 深入了解 JavaScript 應用程式如何呼叫需要來自 Azure Active Directory v2.0 端點存取權杖的 API
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: 1b884571707aab71e8a8d124ba68f938e5a63a43
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063739"
---
# <a name="quickstart-sign-in-users-and-acquire-an-access-token-from-a-javascript-application"></a>快速入門：登入使用者及從 JavaScript 應用程式取得存取權杖

[!INCLUDE [active-directory-develop-applies-v2-msal](../../../includes/active-directory-develop-applies-v2-msal.md)]

在本快速入門中，您會了解如何使用程式碼範例，該範例示範 JavaScript 單頁應用程式 (SPA) 如何登入個人帳戶、公司和學校帳戶、取得存取權杖以呼叫 Microsoft Graph API 或任何 Web API。

![本快速入門所產生之範例應用程式的運作方式](media/quickstart-v2-javascript/javascriptspa-intro.png)

> [!div renderon="docs"]
> ## <a name="register-your-application-and-download-your-quickstart-app"></a>註冊您的應用程式，並下載您的快速入門應用程式
>
> ### <a name="step-1-register-your-application"></a>步驟 1：註冊您的應用程式
>
> 1. 前往 [Microsoft 應用程式註冊入口網站](https://apps.dev.microsoft.com/portal/register-app)註冊應用程式。
> 1. 在 [應用程式名稱] 方塊中，輸入應用程式的名稱。
> 1. 確認已清除 [引導式設定] 核取方塊，然後選取 [建立]。
> 1. 按一下 [新增平台]，然後選取 [Web]。
> 1. 確定 [允許隱含流程] 保持勾選。
> 1. 在 [重新導向 URL] 底下，新增 `http://localhost:30662/`。
> 1. 按一下 [檔案] 。

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-azure-portal"></a>步驟 1：在 Azure 入口網站中設定您的應用程式
> 若要讓此快速入門中的程式碼範例正常運作，您需要將重新導向 URL 新增為 `http://localhost:30662/`，並且啟用 [隱含授與]。
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [為我進行這些變更]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![已設定](media/quickstart-v2-javascript/green-check.png) 您的應用程式已設定了這些屬性

#### <a name="step-2-download-the-project"></a>步驟 2：下載專案

您可以選擇適合您開發環境的其中一個選項。
* [下載核心專案檔案 - 適用於網頁伺服器，例如 Node.js](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip)
* [下載 Visual Studio 專案](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/vsquickstart.zip)

將 zip 檔案解壓縮至本機資料夾 (例如 **C:\Azure-Samples**)。

#### <a name="step-3-configure-your-javascript-app"></a>步驟 3：設定您的 JavaScript 應用程式

> [!div renderon="docs"]
> 編輯 `index.html`，並且使用您剛註冊應用程式的應用程式識別碼取代 `applicationConfig` 之下的 `Enter_the_Application_Id_here`。

> [!div class="sxs-lookup" renderon="portal"]
> 編輯 `index.html` 並將 `applicationConfig` 取代為：

```javascript
var applicationConfig = {
    clientID: "Enter_the_Application_Id_here",
    graphScopes: ["user.read"],
    graphEndpoint: "https://graph.microsoft.com/v1.0/me"
};
```
> [!NOTE]
>如果您使用 [Node.js](https://nodejs.org/en/download/)，則會針對伺服器設定 *server.js* 檔案以開始在連接埠 30662 上接聽。
> 如果您使用 [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/)，則會針對伺服器設定程式碼範例的 *.csproj* 檔案以開始在連接埠 30662 上接聽。
>

#### <a name="step-4-run-the-project"></a>步驟 4：執行專案

如果使用 Node.js，您可以在命令列上從專案目錄執行下列命令，以啟動伺服器：
 ```batch
 npm install
 node server.js
 ```
開啟 web 瀏覽器並巡覽至 `http://localhost:30662/`。 按一下 [登入] 按鈕開始登入，然後呼叫 Microsoft Graph API。

如果您使用 Visual Studio，請務必選取專案解決方案，然後按 **F5** 來執行專案。

## <a name="more-information"></a>相關資訊

### <a name="msaljs"></a>*msal.js*

MSAL 是程式庫，用來登入使用者並要求權杖，該權杖是用來存取受 Microsoft Azure Active Directory (Azure AD) 保護的 API。 快速入門的 index.html 包含程式庫的參考：

```html
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/0.2.3/js/msal.min.js"></script>
```

或者，如果您已安裝節點，可以透過 npm 下載它：

```batch
npm install msal
```

### <a name="msal-initialization"></a>MSAL 初始化

快速入門程式碼也會示範如何初始化程式庫：

```javascript
var myMSALObj = new Msal.UserAgentApplication(applicationConfig.clientID, null, acquireTokenRedirectCallBack, {storeAuthStateInCookie: true, cacheLocation: "localStorage"});
```

> |Where  |  |
> |---------|---------|
> |`ClientId`     |來自註冊於 Azure 入口網站中應用程式的應用程式識別碼|
> |`authority`    |它是授權單位 URL。 傳遞 Null 會將預設授權單位設為 `https://login.microsoftonline.com/common`。 如果您的應用程式是單一租用戶 (目標僅限於單一目錄中的帳戶)，請將此值設為 `https://login.microsoftonline.com/<tenant name or ID>`|
> |`tokenReceivedCallback`| 在驗證重新導向回到應用程式之後所呼叫的回呼方法。 此處已傳遞 `acquireTokenRedirectCallBack`。 如果使用 loginPopup，這是 null 值。|
> |`options`  |選擇性參數的集合。 在此情況下，`storeAuthStateInCookie` 和 `cacheLocation` 都是選擇性組態。 如需選項的詳細資訊，請參閱 [wiki](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/MSAL-basics#configuration-options)。 |

### <a name="sign-in-users"></a>登入使用者

下列程式碼片段顯示如何登入使用者：

```javascript
myMSALObj.loginPopup(applicationConfig.graphScopes).then(function (idToken) {
    //Callback code here
})
```

> |Where  |  |
> |---------|---------|
> | `scopes`   | (選擇性) 包含在登入時針對使用者同意所要求的範圍 (例如：適用於 Microsoft Graph 的 `[ "user.read" ]`，或適用於自訂 Web API 的 `[ "<Application ID URL>/scope" ]`，也就是 `api://<Application ID>/access_as_user`)。 此處已傳遞 `applicationConfig.graphScopes`。 |

> [!TIP]
> 或者，您可以使用 `loginRedirect` 方法，將目前頁面重新導向至登入頁面 (而非快顯視窗)。


### <a name="request-tokens"></a>要求權杖

MSAL 有三種取得權杖的方法，`acquireTokenRedirect`、`acquireTokenPopup` 和 `acquireTokenSilent`：

#### <a name="get-a-user-token-silently"></a>以無訊息方式取得使用者權杖

`acquireTokenSilent` 方法會處理權杖取得和更新作業，不需要與使用者進行任何互動。 在第一次執行 `loginRedirect` 或 `loginPopup` 方法之後，`acquireTokenSilent` 就會成為用來取得權杖的常用方法，以在後續呼叫中使用那些權杖存取受保護的資源。 以無訊息方式進行要求或更新權杖的呼叫。

```javascript
myMSALObj.acquireTokenSilent(applicationConfig.graphScopes).then(function (accessToken) {
    // Callback code here
})
```

> |Where  |  |
> |---------|---------|
> | `scopes`   | 包含要在 API 存取權杖中傳回的所要求範圍 (例如：適用於 Microsoft Graph 的 `[ "user.read" ]`，或適用於自訂 Web API的 `[ "<Application ID URL>/scope" ]`，也就是 `api://<Application ID>/access_as_user`)。 此處已傳遞 `applicationConfig.graphScopes`。|

#### <a name="get-a-user-token-interactively"></a>以互動方式取得使用者權杖

 您有可能必須強制使用者與 Azure AD v2.0 端點互動。 例如︰
* 使用者可能需要重新輸入其認證，因為密碼已過期
* 您的應用程式要求其他資源範圍的存取權，需要使用者同意
* 需要雙因素驗證

大部分應用程式的一般建議模式是先呼叫 `acquireTokenSilent`、捕捉例外狀況，然後呼叫 `acquireTokenRedirect` (或 `acquireTokenPopup`) 以啟動互動式要求。

呼叫 `acquireTokenPopup(scope)` 會導致要登入的快顯視窗 (或者呼叫 `acquireTokenRedirect(scope)` 會導致將使用者重新導向至 Azure AD v2.0 端點)，使用者必須藉由確認其認證、同意必要的資源，或完成雙因素驗證來進行互動。

```javascript
myMSALObj.acquireTokenPopup(applicationConfig.graphScopes).then(function (accessToken) {
    // Callback code here
})
```

> [!NOTE]
> 如果因為 Internet Explorer 瀏覽器處理快顯視窗相關[已知問題](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues)，而使用 Internet Explorer 瀏覽器，則本快速入門會使用 `loginRedirect` 和 `acquireTokenRedirect` 方法。

## <a name="next-steps"></a>後續步驟

如需如何建置本快速入門應用程式的詳細逐步指南，請嘗試以下的 JavaScript 教學課程。

### <a name="learn-the-steps-to-create-the-application-for-this-quickstart"></a>了解建立本快速入門應用程式的步驟

> [!div class="nextstepaction"]
> [呼叫 Graph API 教學課程](https://docs.microsoft.com/azure/active-directory/develop/guidedsetups/active-directory-javascriptspa)

### <a name="browse-the-msal-repo-for-documentation-faq-issues-and-more"></a>瀏覽 MSAL 存放庫的文件、常見問題集問題等等

> [!div class="nextstepaction"]
> [msal.js GitHub 存放庫](https://github.com/AzureAD/microsoft-authentication-library-for-js)


[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]
