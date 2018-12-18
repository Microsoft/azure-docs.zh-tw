---
title: 使用 Azure Application Insights 來診斷 Web 應用程式中的失敗和例外狀況 | Microsoft Docs
description: 擷取從 ASP.NET 應用程式與所要求遙測的例外狀況。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: d1e98390-3ce4-4d04-9351-144314a42aa2
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 09/19/2017
ms.author: mbullwin
ms.openlocfilehash: a3dcf4211df5d40c4b174fd9a818d3268ffaa3a0
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35295705"
---
# <a name="diagnose-exceptions-in-your-web-apps-with-application-insights"></a>使用 Application Insights 在 Web 應用程式中診斷例外狀況
[Application Insights](app-insights-overview.md) 會回報您即時 Web 應用程式中的例外狀況。 您可以在用戶端和伺服器端讓失敗的要求與例外狀況及其他事件相互關聯，以便快速地診斷原因。

## <a name="set-up-exception-reporting"></a>設定例外狀況報告
* 讓伺服器應用程式回報例外狀況︰
  * 在應用程式程式碼中安裝 [Application Insights SDK](app-insights-asp-net.md)，或
  * IIS Web 伺服器：執行 [Application Insights 代理程式](app-insights-monitor-performance-live-website-now.md)；或
  * Azure Web 應用程式：新增 [Application Insights 擴充功能](app-insights-azure-web-apps.md)
  * Java Web 應用程式：安裝 [Java 代理程式](app-insights-java-agent.md)
