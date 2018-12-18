---
title: v2.0 的應用程式類型 | Azure
description: Azure Active Directory v2.0 端點所支援的應用程式類型與案例。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 494a06b8-0f9b-44e1-a7a2-d728cf2077ae
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: saeeda, jmprieur, andret
ms.custom: aaddev
ms.openlocfilehash: 24a9b014028bf99673881904e17ec0911d0b5063
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46952047"
---
# <a name="application-types-for-v20"></a>v2.0 的應用程式類型

Azure Active Directory (Azure AD) v2.0 端點支援各種新型應用程式架構的驗證，它們全都以產業標準通訊協定 [OAuth 2.0 或 OpenID Connect](active-directory-v2-protocols.md) 為基準。 本文描述您可以使用 Azure AD v2.0 來建置的應用程式類型，不論您慣用的語言或平台是哪一種。 本文中的資訊是設計來協助您在[開始使用程式碼](v2-overview.md#getting-started)之前，先了解概要的案例。

> [!NOTE]
> v2.0 端點並未支援所有的 Azure Active Directory 案例和功能。 若要判斷您是否應該使用 v2.0 端點，請參閱 [v2.0 限制](active-directory-v2-limitations.md)。

## <a name="the-basics"></a>基本概念

您必須在 [Microsoft 應用程式註冊入口網站](https://apps.dev.microsoft.com)中註冊每個使用 v2.0 端點的應用程式。 應用程式註冊程序會為您的應用程式收集和指派下列值：

* 可唯一識別應用程式的「應用程式識別碼」
* 可用來將回應導回到應用程式的「重新導向 URI」
* 一些其他案例特定值

如需詳細資訊，請了解如何[註冊應用程式](quickstart-v2-register-an-app.md)。

註冊應用程式之後，應用程式便可傳送要求給 Azure AD v2.0 端點來與 Azure AD 通訊。 我們提供開放原始碼架構，以及可處理這些要求詳細資料的程式庫。 您也可以選擇建立對這些端點的要求，來自行實作驗證邏輯：

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
https://login.microsoftonline.com/common/oauth2/v2.0/token
```

## <a name="single-page-apps-javascript"></a>單頁應用程式 (JavaScript)

許多新式應用程式都有一個單頁應用程式前端，主要是以 JavaScript 撰寫。 通常是使用 AngularJS、Ember.js、Durandal.js 等架構來撰寫它們。 Azure AD v2.0 端點是透過使用 [OAuth 2.0 隱含流程](v2-oauth2-implicit-grant-flow.md)來支援這些應用程式。

在此流程中，應用程式會直接從 v2.0 授權端點接收權杖，而不需要執行任何伺服器對伺服器交換。 所有驗證邏輯和工作階段處理都完全在 JavaScript 用戶端中進行，而不需要執行額外的頁面重新導向。

![隱含驗證流程](./media/v2-app-types/convergence_scenarios_implicit.png)

若要查看此案例的實際運作情形，請在 [v2.0 開始使用](v2-overview.md#getting-started)一節的單頁應用程式程式碼範例中擇一試用。

## <a name="web-apps"></a>Web 應用程式

針對使用者透過瀏覽器存取的 Web 應用程式 (.NET、PHP、Java、Ruby、Python、Node)，您可以使用 [OpenID Connect](active-directory-v2-protocols.md) 來執行使用者登入。 在 OpenID Connect 中，Web 應用程式會收到識別碼權杖。 識別碼權杖是一個安全性權杖，可驗證使用者的身分識別並以宣告形式提供使用者的相關資訊：

```
// Partial raw ID token
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImtyaU1QZG1Cd...

// Partial content of a decoded ID token
{
    "name": "John Smith",
    "email": "john.smith@gmail.com",
    "oid": "d9674823-dffc-4e3f-a6eb-62fe4bd48a58"
    ...
}
```

如需 v2.0 端點中所用不同權杖類型的詳細說明，請參閱[存取權杖](access-tokens.md)參考和 [`id_token` 參考](id-tokens.md)

在 Web 伺服器應用程式中，登入驗證流程會採用下列概要步驟：

![Web 應用程式驗證流程](./media/v2-app-types/convergence_scenarios_webapp.png)

您可以使用接收自 v2.0 端點的公開簽署金鑰來驗證識別碼權杖，來確保使用者的身分識別。 系統會設定工作階段 Cookie，這可在後續的頁面要求上用來識別使用者。

若要查看此案例的實際運作情形，請在 [v2.0 開始使用](v2-overview.md#getting-started)一節的 Web 應用程式登入程式碼範例中擇一試用。

除了簡易登入之外，Web 伺服器應用程式可能也需要存取其他 Web 服務，例如 REST API。 在此情況下，Web 伺服器應用程式可以使用 [OAuth 2.0 授權碼流程](active-directory-v2-protocols.md)，參與結合了 OpenID Connect 與 OAuth 2.0 的流程。 如需有關此案例的詳細資訊，請參閱[開始使用 Web 應用程式和 Web API](active-directory-v2-devquickstarts-webapp-webapi-dotnet.md)。

## <a name="web-apis"></a>Web API

您可以使用 v2.0 端點來保護 Web 服務，例如應用程式的 RESTful Web API。 Web API 使用 OAuth 2.0 存取權杖來保護其資料及驗證連入要求，而不是使用識別碼權杖和工作階段 Cookie。 Web API 的呼叫端會在 HTTP 要求的授權標頭中附加存取權杖，就像這樣：

```
GET /api/items HTTP/1.1
Host: www.mywebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6...
Accept: application/json
...
```

Web API 會使用這個存取權杖來驗證 API 呼叫端的身分識別，並從存取權杖中所編碼的宣告擷取呼叫端的相關資訊。 如需 v2.0 端點中所用不同權杖類型的詳細說明，請參閱[存取權杖](access-tokens.md)參考和 [`id_token` 參考](id-tokens.md)

Web API 可透過公開權限的方式 (亦稱為[範圍](v2-permissions-and-consent.md))，讓使用者能夠選擇加入或退出特定的功能或資料。 為了讓發出呼叫的應用程式取得範圍的權限，使用者必須在流程中對範圍表示同意。 v2.0 端點會向使用者要求權限，然後將這些權限記錄在 Web API 接收的所有存取權杖中。 Web API 會驗證它在每次呼叫所收到的存取權杖，並執行授權檢查。

Web API 可以從所有類型的應用程式接收存取權杖，包括 Web 伺服器應用程式、傳統型應用程式和行動應用程式、單頁應用程式、伺服器端精靈，甚至是其他的 Web API。 Web API 的概要流程看起來像這樣：

![Web API 驗證流程](./media/v2-app-types/convergence_scenarios_webapi.png)

若要了解如何使用 OAuth2 存取權杖來保護 Web API，請查看 [v2.0 開始使用](v2-overview.md#getting-started)一節中的 Web API 程式碼範例。

在許多情況下，Web API 也需要對受 Azure Active Directory 保護的其他下游 Web API 發出傳出要求。 若要這樣做，Web API 可以利用 Azure AD 的「代理者」流程，它能允許 Web API 將傳入存取權杖交換為要在傳出要求中使用的另一個存取權杖。 v2.0 端點的「代理者」流程已[在此詳細說明](v2-oauth2-on-behalf-of-flow.md)。

## <a name="mobile-and-native-apps"></a>行動和原生應用程式

裝置安裝的應用程式 (例如行動應用程式和傳統型應用程式) 通常需要存取儲存資料及代表使用者執行功能的後端服務或 Web API。 這些應用程式可以使用 [OAuth 2.0 授權碼流程](v2-oauth2-auth-code-flow.md)，將登入和授權新增至後端服務。

在此流程中，應用程式會在使用者登入時，從 v2.0 端點接收授權碼。 授權碼代表應用程式具備權限，可代表登入的使用者呼叫後端服務。 應用程式可以在背景中以授權碼交換 OAuth 2.0 存取權杖和重新整理權杖。 應用程式可以使用存取權杖在 HTTP 要求中向 Web API 進行驗證，以及在舊存取權杖到期時，使用重新整理權杖來取得新的存取權杖。

![原生應用程式驗證流程](./media/v2-app-types/convergence_scenarios_native.png)

## <a name="daemons-and-server-side-apps"></a>精靈和伺服器端應用程式

應用程式如果含有長時間執行的程序，或其運作方式不需要與使用者互動，就也需要一個存取受保護資源 (例如 Web API) 的方法。 這些應用程式可以使用應用程式的身分識別 (而非使用者委派的身分識別) 搭配 OAuth 2.0 用戶端認證流程，來驗證及取得權杖。

在此流程中，應用程式會直接與 `/token` 端點互動來取得端點：

![精靈應用程式驗證流程](./media/v2-app-types/convergence_scenarios_daemon.png)

若要建置精靈應用程式，請參閱[用戶端認證文件](v2-oauth2-client-creds-grant-flow.md)，或試試 [.NET 範例應用程式](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2)。
