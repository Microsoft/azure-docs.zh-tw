---
title: 使用 Application Insights 監視 SharePoint 網站
description: 開始使用新的檢測金鑰監視新的應用程式
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 2bfe5910-d673-4cf6-a5c1-4c115eae1be0
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: mbullwin
ms.openlocfilehash: 3a7d657a21b414d51375f912513ae045adec6d6e
ms.sourcegitcommit: df50934d52b0b227d7d796e2522f1fd7c6393478
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38992195"
---
# <a name="monitor-a-sharepoint-site-with-application-insights"></a>使用 Application Insights 監視 SharePoint 網站
Azure Application Insights 會監視應用程式的可用性、效能和使用情況。 您將在這裡深入了解如何針對 SharePoint 網站進行設定。

## <a name="create-an-application-insights-resource"></a>建立 Application Insights 資源
在 [Azure 入口網站](https://portal.azure.com)中，建立新的 Application Insights 資源。 選擇 ASP.NET 做為應用程式類型。

![按一下 [屬性]，選取金鑰，然後按下 CTRL+C](./media/app-insights-sharepoint/001.png)

開啟的視窗是您要查看您的應用程式效能和使用量資料的位置。 若要在下次登入 Azure 時回到此位置，您應該會在開始畫面上發現它的磚。 或者按一下 [瀏覽] 以尋找它。

## <a name="add-the-script-to-your-web-pages"></a>將指令碼新增至您的網頁

```HTML
<!-- 
To collect user behavior analytics tools about your application, 
insert the following script into each page you want to track.
Place this code immediately before the closing </head> tag,
and before any other scripts. Your first data will appear 
automatically in just a few seconds.
-->
<script type="text/javascript">
var appInsights=window.appInsights||function(a){
  function b(a){c[a]=function(){var b=arguments;c.queue.push(function(){c[a].apply(c,b)})}}var c={config:a},d=document,e=window;setTimeout(function(){var b=d.createElement("script");b.src=a.url||"https://az416426.vo.msecnd.net/scripts/a/ai.0.js",d.getElementsByTagName("script")[0].parentNode.appendChild(b)});try{c.cookie=d.cookie}catch(a){}c.queue=[];for(var f=["Event","Exception","Metric","PageView","Trace","Dependency"];f.length;)b("track"+f.pop());if(b("setAuthenticatedUserContext"),b("clearAuthenticatedUserContext"),b("startTrackEvent"),b("stopTrackEvent"),b("startTrackPage"),b("stopTrackPage"),b("flush"),!a.disableExceptionTracking){f="onerror",b("_"+f);var g=e[f];e[f]=function(a,b,d,e,h){var i=g&&g(a,b,d,e,h);return!0!==i&&c["_"+f](a,b,d,e,h),i}}return c
  }({
      instrumentationKey:"<your instrumentation key>"
  });
  
window.appInsights=appInsights,appInsights.queue&&0===appInsights.queue.length&&appInsights.trackPageView();
</script>
```

在您想要追蹤之每一個頁面的 &lt;/head&gt; 標記前插入指令碼。如果您的網站有主版頁面，您可以那裡放入指令碼。 例如，在 ASP.NET MVC 專案中，可放在 View\Shared\_Layout.cshtml 中

指令碼包含檢測金鑰，會將遙測導向您的 Application Insights 資源。

### <a name="add-the-code-to-your-site-pages"></a>將程式碼加入至網站頁面
#### <a name="on-the-master-page"></a>在主要頁面上
如果您可以編輯網站的主要頁面，將會提供網站中每一頁面的監視。

簽出主要頁面，並且使用 SharePoint Designer 或任何其他編輯器來編輯。

![](./media/app-insights-sharepoint/03-master.png)

在 </head> 標記之前加入程式碼。 

![](./media/app-insights-sharepoint/04-code.png)

#### <a name="or-on-individual-pages"></a>或在個別的頁面上
若要監視一組有限的頁面，將指令碼個別加入至每個頁面。 

插入網頁組件並在其中嵌入程式碼片段。

![](./media/app-insights-sharepoint/05-page.png)

## <a name="view-data-about-your-app"></a>檢視應用程式相關的資料
重新部署您的應用程式。

返回 [Azure 入口網站](https://portal.azure.com)中的應用程式刀鋒視窗。

前幾個事件將出現在搜尋中。 

![](./media/app-insights-sharepoint/09-search.png)

如果您預期有更多資料，請在幾秒之後按一下 [重新整理]。

## <a name="capturing-user-id"></a>擷取使用者識別碼
標準網頁程式碼片段不會從 SharePoint 擷取使用者識別碼，但只要稍做修改就能執行此作業。

1. 從 Application Insights 中的 [Essentials] 下拉式清單複製您應用程式的檢測金鑰。 

    ![](./media/app-insights-sharepoint/02-props.png)

1. 使用檢測金鑰替換下列程式碼片段中的 'XXXX'。 
2. 在您的 SharePoint 應用程式中內嵌指令碼，而非您從入口網站取得的程式碼片段。

```


<SharePoint:ScriptLink ID="ScriptLink1" name="SP.js" runat="server" localizable="false" loadafterui="true" /> 
<SharePoint:ScriptLink ID="ScriptLink2" name="SP.UserProfiles.js" runat="server" localizable="false" loadafterui="true" /> 

<script type="text/javascript"> 
var personProperties; 

// Ensure that the SP.UserProfiles.js file is loaded before the custom code runs. 
SP.SOD.executeOrDelayUntilScriptLoaded(getUserProperties, 'SP.UserProfiles.js'); 

function getUserProperties() { 
    // Get the current client context and PeopleManager instance. 
    var clientContext = new SP.ClientContext.get_current(); 
    var peopleManager = new SP.UserProfiles.PeopleManager(clientContext); 

    // Get user properties for the target user. 
    // To get the PersonProperties object for the current user, use the 
    // getMyProperties method. 

    personProperties = peopleManager.getMyProperties(); 

    // Load the PersonProperties object and send the request. 
    clientContext.load(personProperties); 
    clientContext.executeQueryAsync(onRequestSuccess, onRequestFail); 
} 

// This function runs if the executeQueryAsync call succeeds. 
function onRequestSuccess() { 
var appInsights=window.appInsights||function(config){
function s(config){t[config]=function(){var i=arguments;t.queue.push(function(){t[config].apply(t,i)})}}var t={config:config},r=document,f=window,e="script",o=r.createElement(e),i,u;for(o.src=config.url||"//az416426.vo.msecnd.net/scripts/a/ai.0.js",r.getElementsByTagName(e)[0].parentNode.appendChild(o),t.cookie=r.cookie,t.queue=[],i=["Event","Exception","Metric","PageView","Trace"];i.length;)s("track"+i.pop());return config.disableExceptionTracking||(i="onerror",s("_"+i),u=f[i],f[i]=function(config,r,f,e,o){var s=u&&u(config,r,f,e,o);return s!==!0&&t["_"+i](config,r,f,e,o),s}),t
    }({
        instrumentationKey:"XXXX"
    });
    window.appInsights=appInsights;
    appInsights.trackPageView(document.title,window.location.href, {User: personProperties.get_displayName()});
} 

// This function runs if the executeQueryAsync call fails. 
function onRequestFail(sender, args) { 
} 
</script> 


```



## <a name="next-steps"></a>後續步驟
* [Web 測試](app-insights-monitor-web-app-availability.md) 。
* [Application Insights](app-insights-overview.md) 。

<!--Link references-->


