---
title: 工作階段管理 - Microsoft 威脅模型化工具 - Azure | Microsoft Docs
description: 降低威脅模型化工具所暴露的威脅
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.openlocfilehash: 97953779f1132d89c7ad07abdb4e08c0f476f4b9
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43841808"
---
# <a name="security-frame-session-management"></a>安全性架構︰工作階段管理
| 產品/服務 | 文章 |
| --------------- | ------- |
| **Azure AD**    | <ul><li>[在使用 Azure AD 時以 ADAL 方法實作適當的登出](#logout-adal)</li></ul> |
| IoT 裝置 | <ul><li>[為產生的 SaS 權杖使用有限的存留期](#finite-tokens)</li></ul> |
| **Azure Document DB** | <ul><li>[為產生的資源權杖使用最短的權杖存留期](#resource-tokens)</li></ul> |
| **ADFS** | <ul><li>[在使用 ADFS 時以 WsFederation 方法實作適當的登出](#wsfederation-logout)</li></ul> |
| **Identity Server** | <ul><li>[在使用 Identity Server 時實作適當的登出](#proper-logout)</li></ul> |
| **Web 應用程式** | <ul><li>[透過 HTTPS 使用的應用程式必須使用安全 Cookie](#https-secure-cookies)</li><li>[所有 http 型應用程式只應在定義 Cookie 時指定 http](#cookie-definition)</li><li>[避免 ASP.NET 網頁上發生跨網站偽造要求 (CSRF) 攻擊](#csrf-asp)</li><li>[設定工作階段的閒置存留期](#inactivity-lifetime)</li><li>[從應用程式實作適當登出](#proper-app-logout)</li></ul> |
| **Web API** | <ul><li>[避免 ASP.NET Web API 上發生跨網站偽造要求 (CSRF) 攻擊](#csrf-api)</li></ul> |

## <a id="logout-adal"></a>在使用 Azure AD 時以 ADAL 方法實作適當的登出

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure AD | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 如果應用程式依賴 Azure AD 所發出的存取權杖，則應呼叫登出事件處理常式 |

### <a name="example"></a>範例
```csharp
HttpContext.GetOwinContext().Authentication.SignOut(OpenIdConnectAuthenticationDefaults.AuthenticationType, CookieAuthenticationDefaults.AuthenticationType)
```

### <a name="example"></a>範例
它應該也會藉由呼叫 Session.Abandon() 方法終結使用者的工作階段。 下列方法顯示使用者登出的安全實作︰
```csharp
    [HttpPost]
        [ValidateAntiForgeryToken]
        public void LogOff()
        {
            string userObjectID = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
            AuthenticationContext authContext = new AuthenticationContext(Authority + TenantId, new NaiveSessionCache(userObjectID));
            authContext.TokenCache.Clear();
            Session.Clear();
            Session.Abandon();
            Response.SetCookie(new HttpCookie("ASP.NET_SessionId", string.Empty));
            HttpContext.GetOwinContext().Authentication.SignOut(
                OpenIdConnectAuthenticationDefaults.AuthenticationType,
                CookieAuthenticationDefaults.AuthenticationType);
        } 
```

## <a id="finite-tokens"></a>為產生的 SaS 權杖使用有限的存留期

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 裝置 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 為了向 Azure IoT 中樞驗證而產生的 SaS 權杖應擁有有限的到期期限。 保持最短的 SaS 權杖存留期可限制權杖萬一外洩時可供重新執行的時間。|

## <a id="resource-tokens"></a>為產生的資源權杖使用最短的權杖存留期

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure Document DB | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 將資源權杖的時間範圍減少為所需的最小值。 資源權杖有 1 小時的預設有效時間範圍。|

## <a id="wsfederation-logout"></a>在使用 ADFS 時以 WsFederation 方法實作適當的登出

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | ADFS | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 如果應用程式依賴 ADFS 所發出的 STS 權杖，登出事件處理常式應該呼叫 WSFederationAuthenticationModule.FederatedSignOut() 方法來登出使用者。 此外也該終結目前的工作階段，而且應重設工作階段權杖值並讓其變成空值。|

### <a name="example"></a>範例
```csharp
        [HttpPost, ValidateAntiForgeryToken]
        [Authorization]
        public ActionResult SignOut(string redirectUrl)
        {
            if (!this.User.Identity.IsAuthenticated)
            {
                return this.View("LogOff", null);
            }

            // Removes the user profile.
            this.Session.Clear();
            this.Session.Abandon();
            HttpContext.Current.Response.Cookies.Add(new System.Web.HttpCookie("ASP.NET_SessionId", string.Empty)
                {
                    Expires = DateTime.Now.AddDays(-1D),
                    Secure = true,
                    HttpOnly = true
                });

            // Signs out at the specified security token service (STS) by using the WS-Federation protocol.
            Uri signOutUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Issuer);
            Uri replyUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Realm);
            if (!string.IsNullOrEmpty(redirectUrl))
            {
                replyUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Realm + redirectUrl);
            }
           //     Signs out of the current session and raises the appropriate events.
            var authModule = FederatedAuthentication.WSFederationAuthenticationModule;
            authModule.SignOut(false);
        //     Signs out at the specified security token service (STS) by using the WS-Federation
        //     protocol.            
            WSFederationAuthenticationModule.FederatedSignOut(signOutUrl, replyUrl);
            return new RedirectResult(redirectUrl);
        }
```

## <a id="proper-logout"></a>在使用 Identity Server 時實作適當的登出

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Identity Server | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [IdentityServer3-同盟登出](https://identityserver.github.io/Documentation/docsv2/advanced/federated-signout.html) |
| **步驟** | IdentityServer 支援與外部識別提供者同盟的能力。 當使用者登出上游識別提供者時，視使用的通訊協定而定，其可能會在使用者登出時收到通知。這會讓 IdentityServer 通知其用戶端，以便用戶端也可以將使用者登出。請參閱 [參考] 區段中的文件，以取得實作詳細資料。|

## <a id="https-secure-cookies"></a>透過 HTTPS 使用的應用程式必須使用安全 Cookie

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | EnvironmentType - OnPrem |
| **參考**              | [httpCookies 元素 (ASP.NET 設定結構描述)](http://msdn.microsoft.com/library/ms228262(v=vs.100).aspx)、[HttpCookie.Secure 屬性](http://msdn.microsoft.com/library/system.web.httpcookie.secure.aspx) |
| **步驟** | Cookie 通常只能供其範圍所在的網域存取。 不幸的是，「網域」的定義不包含通訊協定，因此透過 HTTPS 所建立的 Cookie 可透過 HTTP 來存取。 「安全」的屬性會對瀏覽器指出 Cookie 只應透過 HTTPS 提供使用。 請確定所有透過 HTTPS 設定的 Cookie 會使用**安全**的屬性。 藉由將 requireSSL 屬性設為 true 可在 web.config 檔案中強制執行這項需求。 這是偏好的方法，因為它會強制對所有目前和未來的 Cookie 執行**安全**的屬性，而不需要進行任何額外的程式碼變更。|

### <a name="example"></a>範例
```csharp
<configuration>
  <system.web>
    <httpCookies requireSSL="true"/>
  </system.web>
</configuration>
```
即使使用 HTTP 來存取應用程式，仍會強制執行這項設定。 如果使用 HTTP 來存取應用程式，這項設定會中斷應用程式，因為 Cookie 會設定使用安全的屬性，而瀏覽器不會將其傳回給應用程式。

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | Web Form、MVC5 |
| **屬性**              | EnvironmentType - OnPrem |
| **參考**              | N/A  |
| **步驟** | 當 Web 應用程式是信賴憑證者，且 IdP 是 ADFS 伺服器時，您可以設定 FedAuth 權杖的安全屬性，方法是在 web.config 的 `system.identityModel.services` 區段中將 requireSSL 設定為 True︰|

### <a name="example"></a>範例
```csharp
  <system.identityModel.services>
    <federationConfiguration>
      <!-- Set requireSsl=true; domain=application domain name used by FedAuth cookies (Ex: .gdinfra.com); -->
      <cookieHandler requireSsl="true" persistentSessionLifetime="0.0:20:0" />
    ....  
    </federationConfiguration>
  </system.identityModel.services>
```

## <a id="cookie-definition"></a>所有 http 型應用程式只應在定義 Cookie 時指定 http

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [安全 Cookie 屬性](https://en.wikipedia.org/wiki/HTTP_cookie#Secure_cookie) |
| **步驟** | 為了降低因跨網站指令碼 (XSS) 攻擊而洩漏資訊的風險，我們在 Cookie 中引進了新的屬性 httpOnly，並已受到所有主要瀏覽器的支援。 這個屬性會指定不可透過指令碼存取的 Cookie。 藉由使用 HttpOnly Cookie，Web 應用程式會降低 Cookie 中所含機密資訊可透過指令碼竊取並傳送至攻擊者網站的可能性。 |

### <a name="example"></a>範例
所有使用 Cookie 的 HTTP 型應用程式應該在 Cookie 定義中指定 HttpOnly，方法是在 web.config 中實作下列組態︰
```XML
<system.web>
.
.
   <httpCookies requireSSL="false" httpOnlyCookies="true"/>
.
.
</system.web>
```

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | Web Form |
| **屬性**              | N/A  |
| **參考**              | [FormsAuthentication.RequireSSL 屬性](https://msdn.microsoft.com/library/system.web.security.formsauthentication.requiressl.aspx) |
| **步驟** | RequireSSL 屬性值是使用組態元素的 requireSSL 屬性來設定於 ASP.NET 應用程式的組態檔中。 您可以藉由設定 requireSSL 屬性，在 ASP.NET 應用程式的 Web.config 檔案中指定 SSL (安全通訊端層) 是否需要將表單驗證 Cookie 傳回給伺服器。|

### <a name="example"></a>範例 
下列程式碼範例會在 Web.config 檔案中設定 requireSSL 屬性。
```XML
<authentication mode="Forms">
  <forms loginUrl="member_login.aspx" cookieless="UseCookies" requireSSL="true"/>
</authentication>
```

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC5 |
| **屬性**              | EnvironmentType - OnPrem |
| **參考**              | [Windows Identity Foundation (WIF) 組態 - 單元二](https://blogs.msdn.microsoft.com/alikl/2011/02/01/windows-identity-foundation-wif-configuration-part-ii-cookiehandler-chunkedcookiehandler-customcookiehandler/) |
| **步驟** | 若要設定 FedAuth Cookie 的 httpOnly 屬性，hideFromCsript 屬性值應該設定為 True。 |

### <a name="example"></a>範例
下列設定顯示正確的設定︰
```XML
<federatedAuthentication>
<cookieHandler mode="Custom"
                       hideFromScript="true"
                       name="FedAuth"
                       path="/"
                       requireSsl="true"
                       persistentSessionLifetime="25">
</cookieHandler>
</federatedAuthentication>
```

## <a id="csrf-asp"></a>避免 ASP.NET 網頁上發生跨網站偽造要求 (CSRF) 攻擊

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 跨網站偽造要求 (CSRF 或 XSRF) 是一種攻擊，攻擊者會在不同使用者對某個網站所建立之工作階段的安全性內容中執行動作。 目標是要修改或刪除內容，如果目標網站完全依賴工作階段 Cookie 來驗證所收到要求的話。 攻擊者可從使用者已登入的有弱點網站，使用命令讓不同使用者的瀏覽器載入 URL，以利用這項弱點。 攻擊者有許多方法可以這麼做，例如裝載不同的網站以從有弱點的伺服器載入資源，或讓使用者按一下連結。 如果伺服器傳送額外的權杖給用戶端，要求用戶端在所有未來的要求中包含該權杖，並確認所有未來的要求皆包含屬於目前工作階段的權杖，例如使用 ASP.NET AntiForgeryToken 或 ViewState，便可防止攻擊。 |

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC5、MVC6 |
| **屬性**              | N/A  |
| **參考**              | [在 ASP.NET MVC 和網頁中防止 XSRF/CSRF](http://www.asp.net/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages) |
| **步驟** | 防 CSRF 和 ASP.NET MVC 表單 - 在檢視上使用 `AntiForgeryToken` 協助程式方法；例如，在表單內放入 `Html.AntiForgeryToken()`。|

### <a name="example"></a>範例
```csharp
@using (Html.BeginForm("UserProfile", "SubmitUpdate")) { 
    @Html.ValidationSummary(true) 
    @Html.AntiForgeryToken()
    <fieldset> 
```

### <a name="example"></a>範例
```csharp
<form action="/UserProfile/SubmitUpdate" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="saTFWpkKN0BYazFtN6c4YbZAmsEwG0srqlUqqloi/fVgeV2ciIFVmelvzwRZpArs" />
    <!-- rest of form goes here -->
</form>
```

### <a name="example"></a>範例
在此同時，Html.AntiForgeryToken() 會給予訪客名為 __RequestVerificationToken 的 Cookie，其值和上面顯示的隨機隱藏值相同。 接下來，若要驗證連入的表單張貼，請將 [ValidateAntiForgeryToken] 篩選新增至目標動作方法。 例如︰
```
[ValidateAntiForgeryToken]
public ViewResult SubmitUpdate()
{
// ... etc.
}
```
授權篩選會確認︰
* 連入要求具有名為 __RequestVerificationToken 的 Cookie
* 連入要求具有名為 __RequestVerificationToken 的 `Request.Form` 項目
* 這些 Cookie 和 `Request.Form` 值符合假設一切正常，要求就會正常通過。 但如果沒有，則授權會失敗並出現訊息「未提供必要的防偽權杖或權杖無效」。 

### <a name="example"></a>範例
防 CSRF 和 AJAX︰表單權杖可能會成為 AJAX 要求的麻煩，因為 AJAX 要求可能會傳送 JSON 資料，而非 HTML 表單資料。 有一個解決方案是在自訂 HTTP 標頭中傳送權杖。 下列程式碼使用 Razor 語法來產生權杖，然後將權杖新增至 AJAX 要求。 
```csharp
<script>
    @functions{
        public string TokenHeaderValue()
        {
            string cookieToken, formToken;
            AntiForgery.GetTokens(null, out cookieToken, out formToken);
            return cookieToken + ":" + formToken;                
        }
    }

    $.ajax("api/values", {
        type: "post",
        contentType: "application/json",
        data: {  }, // JSON data goes here
        dataType: "json",
        headers: {
            'RequestVerificationToken': '@TokenHeaderValue()'
        }
    });
</script>
```

### <a name="example"></a>範例
當您處理要求時，請從要求標頭擷取權杖。 然後呼叫 AntiForgery.Validate 方法來驗證權杖。 如果權杖無效，Validate 方法會擲回例外狀況。
```csharp
void ValidateRequestHeader(HttpRequestMessage request)
{
    string cookieToken = "";
    string formToken = "";

    IEnumerable<string> tokenHeaders;
    if (request.Headers.TryGetValues("RequestVerificationToken", out tokenHeaders))
    {
        string[] tokens = tokenHeaders.First().Split(':');
        if (tokens.Length == 2)
        {
            cookieToken = tokens[0].Trim();
            formToken = tokens[1].Trim();
        }
    }
    AntiForgery.Validate(cookieToken, formToken);
}
```

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | Web Form |
| **屬性**              | N/A  |
| **參考**              | [利用 ASP.NET 內建功能來避開 Web 攻擊](https://msdn.microsoft.com/library/ms972969.aspx#securitybarriers_topic2) |
| **步驟** | 您可以降低 WebForm 型應用程式中的 CSRF 攻擊風險，方法是將 ViewStateUserKey 設為會隨每個使用者而不同的隨機字串，使用者識別碼、或者最好是工作階段識別碼。 基於諸多技術和社交方面的原因，工作階段識別碼會比較適合，因為工作階段識別碼無法預料、會逾時，而且會隨每個使用者而有所不同。|

### <a name="example"></a>範例
您的所有網頁中必須擁有的程式碼如下︰
```csharp
void Page_Init (object sender, EventArgs e) {
   ViewStateUserKey = Session.SessionID;
   :
}
```

## <a id="inactivity-lifetime"></a>設定工作階段的閒置存留期

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [HttpSessionState.Timeout 屬性](https://msdn.microsoft.com/library/system.web.sessionstate.httpsessionstate.timeout(v=vs.110).aspx) |
| **步驟** | 工作階段逾時代表使用者在一段間隔時間內 (由網頁伺服器定義) 未在網站上執行任何動作時所發生的事件。 伺服器端上的事件會將使用者工作階段的狀態變更為「無效」(例如「不再被使用」)，並指示網頁伺服器終結它 (刪除其中包含的所有資料)。 下列程式碼範例會在 Web.config 檔案中將逾時工作階段屬性設定為 15 分鐘。|

### <a name="example"></a>範例
```XML 
<configuration>
  <system.web>
    <sessionState mode="InProc" cookieless="true" timeout="15" />
  </system.web>
</configuration>
```

## <a id="threat-detection"></a>在 Azure SQL 上啟用威脅偵測功能
```

| Title                   | Details      |
| ----------------------- | ------------ |
| **Component**               | Web Application | 
| **SDL Phase**               | Build |  
| **Applicable Technologies** | Web Forms |
| **Attributes**              | N/A  |
| **References**              | [forms Element for authentication (ASP.NET Settings Schema)](https://msdn.microsoft.com/library/1d3t3c61(v=vs.100).aspx) |
| **Steps** | Set the Forms Authentication Ticket cookie timeout to 15 minutes|

### Example
```XML
<forms  name=".ASPXAUTH" loginUrl="login.aspx"  defaultUrl="default.aspx" protection="All" timeout="15" path="/" requireSSL="true" slidingExpiration="true"/>
</forms>
```

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | Web Form、MVC5 |
| **屬性**              | EnvironmentType - OnPrem |
| **參考**              | [asdeqa](https://skf.azurewebsites.net/Mitigations/Details/wefr) |
| **步驟** | 當 Web 應用程式是信賴憑證者，且 ADFS 是 STS 時，驗證 Cookie (FedAuth 權杖) 的存留期即可由 web.config 中的下列組態設定：|

### <a name="example"></a>範例
```XML
  <system.identityModel.services>
    <federationConfiguration>
      <!-- Set requireSsl=true; domain=application domain name used by FedAuth cookies (Ex: .gdinfra.com); -->
      <cookieHandler requireSsl="true" persistentSessionLifetime="0.0:15:0" />
      <!-- Set requireHttps=true; -->
      <wsFederation passiveRedirectEnabled="true" issuer="http://localhost:39529/" realm="https://localhost:44302/" reply="https://localhost:44302/" requireHttps="true"/>
      <!--
      Use the code below to enable encryption-decryption of claims received from ADFS. Thumbprint value varies based on the certificate being used.
      <serviceCertificate>
        <certificateReference findValue="4FBBBA33A1D11A9022A5BF3492FF83320007686A" storeLocation="LocalMachine" storeName="My" x509FindType="FindByThumbprint" />
      </serviceCertificate>
      -->
    </federationConfiguration>
  </system.identityModel.services>
```

### <a name="example"></a>範例
此外，ADFS 所發出之 SAML 宣告權杖的存留期應設為 15 分鐘，方法是在 ADFS 伺服器上執行下列 PowerShell 命令︰
```csharp
Set-ADFSRelyingPartyTrust -TargetName “<RelyingPartyWebApp>” -ClaimsProviderName @(“Active Directory”) -TokenLifetime 15 -AlwaysRequireAuthentication $true
```

## <a id="proper-app-logout"></a>從應用程式實作適當登出

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 當使用者按下 [登出] 按鈕時從應用程式執行正確的登出。 在登出時，應用程式應終結使用者的工作階段，同時重設工作階段 Cookie 值並讓其變成空值，以及重設驗證 cookie 值並讓其變成空值。 此外，當多個工作階段繫結至單一使用者身分識別時，則必須在逾時或登出時於伺服器端將其一起終止。 最後，請確定每個頁面都可使用登出功能。 |

## <a id="csrf-api"></a>避免 ASP.NET Web API 上發生跨網站偽造要求 (CSRF) 攻擊

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 跨網站偽造要求 (CSRF 或 XSRF) 是一種攻擊，攻擊者會在不同使用者對某個網站所建立之工作階段的安全性內容中執行動作。 目標是要修改或刪除內容，如果目標網站完全依賴工作階段 Cookie 來驗證所收到要求的話。 攻擊者可從使用者已登入的有弱點網站，使用命令讓不同使用者的瀏覽器載入 URL，以利用這項弱點。 攻擊者有許多方法可以這麼做，例如裝載不同的網站以從有弱點的伺服器載入資源，或讓使用者按一下連結。 如果伺服器傳送額外的權杖給用戶端，要求用戶端在所有未來的要求中包含該權杖，並確認所有未來的要求皆包含屬於目前工作階段的權杖，例如使用 ASP.NET AntiForgeryToken 或 ViewState，便可防止攻擊。 |

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC5、MVC6 |
| **屬性**              | N/A  |
| **參考**              | [避免 ASP.NET Web API 中發生跨網站偽造要求 (CSRF) 攻擊](http://www.asp.net/web-api/overview/security/preventing-cross-site-request-forgery-csrf-attacks) |
| **步驟** | 防 CSRF 和 AJAX︰表單權杖可能會成為 AJAX 要求的麻煩，因為 AJAX 要求可能會傳送 JSON 資料，而非 HTML 表單資料。 有一個解決方案是在自訂 HTTP 標頭中傳送權杖。 下列程式碼使用 Razor 語法來產生權杖，然後將權杖新增至 AJAX 要求。 |

### <a name="example"></a>範例
```Javascript
<script>
    @functions{
        public string TokenHeaderValue()
        {
            string cookieToken, formToken;
            AntiForgery.GetTokens(null, out cookieToken, out formToken);
            return cookieToken + ":" + formToken;                
        }
    }
    $.ajax("api/values", {
        type: "post",
        contentType: "application/json",
        data: {  }, // JSON data goes here
        dataType: "json",
        headers: {
            'RequestVerificationToken': '@TokenHeaderValue()'
        }
    });
</script>
```

### <a name="example"></a>範例
當您處理要求時，請從要求標頭擷取權杖。 然後呼叫 AntiForgery.Validate 方法來驗證權杖。 如果權杖無效，Validate 方法會擲回例外狀況。
```csharp
void ValidateRequestHeader(HttpRequestMessage request)
{
    string cookieToken = "";
    string formToken = "";

    IEnumerable<string> tokenHeaders;
    if (request.Headers.TryGetValues("RequestVerificationToken", out tokenHeaders))
    {
        string[] tokens = tokenHeaders.First().Split(':');
        if (tokens.Length == 2)
        {
            cookieToken = tokens[0].Trim();
            formToken = tokens[1].Trim();
        }
    }
    AntiForgery.Validate(cookieToken, formToken);
}
```

### <a name="example"></a>範例
防 CSRF 和 ASP.NET MVC 表單 - 在檢視上使用 AntiForgeryToken 協助程式方法；例如，在表單內放入 Html.AntiForgeryToken()。
```csharp
@using (Html.BeginForm("UserProfile", "SubmitUpdate")) { 
    @Html.ValidationSummary(true) 
    @Html.AntiForgeryToken()
    <fieldset> 
}
```

### <a name="example"></a>範例
上述範例會輸出類似下面的結果：
```csharp
<form action="/UserProfile/SubmitUpdate" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="saTFWpkKN0BYazFtN6c4YbZAmsEwG0srqlUqqloi/fVgeV2ciIFVmelvzwRZpArs" />
    <!-- rest of form goes here -->
</form>
```

### <a name="example"></a>範例
在此同時，Html.AntiForgeryToken() 會給予訪客名為 __RequestVerificationToken 的 Cookie，其值和上面顯示的隨機隱藏值相同。 接下來，若要驗證連入的表單張貼，請將 [ValidateAntiForgeryToken] 篩選新增至目標動作方法。 例如︰
```
[ValidateAntiForgeryToken]
public ViewResult SubmitUpdate()
{
// ... etc.
}
```
授權篩選會確認︰
* 連入要求具有名為 __RequestVerificationToken 的 Cookie
* 連入要求具有名為 __RequestVerificationToken 的 `Request.Form` 項目
* 這些 Cookie 和 `Request.Form` 值符合假設一切正常，要求就會正常通過。 但如果沒有，則授權會失敗並出現訊息「未提供必要的防偽權杖或權杖無效」。

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC5、MVC6 |
| **屬性**              | 識別提供者 - ADFS、識別提供者 - Azure AD |
| **參考**              | [在 ASP.NET Web API 2.2 中使用個別帳戶和本機登入保護 Web API](http://www.asp.net/web-api/overview/security/individual-accounts-in-web-api) |
| **步驟** | 如果使用 OAuth 2.0 保護 Web API，則它會預期授權要求標頭中有持有人權杖，並且只會在權杖有效時將存取權授與要求。 不同於以 Cookie 為基礎的驗證，瀏覽器不會在要求中附加持有人權杖。 提出要求的用戶端必須在要求標頭中明確附加持有人權杖。 因此，對於使用 OAuth 2.0 來保護的 ASP.NET Web API，會將持有人權杖視為 CSRF 攻擊的防禦手段。 請注意，如果應用程式的 MVC 部分使用表單驗證 (亦即，使用 Cookie)，MVC Web 應用程式必須使用防偽權杖。 |

### <a name="example"></a>範例
必須通知 Web API 只依賴持有人權杖，而不要依賴 Cookie。 這可透過 `WebApiConfig.Register` 方法中的下列設定來完成：

```csharp
config.SuppressDefaultHostAuthentication();
config.Filters.Add(new HostAuthenticationFilter(OAuthDefaults.AuthenticationType));
```

SuppressDefaultHostAuthentication 方法會指示 Web API 忽略要求透過 IIS 或 OWIN 中介軟體連線到 Web API 管線前發生的任何驗證。 如此一來，我們可以限制 Web API 只驗證使用持有人權杖的要求。
