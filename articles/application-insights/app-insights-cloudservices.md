---
title: Azure 雲端服務的 Application Insights | Microsoft Docs
description: 使用 Application Insights 有效地監視您的 Web 和背景工作角色
services: application-insights
documentationcenter: ''
keywords: WAD2AI, Azure 診斷
author: mrbullwinkle
manager: carmonm
ms.assetid: 5c7a5b34-329e-42b7-9330-9dcbb9ff1f88
ms.service: application-insights
ms.devlang: na
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.workload: tbd
ms.date: 09/05/2018
ms.author: mbullwin
ms.openlocfilehash: 3b06ec3b10edc39d770e5a724125e70afd5e5477
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43783489"
---
# <a name="application-insights-for-azure-cloud-services"></a>Azure 雲端服務的 Application Insights
您可以透過 [Application Insights][start] 來監視 [Microsoft Azure 雲端服務應用程式](https://azure.microsoft.com/services/cloud-services/)，它會結合 Application Insights SDK 的資料與雲端服務的 [Azure 診斷](https://docs.microsoft.com/azure/monitoring-and-diagnostics/azure-diagnostics)資料，來讓您了解應用程式的可用性、效能、失敗和使用情形。 當您取得有關應用程式在現實世界的效能和效率的意見反應時，您可以在每個開發生命週期中針對設計方向做出明智的抉擇。

![概觀儀表板的螢幕擷取畫面](./media/app-insights-cloudservices/overview-graphs.png)

## <a name="before-you-start"></a>開始之前
您需要：

* [Microsoft Azure](http://azure.com) 的訂用帳戶。 使用 Microsoft 帳戶登入，可能是針對 Windows、XBox Live 或其他 Microsoft 雲端服務具備的帳戶。 
* Microsoft Azure 工具 2.9 或更新版本
* 開發人員分析工具 7.10 或更新版本

## <a name="quick-start"></a>快速入門
使用 Application Insights 來監視您雲端服務的最快、最簡單方式就是在將服務發佈到 Azure 時選擇該選項。

![範例](./media/app-insights-cloudservices/azure-cloud-application-insights.png)

此選項會在執行階段檢測您的應用程式，為您提供監視您 Web 角色中的要求、例外狀況及相依性時所需的一切遙測，同時也提供來自背景工作角色的效能計數器。 您應用程式產生的所有診斷追蹤資料也會一併傳送給 Application Insights。

如果這就是您所需的一切，您就已大功告成！ 接下來的步驟包括[檢視來自您應用程式的度量](app-insights-metrics-explorer.md)、[利用分析查詢您的資料](app-insights-analytics.md)，還可能包括設定[儀表板](app-insights-dashboards.md)。 您可以設定[可用性測試](app-insights-monitor-web-app-availability.md)及[在網頁中新增程式碼](app-insights-javascript.md)，以在瀏覽器中監視效能。

但是您也可以取得更多選項：

* 將來自不同元件和組建組態的資料傳送給個別資源。
* 新增來自您應用程式的自訂遙測。

如果您對這些選項感興趣，請繼續閱讀本文。

## <a name="sample-application-instrumented-with-application-insights"></a>使用 Application Insights 檢測的範例應用程式
看一看這個 [範例應用程式](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService) ，其中將 Application Insights 新增到雲端服務，搭配兩個背景工作角色託管於 Azure 中。 

以下將告訴您如何透過相同的方式來調整您的雲端服務專案。

## <a name="plan-resources-and-resource-groups"></a>規劃資源和資源群組
來自您應用程式的遙測將會在 Application Insights 類型的 Azure 資源中儲存、分析及顯示。 

每個資源只屬於一個資源群組。 資源群組可用來在經過協調的單一交易中，管理成本、將存取權授與小組成員，以及部署更新項目。 例如，您可以[撰寫指令碼來部署](../azure-resource-manager/resource-group-template-deploy.md)「Azure 雲端服務」及其 Application Insights 監視資源，全部都在一個作業中完成。

### <a name="resources-for-components"></a>元件的資源
建議的配置是為您應用程式的每個元件 (亦即，每個 Web 角色和背景工作角色) 建立個別的資源。 您可以個別分析每個元件，但建立一個[儀表板](app-insights-dashboards.md)來彙總所有元件的重要圖表，以便將它們一起做比較和進行監視。 

替代配置是將來自多個角色的遙測傳送給同一個資源，但[為每個遙測項目新增一個維度屬性](app-insights-api-filtering-sampling.md#add-properties-itelemetryinitializer)來識別其來源角色。 在此配置中，計量圖表 (例如例外狀況) 通常會顯示來自不同角色之計數的彙總，但您可以視需要依角色識別碼分割該圖表。 您也可以用相同的維度來篩選搜尋。 此替代方案可讓您稍微容易同時檢視每個項目，但也會造成角色之間的一些混淆。

瀏覽器遙測通常包含在與其伺服器端 Web 角色相同的資源中。

請將不同元件的 Application Insights 資源放在一個資源群組中。 這可讓您容易一起管理它們。 

### <a name="separating-development-test-and-production"></a>區分開發、測試及生產環境
如果您是在上一個版本正在運作的情況下，為下一個功能開發自訂事件，您就會想要將開發遙測傳送給個別的 Application Insights 資源。 否則，將會很難在來自即時網站的所有流量中找出您的測試遙測。

為了避免這種情況，請為您系統的每個組建組態或「戳記」(開發、測試、生產環境等) 建立個別的資源。 請將每個組建組態的資源放在個別的資源群組中。 

若要將遙測傳送給適當的資源，您可以安裝 Application Insights SDK，讓它根據組建組態挑選不同的檢測金鑰。 

## <a name="create-an-application-insights-resource-for-each-role"></a>為每個角色建立 Application Insights 資源
如果您已決定為每個角色建立個別的資源，或許也為每個組建組態建立一組個別的資源，則最簡單的方式就是全部都在 Application Insights 入口網站中建立。 (如果您要建立很多資源，您可以[將程序自動化](app-insights-powershell.md))。

1. 在 [Azure 入口網站][portal]中，建立新的 Application Insights 資源。 針對應用程式類型，選擇 ASP.NET 應用程式。 

    ![按一下 [新增]，然後按一下 [Application Insights]](./media/app-insights-cloudservices/01-new.png)
2. 每項資源均會以「檢測金鑰」來識別。 如果您想要手動設定或確認 SDK 的組態，稍後可能會需要此金鑰。


## <a name="set-up-azure-diagnostics-for-each-role"></a>設定每個角色的 Azure 診斷
設定此選項即可使用 Application Insights 來監視您的應用程式。 針對 Web 角色，這除了提供使用情況分析之外，也提供效能監視、警示及診斷。 針對他角色，您可以搜尋和監視 Azure 診斷，例如重新啟動、效能計數器及對 System.Diagnostics.Trace 的呼叫。 

1. 在 Visual Studio [方案總管] 中的 [YourCloudService]&lt;&gt; 角色底下，開啟每個角色的屬性。
2. 在 [組態] 中設定 [傳送診斷資料至 Application Insights]，然後選取您先前建立的適當 Application Insights 資源。

如果您已決定針對每個組建組態使用個別的 Application Insights 資源，請先選取組態。

![在每個 Azure 角色的屬性中，設定 Application Insights](./media/app-insights-cloudservices/configure-azure-diagnostics.png)

這會將您的 Application insights 檢測金鑰插入名為 `ServiceConfiguration.*.cscfg` 的檔案中。 ([範例程式碼](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/AzureEmailService/ServiceConfiguration.Cloud.cscfg))。

如果您想要變更傳送到 Application Insights 的診斷資訊層級，只要[直接編輯 `.cscfg` 檔案即可](app-insights-azure-diagnostics.md)。

## <a name="sdk"></a>在每個專案中安裝 SDK
此選項會新增將自訂業務遙測新增到任何角色的功能，可用來進一步分析您應用程式的使用和執行情況。

在 Visual Studio 中，設定每個雲端應用程式專案的 Application Insights SDK。

1. **Web 角色**：以滑鼠右鍵按一下專案，然後選擇 [設定 Application Insights] 或 [新增 > Application Insights 遙測]。

2. **背景工作角色**： 
 * 以滑鼠右鍵按一下專案，然後選取 [管理 NuGet 套件]。
 * 新增[適用於 Windows 伺服器的 Application Insights](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/)。

    ![搜尋「Application Insights」](./media/app-insights-cloudservices/04-ai-nuget.png)

3. 設定 SDK 以將資料傳送給 Application Insights 資源。

    在適合的啟動函式中，從 ``.cscfg file`` 中的組態設定設定檢測金鑰：
 
    ```csharp
   
     TelemetryConfiguration.Active.InstrumentationKey = RoleEnvironment.GetConfigurationSettingValue("APPINSIGHTS_INSTRUMENTATIONKEY");
    ```
   
    對於應用程式中的每個角色執行這項操作。 請參閱範例：
   
   * [Web 角色](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Global.asax.cs#L27)
   * [背景工作角色](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L232)
   * [針對網頁](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Views/Shared/_Layout.cshtml#L13) 
4. 將 ApplicationInsights.config 設定為一律複製到輸出目錄。 
   
    (在 .config 檔案中，您會看到訊息詢問您將檢測金鑰放至該處。 不過，針對雲端應用程式，最好是從 ``.cscfg file`` 中設定。 這可確保角色會在入口網站中正確識別。)

#### <a name="run-and-publish-the-app"></a>執行和發佈應用程式
執行應用程式，並且登入 Azure。 開啟您建立的 Application Insights 資源，您會看到個別資料點顯示在[搜尋](app-insights-diagnostic-search.md)中，並且在[計量瀏覽器](app-insights-metrics-explorer.md)中彙總資料。 

新增更多遙測 - 請參閱以下章節 - 然後再發佈應用程式以取得即時診斷和使用方式的意見反應。 

#### <a name="no-data"></a>沒有資料？
* 開啟 [[搜尋][diagnostic]] 磚來查看個別事件。
* 使用應用程式、開啟不同頁面，以產生一些遙測。
* 請稍等片刻，然後按一下 [重新整理]。
* 請參閱[疑難排解][qna]。

## <a name="view-azure-diagnostic-events"></a>檢視 Azure 診斷事件
[Azure 診斷](https://docs.microsoft.com/azure/monitoring-and-diagnostics/azure-diagnostics)資訊在 Application Insights 中的位置如下：

* 效能計數器顯示為自訂度量。 
* Windows 事件記錄檔顯示為追蹤和自訂事件。
* 應用程式記錄檔、ETW 記錄檔和任何診斷基礎結構記錄檔顯示為追蹤。

若要查看效能計數器和事件計數，開啟 [計量瀏覽器](app-insights-metrics-explorer.md) 並加入新的圖表：

![Azure 診斷資料](./media/app-insights-cloudservices/23-wad.png)

使用[搜尋](app-insights-diagnostic-search.md)或[分析查詢](app-insights-analytics-tour.md)，在各種 Azure 診斷傳送的追蹤記錄檔中搜尋。 例如，假設您有未處理的例外狀況導致角色損毀並需要回收。 這項資訊會出現在 Windows 事件記錄檔的「應用程式 (Application)」通道。 您可以使用 [搜尋 (Search)] 來查看 Windows 事件記錄檔錯誤，並取得例外狀況的完整堆疊追蹤。 這將協助您找出問題的根本原因。

![Azure 診斷搜尋](./media/app-insights-cloudservices/25-wad.png)

## <a name="more-telemetry"></a>更多遙測
以下各節說明如何從您的應用程式的不同層面取得其他的遙測。

## <a name="track-requests-from-worker-roles"></a>從背景工作角色追蹤要求
在 Web 角色中，要求模組會自動收集有關 HTTP 要求的資料。 如需有關如何覆寫預設收集行為的範例，請參閱 [範例 MVCWebRole](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/MvcWebRole) 。 

您可以藉由與追蹤 HTTP 要求相同的方式追蹤背景工作角色，來擷取背景工作角色呼叫的效能。 在 Application Insights 中，要求遙測類型會測量一個單位的具名伺服器端工作，可以進行計時，而且可以獨立成功或失敗。 由 SDK 會自動擷取 HTTP 要求時，您可以插入自己的程式碼，來追蹤對背景工作角色的要求。

請參閱檢測兩個範例背景工作角色以報告要求：[WorkerRoleA](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/WorkerRoleA) 和 [WorkerRoleB](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/WorkerRoleB)

## <a name="exceptions"></a>例外狀況
如需如何從不同的 Web 應用程式類型收集未處理的例外狀況的資訊，請參閱 [在 Application Insights 中監視例外狀況](app-insights-asp-net-exceptions.md) 。

範例 Web 角色具有 MVC5 以及 Web API 2 控制器。 來自這兩者的未處理例外狀況是使用下列處理常式擷取：

* 針對 MVC5 控制器，[這裡](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/App_Start/FilterConfig.cs#L12)設定的 [AiHandleErrorAttribute](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Telemetry/AiHandleErrorAttribute.cs)
* 針對 Web API 2 控制器，[這裡](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/App_Start/WebApiConfig.cs#L25)設定的 [AiWebApiExceptionLogger](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Telemetry/AiWebApiExceptionLogger.cs)

針對背景工作角色，有兩種方式來追蹤例外狀況：

* TrackException(ex)
* 如果您已新增 Application Insights 追蹤接聽項 NuGet 套件，您可以使用 **System.Diagnostics.Trace** 來記錄例外狀況。 [程式碼範例。](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L107)

## <a name="performance-counters"></a>效能計數器
根據預設會收集下列計數器：

    * \Process(??APP_WIN32_PROC??)\%Processor Time
    * \Memory\Available Bytes
    * \.NET CLR Exceptions(??APP_CLR_PROC??)\# of Exceps Thrown / sec
    * \Process(??APP_WIN32_PROC??)\Private Bytes
    * \Process(??APP_WIN32_PROC??)\IO Data Bytes/sec
    * \Processor(_Total)\% Processor Time

若是 Web 角色，也會收集這些計數器︰

    * \ASP.NET Applications(??APP_W3SVC_PROC??)\Requests/Sec
    * \ASP.NET Applications(??APP_W3SVC_PROC??)\Request Execution Time
    * \ASP.NET Applications(??APP_W3SVC_PROC??)\Requests In Application Queue

您可以藉由編輯 ApplicationInsights.config 來指定額外的自訂或其他 Windows 效能計數器，[如此範例所示](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/ApplicationInsights.config#L14)。

  ![效能計數器](./media/app-insights-cloudservices/002-servers.png)

## <a name="correlated-telemetry-for-worker-roles"></a>背景工作角色的相互關聯遙測
當您可以看見導致失敗或高延遲要求的原因時，診斷體驗會更加豐富。 使用 Web 角色，SDK 會在關聯的遙測間自動設定相互關聯。 針對背景工作角色，您可以使用自訂遙測初始設定式，來設定一個通用 Operation.Id 內容屬性，讓所有的遙測可以達到此目的。 這可讓您查看是否因為相依性或程式碼導致延遲/失敗問題，一目了然！ 

方式如下：

* 設定相互關聯識別碼到 CallContext 中，如 [這裡](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L36)所示。 在此案例中，我們使用「要求 ID」做為相互關聯識別碼
* 新增自訂的 TelemetryInitializer 實作，其會將 Operation.Id 設為前面所設定的 correlationId。 範例如下：[ItemCorrelationTelemetryInitializer](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/Telemetry/ItemCorrelationTelemetryInitializer.cs#L13)
* 新增自訂的遙測初始設定式。 您可以在 ApplicationInsights.config 檔案或程式碼中執行此作業，如[此處](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L233)所示。

## <a name="client-telemetry"></a>用戶端遙測
[將 JavaScript SDK 新增至網頁][client]以取得瀏覽器型遙測 (例如頁面檢視計數、頁面載入時間、指令碼例外狀況)，並讓您在頁面指令碼中撰寫自訂遙測。

## <a name="availability-tests"></a>可用性集合
[設定 Web 測試][availability]，以確認應用程式處於線上狀態且能夠回應。

## <a name="display-everything-together"></a>將所有內容一起顯示
若要了解您系統的整體情況，您可以將重要的監視圖表全部都彙總在一個[儀表板](app-insights-dashboards.md)上。 例如，您可以釘選每個角色的要求和失敗計數。 

如果您的系統使用其他 Azure 服務 (例如「串流分析」)，請將它們的監視圖表一併納入。 

如果您有用戶端行動應用程式，請使用 [App Center](app-insights-mobile-center-quickstart.md)。 請在[分析](app-insights-analytics.md)中建立查詢以顯示事件計數，並將它們釘選到儀表板。

## <a name="example"></a>範例
[此範例](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService) 監視具有 Web 角色和兩個背景工作角色的服務。

## <a name="exception-method-not-found-on-running-in-azure-cloud-services"></a>在 Azure 雲端服務中執行時發生的「找不到方法」例外狀況
您是否已針對 .NET 4.6 組建？ Azure 雲端服務角色不自動支援 4.6。 [在每個角色上安裝 4.6](../cloud-services/cloud-services-dotnet-install-dotnet.md) ，再執行您的 App。

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="next-steps"></a>後續步驟
* [設定將 Azure 診斷傳送至 Application Insights](app-insights-azure-diagnostics.md)
* [自動建立 Application Insights 資源](app-insights-powershell.md)
* [自動執行 Azure 診斷](app-insights-powershell-azure-diagnostics.md)
* [Azure Functions](https://github.com/christopheranderson/azure-functions-app-insights-sample)

[api]: app-insights-api-custom-events-metrics.md
[availability]: app-insights-monitor-web-app-availability.md
[azure]: app-insights-azure.md
[client]: app-insights-javascript.md
[diagnostic]: app-insights-diagnostic-search.md
[netlogs]: app-insights-asp-net-trace-logs.md
[portal]: http://portal.azure.com/
[qna]: app-insights-troubleshoot-faq.md
[redfield]: app-insights-monitor-performance-live-website-now.md
[start]: app-insights-overview.md 
