---
title: 建置 AngularJS 單一頁面應用程式 與 Azure Active Directory 整合進行登入和登出 |Microsoft Docs
description: 了解如何建置 AngularJS 單一頁面應用程式來與 Azure AD 整合進行登入，並使用 OAuth 呼叫受 Azure AD 保護的 API。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: f2991054-8146-4718-a5f7-59b892230ad7
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: javascript
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 23912f9d004d051c422f93e8b10f1aa6cb8b2626
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46959490"
---
# <a name="quickstart-build-an-angularjs-single-page-app-for-sign-in-and-sign-out-with-azure-active-directory"></a>快速入門：建置 AngularJS 單一頁面應用程式 與 Azure Active Directory 整合進行登入和登出

[!INCLUDE [active-directory-develop-applies-v1-adal](../../../includes/active-directory-develop-applies-v1-adal.md)]

Azure Active Directory (Azure AD) 可讓您簡單又直截了當地新增登入、登出，並保護對單一頁面應用程式的 OAuth API 呼叫。 它也可讓您的應用程式以使用者的 Windows Server Active Directory 帳戶來驗證使用者，並取用 Azure AD 保護的任何 Web API，例如 Office 365 API 或 Azure API。

對於在瀏覽器中執行的 JavaScript 應用程式，Azure AD 提供 Active Directory 驗證程式庫 (ADAL)，又稱為 adal.js。 adal.js 的唯一目的是為了讓您的應用程式輕鬆取得存取權杖。

在本快速入門中，您將了解如何建置 AngularJS 待辦事項清單應用程式：

* 使用 Azure AD 做為身分識別提供者，將使用者登入應用程式。
* 顯示使用者的一些相關資訊。
* 使用 Azure AD 簽發的持有人權杖，安全地呼叫應用程式的待辦事項清單 API。
* 將使用者登出應用程式。

若要建置完整且可運作的應用程式，您必須：

1. 向 Azure AD 註冊應用程式.
2. 安裝 ADAL 並設定單一頁面應用程式。
3. 使用 ADAL 來保護單一頁面應用程式中的頁面。

## <a name="prerequisites"></a>必要條件

若要開始，請完成以下必要條件：

