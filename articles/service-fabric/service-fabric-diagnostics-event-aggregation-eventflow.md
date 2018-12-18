---
title: 使用 EventFlow 的 Azure Service Fabric 事件彙總 | Microsoft Docs
description: 深入了解使用 EventFlow 的彙總及收集事件，監視和診斷 Azure Service Fabric 叢集。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/15/2017
ms.author: dekapur
ms.openlocfilehash: 9b851b2d75cf78a02dd223788085ac9a0963376e
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34211349"
---
# <a name="event-aggregation-and-collection-using-eventflow"></a>使用 EventFlow 的事件彙總與集合

[Microsoft Diagnostics EventFlow](https://github.com/Azure/diagnostics-eventflow) 可以將節點的事件路由傳送至一個或多個監視目的地。 由於是以 NuGet 套件形式加入服務專案中，EventFlow 程式碼和組態會隨著服務一起傳送，不會發生稍後提及有關 Azure 診斷的個別節點組態問題。 EventFlow 在您的服務處理序中執行，並直接連線到已設定的輸出。 因為是直接連線，所以 EventFlow 適用於 Azure、容器和內部部署的服務部署。 如果您在高密度情況下執行 EventFlow，例如在容器中，請務必小心，因為每個 EventFlow 管線來建立外部連線。 所以，如果您裝載許多處理序，就會產生大量輸出連線！ 這對 Service Fabric 應用程式來說不會有什麼問題，因為 `ServiceType` 的所有複本都在同一處理序中執行，這會限制輸出連線的數量。 EventFlow 也提供事件篩選，因此只會傳送符合指定篩選的事件。

## <a name="set-up-eventflow"></a>設定 EventFlow

EventFlow 二進位檔是以一組 NuGet 套件的形式提供。 若要將 EventFlow 新增到 Service Fabric 服務專案，請在 [方案總管] 中該專案上按一下滑鼠右鍵，然後選擇 [管理 NuGet 套件]。 切換到 [瀏覽] 索引標籤，然後搜尋 "`Diagnostics.EventFlow`"：

![Visual Studio NuGet 套件管理員 UI 中的 EventFlow NuGet 套件](./media/service-fabric-diagnostics-event-aggregation-eventflow/eventflow-nuget.png)

您會看到一份各種各樣套件的清單，標記為「輸入」和「輸出」。 EventFlow 支援各種不同的記錄提供者和分析器。 視應用程式記錄檔的來源和目的地而定，裝載 EventFlow 的服務應該包含適當的套件。 除了核心 ServiceFabric 套件，您還需要設定至少一個輸入和輸出。 例如，您可以新增下列套件，將 EventSource 事件傳送至 Application Insights：

* `Microsoft.Diagnostics.EventFlow.Inputs.EventSource` 從服務的 EventSource 類別以及從標準 EventSources (例如 *Microsoft-ServiceFabric-Services* 和 *Microsoft-ServiceFabric-Actors*) 擷取資料
* `Microsoft.Diagnostics.EventFlow.Outputs.ApplicationInsights` (我們將把記錄檔傳送給 Azure Application Insights 資源)
* `Microsoft.Diagnostics.EventFlow.ServiceFabric` (允許從 Service Fabric 服務設定將 EventFlow 管線初始化，並以 Service Fabric 健康情況報告的形式傳送診斷資料來回報任何問題)

>[!NOTE]
>`Microsoft.Diagnostics.EventFlow.Inputs.EventSource` 套件會要求服務專案的目標必須是 .NET Framework 4.6 或更新版本。 安裝此套件之前，請先確定您已在專案屬性中設定適當的目標架構。

安裝好所有套件之後，下一個步驟就是設定及啟用服務中的 EventFlow。

## <a name="configure-and-enable-log-collection"></a>設定和啟用記錄集合
EventFlow 管線 (負責傳送記錄檔) 是從儲存在組態檔中的規格建立的。 `Microsoft.Diagnostics.EventFlow.ServiceFabric` 套件會在 `PackageRoot\Config` 方案資料夾底下安裝一個起始的 EventFlow 組態檔，名為 `eventFlowConfig.json`。 此組態檔必須經過修改，才能從預設服務 `EventSource` 類別和任何其他您想設定的輸入擷取資料，然後將資料傳送到適當的位置。

>[!NOTE]
>如果您的專案檔案具有 VisualStudio 2017 格式，則 `eventFlowConfig.json` 檔案不會自動新增。 若要修正這個問題，在 `Config` 資料夾中建立檔案，並且將建置動作設為 `Copy if newer`。 

以下是以前述 NuGet 套件為基礎的範例 *eventFlowConfig.json*：
```json
{
  "inputs": [
    {
      "type": "EventSource",
      "sources": [
        { "providerName": "Microsoft-ServiceFabric-Services" },
        { "providerName": "Microsoft-ServiceFabric-Actors" },
        // (replace the following value with your service's ServiceEventSource name)
        { "providerName": "your-service-EventSource-name" }
      ]
    }
  ],
  "filters": [
    {
      "type": "drop",
      "include": "Level == Verbose"
    }
  ],
  "outputs": [
    {
      "type": "ApplicationInsights",
      // (replace the following value with your AI resource's instrumentation key)
      "instrumentationKey": "00000000-0000-0000-0000-000000000000"
    }
  ],
  "schemaVersion": "2016-08-11"
}
```

服務的 ServiceEventSource 名稱是套用至 ServiceEventSource 類別之 `EventSourceAttribute` 的 Name 屬性值。 全部都在 `ServiceEventSource.cs` 檔案中指定，此檔案是服務程式碼的一部分。 例如，在下列程式碼片段中，ServiceEventSource 的名稱是 *MyCompany-Application1-Stateless1*：

```csharp
[EventSource(Name = "MyCompany-Application1-Stateless1")]
internal sealed class ServiceEventSource : EventSource
{
    // (rest of ServiceEventSource implementation)
}
```

請注意，`eventFlowConfig.json` 檔案是服務組態封裝的一部分。 對此檔案所做的變更可以包含在完整或僅限組態的服務升級中，並受到 Service Fabric 升級健康情況檢查和自動復原 (如果發生升級失敗) 控管。 如需詳細資訊，請參閱 [Service Fabric 應用程式升級](service-fabric-application-upgrade.md)。

組態的「篩選」區段可讓您進一步自訂經過 EventFlow 管線到輸出的資訊，讓您卸除或包含特定資訊，或變更事件資料結構。 如需有關篩選的詳細資訊，請參閱 [EventFlow 篩選](https://github.com/Azure/diagnostics-eventflow#filters)。

最後一個步驟是在服務的啟動程式碼 (位於 `Program.cs` 檔案中) 中將 EventFlow 管線具現化：

```csharp
using System;
using System.Diagnostics;
using System.Threading;
using Microsoft.ServiceFabric;
using Microsoft.ServiceFabric.Services.Runtime;

// **** EventFlow namespace
using Microsoft.Diagnostics.EventFlow.ServiceFabric;

namespace Stateless1
{
    internal static class Program
    {
        /// <summary>
        /// This is the entry point of the service host process.
        /// </summary>
        private static void Main()
        {
            try
            {
                // **** Instantiate log collection via EventFlow
                using (var diagnosticsPipeline = ServiceFabricDiagnosticPipelineFactory.CreatePipeline("MyApplication-MyService-DiagnosticsPipeline"))
                {

                    ServiceRuntime.RegisterServiceAsync("Stateless1Type",
                    context => new Stateless1(context)).GetAwaiter().GetResult();

                    ServiceEventSource.Current.ServiceTypeRegistered(Process.GetCurrentProcess().Id, typeof(Stateless1).Name);

                    Thread.Sleep(Timeout.Infinite);
                }
            }
            catch (Exception e)
            {
                ServiceEventSource.Current.ServiceHostInitializationFailed(e.ToString());
                throw;
            }
        }
    }
}
```

當作 `ServiceFabricDiagnosticsPipelineFactory` 之 `CreatePipeline` 方法的參數來傳遞的名稱是「健康情況實體」的名稱，此實體代表 EventFlow 記錄檔收集管線。 當 EventFlow 發生錯誤並透過 Service Fabric 健康情況子系統回報錯誤時，就會使用此名稱。

### <a name="use-service-fabric-settings-and-application-parameters-in-eventflowconfig"></a>在 eventFlowConfig 中使用 Service Fabric 設定和應用程式參數

EventFlow 支援使用 Service Fabric 設定和應用程式參數來設定 EventFlow 設定。 您可以參考使用這個特殊的語法值的 Service Fabric 設定參數：

```json
servicefabric:/<section-name>/<setting-name>
```

`<section-name>` 是Service Fabric 組態區段的名稱，`<setting-name>` 是提供設定 EventFlow 設定所用值的組態設定。 若要深入了解如何作業，請移至 [Service Fabric 設定和應用程式參數的支援](https://github.com/Azure/diagnostics-eventflow#support-for-service-fabric-settings-and-application-parameters)。

## <a name="verification"></a>驗證

啟動您的服務並觀察 Visual Studio 中的 [偵錯] 輸出視窗。 啟動服務之後，您應該會開始看到服務在向您設定的輸出傳送記錄。 瀏覽至您的事件分析和視覺化平台，確認記錄檔已開始顯示 (可能需要幾分鐘的時間)。

## <a name="next-steps"></a>後續步驟

* [使用 Application Insights 進行事件分析和視覺效果](service-fabric-diagnostics-event-analysis-appinsights.md)
* [使用 Log Analytics 進行事件分析和視覺效果](service-fabric-diagnostics-event-analysis-oms.md)
* [EventFlow 文件](https://github.com/Azure/diagnostics-eventflow)
