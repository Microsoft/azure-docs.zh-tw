---
title: 在 Azure Active Directory B2C 中使用 Node.js 保護保護 Web API | Microsoft Docs
description: 如何建置可接受來自 B2C 租用戶權杖的 Node.js Web API。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 01/07/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 93c3bd3f902f08c8f019744b3f30745c1fd9fa01
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37442418"
---
# <a name="azure-ad-b2c-secure-a-web-api-by-using-nodejs"></a>Azure AD B2C：使用 Node.js 保護 Web API 安全
<!-- TODO [AZURE.INCLUDE [active-directory-b2c-devquickstarts-web-switcher](../../includes/active-directory-b2c-devquickstarts-web-switcher.md)]-->

透過 Azure Active Directory (Azure AD) B2C，您可以使用 OAuth 2.0 存取權杖來保護 Web API。 這些權杖可讓您的用戶端 app 使用 Azure AD B2C 來對 API 進行驗證。 本文說明如何建立「待辦事項清單」API，以便使用者新增和列出工作。 Web API 會使用 Azure AD B2C 進行保護，只允許已驗證的使用者管理他們的待辦事項清單。

> [!NOTE]
> 此範例設計為使用我們的 [iOS B2C 範例應用程式](active-directory-b2c-devquickstarts-ios.md)來連接。 請先執行目前的逐步解說，然後遵循該範例操作。
>
>

**Passport** 是 Node.js 的驗證中介軟體。 您可以暗中將具有彈性且模組化的 Passport 安裝在任何 Express 或 Resitify Web 應用程式中。 有一套完整的策略支援以使用者名稱和密碼、Facebook、Twitter 等來進行驗證。 我們已為 Azure Active Directory (Azure AD) 開發一套策略。 您會安裝此模組，然後新增 Azure AD `passport-azure-ad` 外掛程式。

若要執行此範例，您需要：

1. 向 Azure AD 註冊應用程式。
2. 設定您的應用程式來使用 Passport 的 `passport-azure-ad` 外掛程式。
3. 設定用戶端應用程式呼叫「待辦事項清單」Web API。

## <a name="get-an-azure-ad-b2c-directory"></a>取得 Azure AD B2C 目錄
您必須先建立目錄或租用戶，才可使用 Azure AD B2C。  目錄就是所有使用者、應用程式、群組等項目的容器。  如果您還沒有此資源，請先 [建立 B2C 目錄](active-directory-b2c-get-started.md) 再繼續進行。

## <a name="create-an-application"></a>建立應用程式
接下來，您需要在 B2C 目錄中建立應用程式，以提供一些必要資訊給 Azure AD，讓它與應用程式安全地通訊。 在此案例中，因為用戶端應用程式和 Web API 會組成一個邏輯應用程式，所以由單一 **應用程式識別碼**表示。 如果要建立應用程式，請遵循 [這些指示](active-directory-b2c-app-registration.md)。 請務必：

