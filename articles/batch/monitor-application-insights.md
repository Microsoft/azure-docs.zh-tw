---
title: 使用 Azure Application Insights 監視 Batch | Microsoft Docs
description: 了解如何使用 Azure Application Insights 程式庫檢測 Azure Batch .NET 應用程式。
services: batch
author: dlepow
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: .NET
ms.topic: article
ms.workload: na
ms.date: 04/05/2018
ms.author: danlep
ms.openlocfilehash: 5e0358ebf525c39c09df4268971fa71c02457821
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35766823"
---
# <a name="monitor-and-debug-an-azure-batch-net-application-with-application-insights"></a>使用 Application Insights 監視 Azure Batch .NET 應用程式並進行偵錯

[Application Insights](../application-insights/app-insights-overview.md) 提供開發人員精緻且強大的方法，用來監視部署到 Azure 服務的應用程式並進行偵錯。 使用 Application Insights 監視效能計數器和例外狀況，以及透過自訂計量與追蹤來檢測您的程式碼。 整合 Application Insights 與 Azure Batch 應用程式可讓您取得行為的深入見解，並幾近即時地調查問題。

本文會示範如何將 Application Insights 程式庫新增至您的 Azure Batch .NET 解決方案並加以設定，以及檢測應用程式的程式碼。 同時示範透過 Azure 入口網站監視應用程式和建置自訂儀表板的方法。 如需了解其他語言中的 Application Insights 支援，請查看[語言、平台及整合文件](../application-insights/app-insights-platforms.md)。

隨附於本文的 C# 解決方案範例與程式碼可於 [GitHub](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ApplicationInsights) 中取得。 此範例會將 Application Insights 檢測程式碼新增至 [TopNWords](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/TopNWords) 範例。 如果您不熟悉該範例，請先嘗試建置及執行 TopNWords。 這麼做可協助您了解在多個計算節點上平行處理一組輸入 Blob 的基本 Batch 工作流程。 