* 在您的網頁中安裝 [JavaScript 程式碼片段](app-insights-javascript.md)來攔截瀏覽器例外狀況。
* 在某些應用程式架構中或搭配某些設定時，您必須採取一些額外的步驟，才能攔截較多的例外狀況：
  * [Web Form](#web-forms)
  * [MVC](#mvc)
  * [Web API 1.*](#web-api-1x)
  * [Web API 2.*](#web-api-2x)
  * [WCF](#wcf)

## <a name="diagnosing-exceptions-using-visual-studio"></a>使用 Visual Studio 診斷例外狀況
在 Visual Studio 中開啟應用程式解決方案以協助偵錯。

在您的伺服器上或開發機器上使用 F5 執行應用程式。

在 Visual Studio 中開啟 [Application Insights 搜尋] 視窗，並將它設定為顯示您的應用程式的事件。 當您偵錯時，您只要按一下 [Application Insights] 按鈕即可執行此操作。

![以滑鼠右鍵按一下專案，然後選擇 [Application Insights]、[開啟]。](./media/app-insights-asp-net-exceptions/34.png)

請注意，您可以篩選報告僅顯示例外狀況。

*未顯示例外狀況？請參閱[擷取例外狀況](#exceptions)。*

按一下例外狀況報告以顯示其堆疊追蹤。
按一下堆疊追蹤中的行參考，以開啟相關程式碼檔案。  

在程式碼中，注意 CodeLens 會顯示關於例外狀況的資料︰

![CodeLens 的例外狀況通知。](./media/app-insights-asp-net-exceptions/35.png)

## <a name="diagnosing-failures-using-the-azure-portal"></a>使用 Azure 入口網站診斷失敗
Application Insights 隨附策劃的 APM 體驗，可協助您診斷受監視應用程式中的失敗。 若要開始，請按一下位於 [調查] 區段 Application Insights 資源功能表中的 [失敗] 選項。 您應該會看到全螢幕檢視，其中顯示要求的失敗率趨勢、有多少個失敗的要求，以及多少個使用者受到影響。 在右邊，您會看到所選失敗中作業特定的一些最實用分佈，包括前 3 個回應代碼、前 3 個例外狀況類型，以及前 3 個失敗相依性類型。 

![失敗分級檢視 ([作業] 索引標籤)](./media/app-insights-asp-net-exceptions/FailuresTriageView.png)

只要按一下，您即可以檢閱每個作業子集的代表性範例。 特別是，若要診斷例外狀況，您可以按一下要使用 [例外狀況] 詳細資料刀鋒視窗呈現的特定例外狀況的計數：

![例外狀況詳細資料刀鋒視窗](./media/app-insights-asp-net-exceptions/ExceptionDetailsBlade.png)

**或者，** 不要查看特定失敗中作業的例外狀況，您可以透過切換到 [例外狀況] 索引標籤，從例外狀況的整體檢視開始：

![失敗分級檢視 ([例外狀況] 索引標籤)](./media/app-insights-asp-net-exceptions/FailuresTriageView_Exceptions.png)

您可以在此處查看為您的受監視應用程式收集的所有例外狀況。

*未顯示例外狀況？請參閱[擷取例外狀況](#exceptions)。*


## <a name="custom-tracing-and-log-data"></a>自訂追蹤和記錄資料
若要取得您的 app 的特定診斷資料，您可以插入程式碼以傳送您自己的遙測資料。 這會隨著要求、頁面檢視和其他自動收集的資料顯示在診斷搜尋中。

您有幾種選項：

* [TrackEvent()](app-insights-api-custom-events-metrics.md#trackevent) 通常用來監視使用模式，但它傳送的資料也會出現在診斷搜尋的 [自訂事件] 下。 事件具有名稱，並且可以包含字串屬性和數值度量，您可以對其[篩選診斷搜尋](app-insights-diagnostic-search.md)。
* [TrackTrace()](app-insights-api-custom-events-metrics.md#tracktrace) 可讓您傳送較長的資料，例如 POST 資訊。
* [TrackException()](#exceptions) 會傳送堆疊追蹤。 [深入了解例外狀況](#exceptions)。
* 如果您已經使用 Log4Net 或 NLog 等記錄架構，您可以[擷取這些記錄](app-insights-asp-net-trace-logs.md)，然後在診斷搜尋中將它們連同要求和例外狀況資料一起檢視。

若要查看這些事件，請開啟[搜尋](app-insights-diagnostic-search.md)、開啟 [篩選]，然後選擇 [自訂事件]、[追蹤] 或 [例外狀況]。

![鑽研](./media/app-insights-asp-net-exceptions/viewCustomEvents.png)

> [!NOTE]
> 如果您的應用程式會產生大量遙測，調適性取樣模型會自動藉由僅傳送事件代表性片段，減少傳送到入口網站的量。 為相同作業之一部分的事件會選取或取消選取為群組，讓您可以在相關事件之間瀏覽。 [了解取樣。](app-insights-sampling.md)
>
>

### <a name="how-to-see-request-post-data"></a>如何查看要求 POST 資料
要求詳細資料不包括在 POST 呼叫中傳送至您的應用程式的資料。 若要報告此資料：

* 在您的應用程式專案中[安裝 SDK](app-insights-asp-net.md)。
* 在您的應用程式中插入程式碼來呼叫 [Microsoft.ApplicationInsights.TrackTrace()](app-insights-api-custom-events-metrics.md#tracktrace)。 在訊息參數中傳送 POST 資料。 允許的大小有限制，所以您應該嘗試只傳送基本的資料。
* 當您調查失敗的要求時，會發現相關聯的追蹤。  

![鑽研](./media/app-insights-asp-net-exceptions/060-req-related.png)

## <a name="exceptions"></a> 擷取例外狀況和相關的診斷資料
一開始，您不會在入口網站看到應用程式中造成失敗的所有例外狀況。 您會看到任何瀏覽器例外狀況 (如果您在網頁中使用 [JavaScript SDK](app-insights-javascript.md))。 但是 IIS 會攔截大部分的伺服器例外狀況，而且您必須撰寫一段程式碼才能查看它們。

您可以：

* **明確記錄例外狀況** ，方法是將程式碼插入例外狀況處理常式中，以報告例外狀況。
* **自動擷取例外狀況** ，方法是設定您的 ASP.NET 架構。 架構類型不同，則必要的新增項目也不同。

## <a name="reporting-exceptions-explicitly"></a>明確報告例外狀況
最簡單的方法是在例外狀況處理常式中插入 TrackException() 的呼叫。

```javascript
    try
    { ...
    }
    catch (ex)
    {
      appInsights.trackException(ex, "handler loc",
        {Game: currentGame.Name,
         State: currentGame.State.ToString()});
    }
```

```csharp
    var telemetry = new TelemetryClient();
    ...
    try
    { ...
    }
    catch (Exception ex)
    {
       // Set up some properties:
       var properties = new Dictionary <string, string>
         {{"Game", currentGame.Name}};

       var measurements = new Dictionary <string, double>
         {{"Users", currentGame.Users.Count}};

       // Send the exception telemetry:
       telemetry.TrackException(ex, properties, measurements);
    }
```

```VB
    Dim telemetry = New TelemetryClient
    ...
    Try
      ...
    Catch ex as Exception
      ' Set up some properties:
      Dim properties = New Dictionary (Of String, String)
      properties.Add("Game", currentGame.Name)

      Dim measurements = New Dictionary (Of String, Double)
      measurements.Add("Users", currentGame.Users.Count)

      ' Send the exception telemetry:
      telemetry.TrackException(ex, properties, measurements)
    End Try
```

屬性和度量參數是選用的，但對於[篩選和新增](app-insights-diagnostic-search.md)額外資訊來說，相當有用。 比方說，如果您有一個應用程式可以執行數個遊戲，則您可以找到與特定遊戲相關的所有例外狀況報告。 您可以將許多項目加入至每個字典，且項目數量不限。

## <a name="browser-exceptions"></a>瀏覽器例外狀況
大部分的瀏覽器例外狀況都會報告。

如果您的網頁包含來自內容傳遞網路或其他網域的指令碼檔案，請確定指令碼標籤具有屬性 ```crossorigin="anonymous"```，而且伺服器會傳送 [CORS 標頭](http://enable-cors.org/)。 這可讓您從這些資源取得未處理 JavaScript 例外狀況的堆疊追蹤和詳細資料。

## <a name="web-forms"></a>Web Form
對於 Web Form，HTTP 模組能夠在未使用 CustomErrors 設定重新導向時收集例外狀況。

但是，如果您有使用中的重新導向，將下列程式碼新增至 Global.asax.cs 中的 Application_Error 函式。 (如果您還沒有檔案，請加入 Global.asax 檔案。)

```csharp
    void Application_Error(object sender, EventArgs e)
    {
      if (HttpContext.Current.IsCustomErrorEnabled && Server.GetLastError  () != null)
      {
         var ai = new TelemetryClient(); // or re-use an existing instance

         ai.TrackException(Server.GetLastError());
      }
    }
```

## <a name="mvc"></a>MVC
從 Application Insights Web SDK 2.6 版 (beta3 和更新版本) 開始，Application Insights 會自動收集在 MVC 5+ 控制器方法中擲回的未處理例外狀況。 如果您先前已新增自訂處理常式來追蹤此類例外狀況 (如下列範例中所述)，您可以將其移除，以避免重複追蹤例外狀況。

有一些無法處理的例外狀況篩選器案例。 例如︰

* 從控制器建構函式擲回的例外狀況。
* 從訊息處理常式擲回的例外狀況。
* 在路由期間擲回的例外狀況。
* 在回應內容序列化期間擲回的例外狀況。
* 在應用程式啟動期間擲回的例外狀況。
* 背景工作中擲回的例外狀況。

由應用程式*處理*的所有例外狀況仍然需要手動追蹤。 源自控制器的未處理例外狀況通常會導致 500「內部伺服器錯誤」回應。 如果此類回應是因為未處理的例外狀況 (或根本沒有例外狀況) 而手動建構的，則對應的要求遙測會加以追蹤並產生 `ResultCode` 500，但 Application Insights SDK 無法追蹤對應的例外狀況。

### <a name="prior-versions-support"></a>舊版支援
如果您使用 Application Insights Web SDK 2.5 (和較早版本) 的 MVC 4 (和較早版本)，請參閱以下範例以追蹤例外狀況。

如果 [CustomErrors](https://msdn.microsoft.com/library/h0hfz6fc.aspx) 組態是 `Off`，則例外狀況將可供 [HTTP 模組](https://msdn.microsoft.com/library/ms178468.aspx)收集。 不過，如果它是 `RemoteOnly` (預設值) 或 `On`，則會清除例外狀況，且不適用於讓 Application Insights 自動收集。 您可以藉由覆寫 [System.Web.Mvc.HandleErrorAttribute class](http://msdn.microsoft.com/library/system.web.mvc.handleerrorattribute.aspx)，並且針對以下不同的 MVC 版本如下所示的套用已覆寫的類別，來進行修正 ([github 來源](https://github.com/AppInsightsSamples/Mvc2UnhandledExceptions/blob/master/MVC2App/Controllers/AiHandleErrorAttribute.cs))：

```csharp
    using System;
    using System.Web.Mvc;
    using Microsoft.ApplicationInsights;

    namespace MVC2App.Controllers
    {
      [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited = true, AllowMultiple = true)]
      public class AiHandleErrorAttribute : HandleErrorAttribute
      {
        public override void OnException(ExceptionContext filterContext)
        {
            if (filterContext != null && filterContext.HttpContext != null && filterContext.Exception != null)
            {
                //If customError is Off, then AI HTTPModule will report the exception
                if (filterContext.HttpContext.IsCustomErrorEnabled)
                {   //or reuse instance (recommended!). see note above  
                    var ai = new TelemetryClient();
                    ai.TrackException(filterContext.Exception);
                }
            }
            base.OnException(filterContext);
        }
      }
    }
```

#### <a name="mvc-2"></a>MVC 2
使用您的控制器中的新屬性取代 HandleError 屬性。

```csharp
    namespace MVC2App.Controllers
    {
       [AiHandleError]
       public class HomeController : Controller
       {
    ...
```

[範例](https://github.com/AppInsightsSamples/Mvc2UnhandledExceptions)

#### <a name="mvc-3"></a>MVC 3
註冊 `AiHandleErrorAttribute` 做為 Global.asax.cs 中的全域篩選器：

```csharp
    public class MyMvcApplication : System.Web.HttpApplication
    {
      public static void RegisterGlobalFilters(GlobalFilterCollection filters)
      {
         filters.Add(new AiHandleErrorAttribute());
      }
     ...
```

[範例](https://github.com/AppInsightsSamples/Mvc3UnhandledExceptionTelemetry)

#### <a name="mvc-4-mvc5"></a>MVC 4、MVC5
註冊 AiHandleErrorAttribute 做為 FilterConfig.cs 中的全域篩選器：

```csharp
    public class FilterConfig
    {
      public static void RegisterGlobalFilters(GlobalFilterCollection filters)
      {
        // Default replaced with the override to track unhandled exceptions
        filters.Add(new AiHandleErrorAttribute());
      }
    }
```

[範例](https://github.com/AppInsightsSamples/Mvc5UnhandledExceptionTelemetry)

## <a name="web-api"></a>Web API
從 Application Insights Web SDK 2.6 版 (beta3 和更新版本) 開始，Application Insights 會自動針對 WebAPI 2+ 收集在控制器方法中擲回的未處理例外狀況。 如果您先前已新增自訂處理常式來追蹤此類例外狀況 (如下列範例中所述)，您可以將其移除，以避免重複追蹤例外狀況。

有一些無法處理的例外狀況篩選器案例。 例如︰

* 從控制器建構函式擲回的例外狀況。
* 從訊息處理常式擲回的例外狀況。
* 在路由期間擲回的例外狀況。
* 在回應內容序列化期間擲回的例外狀況。
* 在應用程式啟動期間擲回的例外狀況。
* 背景工作中擲回的例外狀況。

由應用程式*處理*的所有例外狀況仍然需要手動追蹤。 源自控制器的未處理例外狀況通常會導致 500「內部伺服器錯誤」回應。 如果此類回應是因為未處理的例外狀況 (或根本沒有例外狀況) 而手動建構的，則對應的要求遙測會加以追蹤並產生 `ResultCode` 500，但 Application Insights SDK 無法追蹤對應的例外狀況。

### <a name="prior-versions-support"></a>舊版支援
如果您使用 Application Insights Web SDK 2.5 (和較早版本) 的 WebAPI 1 (和較早版本)，請參閱以下範例以追蹤例外狀況。

#### <a name="web-api-1x"></a>Web API 1.x
覆寫 System.Web.Http.Filters.ExceptionFilterAttribute：

```csharp
    using System.Web.Http.Filters;
    using Microsoft.ApplicationInsights;

    namespace WebAPI.App_Start
    {
      public class AiExceptionFilterAttribute : ExceptionFilterAttribute
      {
        public override void OnException(HttpActionExecutedContext actionExecutedContext)
        {
            if (actionExecutedContext != null && actionExecutedContext.Exception != null)
            {  //or reuse instance (recommended!). see note above
                var ai = new TelemetryClient();
                ai.TrackException(actionExecutedContext.Exception);    
            }
            base.OnException(actionExecutedContext);
        }
      }
    }
```

您可以將此覆寫的屬性加入到特定的控制器，或將它加入至 WebApiConfig 類別中的全域篩選組態：

```csharp
    using System.Web.Http;
    using WebApi1.x.App_Start;

    namespace WebApi1.x
    {
      public static class WebApiConfig
      {
        public static void Register(HttpConfiguration config)
        {
            config.Routes.MapHttpRoute(name: "DefaultApi", routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional });
            ...
            config.EnableSystemDiagnosticsTracing();

            // Capture exceptions for Application Insights:
            config.Filters.Add(new AiExceptionFilterAttribute());
        }
      }
    }
```

[範例](https://github.com/AppInsightsSamples/WebApi_1.x_UnhandledExceptions)

#### <a name="web-api-2x"></a>Web API 2.x
新增 IExceptionLogger 的實作：

```csharp
    using System.Web.Http.ExceptionHandling;
    using Microsoft.ApplicationInsights;

    namespace ProductsAppPureWebAPI.App_Start
    {
      public class AiExceptionLogger : ExceptionLogger
      {
        public override void Log(ExceptionLoggerContext context)
        {
            if (context !=null && context.Exception != null)
            {//or reuse instance (recommended!). see note above
                var ai = new TelemetryClient();
                ai.TrackException(context.Exception);
            }
            base.Log(context);
        }
      }
    }
```

將其新增至 WebApiConfig 中的服務：

```csharp
    using System.Web.Http;
    using System.Web.Http.ExceptionHandling;
    using ProductsAppPureWebAPI.App_Start;

    namespace WebApi2WithMVC
    {
      public static class WebApiConfig
      {
        public static void Register(HttpConfiguration config)
        {
            // Web API configuration and services

            // Web API routes
            config.MapHttpAttributeRoutes();

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
            config.Services.Add(typeof(IExceptionLogger), new AiExceptionLogger());
        }
      }
     }
```

[範例](https://github.com/AppInsightsSamples/WebApi_2.x_UnhandledExceptions)

做為替代方案，您可以：

1. 以 IExceptionHandler 的自訂實作取代唯一的 ExceptionHandler。 這只會在架構仍然可以選擇要傳送的回應訊息時呼叫 (不會在針對執行個體中止連接時呼叫)
2. 例外狀況篩選器 (如以上的 Web API 1.x 控制器章節所述) - 在所有案例中均不會呼叫。

## <a name="wcf"></a>WCF
新增類別，該類別會擴充屬性和實作 IErrorHandler 和 IServiceBehavior。

```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.ServiceModel.Description;
    using System.ServiceModel.Dispatcher;
    using System.Web;
    using Microsoft.ApplicationInsights;

    namespace WcfService4.ErrorHandling
    {
      public class AiLogExceptionAttribute : Attribute, IErrorHandler, IServiceBehavior
      {
        public void AddBindingParameters(ServiceDescription serviceDescription,
            System.ServiceModel.ServiceHostBase serviceHostBase,
            System.Collections.ObjectModel.Collection<ServiceEndpoint> endpoints,
            System.ServiceModel.Channels.BindingParameterCollection bindingParameters)
        {
        }

        public void ApplyDispatchBehavior(ServiceDescription serviceDescription,
            System.ServiceModel.ServiceHostBase serviceHostBase)
        {
            foreach (ChannelDispatcher disp in serviceHostBase.ChannelDispatchers)
            {
                disp.ErrorHandlers.Add(this);
            }
        }

        public void Validate(ServiceDescription serviceDescription,
            System.ServiceModel.ServiceHostBase serviceHostBase)
        {
        }

        bool IErrorHandler.HandleError(Exception error)
        {//or reuse instance (recommended!). see note above
            var ai = new TelemetryClient();

            ai.TrackException(error);
            return false;
        }

        void IErrorHandler.ProvideFault(Exception error,
            System.ServiceModel.Channels.MessageVersion version,
            ref System.ServiceModel.Channels.Message fault)
        {
        }
      }
    }

Add the attribute to the service implementations:

    namespace WcfService4
    {
        [AiLogException]
        public class Service1 : IService1
        {
         ...
```

[範例](https://github.com/AppInsightsSamples/WCFUnhandledExceptions)

## <a name="exception-performance-counters"></a>例外狀況效能計數器
如果您已在伺服器上[安裝 Application Insights代理程式](app-insights-monitor-performance-live-website-now.md)，您便可取得由 .NET 測量的例外狀況比率圖表。 這包括已處理和未處理的 .NET 例外狀況。

開啟 [計量瀏覽器] 刀鋒視窗、加入新圖表，然後選取 [效能計數器] 下方所列的 [例外狀況比率] 。

.NET framework 會計算間隔中的例外狀況次數並除以間隔長度，以計算得出例外狀況比率。

這與 Application Insights 入口網站執行 TrackException 報告計數算得的「例外狀況」計數不同。 取樣間隔不同，且 SDK 不會針對所有已處理與未處理的例外狀況傳送 TrackException 報告。

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player] 

## <a name="next-steps"></a>後續步驟
* [監視 REST、SQL 及其他對相依性的呼叫](app-insights-asp-net-dependencies.md)
* [監視頁面載入時間、瀏覽器例外狀況及 AJAX 呼叫](app-insights-javascript.md)
* [監視效能計數器](app-insights-performance-counters.md)
