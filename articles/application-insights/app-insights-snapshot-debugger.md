---
title: .NET 應用程式的 Azure Application Insights 快照集偵錯工具 | Microsoft Docs
description: 在生產環境 .NET 應用程式中擲回例外狀況時，會自動收集偵錯快照集
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 05/08/2018
ms.reviewer: pharring
ms.author: mbullwin
ms.openlocfilehash: d4c27c8297fb5a2ad13a245279a206d00fc4f8b1
ms.sourcegitcommit: a1140e6b839ad79e454186ee95b01376233a1d1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43144120"
---
# <a name="debug-snapshots-on-exceptions-in-net-apps"></a>.NET 應用程式中的例外狀況偵錯快照集

發生例外狀況時，您可以自動從即時 Web 應用程式收集偵錯快照集。 快照集會顯示擲回例外狀況時原始程式碼和變數的狀態。 [Application Insights](app-insights-overview.md) 中的快照集偵錯工具 (預覽) 會監視 web 應用程式的例外狀況遙測。 它會收集前幾個擲回例外狀況的快照集，讓您取得診斷生產環境中問題所需的資訊。 將[快照集收集器 NuGet 套件](http://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector)納入您的應用程式，並選擇性地設定 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md) 中的集合參數。快照集會顯示在 Application Insights 入口網站中的[例外狀況](app-insights-asp-net-exceptions.md)。

您可以檢視入口網站中的偵錯快照集，以查看呼叫堆疊並檢查每個呼叫堆疊框架的變數。 若要使用原始程式碼取得更強大的偵錯體驗，可[下載 Visual Studio 的快照集偵錯工具擴充功能](https://aka.ms/snapshotdebugger)，以 Visual Studio 2017 Enterprise 開啟快照集。 在 Visual Studio 中，您也可以[設定貼齊點以互動方式建立快照集](https://aka.ms/snappoint)，而不需等待例外狀況。

快照集集合適用於：
* 執行 .NET Framework 4.5 或更新版本的 .NET Framework 和 ASP.NET 應用程式。
* 在 Windows 上執行的 .NET Core 2.0 和 ASP.NET Core 2.0 應用程式。

支援下列環境：
* Azure App Service。
* 執行作業系統系列 4 或更新版本的 Azure 雲端服務。
* 在 Windows Server 2012 R2 或更新版本上執行的 Azure Service Fabric 服務。
* 執行 Windows Server 2012 R2 或更新版本的 Azure 虛擬機器。
* 執行 Windows Server 2012 R2 或更新版本的內部部署虛擬或實體機器。

> [!NOTE]
> 不支援用戶端應用程式 (例如，WPF、Windows Forms 或 UWP)。

### <a name="configure-snapshot-collection-for-aspnet-applications"></a>設定 ASP.NET 應用程式的快照集集合

1. 如果您尚未這麼做，請[在 Web 應用程式中啟用 Application Insights](app-insights-asp-net.md)。

2. 將 [Microsoft.ApplicationInsights.SnapshotCollector](http://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) NuGet 套件納入您的應用程式。

3. 檢閱套件已新增至 [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md) 的預設選項：

    ```xml
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.SnapshotCollector.SnapshotCollectorTelemetryProcessor, Microsoft.ApplicationInsights.SnapshotCollector">
        <!-- The default is true, but you can disable Snapshot Debugging by setting it to false -->
        <IsEnabled>true</IsEnabled>
        <!-- Snapshot Debugging is usually disabled in developer mode, but you can enable it by setting this to true. -->
        <!-- DeveloperMode is a property on the active TelemetryChannel. -->
        <IsEnabledInDeveloperMode>false</IsEnabledInDeveloperMode>
        <!-- How many times we need to see an exception before we ask for snapshots. -->
        <ThresholdForSnapshotting>1</ThresholdForSnapshotting>
        <!-- The maximum number of examples we create for a single problem. -->
        <MaximumSnapshotsRequired>3</MaximumSnapshotsRequired>
        <!-- The maximum number of problems that we can be tracking at any time. -->
        <MaximumCollectionPlanSize>50</MaximumCollectionPlanSize>
        <!-- How often we reconnect to the stamp. The default value is 15 minutes.-->
        <ReconnectInterval>00:15:00</ReconnectInterval>
        <!-- How often to reset problem counters. -->
        <ProblemCounterResetInterval>1.00:00:00</ProblemCounterResetInterval>
        <!-- The maximum number of snapshots allowed in ten minutes.The default value is 1. -->
        <SnapshotsPerTenMinutesLimit>1</SnapshotsPerTenMinutesLimit>
        <!-- The maximum number of snapshots allowed per day. -->
        <SnapshotsPerDayLimit>30</SnapshotsPerDayLimit>
        <!-- Whether or not to collect snapshot in low IO priority thread. The default value is true. -->
        <SnapshotInLowPriorityThread>true</SnapshotInLowPriorityThread>
        <!-- Agree to send anonymous data to Microsoft to make this product better. -->
        <ProvideAnonymousTelemetry>true</ProvideAnonymousTelemetry>
        <!-- The limit on the number of failed requests to request snapshots before the telemetry processor is disabled. -->
        <FailedRequestLimit>3</FailedRequestLimit>
        </Add>
    </TelemetryProcessors>
    ```

4. 快照集只會收集向 Application Insights 回報的例外狀況。 在某些情況下 (例如，舊版的 .NET 平台)，您可能需要[設定例外狀況集合](app-insights-asp-net-exceptions.md#exceptions)，才能在入口網站中查看快照集的例外狀況。


### <a name="configure-snapshot-collection-for-aspnet-core-20-applications"></a>設定 ASP.NET Core 2.0 應用程式的快照集集合

1. 如果您尚未這麼做，請[在 ASP.NET Core Web 應用程式中啟用 Application Insights](app-insights-asp-net-core.md)。

    > [!NOTE]
    > 請確定您的應用程式參考的是 2.1.1 版或更新版本的 Microsoft.ApplicationInsights.AspNetCore 封裝。

2. 將 [Microsoft.ApplicationInsights.SnapshotCollector](http://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) NuGet 套件納入您的應用程式。

3. 修改您應用程式的 `Startup` 類別，以新增和設定快照集收集器的遙測處理器。

    將下列 using 陳述式新增至 `Startup.cs`

   ```csharp
   using Microsoft.ApplicationInsights.SnapshotCollector;
   using Microsoft.Extensions.Options;
   using Microsoft.ApplicationInsights.AspNetCore;
   using Microsoft.ApplicationInsights.Extensibility;
   ```

   將下列 `SnapshotCollectorTelemetryProcessorFactory` 類別新增至 `Startup` 類別。

   ```csharp
   class Startup
   {
       private class SnapshotCollectorTelemetryProcessorFactory : ITelemetryProcessorFactory
       {
           private readonly IServiceProvider _serviceProvider;

           public SnapshotCollectorTelemetryProcessorFactory(IServiceProvider serviceProvider) =>
               _serviceProvider = serviceProvider;

           public ITelemetryProcessor Create(ITelemetryProcessor next)
           {
               var snapshotConfigurationOptions = _serviceProvider.GetService<IOptions<SnapshotCollectorConfiguration>>();
               return new SnapshotCollectorTelemetryProcessor(next, configuration: snapshotConfigurationOptions.Value);
           }
       }
       ...
    ```
    將 `SnapshotCollectorConfiguration` 和 `SnapshotCollectorTelemetryProcessorFactory` 服務新增至啟動管線：

    ```csharp
       // This method gets called by the runtime. Use this method to add services to the container.
       public void ConfigureServices(IServiceCollection services)
       {
           // Configure SnapshotCollector from application settings
           services.Configure<SnapshotCollectorConfiguration>(Configuration.GetSection(nameof(SnapshotCollectorConfiguration)));

           // Add SnapshotCollector telemetry processor.
           services.AddSingleton<ITelemetryProcessorFactory>(sp => new SnapshotCollectorTelemetryProcessorFactory(sp));

           // TODO: Add other services your application needs here.
       }
   }
   ```

4. 在 appsettings.json 中新增 SnapshotCollectorConfiguration 區段以設定快照集收集器。 例如︰

   ```json
   {
     "ApplicationInsights": {
       "InstrumentationKey": "<your instrumentation key>"
     },
     "SnapshotCollectorConfiguration": {
       "IsEnabledInDeveloperMode": false,
       "ThresholdForSnapshotting": 1,
       "MaximumSnapshotsRequired": 3,
       "MaximumCollectionPlanSize": 50,
       "ReconnectInterval": "00:15:00",
       "ProblemCounterResetInterval":"1.00:00:00",
       "SnapshotsPerTenMinutesLimit": 1,
       "SnapshotsPerDayLimit": 30,
       "SnapshotInLowPriorityThread": true,
       "ProvideAnonymousTelemetry": true,
       "FailedRequestLimit": 3
     }
   }
   ```

### <a name="configure-snapshot-collection-for-other-net-applications"></a>設定其他 .NET 應用程式的快照集集合

1. 如果您的應用程式尚未使用 Application Insights 進行檢測，請從[啟用 Application Insights 及設定檢測金鑰](app-insights-windows-desktop.md)著手。

2. 在您的應用程式中新增 [Microsoft.ApplicationInsights.SnapshotCollector](http://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) NuGet 套件。

3. 快照集只會收集向 Application Insights 回報的例外狀況。 您可能需要修改程式碼才能回報例外狀況。 例外狀況處理程式碼取決於應用程式的結構，但有一個範例如下：
    ```csharp
   TelemetryClient _telemetryClient = new TelemetryClient();

   void ExampleRequest()
   {
        try
        {
            // TODO: Handle the request.
        }
        catch (Exception ex)
        {
            // Report the exception to Application Insights.
            _telemetryClient.TrackException(ex);

            // TODO: Rethrow the exception if desired.
        }
   }
    ```

## <a name="grant-permissions"></a>授與權限

Azure 訂用帳戶的擁有者可以檢查快照集。 其他使用者必須由擁有者授與權限。

若要授與權限，請指派 `Application Insights Snapshot Debugger` 角色給要檢查快照集的使用者。 這個角色可以由目標 Application Insights 資源或其資源群組或訂用帳戶的訂用帳戶擁有者，指派給個別使用者或群組。

1. 在 Azure 入口網站中巡覽到 Application Insights 資源。
1. 按一下 [存取控制 (IAM)]。
1. 按一下 [+新增] 按鈕。
1. 從 [角色] 下拉式清單中選取 [Application Insights 快照集偵錯工具]。
1. 搜尋並輸入要新增的使用者名稱。
1. 按一下 [儲存] 按鈕，將使用者新增至角色。


> [!IMPORTANT]
> 快照集可能會在變數和參數值中包含個人和其他機密資訊。

## <a name="debug-snapshots-in-the-application-insights-portal"></a>Application Insights 入口網站中的偵錯快照集

如果快照集可用於指定的例外狀況或問題識別碼，則 Application Insights 入口網站中的[例外狀況](app-insights-asp-net-exceptions.md)會出現 [開啟偵錯快照集] 按鈕。

![例外狀況的 [開啟偵錯快照集] 按鈕](./media/app-insights-snapshot-debugger/snapshot-on-exception.png)

在 [偵錯快照集] 檢視中，您會看到呼叫堆疊和變數窗格。 當您在 [呼叫堆疊] 窗格中選取呼叫堆疊的框架時，您可以檢視 [變數] 窗格中的本機變數和該函式呼叫的參數。

![檢視入口網站中的偵錯快照集](./media/app-insights-snapshot-debugger/open-snapshot-portal.png)

快照集可能包含機密資訊，依預設為不可檢視。 若要檢視快照集，您必須有指派給您的 `Application Insights Snapshot Debugger` 角色。

## <a name="debug-snapshots-with-visual-studio-2017-enterprise"></a>Visual Studio 2017 Enterprise 的偵錯快照集
1. 按一下 [下載快照集] 按鈕，下載可利用 Visual Studio 2017 Enterprise 開啟的 `.diagsession` 檔案。

2. 若要開啟 `.diagsession` 檔案，您必須先[下載並安裝 Visual Studio 的快照集偵錯工具擴充功能](https://aka.ms/snapshotdebugger)。

3. 開啟快照集檔案之後，Visual Studio 中的 [小型傾印偵錯] 分頁隨即出現。 按一下 [偵錯受控碼] 以開始偵錯快照集。 快照集會開啟至擲回例外狀況的程式碼行，您可將程序的目前狀態進行偵錯。

    ![檢視 Visual Studio 中的偵錯快照集](./media/app-insights-snapshot-debugger/open-snapshot-visualstudio.png)

下載的快照集會包含 Web 應用程式伺服器上找到的任何符號檔。 若要建立快照集資料與原始程式碼的關聯，就需要這些符號檔。 對於 App Service 應用程式，當您發佈 Web 應用程式時請務必啟用符號部署。

## <a name="how-snapshots-work"></a>快照集運作方式

快照集收集器會實作為 [Application Insights 遙測處理器](app-insights-configuration-with-applicationinsights-config.md#telemetry-processors-aspnet)。 當您的應用程式執行時，快照集收集器的遙測處理器會新增至您應用程式的遙測管線中。
每次應用程式呼叫 [TrackException](app-insights-asp-net-exceptions.md#exceptions) 時，快照集收集器會根據所擲回例外狀況的類型和擲回方法計算問題識別碼。
每次應用程式呼叫 TrackException 時，適當問題識別碼的計數器即會遞增。 當計數器達到 `ThresholdForSnapshotting` 值時，問題識別碼就會新增至收集計畫。

快照集收集器還會藉由訂閱 [AppDomain.CurrentDomain.FirstChanceException](https://docs.microsoft.com/dotnet/api/system.appdomain.firstchanceexception) 事件來監視擲回的例外狀況。 當該事件引發時，系統會計算例外狀況的問題識別碼，並將其與收集計畫中的問題識別碼進行比較。
如果沒有相符項目，則會建立執行中處理序的快照集。 快照集獲指派一個唯一的識別碼，而例外狀況則標有該識別碼的戳記。 FirstChanceException 處理常式返回之後，所擲回的例外狀況會以正常方式處理。 最後，例外狀況會再次到達 TrackException 方法，它將在其中與快照集識別碼一起回報給 Application Insights。

主要處理序會繼續執行，並在很少中斷的情況下提供流量給使用者。 同時，快照集會遞交給快照集上傳程式處理序。 接著，快照集上傳程式會建立小型傾印，並將其與任何相關符號檔 (.pdb) 一起上傳至 Application Insights。

> [!TIP]
> - 處理序快照集是執行中處理序的已暫止複製品。
> - 建立快照集大約需要 10 到 20 毫秒的時間。
> - `ThresholdForSnapshotting` 的預設值為 1。 這也是最小值。 因此，在建立快照集之前，您的應用程式必須觸發相同的例外狀況**兩次**。
> - 如果您想要在 Visual Studio 中進行偵錯時產生快照集，請將 `IsEnabledInDeveloperMode` 設定為 true。
> - 快照集建立速率受限於 `SnapshotsPerTenMinutesLimit` 設定。 根據預設，限制為每隔十分鐘建立一個快照集。
> - 每日可上傳的快照集不得超過 50 個。

## <a name="current-limitations"></a>目前的限制

### <a name="publish-symbols"></a>發佈符號
快照集偵錯工具需要符號檔出現在生產環境伺服器上，才可將變數解碼並提供 Visual Studio 中的偵錯體驗。
Visual Studio 2017 的 15.2 版 (或更新版本) 在發佈至 App Service 時，依預設會發佈版本組建的符號。 在舊版中，您必須將下列這一行新增至發佈設定檔 `.pubxml` 檔案，才會在發行模式中將符號發佈：

```xml
    <ExcludeGeneratedDebugSymbol>False</ExcludeGeneratedDebugSymbol>
```

針對 Azure Compute 和其他類型，請確定符號檔案與主要應用程式 .dll 位於相同資料夾 (通常為 `wwwroot/bin`)，或可在目前的路徑使用。

### <a name="optimized-builds"></a>最佳化的組建
在某些情況下，由於 JIT 編譯器所套用的最佳化，使版本組建無法檢視本機變數。
不過，在 Azure 應用程式服務中，快照集收集器可以將屬於其收集計劃一部分的擲回方法取消最佳化。

> [!TIP]
> 在應用程式服務中安裝 Application Insights 網站延伸模組，以取得取消最佳化支援。

## <a name="troubleshooting"></a>疑難排解

這些提示可協助您針對快照集偵錯工具的問題進行疑難排解。

### <a name="use-the-snapshot-health-check"></a>使用快照集健康情況檢查
一些常見的問題會導致不會顯示開啟偵錯快照集。 例如，使用過期的快照集收集器；達到每日上傳限制；或者快照集可能會花很長的時間來上傳。 請使用快照集健康情況檢查，針對常見的問題進行疑難排解。

端對端追蹤檢視的例外狀況窗格中有一個連結，可帶領您前往快照集健康情況檢查。

![輸入快照集健康情況檢查](./media/app-insights-snapshot-debugger/enter-snapshot-health-check.png)

互動式的聊天型介面可尋找常見的問題，並引導您修正它們。

![健康情況檢查](./media/app-insights-snapshot-debugger/healthcheck.png)

如果這樣無法解決問題，則請參閱下列手動疑難排解步驟。

### <a name="verify-the-instrumentation-key"></a>驗證檢測金鑰

請確定您在已發佈的應用程式中使用正確的檢測金鑰。 通常，會從 ApplicationInsights.config 檔案中讀取檢測金鑰。 請確認此值與您在入口網站中看到之 Application Insights 資源的檢測金鑰相同。

### <a name="upgrade-to-the-latest-version-of-the-nuget-package"></a>升級至最新版本的 NuGet 套件

請使用 Visual Studio 的 NuGet 套件管理員，以確定您使用的是最新版的 Microsoft.ApplicationInsights.SnapshotCollector。 如需版本資訊，請參閱 https://github.com/Microsoft/ApplicationInsights-Home/issues/167

### <a name="check-the-uploader-logs"></a>請檢查上傳程式記錄檔

建立快照集之後，磁碟上會建立小型傾印檔案 (.dmp)。 個別的上傳程式處理序會建立該小型傾印檔案，並將它 (以及任何相關聯的 PDB) 上傳至 Application Insights 快照集偵錯工具儲存體。 成功上傳小型傾印之後，它就會從磁碟中刪除。 上傳程式處理序的記錄檔會保留在磁碟上。 在 App Service 環境中，您可以在 `D:\Home\LogFiles` 中找到這些記錄。 使用 App Service 的 Kudu 管理網站來尋找這些記錄檔。

1. 在 Azure 入口網站中開啟您的 App Service 應用程式。
2. 按一下 [進階工具]，或搜尋 **Kudu**。
3. 按一下 [執行]。
4. 在 [偵錯主控台] 下拉式清單方塊中，選取 [CMD]。
5. 按一下 [LogFiles]。

您應會看到至少有一個檔案的名稱開頭為 `Uploader_` 或 `SnapshotUploader_` 且副檔名為 `.log`。 按一下適當的圖示，以下載任何記錄檔，或在瀏覽器中開啟它們。
檔案名稱包含可識別 App Service 執行個體的唯一尾碼。 如果 App Service 執行個體裝載於一部以上的電腦，每部電腦會有個別的記錄檔。 當上傳程式偵測到新的小型傾印檔案時，該檔案會記錄在記錄檔中。 以下是成功快照集和上傳的範例︰

```
SnapshotUploader.exe Information: 0 : Received Fork request ID 139e411a23934dc0b9ea08a626db16c5 from process 6368 (Low pri)
    DateTime=2018-03-09T01:42:41.8571711Z
SnapshotUploader.exe Information: 0 : Creating minidump from Fork request ID 139e411a23934dc0b9ea08a626db16c5 from process 6368 (Low pri)
    DateTime=2018-03-09T01:42:41.8571711Z
SnapshotUploader.exe Information: 0 : Dump placeholder file created: 139e411a23934dc0b9ea08a626db16c5.dm_
    DateTime=2018-03-09T01:42:41.8728496Z
SnapshotUploader.exe Information: 0 : Dump available 139e411a23934dc0b9ea08a626db16c5.dmp
    DateTime=2018-03-09T01:42:45.7525022Z
SnapshotUploader.exe Information: 0 : Successfully wrote minidump to D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp
    DateTime=2018-03-09T01:42:45.7681360Z
SnapshotUploader.exe Information: 0 : Uploading D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp, 214.42 MB (uncompressed)
    DateTime=2018-03-09T01:42:45.7681360Z
SnapshotUploader.exe Information: 0 : Upload successful. Compressed size 86.56 MB
    DateTime=2018-03-09T01:42:59.6184651Z
SnapshotUploader.exe Information: 0 : Extracting PDB info from D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp.
    DateTime=2018-03-09T01:42:59.6184651Z
SnapshotUploader.exe Information: 0 : Matched 2 PDB(s) with local files.
    DateTime=2018-03-09T01:42:59.6809606Z
SnapshotUploader.exe Information: 0 : Stamp does not want any of our matched PDBs.
    DateTime=2018-03-09T01:42:59.8059929Z
SnapshotUploader.exe Information: 0 : Deleted D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp
    DateTime=2018-03-09T01:42:59.8530649Z
```

> [!NOTE]
> 上述範例來自於 1.2.0 版的 Microsoft.ApplicationInsights.SnapshotCollector NuGet 套件。 在較舊的版本中，上傳程式程序稱為 `MinidumpUploader.exe`，且記錄較不詳細。

在上述範例中，檢測金鑰為 `c12a605e73c44346a984e00000000000`。 這個值應該符合您應用程式的檢測金鑰。
小型傾印會與識別碼為 `139e411a23934dc0b9ea08a626db16c5` 的快照集相關聯。 您稍後可以使用這個識別碼，在 Application Insights Analytics 中找出相關聯的例外狀況遙測。

上載程式約每隔 15 分鐘掃描一次新的 PDB。 以下是範例：

```
SnapshotUploader.exe Information: 0 : PDB rescan requested.
    DateTime=2018-03-09T01:47:19.4457768Z
SnapshotUploader.exe Information: 0 : Scanning D:\home\site\wwwroot for local PDBs.
    DateTime=2018-03-09T01:47:19.4457768Z
SnapshotUploader.exe Information: 0 : Local PDB scan complete. Found 2 PDB(s).
    DateTime=2018-03-09T01:47:19.4614027Z
SnapshotUploader.exe Information: 0 : Deleted PDB scan marker : D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\6368.pdbscan
    DateTime=2018-03-09T01:47:19.4614027Z
```

若為「未」裝載於 App Service 中的應用程式，上傳程式記錄位於與小型傾印相同的資料夾中：`%TEMP%\Dumps\<ikey>` (其中 `<ikey>` 是您的檢測金鑰)。

### <a name="troubleshooting-cloud-services"></a>針對雲端服務進行疑難排解
針對雲端服務中的角色，預設暫存資料夾可能太小，無法保存小型傾印檔案，進而導致遺失快照集。
所需的空間取決於您應用程式的總工作集以及並行快照集數目。
32 位元 ASP.NET Web 角色的工作集一般介於 200 MB 與 500 MB 之間。
允許至少兩個並行快照集。
例如，如果您的應用程式使用 1 GB 的總工作集，則應確定至少有 2 GB 的磁碟空間可儲存快照集。
請遵循下列步驟，以使用快照集的專用本機資源來設定您的雲端服務角色。

1. 編輯雲端服務定義 (.csdef) 檔案，以將新的本機資源新增至雲端服務。 下列範例定義稱為 `SnapshotStore` 且大小為 5 GB 的資源。
   ```xml
   <LocalResources>
     <LocalStorage name="SnapshotStore" cleanOnRoleRecycle="false" sizeInMB="5120" />
   </LocalResources>
   ```

2. 修改角色的啟動程式碼，新增指向 `SnapshotStore` 本機資源的環境變數。 對於背景工作角色，程式碼應新增至您角色的 `OnStart` 方法：
   ```csharp
   public override bool OnStart()
   {
       Environment.SetEnvironmentVariable("SNAPSHOTSTORE", RoleEnvironment.GetLocalResource("SnapshotStore").RootPath);
       return base.OnStart();
   }
   ```
   對於 Web 角色 (ASP.NET)，程式碼應新增至您 Web 應用程式的 `Application_Start` 方法：
   ```csharp
   using Microsoft.WindowsAzure.ServiceRuntime;
   using System;

   namespace MyWebRoleApp
   {
       public class MyMvcApplication : System.Web.HttpApplication
       {
          protected void Application_Start()
          {
             Environment.SetEnvironmentVariable("SNAPSHOTSTORE", RoleEnvironment.GetLocalResource("SnapshotStore").RootPath);
             // TODO: The rest of your application startup code
          }
       }
   }
   ```

3. 更新您角色的 ApplicationInsights.config 檔案，以覆寫 `SnapshotCollector` 所使用的暫存資料夾位置
   ```xml
   <TelemetryProcessors>
    <Add Type="Microsoft.ApplicationInsights.SnapshotCollector.SnapshotCollectorTelemetryProcessor, Microsoft.ApplicationInsights.SnapshotCollector">
      <!-- Use the SnapshotStore local resource for snapshots -->
      <TempFolder>%SNAPSHOTSTORE%</TempFolder>
      <!-- Other SnapshotCollector configuration options -->
    </Add>
   </TelemetryProcessors>
   ```

### <a name="overriding-the-shadow-copy-folder"></a>覆寫陰影複製資料夾

快照集收集器啟動時，它會嘗試在磁碟上尋找適合用來執行快照集上傳程式程序的資料夾。 選擇的資料夾稱為陰影複製資料夾。

快照集收集器會檢查幾個已知的位置，確定它有權複製快照集上傳程式二進位檔。 使用的環境變數如下：
- Fabric_Folder_App_Temp
- LOCALAPPDATA
- APPDATA
- TEMP

如果找不到適合的資料夾，快照集收集器會報告錯誤，指出「找不到適合的陰影複製資料夾。」

如果複製失敗，快照集收集器會報告 `ShadowCopyFailed` 錯誤。

如果上傳程式無法啟動，快照集收集器會報告 `UploaderCannotStartFromShadowCopy` 錯誤。 訊息內文通常會包含 `System.UnauthorizedAccessException`。 之所以發生此錯誤，通常是因為應用程式執行於權限降低的帳戶下。 該帳戶有權寫入陰影複製資料夾，但沒有執行程式碼的權限。

這些錯誤通常是在啟動期間發生的，因此通常會尾隨 `ExceptionDuringConnect` 錯誤，指出「上傳程式無法啟動。」

若要解決這些錯誤，您可以透過 `ShadowCopyFolder` 組態選項以手動方式指定陰影複製資料夾。 例如，使用 ApplicationInsights.config：

   ```xml
   <TelemetryProcessors>
    <Add Type="Microsoft.ApplicationInsights.SnapshotCollector.SnapshotCollectorTelemetryProcessor, Microsoft.ApplicationInsights.SnapshotCollector">
      <!-- Override the default shadow copy folder. -->
      <ShadowCopyFolder>D:\SnapshotUploader</ShadowCopyFolder>
      <!-- Other SnapshotCollector configuration options -->
    </Add>
   </TelemetryProcessors>
   ```

或者，如果您搭配使用 appsettings.json 與 .NET Core 應用程式：

   ```json
   {
     "ApplicationInsights": {
       "InstrumentationKey": "<your instrumentation key>"
     },
     "SnapshotCollectorConfiguration": {
       "ShadowCopyFolder": "D:\\SnapshotUploader"
     }
   }
   ```

### <a name="use-application-insights-search-to-find-exceptions-with-snapshots"></a>使用 Application Insights 搜尋來尋找快照集例外狀況的

建立快照集後，擲回中的例外狀況會以快照集識別碼標記。 向 Application Insights 回報例外狀況遙測後，該快照集識別碼就會包含為自訂屬性。 使用 Application Insights 中的 [搜尋] 刀鋒視窗，您可以找到具有 `ai.snapshot.id` 自訂屬性的所有遙測。

1. 在 Azure 入口網站中瀏覽至您的 Application Insights 資源。
2. 按一下 [搜尋] 。
3. 在 [搜尋] 文字方塊中輸入 `ai.snapshot.id`，然後按 Enter 鍵。

![在入口網站中使用快照集識別碼搜尋遙測](./media/app-insights-snapshot-debugger/search-snapshot-portal.png)

如果此搜尋未傳回任何結果，則不會針對所選時間範圍中您的應用程式向 Application Insights 回報任何快照集。

若要搜尋從上傳程式記錄檔中的特定快照集識別碼，請在 [搜尋] 方塊中輸入該識別碼。 如果您找不到已上傳快照集的遙測，請遵循下列步驟：

1. 請驗證檢測金鑰，仔細檢查您查看的是正確的 Application Insights 資源。

2. 使用上傳程式記錄中的時間戳記，調整搜尋的時間範圍篩選條件以涵蓋該時間範圍。

如果仍未看到具有該快照集識別碼的例外狀況，則未向 Application Insights 回報此例外狀況遙測。 如果您的應用程式在採用快照集之後，但回報例外狀況遙測之前損毀，可能會發生這種情況。 在此情況下，檢查 `Diagnose and solve problems` 之下的 App Service 記錄，查看是否發生非預期的重新啟動或未處理的例外狀況。

### <a name="edit-network-proxy-or-firewall-rules"></a>編輯網路 Proxy 或防火牆規則

如果應用程式透過 Proxy 或防火牆連線至網際網路，您可能需要編輯規則以允許應用程式與快照集偵錯工具服務進行通訊。 以下是[快照集偵錯工具使用的 IP 位址和連接埠清單](app-insights-ip-addresses.md#snapshot-debugger)。

## <a name="next-steps"></a>後續步驟

* [在您的程式碼中設定 Snappoint](https://docs.microsoft.com/visualstudio/debugger/debug-live-azure-applications) 以取得快照集，而不需等待例外狀況。
* [診斷 Web Apps 中的例外狀況](app-insights-asp-net-exceptions.md)說明如何讓 Application Insights 看見更多的例外狀況。
* [智慧型偵測](app-insights-proactive-diagnostics.md)會自動探索效能異常。