## <a name="prerequisites"></a>必要條件
* [Visual Studio 2017](https://www.visualstudio.com/vs)

* [Batch 帳戶和連結的 Azure 儲存體帳戶](batch-account-create-portal.md)

* [Application Insights 資源](../application-insights/app-insights-create-new-resource.md)
  
   * 使用 Azure 入口網站建立 Application Insights 資源。 選取「一般」**應用程式類型**。

   * 從入口網站複製[檢測金鑰](../application-insights/app-insights-create-new-resource.md#copy-the-instrumentation-key)。 本文稍後會需要此項目。
  
  > [!NOTE]
  > 您可能需要為儲存在 Application Insights 中的資料[支付費用](https://azure.microsoft.com/pricing/details/application-insights/)。 這其中包括本文討論的診斷和監視記錄。
  > 

## <a name="add-application-insights-to-your-project"></a>將 Application Insights 加入至專案

您的專案需要 **Microsoft.ApplicationInsights.WindowsServer** NuGet 套件和其相依項目。 可將其新增或還原到您應用程式的專案。 若要安裝套件，請使用 `Install-Package` 命令或 NuGet 套件管理員。

```powershell
Install-Package Microsoft.ApplicationInsights.WindowsServer
```
您可以使用 **Microsoft.ApplicationInsights** 命名空間從您的 .NET 應用程式參考 Application Insights。

## <a name="instrument-your-code"></a>實作您的程式碼

若要檢測您的程式碼，您的解決方案必須建立 Application Insights [TelemetryClient](/dotnet/api/microsoft.applicationinsights.telemetryclient)。 在範例中，TelemetryClient 會從 [ApplicationInsights.config](../application-insights/app-insights-configuration-with-applicationinsights-config.md) 檔案載入其組態。 請務必使用您的 Application Insights 檢測金鑰來更新下列專案中的 ApplicationInsights.config：Microsoft.Azure.Batch.Samples.TelemetryStartTask and TopNWordsSample。

```xml
<InstrumentationKey>YOUR-IKEY-GOES-HERE</InstrumentationKey>
```
也請將檢測金鑰新增至 TopNWords.cs 檔案。

TopNWords.cs 中的範例會使用 Application Insights API 中的下列[檢測呼叫](../application-insights/app-insights-api-custom-events-metrics.md)：
* `TrackMetric()` - 追蹤計算節點下載必要文字檔案要多久時間 (平均值)。
* `TrackTrace()` - 將偵錯呼叫新增至您的程式碼。
* `TrackEvent()` - 追蹤要擷取的關注事件。

此範例刻意省去例外狀況處理。 改為 Application Insights 會自動報告未處理的例外狀況，進而大幅改善偵錯體驗。 

下列程式碼片段說明如何使用這些方法。

```csharp
public void CountWords(string blobName, int numTopN, string storageAccountName, string storageAccountKey)
{
    // simulate exception for some set of tasks
    Random rand = new Random();
    if (rand.Next(0, 10) % 10 == 0)
    {
        blobName += ".badUrl";
    }

    // log the url we are downloading the file from
    insightsClient.TrackTrace(new TraceTelemetry(string.Format("Task {0}: Download file from: {1}", this.taskId, blobName), SeverityLevel.Verbose));

    // open the cloud blob that contains the book
    var storageCred = new StorageCredentials(storageAccountName, storageAccountKey);
    CloudBlockBlob blob = new CloudBlockBlob(new Uri(blobName), storageCred);
    using (Stream memoryStream = new MemoryStream())
    {
        // calculate blob download time
        DateTime start = DateTime.Now;
        blob.DownloadToStream(memoryStream);
        TimeSpan downloadTime = DateTime.Now.Subtract(start);

        // track how long the blob takes to download on this node
        // this will help debug timing issues or identify poorly performing nodes
        insightsClient.TrackMetric("Blob download in seconds", downloadTime.TotalSeconds, this.CommonProperties);

        memoryStream.Position = 0; //Reset the stream
        var sr = new StreamReader(memoryStream);
        var myStr = sr.ReadToEnd();
        string[] words = myStr.Split(' ');

        // log how many words were found in the text file
        insightsClient.TrackTrace(new TraceTelemetry(string.Format("Task {0}: Found {1} words", this.taskId, words.Length), SeverityLevel.Verbose));
        var topNWords =
            words.
                Where(word => word.Length > 0).
                GroupBy(word => word, (key, group) => new KeyValuePair<String, long>(key, group.LongCount())).
                OrderByDescending(x => x.Value).
                Take(numTopN).
                ToList();
        foreach (var pair in topNWords)
        {
            Console.WriteLine("{0} {1}", pair.Key, pair.Value);
        }

        // emit an event to track the completion of the task
        insightsClient.TrackEvent("Done counting words");
    }
}
```

### <a name="azure-batch-telemetry-initializer-helper"></a>Azure Batch 遙測初始設定式的協助程式
報告指定伺服器和執行個體的遙測時，Application Insights 會使用 Azure VM 角色和 VM 名稱作為預設值。 在 Azure Batch 的執行內容中，此範例會改為示範如何使用集區名稱和計算節點名稱。 使用[遙測初始設定式](../application-insights/app-insights-api-filtering-sampling.md#add-properties)覆寫預設值。 

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;
using System;
using System.Threading;

namespace Microsoft.Azure.Batch.Samples.TelemetryInitializer
{
    public class AzureBatchNodeTelemetryInitializer : ITelemetryInitializer
    {
        // Azure Batch environment variables
        private const string PoolIdEnvironmentVariable = "AZ_BATCH_POOL_ID";
        private const string NodeIdEnvironmentVariable = "AZ_BATCH_NODE_ID";

        private string roleInstanceName;
        private string roleName;

        public void Initialize(ITelemetry telemetry)
        {
            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleName))
            {
                // override the role name with the Azure Batch Pool name
                string name = LazyInitializer.EnsureInitialized(ref this.roleName, this.GetPoolName);
                telemetry.Context.Cloud.RoleName = name;
            }

            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleInstance))
            {
                // override the role instance with the Azure Batch Compute Node name
                string name = LazyInitializer.EnsureInitialized(ref this.roleInstanceName, this.GetNodeName);
                telemetry.Context.Cloud.RoleInstance = name;
            }
        }

        private string GetPoolName()
        {
            return Environment.GetEnvironmentVariable(PoolIdEnvironmentVariable) ?? string.Empty;
        }

        private string GetNodeName()
        {
            return Environment.GetEnvironmentVariable(NodeIdEnvironmentVariable) ?? string.Empty;
        }
    }
}
```

若要啟用遙測初始設定式，TopNWordsSample 專案中的 ApplicationInsights.config 檔案包含下列內容：

```xml
<TelemetryInitializers>
    <Add Type="Microsoft.Azure.Batch.Samples.TelemetryInitializer.AzureBatchNodeTelemetryInitializer, Microsoft.Azure.Batch.Samples.TelemetryInitializer"/>
