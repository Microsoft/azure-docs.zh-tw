---
title: 自訂 Azure App Service 中的驗證和授權 | Microsoft Docs
description: 示範如何自訂 App Service 中的驗證與授權，以及取得使用者宣告和不同的權杖。
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.service: app-service
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 03/14/2018
ms.author: cephalin
ms.openlocfilehash: 629a76ab5610625e14780d7b5c57d3979c2224c9
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43344165"
---
# <a name="customize-authentication-and-authorization-in-azure-app-service"></a>自訂 Azure App Service 中的驗證與授權

本文會說明如何自訂 [App Service 中的驗證與授權](app-service-authentication-overview.md)，以及管理您應用程式的身分識別。 

若要快速開始，請參閱下列其中一個教學課程︰

* [教學課程：在 Azure App Service 中端對端驗證和授權使用者 (Windows)](app-service-web-tutorial-auth-aad.md)
* [教學課程：在適用於 Linux 的 Azure App Service 中端對端驗證和授權使用者](containers/tutorial-auth-aad.md)
* [如何設定您的 App 以使用 Azure Active Directory 登入](app-service-mobile-how-to-configure-active-directory-authentication.md)
* [如何設定 App 以使用 Facebook 登入](app-service-mobile-how-to-configure-facebook-authentication.md)
* [如何設定 App 以使用 Google 登入](app-service-mobile-how-to-configure-google-authentication.md)
* [如何設定 App 以使用 Microsoft 帳戶登入](app-service-mobile-how-to-configure-microsoft-authentication.md)
* [如何設定 App 以使用 Twitter 登入](app-service-mobile-how-to-configure-twitter-authentication.md)

## <a name="use-multiple-sign-in-providers"></a>使用多個登入提供者

入口網站設定不會提供周全的方式，向您的使用者顯示多個登入提供者 (例如 Facebook 和 Twitter)。 不過，要將功能新增至您的 Web 應用程式並不困難。 步驟概述如下：

首先，在 Azure 入口網站的 [驗證/授權] 頁面中，設定您需要啟用的每一個識別提供者。

在 [當要求未經驗證時所要採取的動作] 中，選取 [允許匿名要求 (無動作)]。

在登入頁面或導覽列、或是您 Web 應用程式的任何其他位置中，將登入連結新增至您啟用的每個提供者 (`/.auth/login/<provider>`)。 例如︰

```HTML
<a href="/.auth/login/aad">Log in with Azure AD</a>
<a href="/.auth/login/microsoftaccount">Log in with Microsoft Account</a>
<a href="/.auth/login/facebook">Log in with Facebook</a>
<a href="/.auth/login/google">Log in with Google</a>
<a href="/.auth/login/twitter">Log in with Twitter</a>
```

當使用者按一下其中一個連結時，隨即會開啟個別登入頁面將使用者登入。

若要將登入後的使用者重新導向至自訂 URL，請使用 `post_login_redirect_url` 查詢字串參數 (請勿與您身分識別提供者組態中的重新導向 URI 混淆)。 例如，若要在使用者登入之後，將他們導向 `/Home/Index`，請使用下列 HTML 程式碼：

```HTML
<a href="/.auth/login/<provider>?post_login_redirect_url=/Home/Index">Log in</a>
```

## <a name="sign-out-of-a-session"></a>登出工作階段

使用者可以將 `GET` 要求傳送到應用程式的 `/.auth/logout` 端點起始登出。 `GET` 要求會執行下列作業：

- 清除目前工作階段的驗證 Cookie。
- 從安全性權杖存放區中刪除目前使用者的安全性權杖。
- 對於 Azure Active Directory 和 Google，對身分識別提供者執行伺服器端登出。

以下是網頁中的簡單登出連結：

```HTML
<a href="/.auth/logout">Sign out</a>
```

成功登出預設會將用戶端重新導向到 URL `/.auth/logout/done`。 您可以新增 `post_logout_redirect_uri` 查詢參數來變更登出後重新導向頁面。 例如︰

```
GET /.auth/logout?post_logout_redirect_uri=/index.html
```