* 在應用程式中加入 **Web 應用程式/Web API**
* 在 [回覆 URL] 中輸入 `http://localhost/TodoListService`。 這是此程式碼範例的預設 URL。
* 為您的應用程式建立 **應用程式密碼** ，並複製起來。 您稍後需要此資料。 請注意，這個值在使用之前必須經過 [XML 逸出](https://www.w3.org/TR/2006/REC-xml11-20060816/#dt-escape) 。
* 複製指派給您的應用程式的 **應用程式識別碼** 。 您稍後需要此資料。

## <a name="create-your-policies"></a>建立您的原則
在 Azure AD B2C 中，每個使用者體驗皆是由某個 [原則](active-directory-b2c-reference-policies.md)所定義。 此應用程式包含兩種身分識別體驗：註冊和登入。 您必須為每個類型建立一個原則，如 [原則參考文章](active-directory-b2c-reference-policies.md#create-a-sign-up-policy)所述。  建立您的三個原則時，請務必：

* 在註冊原則中，選擇 [顯示名稱]  和其他註冊屬性。
* 在每個原則中，選擇 [顯示名稱] 和 [物件識別碼] 應用程式宣告。  您也可以選擇其他宣告。
* 建立每個原則後，請複製原則的 **名稱** 。 其前置詞應該為 `b2c_1_`。  您稍後需要這些原則名稱。

[!INCLUDE [active-directory-b2c-devquickstarts-policy](../../includes/active-directory-b2c-devquickstarts-policy.md)]

建立您的三個原則後，就可以開始建置您的應用程式。

如需了解 Azure AD B2C 中原則的運作方式，請從 [.NET Web 應用程式快速入門教學課程](active-directory-b2c-devquickstarts-web-dotnet.md)開始。

## <a name="download-the-code"></a>下載程式碼
本教學課程的程式碼保留在 [GitHub](https://github.com/AzureADQuickStarts/B2C-WebAPI-NodeJS)。 如要依照指示建置範例，請 [下載 .zip 檔案格式的基本架構專案](https://github.com/AzureADQuickStarts/B2C-WebAPI-NodeJS/archive/skeleton.zip)。 您也可以複製基本架構：

```
git clone --branch skeleton https://github.com/AzureADQuickStarts/B2C-WebAPI-NodeJS.git
```

完整的 App 也[提供 .zip 檔案格式](https://github.com/AzureADQuickStarts/B2C-WebAPI-NodeJS/archive/complete.zip)，或放在相同儲存機制的 `complete` 分支中。

## <a name="download-nodejs-for-your-platform"></a>下載適用於您平台的 Node.js
若要成功使用此範例，您需要已成功安裝的 Node.js。

從 [nodejs.org](http://nodejs.org)安裝 Node.js。

## <a name="install-mongodb-for-your-platform"></a>安裝適用於您平台的 MongoDB
若要成功使用此範例，您需要已成功安裝的 MongoDB。 您會使用 MongoDB，讓 REST API 得以在不同伺服器執行個體之間持續使用。

從 [mongodb.org](http://www.mongodb.org)安裝 MongoDB。

> [!NOTE]
> 本逐步解說假設您會使用 MongoDB 的預設安裝和伺服器端點，在撰寫本文時為 `mongodb://localhost`。
>
>

## <a name="install-the-restify-modules-in-your-web-api"></a>在您的 Web API 中安裝 Restify 模組
您會使用 Restify 來建置 REST API。 Restify 是衍生自 Express 的最小且具彈性的 Node.js 應用程式架構。 它有一組強大的功能，可在 Connect 之上建置 REST API。

### <a name="install-restify"></a>安裝 Restify
從命令列，切換至 `azuread`目錄。 如果 `azuread` 目錄不存在，請建立一個。

`cd azuread` 或 `mkdir azuread;`

輸入下列命令：

`npm install restify`

此命令會安裝 Restify。

#### <a name="did-you-get-an-error"></a>您有收到錯誤訊息嗎？
在某些作業系統上使用 `npm` 時，您可能會收到 `Error: EPERM, chmod '/usr/local/bin/..'` 的錯誤，並且要求您以系統管理員身分執行帳戶。 若發生此問題，請使用 `sudo` 命令，以更高的權限層級執行 `npm`。

#### <a name="did-you-get-a-dtrace-error"></a>有發生 DTrace 錯誤嗎？
安裝 Restify 時，您可能會看到類似下面的文字：

```Shell
clang: error: no such file or directory: 'HD/azuread/node_modules/restify/node_modules/dtrace-provider/libusdt'
make: *** [Release/DTraceProviderBindings.node] Error 1
gyp ERR! build error
gyp ERR! stack Error: `make` failed with exit code: 2
gyp ERR! stack     at ChildProcess.onExit (/usr/local/lib/node_modules/npm/node_modules/node-gyp/lib/build.js:267:23)
gyp ERR! stack     at ChildProcess.EventEmitter.emit (events.js:98:17)
gyp ERR! stack     at Process.ChildProcess._handle.onexit (child_process.js:789:12)
gyp ERR! System Darwin 13.1.0
gyp ERR! command "node" "/usr/local/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
gyp ERR! cwd /Volumes/Development HD/azuread/node_modules/restify/node_modules/dtrace-provider
gyp ERR! node -v v0.10.11
gyp ERR! node-gyp -v v0.10.0
gyp ERR! not ok
npm WARN optional dep failed, continuing dtrace-provider@0.2.8
```

Restify 提供強大機制來使用 DTrace 追蹤 REST 呼叫。 不過，許多作業系統沒有 DTrace 可以使用。 您可以放心地忽略這些錯誤。

此命令的輸出應類似以下文字：

    restify@2.6.1 node_modules/restify
    ├── assert-plus@0.1.4
    ├── once@1.3.0
    ├── deep-equal@0.0.0
    ├── escape-regexp-component@1.0.2
    ├── qs@0.6.5
    ├── tunnel-agent@0.3.0
    ├── keep-alive-agent@0.0.1
    ├── lru-cache@2.3.1
    ├── node-uuid@1.4.0
    ├── negotiator@0.3.0
    ├── mime@1.2.11
    ├── semver@2.2.1
    ├── spdy@1.14.12
    ├── backoff@2.3.0
    ├── formidable@1.0.14
    ├── verror@1.3.6 (extsprintf@1.0.2)
    ├── csv@0.3.6
    ├── http-signature@0.10.0 (assert-plus@0.1.2, asn1@0.1.11, ctype@0.5.2)
    └── bunyan@0.22.0 (mv@0.0.5)

## <a name="install-passport-in-your-web-api"></a>在您的 Web API 中安裝 Passport
從命令列，切換至 `azuread`目錄 (如果還不在此目錄下)。

使用下列命令安裝 Passport：

`npm install passport`

此命令的輸出應類似以下文字：

    passport@0.1.17 node_modules\passport
    ├── pause@0.0.1
    └── pkginfo@0.2.3

## <a name="add-passport-azuread-to-your-web-api"></a>將 passport-azuread 加入您的 Web API
接下來，使用 `passport-azuread`來新增 OAuth 策略，這是一套將 Azure AD 連接到 Passport 的策略。 在這個 Rest API 範例中，請針對持有人權杖使用此策略。

> [!NOTE]
> 雖然 OAuth2 提供可發行任何已知權杖類型的架構，但只會普遍使用特定的權杖類型。 用於保護端點的權杖是持有者權杖。 這些是 OAuth2 中最普遍發行的權杖類型。 許多實作假設持有者權杖會是唯一發行的權杖類型。
>
>

從命令列，切換至 `azuread`目錄 (如果還不在此目錄下)。

使用下列命令安裝 Passport `passport-azure-ad` 模組：

`npm install passport-azure-ad`

此命令的輸出應類似以下文字：

``
passport-azure-ad@1.0.0 node_modules/passport-azure-ad
├── xtend@4.0.0
├── xmldom@0.1.19
├── passport-http-bearer@1.0.1 (passport-strategy@1.0.0)
├── underscore@1.8.3
├── async@1.3.0
├── jsonwebtoken@5.0.2
├── xml-crypto@0.5.27 (xpath.js@1.0.6)
├── ursa@0.8.5 (bindings@1.2.1, nan@1.8.4)
├── jws@3.0.0 (jwa@1.0.1, base64url@1.0.4)
├── request@2.58.0 (caseless@0.10.0, aws-sign2@0.5.0, forever-agent@0.6.1, stringstream@0.0.4, tunnel-agent@0.4.1, oauth-sign@0.8.0, isstream@0.1.2, extend@2.0.1, json-stringify-safe@5.0.1, node-uuid@1.4.3, qs@3.1.0, combined-stream@1.0.5, mime-types@2.0.14, form-data@1.0.0-rc1, http-signature@0.11.0, bl@0.9.4, tough-cookie@2.0.0, hawk@2.3.1, har-validator@1.8.0)
└── xml2js@0.4.9 (sax@0.6.1, xmlbuilder@2.6.4)
``

## <a name="add-mongodb-modules-to-your-web-api"></a>將 MongoDB 模組加入您的 Web API
此範例會使用 MongoDB 作為資料存放區。 為此安裝 Mongoose，這是廣泛用於管理模型和結構描述的外掛程式。

* `npm install mongoose`

## <a name="install-additional-modules"></a>安裝其他模組
接下來，安裝其餘所需的模組。

從命令列，切換至 `azuread`目錄 (如果還不在此目錄下)：

`cd azuread`

在您的 `node_modules` 目錄中安裝模組：

* `npm install assert-plus`
* `npm install ejs`
* `npm install ejs-locals`
* `npm install express`
* `npm install bunyan`

## <a name="create-a-serverjs-file-with-your-dependencies"></a>使用您的相依性建立 server.js 檔案
`server.js` 檔案會提供您的 Web API 伺服器的大部分功能。

從命令列，切換至 `azuread`目錄 (如果還不在此目錄下)：

`cd azuread`

在編輯器中建立 `server.js` 檔案。 加入下列資訊：

```Javascript
'use strict';
/**
* Module dependencies.
*/
var fs = require('fs');
var path = require('path');
var util = require('util');
var assert = require('assert-plus');
var mongoose = require('mongoose/');
var bunyan = require('bunyan');
var restify = require('restify');
var config = require('./config');
var passport = require('passport');
var OIDCBearerStrategy = require('passport-azure-ad').BearerStrategy;
```

儲存檔案。 您稍後會回到此檔案。

## <a name="create-a-configjs-file-to-store-your-azure-ad-settings"></a>建立 config.js 檔案來儲存您的 Azure AD 設定
這個程式碼檔會將設定參數從您的 Azure AD 入口網站傳遞到 `Passport.js` 檔案。 在本逐步解說的第一個部分，您在將 Web API 加入入口網站時已建立這些組態值。 在您複製程式碼之後，我們會說明要放入這些參數值的內容。

從命令列，切換至 `azuread`目錄 (如果還不在此目錄下)：

`cd azuread`

在編輯器中建立 `config.js` 檔案。 加入下列資訊：

```Javascript
// Don't commit this file to your public repos. This config is for first-run
exports.creds = {
clientID: <your client ID for this Web API you created in the portal>
mongoose_auth_local: 'mongodb://localhost/tasklist', // Your mongo auth uri goes here
audience: '<your audience URI>', // the Client ID of the application that is calling your API, usually a web API or native client
identityMetadata: 'https://login.microsoftonline.com/<tenant name>/.well-known/openid-configuration', // Make sure you add the B2C tenant name in the <tenant name> area
tenantName:'<tenant name>',
policyName:'b2c_1_<sign in policy name>' // This is the policy you'll want to validate against in B2C. Usually this is your Sign-in policy (as users sign in to this API)
passReqToCallback: false // This is a node.js construct that lets you pass the req all the way back to any upstream caller. We turn this off as there is no upstream caller.
};

```

[!INCLUDE [active-directory-b2c-devquickstarts-tenant-name](../../includes/active-directory-b2c-devquickstarts-tenant-name.md)]

### <a name="required-values"></a>必要值
`clientID`您的 Web API 應用程式的用戶端識別碼。

`IdentityMetadata`：`passport-azure-ad` 會在這裡尋找識別提供者的組態資料。 它也會尋找金鑰以驗證 JSON Web 權杖。

`audience`：入口網站上的統一資源識別項 (URI)，用於識別您的呼叫端應用程式。

`tenantName`：您的租用戶名稱 (例如 **contoso.onmicrosoft.com**)。

`policyName`：您想用來驗證傳入伺服器之權杖的原則。 此原則應與您針對用戶端應用程式登入所用的原則相同。

> [!NOTE]
> 現在，請將相同的原則用於用戶端與伺服器設定。 如果您已完成逐步解說並建立這些原則，則不需要再做一次。 由於您已完成此逐步解說，您應該不需要在網站上為用戶端逐步解說建立新的原則。
>
>

## <a name="add-configuration-to-your-serverjs-file"></a>將設定加入 server.js 檔案
若要從您建立的 `config.js`檔案讀取值，請在應用程式中新增 `.config` 檔案作為必要資源，然後將全域變數設定為 `config.js` 文件中的那些值。

從命令列，切換至 `azuread`目錄 (如果還不在此目錄下)：

`cd azuread`

在編輯器中開啟 `server.js` 檔案。 加入下列資訊：

```Javascript
var config = require('./config');
```
將包含下列程式碼的新區段新增至 `server.js` ：

```Javascript
// We pass these options in to the ODICBearerStrategy.

var options = {
    // The URL of the metadata document for your app. We put the keys for token validation from the URL found in the jwks_uri tag of the in the metadata.
    identityMetadata: config.creds.identityMetadata,
    clientID: config.creds.clientID,
    tenantName: config.creds.tenantName,
    policyName: config.creds.policyName,
    validateIssuer: config.creds.validateIssuer,
    audience: config.creds.audience,
    passReqToCallback: config.creds.passReqToCallback

};
```

接下來，我們會為使用者新增一些從呼叫端應用程式收到的預留位置。

```Javascript
// array to hold logged in users and the current logged in user (owner)
var users = [];
var owner = null;
```

不用猶豫，讓我們同時建立記錄器。

```Javascript
// Our logger
var log = bunyan.createLogger({
    name: 'Microsoft Azure Active Directory Sample'
});
```

## <a name="add-the-mongodb-model-and-schema-information-by-using-mongoose"></a>使用 Moongoose 新增 MongoDB 模型和結構描述資訊
及早準備有助於在 REST API 服務中組合這三個檔案。

在此逐步解說中，請使用 MongoDB 來儲存工作，稍早所述。

在 `config.js` 檔案中，您將資料庫命名為 **tasklist**。 該名稱也是您在 `mongoose_auth_local` 連線 URL 結尾處輸入的內容。 您不需要在 MongoDB 中事先建立此資料庫。 第一次執行應用程式伺服器時，它會為您建立資料庫。

在您告訴伺服器要使用哪個 MongoDB 資料庫之後，您必須撰寫一些額外程式碼，以建立伺服器工作的模型和結構描述。

### <a name="expand-the-model"></a>擴充模型
此結構描述模型很簡單。 您可以視需要來擴充它。

`owner`：指派給工作的人員。 此物件是 **字串**。  

`Text`：工作本身。 此物件是 **字串**。

`date`：工作到期的日期。 此物件是 **日期時間**。

`completed`：工作是否已完成。 此物件是 **布林值**。

### <a name="create-the-schema-in-the-code"></a>在程式碼中建立結構描述
從命令列，切換至 `azuread`目錄 (如果還不在此目錄下)：

`cd azuread`

在編輯器中開啟 `server.js` 檔案。 在組態項目下方加入下列資訊：

```Javascript
// MongoDB setup
// Setup some configuration
var serverPort = process.env.PORT || 3000; // Note we are hosting our API on port 3000
var serverURI = (process.env.PORT) ? config.creds.mongoose_auth_mongohq : config.creds.mongoose_auth_local;

// Connect to MongoDB
global.db = mongoose.connect(serverURI);
var Schema = mongoose.Schema;
log.info('MongoDB Schema loaded');

// Here we create a schema to store our tasks and users. Pretty simple schema for now.
var TaskSchema = new Schema({
    owner: String,
    Text: String,
    completed: Boolean,
    date: Date
});

// Use the schema to register a model
mongoose.model('Task', TaskSchema);
var Task = mongoose.model('Task');
```
您首先建立結構描述，然後建立模型物件，在您定義 **路由**時用來儲存整個程式碼中的資料。

## <a name="add-routes-for-your-rest-api-task-server"></a>新增 REST API 工作伺服器的路由
既然您已經有可以使用的資料庫模型，請新增要用於 REST API 伺服器的路由。

### <a name="about-routes-in-restify"></a>關於 Restify 中的路由
路由在 Restify 中的運作與它們使用 Express 堆疊時的運作方式相同。 您可以使用您預期用戶端應用程式呼叫的 URI 來定義路由。

Restify 路由的典型模式是：

```Javascript
function createObject(req, res, next) {
// do work on Object
_object.name = req.params.object; // passed value is in req.params under object
///...
return next(); // keep the server going
}
....
server.post('/service/:add/:object', createObject); // calls createObject on routes that match this.
```

Resitfy 及 Express 可提供更深入的功能，例如定義應用程式類型和執行不同端點之間的複雜路由。 基於本教學課程的目的，我們會將這些路由保持簡單。

#### <a name="add-default-routes-to-your-server"></a>將預設路由加入伺服器
您現在可為 REST API 新增 **create** 和 **list** 的基本 CRUD 路由。 在範例的 `complete` 分支中可以找到其他路由。

從命令列，切換至 `azuread`目錄 (如果還不在此目錄下)：

`cd azuread`

在編輯器中開啟 `server.js` 檔案。 在您先前輸入的資料庫項目底下，新增下列資訊：

```Javascript
/**
 *
 * APIs for our REST Task server
 */

// Create a task

function createTask(req, res, next) {

    // Resitify currently has a bug which doesn't allow you to set default headers
    // This headers comply with CORS and allow us to mongodbServer our response to any origin

    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "X-Requested-With");

    // Create a new task model, fill it up and save it to Mongodb
    var _task = new Task();

    if (!req.params.Text) {
        req.log.warn({
            params: req.params
        }, 'createTodo: missing task');
        next(new MissingTaskError());
        return;
    }

    _task.owner = owner;
    _task.Text = req.params.Text;
    _task.date = new Date();

    _task.save(function(err) {
        if (err) {
            req.log.warn(err, 'createTask: unable to save');
            next(err);
        } else {
            res.send(201, _task);

        }
    });

    return next();

}
```

```Javascript
/// Simple returns the list of TODOs that were loaded.

function listTasks(req, res, next) {
    // Resitify currently has a bug which doesn't allow you to set default headers
    // This headers comply with CORS and allow us to mongodbServer our response to any origin

    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "X-Requested-With");

    log.info("listTasks was called for: ", owner);

    Task.find({
        owner: owner
    }).limit(20).sort('date').exec(function(err, data) {

        if (err)
            return next(err);

        if (data.length > 0) {
            log.info(data);
        }

        if (!data.length) {
            log.warn(err, "There is no tasks in the database. Add one!");
        }

        if (!owner) {
            log.warn(err, "You did not pass an owner when listing tasks.");
        } else {

            res.json(data);

        }
    });

    return next();
}
```


#### <a name="add-error-handling-for-the-routes"></a>新增路由的錯誤處理
新增一些錯誤處理，以便您將遇到的任何問題，以用戶端可了解的方式傳回給用戶端。

新增下列程式碼：

```Javascript
///--- Errors for communicating something interesting back to the client
function MissingTaskError() {
restify.RestError.call(this, {
statusCode: 409,
restCode: 'MissingTask',
message: '"task" is a required parameter',
constructorOpt: MissingTaskError
});
this.name = 'MissingTaskError';
}
util.inherits(MissingTaskError, restify.RestError);
function TaskExistsError(owner) {
assert.string(owner, 'owner');
restify.RestError.call(this, {
statusCode: 409,
restCode: 'TaskExists',
message: owner + ' already exists',
constructorOpt: TaskExistsError
});
this.name = 'TaskExistsError';
}
util.inherits(TaskExistsError, restify.RestError);
function TaskNotFoundError(owner) {
assert.string(owner, 'owner');
restify.RestError.call(this, {
statusCode: 404,
restCode: 'TaskNotFound',
message: owner + ' was not found',
constructorOpt: TaskNotFoundError
});
this.name = 'TaskNotFoundError';
}
util.inherits(TaskNotFoundError, restify.RestError);
```


## <a name="create-your-server"></a>建立伺服器
您現在已定義資料庫並備妥路由。 最後一件事是新增伺服器執行個體，以管理您的呼叫。

Restify 及 Express 為 REST API 伺服器提供進階的自訂功能，但我們會使用最基本的設定。

```Javascript

/**
 * Our Server
 */


var server = restify.createServer({
    name: "Microsoft Azure Active Directroy TODO Server",
    version: "2.0.1"
});

// Ensure we don't drop data on uploads
server.pre(restify.pre.pause());

// Clean up sloppy paths like //todo//////1//
server.pre(restify.pre.sanitizePath());

// Handles annoying user agents (curl)
server.pre(restify.pre.userAgentConnection());

// Set a per request bunyan logger (with requestid filled in)
server.use(restify.requestLogger());

// Allow 5 requests/second by IP, and burst to 10
server.use(restify.throttle({
    burst: 10,
    rate: 5,
    ip: true,
}));

// Use the common stuff you probably want
server.use(restify.acceptParser(server.acceptable));
server.use(restify.dateParser());
server.use(restify.queryParser());
server.use(restify.gzipResponse());
server.use(restify.bodyParser({
    mapParams: true
})); // Allows for JSON mapping to REST
server.use(restify.authorizationParser()); // Looks for authorization headers

// Let's start using Passport.js

server.use(passport.initialize()); // Starts passport
server.use(passport.session()); // Provides session support


```
## <a name="add-the-routes-to-the-server-without-authentication"></a>將路由加入伺服器 (不含驗證)
```Javascript
server.get('/api/tasks', passport.authenticate('oauth-bearer', {
    session: false
}), listTasks);
server.get('/api/tasks', passport.authenticate('oauth-bearer', {
    session: false
}), listTasks);
server.get('/api/tasks/:owner', passport.authenticate('oauth-bearer', {
    session: false
}), getTask);
server.head('/api/tasks/:owner', passport.authenticate('oauth-bearer', {
    session: false
}), getTask);
server.post('/api/tasks/:owner/:task', passport.authenticate('oauth-bearer', {
    session: false
}), createTask);
server.post('/api/tasks', passport.authenticate('oauth-bearer', {
    session: false
}), createTask);
server.del('/api/tasks/:owner/:task', passport.authenticate('oauth-bearer', {
    session: false
}), removeTask);
server.del('/api/tasks/:owner', passport.authenticate('oauth-bearer', {
    session: false
}), removeTask);
server.del('/api/tasks', passport.authenticate('oauth-bearer', {
    session: false
}), removeTask);
server.del('/api/tasks', passport.authenticate('oauth-bearer', {
    session: false
}), removeAll, function respond(req, res, next) {
    res.send(204);
    next();
});


// Register a default '/' handler

server.get('/', function root(req, res, next) {
    var routes = [
        'GET     /',
        'POST    /api/tasks/:owner/:task',
        'POST    /api/tasks (for JSON body)',
        'GET     /api/tasks',
        'PUT     /api/tasks/:owner',
        'GET     /api/tasks/:owner',
        'DELETE  /api/tasks/:owner/:task'
    ];
    res.send(200, routes);
    next();
});
```

```Javascript

server.listen(serverPort, function() {

    var consoleMessage = '\n Microsoft Azure Active Directory Tutorial';
    consoleMessage += '\n +++++++++++++++++++++++++++++++++++++++++++++++++++++';
    consoleMessage += '\n %s server is listening at %s';
    consoleMessage += '\n Open your browser to %s/api/tasks\n';
    consoleMessage += '+++++++++++++++++++++++++++++++++++++++++++++++++++++ \n';
    consoleMessage += '\n !!! why not try a $curl -isS %s | json to get some ideas? \n';
    consoleMessage += '+++++++++++++++++++++++++++++++++++++++++++++++++++++ \n\n';

    //log.info(consoleMessage, server.name, server.url, server.url, server.url);

});

```

## <a name="add-authentication-to-your-rest-api-server"></a>將驗證加入 REST API 伺服器
既然您已經有執行中的 REST API 伺服器，您可以讓它在 Azure AD 中發揮價值。

從命令列，切換至 `azuread`目錄 (如果還不在此目錄下)：

`cd azuread`

### <a name="use-the-oidcbearerstrategy-that-is-included-with-passport-azure-ad"></a>使用 passport-azure-ad 隨附的 OIDCBearerStrategy
> [!TIP]
> 撰寫 API 時，您應一律將資料連結到使用者無法證明其在權杖中是唯一的項目。 伺服器儲存 ToDo 項目時，它會根據放在 [擁有者] 欄位的權杖 (透過 token.oid 呼叫) 中使用者的 **oid** 來儲存這些項目。 此值可確保只有該使用者可以存取自己的 ToDo 項目。 不會在「擁有者」API 中公開，因此，外部使用者可以要求其他人的 ToDo 項目，即使它們已經過驗證也一樣。
>
>

接下來，使用隨附於 `passport-azure-ad`的持有人策略。

```Javascript
var findById = function(id, fn) {
    for (var i = 0, len = users.length; i < len; i++) {
        var user = users[i];
        if (user.oid === id) {
            log.info('Found user: ', user);
            return fn(null, user);
        }
    }
    return fn(null, null);
};


var oidcStrategy = new OIDCBearerStrategy(options,
    function(token, done) {
        log.info('verifying the user');
        log.info(token, 'was the token retreived');
        findById(token.sub, function(err, user) {
            if (err) {
                return done(err);
            }
            if (!user) {
                // "Auto-registration"
                log.info('User was added automatically as they were new. Their sub is: ', token.oid);
                users.push(token);
                owner = token.oid;
                return done(null, token);
            }
            owner = token.sub;
            return done(null, user, token);
        });
    }
);

passport.use(oidcStrategy);
```

Passport 對其所有的策略使用相同的模式。 您需要傳遞以 `token` 和 `done` 為參數的 `function()` 給它。 策略在完成所有工作之後會回到您這邊。 然後，您應該儲存使用者和權杖，如此就不需再次要求它。

> [!IMPORTANT]
> 上述程式碼會接受正好向您的伺服器驗證的任何使用者。 此程序稱為自動註冊。 在生產伺服器中，除非使用者先完成註冊程序，否則不要讓他們存取 API。 此程序常見於一些取用者應用程式中，先是可讓您使用 Facebook 來註冊，但接著會要求您填寫其他資訊。 如果此程式不是命令列程式，我們可以從傳回的權杖物件中擷取電子郵件，然後要求使用者填寫其他資訊。 因為這是範例，所以我們會將其新增至記憶體中的資料庫。
>
>

## <a name="run-your-server-application-to-verify-that-it-rejects-you"></a>執行伺服器應用程式，確認它會拒絕您的要求
您可以使用 `curl` ，檢查現在是否以 OAuth2 保護您的端點。 傳回的標頭應該足以說明您執行的作業正確無誤。

請確定您的 MongoDB 執行個體正在執行：

    $sudo mongodb

切換至目錄，然後執行伺服器：

    $ cd azuread
    $ node server.js

在新的終端機視窗中，執行 `curl`

嘗試基本的 POST 方法：

`$ curl -isS -X POST http://127.0.0.1:3000/api/tasks/brandon/Hello`

```Shell
HTTP/1.1 401 Unauthorized
Connection: close
WWW-Authenticate: Bearer realm="Users"
Date: Tue, 14 Jul 2015 05:45:03 GMT
Transfer-Encoding: chunked
```

401 錯誤是您想要的回應。 它指出 Passport 層正嘗試重新導向到授權端點。

## <a name="you-now-have-a-rest-api-service-that-uses-oauth2"></a>您現在擁有一項使用 OAuth2 的 REST API 服務
您已使用 Restify 和 OAuth 實作 REST API！ 您現在已經有足夠的程式碼可以繼續開發服務，並以此範例為基礎進行建置。 在未使用 OAuth2 相容用戶端的情況下，您已經儘可能地使用此伺服器的所有功能。 接下來使用其他逐步解說，例如我們的 [使用 iOS 搭配 B2C 連線至 Web API](active-directory-b2c-devquickstarts-ios.md) 逐步解說。

## <a name="next-steps"></a>後續步驟
您現在可以進入更進階的主題，例如：

[使用 iOS 搭配 B2C 連線至 Web API](active-directory-b2c-devquickstarts-ios.md)
