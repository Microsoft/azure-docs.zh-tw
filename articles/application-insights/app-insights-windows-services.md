---
title: 適用於 Windows 伺服器和背景工作角色的 Azure Application Insights | Microsoft Docs
description: 將 Application Insights SDK 手動新增至您的 ASP.NET 應用程式，以分析使用情況、可用性和效能。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: 106ba99b-b57a-43b8-8866-e02f626c8190
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 05/15/2017
ms.author: mbullwin
ms.openlocfilehash: 3f2f0925cf6ec612b84765e42217723d8c8729c6
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2018
ms.locfileid: "35632254"
---
# <a name="manually-configure-application-insights-for-net-applications"></a>為 .NET 應用程式手動設定 Application Insights

您可以設定 [Application Insights](app-insights-overview.md) 以監視各種不同的應用程式或應用程式角色、元件或微服務。 對於 Web 應用程式和服務，Visual Studio 會提供[單步驟設定](app-insights-asp-net.md)。 對於其他類型的 .NET 應用程式 (例如後端伺服器角色或桌面應用程式)，您可以手動設定 Application Insights。

![範例效能監視圖表](./media/app-insights-windows-services/10-perf.png)

#### <a name="before-you-start"></a>開始之前

您需要：

* [Microsoft Azure](http://azure.com)訂用帳戶。 如果您的小組或組織擁有 Azure 訂用帳戶，擁有者就可以使用您的 [Microsoft 帳戶](http://live.com)將您加入。
* Visual Studio 2013 或更新版本。

## <a name="add"></a>1.選擇 Application Insights 資源

「資源」是在 Azure 入口網站中收集和顯示您資料的位置。 您需要決定是否要建立新的資源，或共用現有資源。

### <a name="part-of-a-larger-app-use-existing-resource"></a>屬於大型應用程式︰使用現有資源

如果您的 Web 應用程式有多個元件 (例如，前端 Web 應用程式和一或多個後端服務)，您應該將來自所有元件的遙測傳送至相同的資源。 這會讓它們顯示在單一應用程式對應上，並可追蹤元件之間的要求。

因此，如果您已監視此應用程式的其他元件，只要使用相同的資源即可。

在 [Azure 入口網站](https://portal.azure.com/)中開啟資源。 

### <a name="self-contained-app-create-a-new-resource"></a>獨立應用程式：建立新的資源

如果新的應用程式與其他應用程式無關，則應該有自己的資源。

登入 [Azure 入口網站](https://portal.azure.com/)，並建立新的 Application Insights 資源。 選擇 ASP.NET 做為應用程式類型。

![按一下 [新增]，然後按一下 [Application Insights]](./media/app-insights-windows-services/01-new-asp.png)

所選的應用程式類型會設定資源刀鋒視窗的預設內容。

## <a name="2-copy-the-instrumentation-key"></a>2.複製檢測金鑰
可識別資源的金鑰。 您很快就會將它安裝在 SDK 中，以便將資料導向資源。

![按一下 [屬性]，選取金鑰，然後按下 CTRL+C](./media/app-insights-windows-services/02-props-asp.png)

## <a name="sdk"></a>3.在您的應用程式中安裝 Application Insights 套件
安裝和設定 Application Insights 套件會因您所正在使用的平台而有所不同。 

1. 在 Visual Studio 中，以滑鼠右鍵按一下專案，然後選擇 [管理 Nuget 套件]。
   
    ![以滑鼠右鍵按一下專案，然後選取 [管理 NuGet 套件]](./media/app-insights-windows-services/03-nuget.png)
2. 針對 Windows Server 應用程式安裝 Application Insights 套件 "Microsoft.ApplicationInsights.WindowsServer"。
   
    ![搜尋「Application Insights」](./media/app-insights-windows-services/04-ai-nuget.png)
   
    *哪個版本？*

    如果您想要嘗試最新的功能，請勾選 [包括搶鮮版]。 相關的文件或部落格會指明您是否需要搶鮮版。
    
    *可以使用其他封裝嗎？*
   
    是。 如果您只想要使用 API 來傳送自己的遙測，請選擇 "Microsoft.ApplicationInsights"。 Windows Server 套件會包含 API 及其他套件，例如效能計數器收集和相依性監視。 

### <a name="to-upgrade-to-future-package-versions"></a>升級至未來的套件版本
我們隨時會發行新版的 SDK。

若要升級至[新版的套件](https://github.com/Microsoft/ApplicationInsights-dotnet-server/releases/)，請再次開啟 NuGet 套件管理員，並篩選出已安裝的套件。 選取 **Microsoft.ApplicationInsights.WindowsServer**，然後選擇 [升級]。

如果您已對 ApplicationInsights.config 進行任何的自訂，請在升級前儲存複本，並在升級後合併您的變更到新版本中。

## <a name="4-send-telemetry"></a>4.傳送遙測
**如果您只安裝 API 套件︰**

* 在程式碼中設定檢測金鑰，例如在 `main()`中： 
  
    `TelemetryConfiguration.Active.InstrumentationKey = "`您的金鑰 `";` 
* [使用 API 撰寫自己的遙測](app-insights-api-custom-events-metrics.md#ikey)。

**如果您安裝了其他 Application Insights 封裝** ，您可以視需要使用 .config 檔案來設定檢測金鑰︰

* 編輯 ApplicationInsights.config (已由 NuGet 安裝加入)。 在結尾標記前面插入此內容：
  
    `<InstrumentationKey>`您複製的檢測金鑰 `</InstrumentationKey>`
* 確定 [方案總管] 中 ApplicationInsights.config 的屬性已設定為 [建置動作] = [內容]、[複製到輸出目錄] = [複製] 。

如果您想要[針對不同建置組態切換金鑰](app-insights-separate-resources.md)，在程式碼中設定檢測金鑰便很實用。 如果您在程式碼中設定金鑰，您不必在 `.config` 檔案中設定它。

## <a name="run"></a> 執行專案
使用 **F5** 執行應用程式並立即試用：開啟不同的頁面來產生一些遙測。

在 Visual Studio 中，您可以看見已傳送到的事件計數。

![Visual Studio 中的事件計數](./media/app-insights-windows-services/appinsights-09eventcount.png)

## <a name="monitor"></a> 檢視遙測
返回 [Azure 入口網站](https://portal.azure.com/) ，並且瀏覽至您的 Application Insights 資源。

在 [概觀] 圖表中尋找資料。 剛開始的時候，您只會看見一或兩個資料點。 例如︰

![Click through to more data](./media/app-insights-windows-services/12-first-perf.png)

逐一點選任何圖表以查看更詳細的度量。 [深入了解度量。](app-insights-web-monitor-performance.md)

### <a name="no-data"></a>沒有資料？
* 使用應用程式、開啟不同頁面，以產生一些遙測。
* 開啟 [ [搜尋](app-insights-diagnostic-search.md) ] 磚來查看個別事件。 有時候，事件通過計量管線所需的時間較長。
* 請稍等片刻，然後按一下 [重新整理 ]。 圖表會定期自行重新整理，但是如果您在等待一些要顯示的資料，您可以手動重新整理。
* 請參閱 [疑難排解](app-insights-troubleshoot-faq.md)。

## <a name="publish-your-app"></a>發佈您的應用程式
現在將應用程式部署至您的伺服器或 Azure，並觀看資料累積情形。

![使用 Visual Studio 來發佈您的應用程式](./media/app-insights-windows-services/15-publish.png)

以偵錯模式執行時，系統會透過管線迅速傳送遙測資料，因此您應該可以在幾秒內看見資料。 以發行組態部署應用程式時，資料累積會較為緩慢。

### <a name="no-data-after-you-publish-to-your-server"></a>發佈資料到伺服器之後，卻沒有資料？
請在您的伺服器防火牆中，開啟連出流量的連接埠。 請參閱[本頁](https://docs.microsoft.com/azure/application-insights/app-insights-ip-addresses)以取得必要位址清單 

### <a name="trouble-on-your-build-server"></a>組建伺服器發生問題？
請參閱 [此疑難排解項目](app-insights-asp-net-troubleshoot-no-data.md#NuGetBuild)。

> [!NOTE]
> 如果您的應用程式會產生大量遙測，調適性取樣模型會自動藉由僅傳送事件代表性片段，減少傳送到入口網站的量。 不過，與同一個要求相關的事件會選取或取消選取為群組，讓您可以在相關事件之間瀏覽。 
> [了解取樣](app-insights-sampling.md)。
> 
> 

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="next-steps"></a>後續步驟
* [新增更多遙測](app-insights-asp-net-more.md) 可取得應用程式的完整 360 度檢視。