建議您將值 `post_logout_redirect_uri` [編碼](https://wikipedia.org/wiki/Percent-encoding)。

使用完整的 URL，URL 必須裝載於相同的網域，或設定為應用程式允許的外部重新導向 URL。 在下列範例中，將重新導向至不在相同的網域中裝載的 `https://myexternalurl.com`：

```
GET /.auth/logout?post_logout_redirect_uri=https%3A%2F%2Fmyexternalurl.com
```

您必須在 [Azure Cloud Shell](../cloud-shell/quickstart.md) 中輸入下列命令︰

```azurecli-interactive
az webapp auth update --name <app_name> --resource-group <group_name> --allowed-external-redirect-urls "https://myexternalurl.com"
```

## <a name="preserve-url-fragments"></a>保留 URL 片段

使用者登入您的應用程式之後，通常想要重新導向至相同頁面的相同區段，例如 `/wiki/Main_Page#SectionZ`。 不過，因為 [URL 片段](https://wikipedia.org/wiki/Fragment_identifier) (例如，`#SectionZ`) 不會傳送到伺服器，因此 OAuth 登入完成之後預設不會保留，而會重新導向回到您的應用程式。 使用者會在需要再次瀏覽至所需的錨點時感受到欠佳的體驗。 這項限制適用於所有的伺服器端驗證解決方案。

在 App Service 驗證中，您可以在 OAuth 登入保留 URL 片段。 若要這麼做，將稱為 `WEBSITE_AUTH_PRESERVE_URL_FRAGMENT` 的應用程式設定設為 `true`。 您可以在 [Azure 入口網站](https://portal.azure.com)中這麼做，也可以直接在 [Azure Cloud Shell](../cloud-shell/quickstart.md) 中執行下列命令：

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group <group_name> --settings WEBSITE_AUTH_PRESERVE_URL_FRAGMENT="true"
```

## <a name="access-user-claims"></a>存取使用者宣告

App Service 會使用特殊標頭，將使用者宣告傳遞至您的應用程式。 外部要求不得設定這些標頭，因此僅在由 App Service 設定時，這些標頭才會出現。 某些範例標頭包括︰

* X-MS-CLIENT-PRINCIPAL-NAME
* X-MS-CLIENT-PRINCIPAL-ID

以任何語言或架構撰寫的程式碼可以從這些標頭中取得所需的資訊。 針對 ASP.NET 4.6 應用程式， **ClaimsPrincipal** 會自動設定適當的值。

您的應用程式也可以藉由呼叫 `/.auth/me` 來取得關於已驗證使用者的其他詳細資料。 Mobile Apps 伺服器 SDK 提供 Helper 方法來處理此資料。 如需詳細資訊，請參閱[如何使用 Azure Mobile Apps Node.js SDK ](../app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk.md#howto-tables-getidentity)和[使用適用於 Azure Mobile Apps 的 .NET 後端伺服器 SDK](../app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#user-info)。

## <a name="retrieve-tokens-in-app-code"></a>在應用程式程式碼中取出權杖

提供者專屬權杖會從伺服器程式碼插入到要求標頭，以供您輕鬆地存取。 下表顯示可能的權杖標頭名稱：

| | |
|-|-|
| Azure Active Directory | `X-MS-TOKEN-AAD-ID-TOKEN` <br/> `X-MS-TOKEN-AAD-ACCESS-TOKEN` <br/> `X-MS-TOKEN-AAD-EXPIRES-ON`  <br/> `X-MS-TOKEN-AAD-REFRESH-TOKEN` |
| Facebook 權杖 | `X-MS-TOKEN-FACEBOOK-ACCESS-TOKEN` <br/> `X-MS-TOKEN-FACEBOOK-EXPIRES-ON` |
| Google | `X-MS-TOKEN-GOOGLE-ID-TOKEN` <br/> `X-MS-TOKEN-GOOGLE-ACCESS-TOKEN` <br/> `X-MS-TOKEN-GOOGLE-EXPIRES-ON` <br/> `X-MS-TOKEN-GOOGLE-REFRESH-TOKEN` |
| Microsoft 帳戶 | `X-MS-TOKEN-MICROSOFTACCOUNT-ACCESS-TOKEN` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-EXPIRES-ON` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-AUTHENTICATION-TOKEN` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-REFRESH-TOKEN` |
| Twitter | `X-MS-TOKEN-TWITTER-ACCESS-TOKEN` <br/> `X-MS-TOKEN-TWITTER-ACCESS-TOKEN-SECRET` |
|||

從用戶端程式碼 (例如行動應用程式或瀏覽器內的 JavaScript) 將 HTTP `GET` 要求傳送至 `/.auth/me`。 傳回的 JSON 具有提供者專屬權杖。

> [!NOTE]
> 存取權杖是用於存取提供者資源，因此僅在您使用用戶端密碼設定您的提供者時，這些權杖才會出現。 若要了解如何取得重新整理權杖，請參閱[重新整理存取權杖](#refresh-access-tokens)。

## <a name="refresh-access-tokens"></a>重新整理存取權杖

當提供者的存取權杖到期時，您必須重新驗證使用者。 您可以向應用程式的 `/.auth/refresh` 端點發出 `GET` 呼叫，以避免權杖到期。 一經呼叫，App Service 就會自動重新整理已驗證使用者在權杖放區中的存取權杖。 您應用程式程式碼的後續權杖要求會取得重新整理過的權杖。 不過，若要進行權杖重新整理，權杖存放區必須包含您提供者的[重新整理權杖](https://auth0.com/learn/refresh-tokens/)。 每個提供者會記載其重新整理權杖的取得方法，而下列清單僅為簡短摘要：

- **Google**：將 `access_type=offline` 查詢字串參數附加至您的 `/.auth/login/google` API 呼叫。 如果是使用 Mobile Apps SDK，您可以將參數新增至其中一個 `LogicAsync` 多載 (請參閱 [Google 重新整理權杖](https://developers.google.com/identity/protocols/OpenIDConnect#refresh-tokens))。
- **Facebook**：不提供重新整理權杖。 長時間執行的權杖會在 60 天內到期 (請參閱 [Facebook 到期和存取權杖的擴充功能](https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension))。
- **Twitter**：存取權杖不會到期 (請參閱 [Twitter OAuth 常見問題集](https://developer.twitter.com/en/docs/basics/authentication/guides/oauth-faq))。
- **Microsoft 帳戶**：當您[設定 Microsoft 帳戶驗證設定](app-service-mobile-how-to-configure-microsoft-authentication.md)時，請選取 `wl.offline_access` 範圍。
- **Azure Active Directory**：在 [https://resources.azure.com](https://resources.azure.com) 中，執行下列步驟：
    1. 在頁面的頂端，選取 [讀取/寫入]。
    1. 在左側瀏覽器中，巡覽至 [訂用帳戶] > **_\<subscription\_name_** > **resourceGroups** > _**\<resource\_group\_name>**_ > **providers** > **Microsoft.Web** > **sites** > _**\<app\_name>**_ > **config** > **authsettings**。 
    1. 按一下 [編輯]。
    1. 修改下列屬性。 將 _\<app\_id>_ 取代為所要存取服務的 Azure Active Directory 應用程式識別碼。

        ```json
        "additionalLoginParams": ["response_type=code id_token", "resource=<app_id>"]
        ```

    1. 按一下 [放置]。 

設定好提供者之後，您可以在權杖存放區中[尋找重新整理權杖和存取權杖的到期時間](#retrieve-tokens-in-app-code)。 

若要隨時重新整理您的存取權杖，只需要以任何語言呼叫 `/.auth/refresh`。 下列程式碼片段會使用 jQuery 來重新整理 JavaScript 用戶端的存取權杖。

```JavaScript
function refreshTokens() {
  var refreshUrl = "/.auth/refresh";
  $.ajax(refreshUrl) .done(function() {
    console.log("Token refresh completed successfully.");
  }) .fail(function() {
    console.log("Token refresh failed. See application logs for details.");
  });
}
```

如果使用者將授予您應用程式的權限撤銷，您對 `/.auth/me` 的呼叫可能會失敗，並出現 `403 Forbidden` 回應。 若要診斷錯誤，請檢查您的應用程式記錄以取得詳細資訊。

## <a name="extend-session-expiration-grace-period"></a>延長工作階段到期寬限期

已驗證的工作階段到期之後，依預設會有 72 小時的寬限期。 在此寬限期內，您可以使用 App Service 重新整理工作階段 Cookie 或工作階段權杖，而無須重新驗證使用者。 當您的工作階段 Cookie 或工作階段權杖失效時，您只需要呼叫 `/.auth/refresh`，而不必自行追蹤權杖到期日。 一旦 72 小時寬限期結束後，使用者必須重新登入，才能取得有效的工作階段 Cookie 或工作階段權杖。

如果 72 小時的時間不夠您使用，您可以延長此到期時間範圍。 將到期日延長超過一段很長的期間可能會造成重大的安全性影響 (例如，當驗證權杖外洩或遭竊時)。 因此，請將它保留為預設的 72 小時，或將延長期間設為最小值。

若要延長預設的到期時間範圍，請在 [Cloud Shell](../cloud-shell/overview.md) 中執行下列命令。

```azurecli-interactive
az webapp auth update --resource-group <group_name> --name <app_name> --token-refresh-extension-hours <hours>
```

> [!NOTE]
> 寬限期僅適用於 App Service 的已驗證工作階段，而不適用於識別提供者的權杖。 已過期的提供者權杖沒有任何寬限期。 
>

## <a name="limit-the-domain-of-sign-in-accounts"></a>限制登入帳戶的網域

Microsoft 帳戶和 Azure Active Directory 都可讓您從多個網域登入。 例如，Microsoft 帳戶允許 _outlook.com_、_live.com_ 和 _hotmail.com_ 帳戶。 Azure Active Directory 對於登入帳戶可允許任何數目的自訂網域。 這種行為可能不適合內部應用程式，因為您不會想讓具有 outlook.com 帳戶的任何使用者存取內部應用程式。 若要限制登入帳戶的網域名稱，請遵循下列步驟。

在 [https://resources.azure.com](https://resources.azure.com) 中，巡覽至 [訂用帳戶] > **_\<subscription\_name_** > **resourceGroups** > _**\<resource\_group\_name>**_ > **providers** > **Microsoft.Web** > **sites** > _**\<app\_name>**_ > **config** > **authsettings**。 

按一下 [編輯]、修改下列屬性，然後按一下 [放置]。 請務必將 _\<domain\_name>_ 取代為您需要的網域。

```json
"additionalLoginParams": ["domain_hint=<domain_name>"]
```
## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [教學課程：端對端驗證和授權使用者 (Windows)](app-service-web-tutorial-auth-aad.md)
> [教學課程：端對端驗證和授權使用者 (Linux)](containers/tutorial-auth-aad.md)