* [下載應用程式基本架構](https://github.com/AzureADQuickStarts/SinglePageApp-AngularJS-DotNet/archive/skeleton.zip)或[下載完整的範例](https://github.com/AzureADQuickStarts/SinglePageApp-AngularJS-DotNet/archive/complete.zip)。
* 您需要一個可以建立使用者並註冊應用程式的 Azure AD 租用戶。 如果您還沒有租用戶， [了解如何取得租用戶](quickstart-create-new-tenant.md)。

## <a name="step-1-register-the-directorysearcher-application"></a>步驟 1：註冊 DirectorySearcher 應用程式

若要讓應用程式能夠驗證使用者並取得權杖，您必須先在 Azure AD 租用戶中註冊這個應用程式：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 如果登入多個目錄，您可能需要確定檢視正確的目錄。 若要這樣做，請在頂端列上，按一下您的帳戶。 在 [目錄] 清單下，選擇您要註冊應用程式的 Azure AD 租用戶。
1. 按一下左窗格中的 [所有服務]，然後選取 [Azure Active Directory]。
1. 按一下 [應用程式註冊]，然後選取 [新增]。
1. 遵照提示進行，並建立新的 Web 應用程式和/或 Web API：

    * **名稱**向使用者描述您的應用程式。
    * **登入 URL** 是供 Azure AD 傳回權杖的位置。 此範例中的預設位置是 `https://localhost:44326/`。

1. 完成註冊之後，Azure AD 會為應用程式指派一個唯一的應用程式識別碼。 您會在後續小節中用到這個值，所以請從應用程式索引標籤中複製此值。
1. Adal.js 會使用 OAuth 隱含流程來與 Azure AD 通訊。 您必須為您的應用程式啟用隱含流程：

    1. 按一下應用程式，選取 [資訊清單] 以開啟內嵌資訊清單編輯器。
    1. 找出 `oauth2AllowImplicitFlow` 屬性。 將值設為 `true`。
    1. 按一下 [儲存]  以儲存資訊清單。

1. 在您的應用程式租用戶上授予權限。 前往 [設定] > [必要的權限]，並選取頂端列中的 [授與權限] 按鈕。
1. 選取 [是]  加以確認。

## <a name="step-2-install-adal-and-configure-the-single-page-app"></a>步驟 2︰安裝 ADAL 並設定單一頁面應用程式

既然您在 Azure AD 中已有應用程式，您可以安裝 adal.js，並撰寫身分識別相關的程式碼。

### <a name="configure-the-javascript-client"></a>設定 JavaScript 用戶端

首先，使用 [套件管理主控台] 將 adal.js 新增至 TodoSPA 專案：

1. 下載 [adal.js](https://raw.githubusercontent.com/AzureAD/azure-activedirectory-library-for-js/master/lib/adal.js) 並將它新增至 `App/Scripts/` 專案目錄。
2. 下載 [adal-angular.js](https://raw.githubusercontent.com/AzureAD/azure-activedirectory-library-for-js/master/lib/adal-angular.js) 並將它新增至 `App/Scripts/` 專案目錄。
3. 在 `index.html` 中的 `</body>` 結尾之前，載入每個指令碼：

    ```js
    ...
    <script src="App/Scripts/adal.js"></script>
    <script src="App/Scripts/adal-angular.js"></script>
    ...
    ```

### <a name="configure-the-back-end-server"></a>設定後端伺服器

為了讓單一頁面應用程式的後端待辦事項清單 API 從瀏覽器接受權杖，後端需要應用程式註冊的組態資訊。 在 TodoSPA 專案中，開啟 `web.config`。 取代 `<appSettings>` 區段中的元素值，以反映您在 Azure 入口網站中所使用的值。 每當使用 ADAL 時，您的程式碼便會參考這些值。

   * `ida:Tenant` 是指您的 Azure AD 租用戶網域，例如 contoso.onmicrosoft.com。
   * `ida:Audience` 是您從入口網站複製的應用程式用戶端識別碼。

## <a name="step-3-use-adal-to-help-secure-pages-in-the-single-page-app"></a>步驟 3︰使用 ADAL 來保護單一頁面應用程式中的頁面

adal.js 與 AngularJS 路由和 HTTP 提供者整合，因此您可以保護單一頁面應用程式中的個別檢視。

1. 在 `App/Scripts/app.js` 中，載入 adal.js 模組：

    ```js
    angular.module('todoApp', ['ngRoute','AdalAngular'])
    .config(['$routeProvider','$httpProvider', 'adalAuthenticationServiceProvider',
     function ($routeProvider, $httpProvider, adalProvider) {
    ...
    ```
2. 使用應用程式註冊的組態值將 `adalProvider` 初始化 (也在 `App/Scripts/app.js` 中)：

    ```js
    adalProvider.init(
      {
          instance: 'https://login.microsoftonline.com/',
          tenant: 'Enter your tenant name here e.g. contoso.onmicrosoft.com',
          clientId: 'Enter your client ID here e.g. e9a5a8b6-8af7-4719-9821-0deef255f68e',
          extraQueryParameter: 'nux=1',
          //cacheLocation: 'localStorage', // enable this for IE, as sessionStorage does not work for localhost.
      },
      $httpProvider
    );
    ```
3. 只需要一行程式碼 - `requireADLogin`，就能保護應用程式中的 `TodoList` 檢視。

    ```js
    ...
    }).when("/TodoList", {
            controller: "todoListCtrl",
            templateUrl: "/App/Views/TodoList.html",
            requireADLogin: true,
    ...
    ```

## <a name="summary"></a>總結

您現在有了一個安全的單一頁面應用程式，能夠將使用者登入，並將以持有人權杖保護的要求發給其後端 API。 當使用者按一下 **TodoList** 連結時，需要的話，adal.js 會自動重新導向至 Azure AD 進行登入。 此外，adal.js 將會自動將 access_token 附加至任何傳送至應用程式後端的 Ajax 要求。

前述步驟是使用 adal.js 建置單一頁面應用程式的最低必要程序。 但單一頁面應用程式中還有其他幾項很有用的功能︰

* 若要明確發出登入和登出要求，您可以在控制器中定義函式來叫用 adal.js。 在 `App/Scripts/homeCtrl.js`中：

    ```js
    ...
    $scope.login = function () {
        adalService.login();
    };
    $scope.logout = function () {
        adalService.logOut();
    };
    ...
    ```
* 您可以在應用程式的 UI 中顯示使用者資訊。 ADAL 服務已經加入 `userDataCtrl` 控制器，您現在可以在相關聯檢視 `App/Views/UserData.html` 中存取 `userInfo` 物件：

    ```js
    <p>{{userInfo.userName}}</p>
    <p>aud:{{userInfo.profile.aud}}</p>
    <p>iss:{{userInfo.profile.iss}}</p>
    ...
    ```

* 在許多情況下，您也可能想知道使用者是否已登入。 您也可以使用 `userInfo` 物件來收集此資訊。 例如，在 `index.html` 中，您可以根據驗證狀態顯示 [登入] 或 [登出] 按鈕：

    ```js
    <li><a class="btn btn-link" ng-show="userInfo.isAuthenticated" ng-click="logout()">Logout</a></li>
    <li><a class="btn btn-link" ng-hide=" userInfo.isAuthenticated" ng-click="login()">Login</a></li>
    ```

您的已整合 Azure AD 的單一頁面應用程式可以驗證使用者、使用 OAuth 2.0 安全地呼叫其後端，以及取得使用者的基本資訊。 如果您還沒有這麼做，現在是將一些使用者植入租用戶的時候。 執行待辦事項清單單一頁面應用程式，並使用其中一個使用者登入。 將工作新增至使用者的待辦事項清單、登出，再重新登入。

adal.js 可讓您輕鬆地將常見的身分識別功能納入您的應用程式。 它會為您處理一切麻煩的事：快取管理、OAuth 通訊協定支援、向使用者顯示登入 UI、重新整理過期權杖等等。

如需參考資料，請參閱 [GitHub](https://github.com/AzureADQuickStarts/SinglePageApp-AngularJS-DotNet/archive/complete.zip) 中的完整範例 (不含您的組態值)。

## <a name="next-steps"></a>後續步驟

您現在可以繼續探索其他案例。

> [!div class="nextstepaction"]
> [從單一頁面應用程式呼叫 CORS Web API](https://github.com/AzureAdSamples/SinglePageApp-WebAPI-AngularJS-DotNet)。