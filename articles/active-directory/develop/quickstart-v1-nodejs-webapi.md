---
title: 使用 Azure AD 保護 Web API | Microsoft Docs
description: 了解如何建置能與 Azure AD 整合以進行驗證的 Node.js REST Web API。
services: active-directory
documentationcenter: nodejs
author: CelesteDG
manager: mtillman
ms.assetid: 7654ab4c-4489-4ea5-aba9-d7cdc256e42a
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: javascript
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: celested
ms.custom: aaddev
ms.openlocfilehash: f6f804ea9121d1728e31f1e694280e841f4b7f4e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946539"
---
# <a name="quickstart-secure-a-web-api-with-azure-active-directory"></a>快速入門：使用 Azure Active Directory 保護 Node.js Web API

[!INCLUDE [active-directory-develop-applies-v1](../../../includes/active-directory-develop-applies-v1.md)]

在本快速入門中，您將了解如何使用 [passport-azure-ad](https://github.com/AzureAD/passport-azure-ad) \(英文\) 模組處理與 Azure Active Directory (Azure AD) 的通訊，進而透過 [Passport](http://passportjs.org/) \(英文\) 保護 [Restify](http://restify.com/) \(英文\) API 端點。

本快速入門的範圍涵蓋保護 API 端點的相關考量。 登入和保留驗證權杖的考量未在此實作，那是由用戶端應用程式負責管理。 如需用戶端實作的相關詳細資訊，請檢閱[搭配 Azure AD 的 Node.js Web 應用程式登入和登出](quickstart-v1-openid-connect-code.md)。

與此文章相關聯的完整程式碼範例可從 [GitHub](https://github.com/Azure-Samples/active-directory-node-webapi-basic) 取得。

## <a name="prerequisites"></a>必要條件

若要開始，請完成以下先決條件。

### <a name="create-the-sample-project"></a>建立範例專案

伺服器應用程式需要一些套件相依性來支援 Restify 和 Passport，以及傳遞至 Azure AD 的帳戶資訊。

若要開始，請將下列程式碼新增到名為 `package.json` 的檔案中：

```Shell
{
  "name": "node-aad-demo",
  "version": "0.0.1",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "passport": "0.4.0",
    "passport-azure-ad": "3.0.8",
    "restify": "6.0.1",
    "restify-plugins": "1.6.0"
  }
}
```

建立 `package.json` 後，在命令提示字元中執行 `npm install`，以安裝套件相依性。 

#### <a name="configure-the-project-to-use-active-directory"></a>設定專案以使用 Active Directory

若要開始設定應用程式，有幾個帳戶專屬值可從 Azure CLI 取得。 開始使用 CLI 最簡單方式是使用 Azure Cloud Shell。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

在 Cloud Shell 中輸入下列命令︰

```azurecli-interactive
az ad app create --display-name node-aad-demo --homepage http://localhost --identifier-uris http://node-aad-demo
```

用於 `create` 命令的[引數](/cli/azure/ad/app?view=azure-cli-latest#az-ad-app-create)包括：

| 引數  | 說明 |
|---------|---------|
|`display-name` | 註冊的易記名稱 |
|`homepage` | 使用者可登入並使用您應用程式的 URL |
|`identifier-uris` | 可讓 Azure AD 用在此應用程式的唯一 URI (空格分隔清單) |

連線到 Azure Active Directory 之前，您需要下列資訊：

| 名稱  | 說明 | 組態檔中的變數名稱 |
| ------------- | ------------- | ------------- |
| 租用戶名稱  | 您想要用來進行驗證的[租用戶名稱](quickstart-create-new-tenant.md) | `tenantName`  |
| 用戶端識別碼  | 用戶端識別碼是用於 AAD _應用程式識別碼_ 的 OAuth 字詞。 |  `clientID`  |

從 Azure Cloud Shell 中的註冊回應複製 `appId` 值，並建立新的檔案，命名為 `config.js`。 接下來，新增下列程式碼，並使用括弧括住的權杖取代您的值：

```JavaScript
const tenantName    = //<YOUR_TENANT_NAME>;
const clientID      = //<YOUR_APP_ID_FROM_CLOUD_SHELL>;
const serverPort    = 3000;

module.exports.serverPort = serverPort;

module.exports.credentials = {
  identityMetadata: `https://login.microsoftonline.com/${tenantName}.onmicrosoft.com/.well-known/openid-configuration`, 
  clientID: clientID
};
```

如需有關個別組態設定的詳細資訊，請檢閱 [passport-azure-ad](https://github.com/AzureAD/passport-azure-ad#5-usage) 模組文件。

### <a name="implement-the-server"></a>實作伺服器

[Passport azure ad](https://github.com/AzureAD/passport-azure-ad#5-usage) 模組功能有兩種驗證策略：[OIDC](https://github.com/AzureAD/passport-azure-ad#51-oidcstrategy) 和[持有人權杖](https://github.com/AzureAD/passport-azure-ad#52-bearerstrategy) 策略。 本文中實作的伺服器使用「持有人權杖」策略來保護 API 端點。

### <a name="step-1-import-dependencies"></a>步驟 1：匯入相依性

建立名為 `app.js` 的新檔案，並貼上下列文字︰

```JavaScript
const
      restify = require('restify')
    , restifyPlugins = require('restify-plugins')
    , passport = require('passport')
    , BearerStrategy = require('passport-azure-ad').BearerStrategy
    , config = require('./config')
    , authenticatedUserTokens = []
    , serverPort = process.env.PORT || config.serverPort
;
```

在這段程式碼中：

- `restify` 和 `restify-plugins` 是為了設定 Restify 伺服器所參考的模組。
- `passport` 和 `passport-azure-ad` 模組負責與 Azure AD 通訊。
- 檔案 `config.js` (於先前步驟中建立) 中的值會用來初始化 `config` 變數。
- 當使用者權杖傳遞至安全的端點時，會建立 `authenticatedUserTokens` 的陣列來儲存使用者權杖。
- `serverPort` 會從流程環境的連接埠或從組態檔進行定義。

### <a name="step-2-instantiate-an-authentication-strategy"></a>步驟 2：具現化驗證策略

當您要保護端點時，您必須提供策略來判斷目前要求是否來自已驗證的使用者。 這裡的 `authenticatonStrategy` 變數是 `passport-azure-ad` `BearerStrategy` 類別的執行個體。 在 `require` 陳述式之後新增下列程式碼。

```JavaScript
const authenticationStrategy = new BearerStrategy(config.credentials, (token, done) => {
    let currentUser = null;

    let userToken = authenticatedUserTokens.find((user) => {
        currentUser = user;
        user.sub === token.sub;
    });

    if(!userToken) {
        authenticatedUserTokens.push(token);
    }

    return done(null, currentUser, token);
});
```

此實作會藉由將驗證權杖新增至 `authenticatedUserTokens` 陣列 (如果驗證權杖不存在的話) 來使用自動註冊。

建立策略的新執行個體後，您必須透過 `use` 方法將此執行個體傳遞至 Passport。 將下列程式碼新增至 `app.js`，以使用 Passport 中的策略。

```JavaScript
passport.use(authenticationStrategy);
```

### <a name="step-3-server-configuration"></a>步驟 3：伺服器組態

定義驗證策略後，您現在可以設定具備一些基本設定的 Restify 伺服器，並針對安全性設定為使用 Passport。

```JavaScript
const server = restify.createServer({ name: 'Azure Active Directroy with Node.js Demo' });
server.use(restifyPlugins.authorizationParser());
server.use(passport.initialize());
server.use(passport.session());
```
此伺服器已初始化並設定為剖析授權標頭，然後設定為使用 Passport。

### <a name="step-4-define-routes"></a>步驟 4：定義路由

您現在可以定義路由，並決定要使用 Azure AD 保護哪一個路由。 此專案包含兩個路由，其中根層級是開放的，且 `/api` 路由設定為需要驗證。

在 `app.js` 新增下列程式碼以進行根層級路由：

```JavaScript
server.get('/', (req, res, next) => {
    res.send(200, 'Try: curl -isS -X GET http://127.0.0.1:3000/api');
    next();
});
```

根路由可讓所有要求通過路由並傳回訊息，訊息中包含要測試 `/api` 路由的命令。 相反地，`/api` 路由會鎖定使用 [ `passport.authenticate` ](http://passportjs.org/docs/authenticate)。 在根路由之後新增下列程式碼。

```JavaScript
server.get('/api', passport.authenticate('oauth-bearer', { session: false }), (req, res, next) => {
    res.json({ message: 'response from API endpoint' });
    return next();
});
```

此設定只允許內含 `/api` 持有人權杖存取權的已驗證要求。 `session: false` 選項是用來停用要求權杖隨著每個要求傳遞至 API 的工作階段。

最後，藉由呼叫 `listen` 方法，伺服器會設定為在設定的連接埠上接聽。

```JavaScript
server.listen(serverPort);
```

## <a name="run-the-sample"></a>執行範例

伺服器現已實作，您可以藉由開啟命令提示字元並輸入下列字串來啟動伺服器：

```shell
npm start
```

伺服器執行時，您可以提交要求到伺服器以測試結果。 若要示範來自根路由的回應，請開啟 Bash 殼層並輸入下列程式碼：

```shell
curl -isS -X GET http://127.0.0.1:3000/
```

如果您已正確設定您的伺服器，回應看起來應該像下面這樣：

```shell
HTTP/1.1 200 OK
Server: Azure Active Directroy with Node.js Demo
Content-Type: application/json
Content-Length: 49
Date: Tue, 10 Oct 2017 18:35:13 GMT
Connection: keep-alive

Try: curl -isS -X GET http://127.0.0.1:3000/api
```

接下來，您可以藉由在 Bash 殼層輸入下列命令，來測試需要驗證的路由：

```shell
curl -isS -X GET http://127.0.0.1:3000/api
```

如果您已正確設定伺服器，則伺服器應回應 `Unauthorized` 狀態。

```shell
HTTP/1.1 401 Unauthorized
Server: Azure Active Directroy with Node.js Demo
WWW-Authenticate: token is not found
Date: Tue, 10 Oct 2017 16:22:03 GMT
Connection: keep-alive
Content-Length: 12

Unauthorized
```

現在您已建立安全的 API，您可以實作能夠將驗證權杖傳遞給 API 的用戶端。

## <a name="next-steps"></a>後續步驟

* 您必須實作用戶端對應項目來連線到負責處理登入、登出和管理權杖的伺服器。 如需以程式碼為基礎的範例，請參考 [iOS](https://github.com/MSOpenTech/azure-activedirectory-library-for-ios) \(英文\) 和 [Android](https://github.com/MSOpenTech/azure-activedirectory-library-for-android) \(英文\) 中的用戶端應用程式。
* 如需逐步教學課程，請參閱[搭配 Azure AD 的 Node.js Web 應用程式登入和登出](quickstart-v1-openid-connect-code.md)。