</TelemetryInitializers>
``` 

## <a name="update-the-job-and-tasks-to-include-application-insights-binaries"></a>更新作業和工作，以包含 Application Insights 二進位檔

為了讓 Application Insights 在節點上正確地執行，請確定二進位檔位於正確位置。 請將所需的二進位檔新增至您工作資源檔案的集合，使其能在工作執行時加以下載。 下列程式碼片段類似於 Job.cs 中的程式碼。

首先，建立要上傳的 Application Insights 檔案靜態清單。

```csharp
private static readonly List<string> AIFilesToUpload = new List<string>()
{
    // Application Insights config and assemblies
    "ApplicationInsights.config",
    "Microsoft.ApplicationInsights.dll",
    "Microsoft.AI.Agent.Intercept.dll",
    "Microsoft.AI.DependencyCollector.dll",
    "Microsoft.AI.PerfCounterCollector.dll",
    "Microsoft.AI.ServerTelemetryChannel.dll",
    "Microsoft.AI.WindowsServer.dll",
    
    // custom telemetry initializer assemblies
    "Microsoft.Azure.Batch.Samples.TelemetryInitializer.dll",
 };
...
```

接下來，建立工作要使用的暫存檔案。
```csharp
...
// create file staging objects that represent the executable and its dependent assembly to run as the task.
// These files are copied to every node before the corresponding task is scheduled to run on that node.
FileToStage topNWordExe = new FileToStage(TopNWordsExeName, stagingStorageAccount);
FileToStage storageDll = new FileToStage(StorageClientDllName, stagingStorageAccount);

// Upload Application Insights assemblies
List<FileToStage> aiStagedFiles = new List<FileToStage>();
foreach (string aiFile in AIFilesToUpload)
{
    aiStagedFiles.Add(new FileToStage(aiFile, stagingStorageAccount));
}
...
```

`FileToStage` 方法是程式碼範例中的協助程式函式，可讓您輕鬆地將檔案從本機磁碟中上傳至 Azure 儲存體 Blob。 稍後每個檔都會下載到計算節點，並且讓工作參考。

最後，將工作新增至作業，並包含必要的 Application Insights 二進位檔。
```csharp
...
// initialize a collection to hold the tasks that will be submitted in their entirety
List<CloudTask> tasksToRun = new List<CloudTask>(topNWordsConfiguration.NumberOfTasks);
for (int i = 1; i <= topNWordsConfiguration.NumberOfTasks; i++)
{
    CloudTask task = new CloudTask("task_no_" + i, String.Format("{0} --Task {1} {2} {3} {4}",
        TopNWordsExeName,
        string.Format("https://{0}.blob.core.windows.net/{1}",
            accountSettings.StorageAccountName,
            documents[i]),
        topNWordsConfiguration.TopWordCount,
        accountSettings.StorageAccountName,
        accountSettings.StorageAccountKey));

    //This is the list of files to stage to a container -- for each job, one container is created and 
    //files all resolve to Azure Blobs by their name (so two tasks with the same named file will create just 1 blob in
    //the container).
    task.FilesToStage = new List<IFileStagingProvider>
                        {
                            // required application binaries
                            topNWordExe,
                            storageDll,
                        };
    foreach (FileToStage stagedFile in aiStagedFiles)
   {
        task.FilesToStage.Add(stagedFile);
   }    
    task.RunElevated = false;
    tasksToRun.Add(task);
}
```

## <a name="view-data-in-the-azure-portal"></a>在 Azure 入口網站中檢視資料

現在您已設定了使用 Application Insights 的作業和工作，接下來請在工作集區中執行範例。 瀏覽至 Azure 入口網站，並開啟您佈建的 Application Insights 資源。 集區佈建好之後，您應會開始看到資料的流動與記錄。 本文其餘部分只會說明幾個 Application Insights 功能，但您可以隨意地探索完整的功能集。

### <a name="view-live-stream-data"></a>檢視即時資料流

若要檢視 Application Insights 資源中的追蹤記錄，請按一下 [即時資料流]。 下列螢幕擷取畫面說明如何檢視來自集區中計算節點的即時資料，例如每個計算節點的 CPU 使用量。

![即時資料流計算節點資料](./media/monitor-application-insights/applicationinsightslivestream.png)

### <a name="view-trace-logs"></a>檢視追蹤記錄

若要檢視 Application Insights 資源中的追蹤記錄，請按一下 [搜尋]。 此檢視會顯示 Application Insights 所擷取的診斷資料清單，其中包括追蹤、事件和例外狀況。 

下列螢幕擷取畫面說明單一工作追蹤的記錄方式，以及之後進行偵錯所需的查詢方式。

![追蹤記錄影像](./media/monitor-application-insights/tracelogsfortask.png)

### <a name="view-unhandled-exceptions"></a>檢視未處理的例外狀況

下列螢幕擷取畫面說明 Application Insights 如何記錄您應用程式擲回的例外狀況。 在此案例中，在應用程式擲回例外狀況的幾秒內，您就可以深入了解特定例外狀況並診斷問題。

![未處理的例外狀況](./media/monitor-application-insights/exception.png)

### <a name="measure-blob-download-time"></a>測量 Blob 下載時間

自訂計量也是入口網站中的實用工具。 例如，您可以顯示每個計算節點下載所處理的必要文字檔案時，所花費的平均時間。

建立範例圖表：
1. 在 Application Insights 資源中，按一下 [計量瀏覽器] > [新增圖表]。
2. 在新增的圖表上按一下 [編輯]。
2. 更新圖表的詳細資料，如下所示：
   * 將 [圖表類型] 設定為 [方格]。
   * 將 [彙總] 設定為 [平均]。
   * 將 [分組依據] 設定為 [NodeId]。
   * 在 [計量] 中，選取 [自訂] > [Blob 下載 (以秒為單位)]。
   * 您可以依喜好調整顯示的 [色彩調色盤]。 

![每個節點的 Blob 下載時間](./media/monitor-application-insights/blobdownloadtime.png)


## <a name="monitor-compute-nodes-continuously"></a>持續監視計算節點

您可能已注意到，只有在工作執行時才會記錄所有計量 (包括效能計數器)。 此行為是非常實用，因為這會限制 Application Insights 記錄的資料量。 不過，在某些情況下，您可能需要一直監視計算節點。 例如，這些計算節點執行的背景工作可能未透過 Batch 服務進行排程。 在此情況下，請針對計算節點的存留期，設定要執行的監視程序。 

要完成此行為的方法之一是繁衍程序來載入 Application Insights 程式庫，並在背景中執行。 在範例中，啟動工作會在機器上載入二進位檔，並會無限期地持續執行處理程序。 設定此處理程序的 Application Insights 組態檔，以發出您關注的其他資料，例如效能計數器。

```csharp
...
 // Batch start task telemetry runner
