---
title: 例外狀況管理 - Microsoft 威脅模型化工具 - Azure | Microsoft Docs
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
ms.openlocfilehash: ce748be7f11d440e656e4af5cdd3cee3bbc9e313
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43302144"
---
# <a name="security-frame-exception-management--mitigations"></a>安全性架構︰例外狀況管理 | 風險降低 
| 產品/服務 | 文章 |
| --------------- | ------- |
| **WCF** | <ul><li>[WCF - 請勿在組態檔中包含 serviceDebug 節點](#servicedebug)</li><li>[WCF - 請勿在組態檔中包含 serviceMetadata 節點](#servicemetadata)</li></ul> |
| **Web API** | <ul><li>[確定有在 ASP.NET Web API 中正確處理例外狀況](#exception)</li></ul> |
| **Web 應用程式** | <ul><li>[請勿在錯誤訊息中公開安全性詳細資料](#messages)</li><li>[實作預設的錯誤處理頁面](#default)</li><li>[設定要在 IIS 中零售的部署方法](#deployment)</li><li>[例外狀況應安全地失敗](#fail)</li></ul> |

## <a id="servicedebug"></a>WCF - 請勿在組態檔中包含 serviceDebug 節點

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型、NET Framework 3 |
| **屬性**              | N/A  |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_debug_information) |
| **步驟** | Windows Communication Framework (WCF) 服務可以設定為公開偵錯資訊。 偵錯資訊不應用於生產環境。 `<serviceDebug>` 標籤可定義是否啟用 WCF 服務的偵錯資訊功能。 如果屬性 includeExceptionDetailInFaults 設為 true，來自應用程式的例外狀況資訊會傳回給用戶端。 攻擊者可以利用他們從偵錯輸出取得的其他資訊，裝載以架構、資料庫或應用程式所用其他資源做為目標的攻擊。 |

### <a name="example"></a>範例
下列組態檔包含 `<serviceDebug>` 標籤︰ 
```
<configuration> 
<system.serviceModel> 
<behaviors> 
<serviceBehaviors> 
<behavior name=""MyServiceBehavior""> 
<serviceDebug includeExceptionDetailInFaults=""True"" httpHelpPageEnabled=""True""/> 
... 
```
停用服務中的偵錯資訊。 這可藉由從應用程式組態檔移除 `<serviceDebug>` 標籤來完成。 

## <a id="servicemetadata"></a>WCF - 請勿在組態檔中包含 serviceMetadata 節點

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | WCF | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | 泛型、NET Framework 3 |
| **參考**              | [MSDN](https://msdn.microsoft.com/library/ff648500.aspx)、[Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_service_enumeration) |
| **步驟** | 對外公開的服務相關資訊可提供重要見解給攻擊者，讓其了解如何利用服務。 `<serviceMetadata>` 標籤會啟用中繼資料發佈功能。 服務中繼資料可能包含不應開放存取的敏感性資訊。 至少，請只允許受信任使用者存取中繼資料，並確定不會公開不必要的資訊。 最好是完全停用發佈中繼資料的功能。 安全的 WCF 組態不會包含 `<serviceMetadata>` 標籤。 |

## <a id="exception"></a>確定有在 ASP.NET Web API 中正確處理例外狀況

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web API | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | MVC 5、MVC 6 |
| **屬性**              | N/A  |
| **參考**              | [ASP.NET Web API 中的例外狀況處理](http://www.asp.net/web-api/overview/error-handling/exception-handling)、[ASP.NET Web API 中的模型驗證](http://www.asp.net/web-api/overview/formats-and-model-binding/model-validation-in-aspnet-web-api) |
| **步驟** | 根據預設，ASP.NET Web API 中大多數未攔截到的例外狀況會轉譯成狀態碼為 `500, Internal Server Error` 的 HTTP 回應|

### <a name="example"></a>範例
若要控制 API 傳回的狀態碼，可如下所示使用 `HttpResponseException`︰ 
```csharp
public Product GetProduct(int id)
{
    Product item = repository.Get(id);
    if (item == null)
    {
        throw new HttpResponseException(HttpStatusCode.NotFound);
    }
    return item;
}
```

### <a name="example"></a>範例
若要進一步控制例外狀況回應，可如下所示使用 `HttpResponseMessage` 類別︰ 
```csharp
public Product GetProduct(int id)
{
    Product item = repository.Get(id);
    if (item == null)
    {
        var resp = new HttpResponseMessage(HttpStatusCode.NotFound)
        {
            Content = new StringContent(string.Format("No product with ID = {0}", id)),
            ReasonPhrase = "Product ID Not Found"
        }
        throw new HttpResponseException(resp);
    }
    return item;
}
```
若要攔截類型不是 `HttpResponseException` 的未處理例外狀況，可使用例外狀況篩選。 例外狀況篩選會實作 `System.Web.Http.Filters.IExceptionFilter` 介面。 撰寫例外狀況篩選的最簡單方式是從 `System.Web.Http.Filters.ExceptionFilterAttribute` 類別衍生並覆寫 OnException 方法。 

### <a name="example"></a>範例
以下是會將 `NotImplementedException` 例外狀況轉換成 HTTP 狀態碼 `501, Not Implemented` 的篩選： 
```csharp
namespace ProductStore.Filters
{
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Web.Http.Filters;

    public class NotImplExceptionFilterAttribute : ExceptionFilterAttribute 
    {
        public override void OnException(HttpActionExecutedContext context)
        {
            if (context.Exception is NotImplementedException)
            {
                context.Response = new HttpResponseMessage(HttpStatusCode.NotImplemented);
            }
        }
    }
}
```

有數種方式可以註冊 Web API 例外狀況篩選︰
- 透過動作
- 透過控制器
- 全域

### <a name="example"></a>範例
若要將篩選套用至特定動作，請在動作中新增篩選來做為屬性︰ 
```csharp
public class ProductsController : ApiController
{
    [NotImplExceptionFilter]
    public Contact GetContact(int id)
    {
        throw new NotImplementedException("This method is not implemented");
    }
}
```
### <a name="example"></a>範例
若要將篩選套用至 `controller` 上的所有動作，請在 `controller` 類別中新增篩選來做為屬性︰ 

```csharp
[NotImplExceptionFilter]
public class ProductsController : ApiController
{
    // ...
}
```

### <a name="example"></a>範例
若要將篩選全域套用至所有 Web API 控制器，請在 `GlobalConfiguration.Configuration.Filters` 集合中新增篩選執行個體。 此集合中的例外狀況篩選會套用至任何 Web API 控制器動作。 
```csharp
GlobalConfiguration.Configuration.Filters.Add(
    new ProductStore.NotImplExceptionFilterAttribute());
```

### <a name="example"></a>範例
若要驗證模型，可將模型狀態傳遞至 CreateErrorResponse 方法，如下所示︰ 
```csharp
public HttpResponseMessage PostProduct(Product item)
{
    if (!ModelState.IsValid)
    {
        return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ModelState);
    }
    // Implementation not shown...
}
```

請參閱 [參考] 區段中的連結，以取得在 ASP.Net Web API 中處理例外狀況和驗證模型的其他詳細資料 

## <a id="messages"></a>請勿在錯誤訊息中公開安全性詳細資料

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | N/A  |
| **步驟** | <p>泛型錯誤訊息會直接提供給使用者，但不會包含敏感性應用程式資料。 敏感性資料的範例包括︰</p><ul><li>伺服器名稱</li><li>連接字串</li><li>使用者名稱</li><li>密碼</li><li>SQL 程序</li><li>動態 SQL 失敗的詳細資料</li><li>堆疊追蹤和程式碼行</li><li>儲存在記憶體中的變數</li><li>磁碟機和資料夾的位置</li><li>應用程式的安裝點</li><li>主機組態設定</li><li>其他內部應用程式詳細資料</li></ul><p>捕捉應用程式中的所有錯誤並提供泛型錯誤訊息，以及啟用 IIS 中的自訂錯誤，將有助於避免資訊外洩。 除了錯誤處理架構外，SQL Server 資料庫和 .NET 例外狀況處理尤其詳盡，非常適合用於在應用程式中分析惡意使用者。 請勿直接顯示衍生自 .NET 例外狀況類別的類別內容，並確定您有適當的例外狀況處理手段，以便不會直接對使用者意外產生非預期的例外狀況。</p><ul><li>直接提供泛型錯誤訊息給使用者，此訊息是從直接在例外狀況/錯誤訊息中找到的特定詳細資料所抽離出來</li><li>請勿直接向使用者顯示 .NET 例外狀況類別的內容</li><li>捕捉所有錯誤訊息，並在情況允許時，透過傳送至應用程式用戶端的泛型錯誤訊息通知使用者</li><li>請勿直接對使用者顯示例外狀況類別的內容，特別是來自 `.ToString()` 的傳回值，或是 Message 或 StackTrace 屬性的值。 安全地記錄這項資訊，並向使用者顯示較無害的訊息</li></ul>|

## <a id="default"></a>實作預設的錯誤處理頁面

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [編輯 ASP.NET 錯誤頁面設定對話方塊](https://technet.microsoft.com/library/dd569096(WS.10).aspx) |
| **步驟** | <p>當 ASP.NET 應用程式失敗並造成 HTTP/1.x 500 內部伺服器錯誤時，或當功能組態 (例如要求篩選) 避免顯示頁面時，就會產生錯誤訊息。 系統管理員可以選擇應用程式應該對用戶端顯示容易理解的訊息、對用戶端顯示詳細的錯誤訊息，或只對本機主機顯示詳細的錯誤訊息。 web.config 中的 <customErrors> 標籤有三種模式︰</p><ul><li>**開啟︰** 指定要啟用自訂錯誤。 如果未指定任何 defaultRedirect 屬性，則使用者會看到泛型錯誤。 自訂錯誤會顯示給遠端用戶端和本機主機</li><li>**關閉︰** 指定要停用自訂錯誤。 詳細的 ASP.NET 錯誤會顯示給遠端用戶端和本機主機</li><li>**RemoteOnly：** 指定自訂錯誤只顯示給遠端用戶端，而將 ASP.NET 錯誤顯示給本機主機。 這是預設值</li></ul><p>開啟應用程式/網站的 `web.config` 檔案，並確定標籤已定義 `<customErrors mode="RemoteOnly" />` 或 `<customErrors mode="On" />`。</p>|

## <a id="deployment"></a>設定要在 IIS 中零售的部署方法

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 部署 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [部署元素 (ASP.NET 設定結構描述)](https://msdn.microsoft.com/library/ms228298(VS.80).aspx) |
| **步驟** | <p>`<deployment retail>` 參數是要供生產 IIS 伺服器使用。 這個參數可用來協助應用程式以最佳效能和最低安全性資訊外洩可能性來執行，方法是停用應用程式對頁面產生追蹤輸出的能力、停用對使用者顯示詳細錯誤訊息的能力，以及停用偵錯參數。</p><p>在開發期間，經常會啟用專供開發人員使用的參數和選項，例如失敗要求追蹤和偵錯。 建議您將任何生產伺服器上的部署方法設為零售。 開啟 machine.config 檔，並確定 `<deployment retail="true" />` 維持設定為 true。</p>|

## <a id="fail"></a>例外狀況應安全地失敗

| 標題                   | 詳細資料      |
| ----------------------- | ------------ |
| **元件**               | Web 應用程式 | 
| **SDL 階段**               | 建置 |  
| **適用的技術** | 泛型 |
| **屬性**              | N/A  |
| **參考**              | [安全地失敗](https://www.owasp.org/index.php/Fail_securely) |
| **步驟** | 應用程式應安全地失敗。 任何會傳回布林值的方法 (根據所做出的特定決策) 皆應小心地建立例外狀況區塊。 若草率撰寫例外狀況區塊，會因為不知不覺潛入的安全性問題而產生許多邏輯錯誤。|

### <a name="example"></a>範例
```csharp
        public static bool ValidateDomain(string pathToValidate, Uri currentUrl)
        {
            try
            {
                if (!string.IsNullOrWhiteSpace(pathToValidate))
                {
                    var domain = RetrieveDomain(currentUrl);
                    var replyPath = new Uri(pathToValidate);
                    var replyDomain = RetrieveDomain(replyPath);

                    if (string.Compare(domain, replyDomain, StringComparison.OrdinalIgnoreCase) != 0)
                    {
                        //// Adding additional check to enable CMS urls if they are not hosted on same domain.
                        if (!string.IsNullOrWhiteSpace(Utilities.CmsBase))
                        {
                            var cmsDomain = RetrieveDomain(new Uri(Utilities.Base.Trim()));
                            if (string.Compare(cmDomain, replyDomain, StringComparison.OrdinalIgnoreCase) != 0)
                            {
                                return false;
                            }
                            else
                            {
                                return true;
                            }
                        }

                        return false;
                    }
                }

                return true;
            }
            catch (UriFormatException ex)
            {
                LogHelper.LogException("Utilities:ValidateDomain", ex);
                return true;
            }
        }
```
如果發生某些例外狀況，上述方法一律會傳回 True。 如果使用者提供格式不正確的 URL，瀏覽器加以採用但 `Uri()` 建構函式未採用，就會擲回例外狀況，並將受害者導向有效但格式不正確的 URL。 