private const string BatchStartTaskFolderName = "StartTask";
private const string BatchStartTaskTelemetryRunnerName = "Microsoft.Azure.Batch.Samples.TelemetryStartTask.exe";
private const string BatchStartTaskTelemetryRunnerAIConfig = "ApplicationInsights.config";
...
CloudPool pool = client.PoolOperations.CreatePool(
    topNWordsConfiguration.PoolId,
    targetDedicated: topNWordsConfiguration.PoolNodeCount,
    virtualMachineSize: "standard_d1_v2",
    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));
...

// Create a start task which will run a dummy exe in background that simply emits performance
// counter data as defined in the relevant ApplicationInsights.config.
// Note that the waitForSuccess on the start task was not set so the Compute Node will be
// available immediately after this command is run.
pool.StartTask = new StartTask()
{
    CommandLine = string.Format("cmd /c {0}", BatchStartTaskTelemetryRunnerName),
    ResourceFiles = resourceFiles
};
...
```

> [!TIP]
> 若要增加解決方案的可管理性，您可以搭配[應用程式套件](./batch-application-packages.md)中的組件。 然後，若要讓應用程式套件自動部署至您的集區，請將應用程式套件參考新增至集區組態。
>

## <a name="throttle-and-sample-data"></a>節流和範例資料 

由於在生產環境中執行的 Azure Batch 應用程式本質上都是大規模運作，因此建議您藉由限制 Application Insights 所收集的資料量來管理成本。 請參閱 [Application Insights 中的取樣](../application-insights/app-insights-sampling.md)，以了解完成此操作的一些機制。


## <a name="next-steps"></a>後續步驟
* 深入了解 [Application Insights](../application-insights/app-insights-overview.md)。

* 如需了解其他語言中的 Application Insights 支援，請查看[語言、平台及整合文件](../application-insights/app-insights-platforms.md)。


