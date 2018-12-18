---
title: 如何以 WebJobs 的形式執行長期函式 - Azure
description: 了解如何使用 WebJobs SDK，對 Durable Functions 進行編碼並設定為在 WebJobs 中執行。
services: functions
author: ggailey777
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 04/25/2018
ms.author: azfuncdf
ms.openlocfilehash: 94d443505c5c1634c9da00d455e0163b2762d969
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39426021"
---
# <a name="how-to-run-durable-functions-as-webjobs"></a>如何以 WebJobs 的形式執行長期函式

[Azure Functions](functions-overview.md) 和 [Durable Functions](durable-functions-overview.md) 擴充功能是以 [WebJobs SDK](../app-service/web-sites-create-web-jobs.md) 為基礎所建置的。 WebJobs SDK 中的 `JobHost` 是 Azure Functions 中的執行階段。 如果您需要以無法在 Azure Functions 中實現的方式控制 `JobHost` 行為，則可自行使用 WebJobs SDK 來開發及執行長期函式。 然後，您可以在 Azure WebJob 或主控台應用程式可於其中執行的任何位置執行長期函式。

鏈結的 Durable Functions 範例有提供 WebJobs SDK 版本：下載或複製 [Durable Functions 存放庫](https://github.com/azure/azure-functions-durable-extension/)，並瀏覽至 samples\\webjobssdk\\chaining 資料夾。

## <a name="prerequisites"></a>必要條件

本文假設您已熟悉 WebJobs SDK、Azure Functions 的C# 類別庫開發和 Durable Functions 等項目的基本概念。 如果您需要這些主題的簡介，請參閱下列資源：

* [開始使用 WebJobs SDK](../app-service/webjobs-sdk-get-started.md)
* [使用 Visual Studio 建立第一個函數](functions-create-your-first-function-visual-studio.md)
* [Durable Functions](durable-functions-sequence.md)

若要完成本文中的步驟：

* [安裝 Visual Studio 2017 15.6 版或更新版本](https://docs.microsoft.com/visualstudio/install/)與 **Azure 開發**工作負載。

  如果您已有 Visual Studio 但沒有該工作負載，請選取 [工具] > [取得工具和功能] 來新增該工作負載。 

  (您可以改為使用 [Visual Studio Code](https://code.visualstudio.com/)，但某些指示是 Visual Studio 特有的。)

* 安裝並執行 [Azure 儲存體模擬器](../storage/common/storage-use-emulator.md) 5.2 版或更新版本。 替代方法是使用 Azure 儲存體連接字串來更新 App.config 檔案。

## <a name="webjobs-sdk-versions"></a>WebJobs SDK 版本

本文說明如何開發 WebJobs SDK 2.x 專案 (相當於 Azure Functions 1.x 版)。 如需 3.x 版的資訊，請參閱本文稍後的 [WebJobs SDK 3.x](#webjobs-sdk-3x)。 

## <a name="create-console-app"></a>建立主控台應用程式

WebJobs SDK 專案只是安裝了適當 NuGet 套件的主控台應用程式專案。

在 Visual Studio 的 [新增專案] 對話方塊中，選取 [Windows 傳統桌面] > [主控台應用程式] \(.NET Framework\)。 在專案檔中，`TargetFrameworkVersion` 應該是 `v4.6.1`。

Visual Studio 也有 WebJob 專案範本，選取 [雲端] > [Azure WebJob] \(.NET Framework\) 即可使用。 此範本會安裝許多套件，但您可能不需要其中的某些套件。

## <a name="install-nuget-packages"></a>安裝 NuGet 套件

您需要 NuGet 套件中的 WebJobs SDK、核心繫結、記錄架構和長期工作擴充功能。 以下是這些套件的**套件管理員主控台**命令，以及截至本文撰寫當下的最新穩定版本號碼：

```powershell
Install-Package Microsoft.Azure.WebJobs.Extensions -version 2.2.0
Install-Package Microsoft.Extensions.Logging -version 2.0.1
Install-Package Microsoft.Azure.WebJobs.Extensions.DurableTask -version 1.4.0
```

您也需要記錄提供者。 下列命令會安裝 Application Insights 提供者和 `ConfigurationManager`。 `ConfigurationManager` 可讓您從應用程式設定中取得 Application Insights 檢測金鑰。

```powershell
Install-Package Microsoft.Azure.WebJobs.Logging.ApplicationInsights -version 2.2.0
Install-Package System.Configuration.ConfigurationManager -version 4.4.1
```

下列命令會安裝主控台提供者：

```powershell
Install-Package Microsoft.Extensions.Logging.Console -version 2.0.1
```

## <a name="jobhost-code"></a>JobHost 程式碼

若要使用 Durable Functions 擴充功能，請在 `Main` 方法中的 `JobHostConfiguration` 物件上呼叫 `UseDurableTask`：

```cs
var config = new JobHostConfiguration();
config.UseDurableTask(new DurableTaskExtension
{
    HubName = "MyTaskHub",
};
```

如需可在 `DurableTaskExtension` 物件中設定的屬性清單，請參閱 [host.json](functions-host-json.md#durabletask)。

`Main` 方法也是用來設定記錄提供者的位置。 下列範例會設定主控台和 Application Insights 提供者。

```cs
static void Main(string[] args)
{
    using (var loggerFactory = new LoggerFactory())
    {
        var config = new JobHostConfiguration();

        config.DashboardConnectionString = "";

        var instrumentationKey =
            ConfigurationManager.AppSettings["APPINSIGHTS_INSTRUMENTATIONKEY"];

        config.LoggerFactory = loggerFactory
            .AddApplicationInsights(instrumentationKey, null)
            .AddConsole();

        config.UseTimers();
        config.UseDurableTask(new DurableTaskExtension
        {
            HubName = "MyTaskHub",
        });
        var host = new JobHost(config);
        host.RunAndBlock();
    }
}
```

## <a name="functions"></a>Functions

相較之下，您為 WebJobs SDK 函式所撰寫的程式碼與您為 Azure Functions 服務所撰寫的程式碼有幾項差異。

WebJobs SDK 不支援下列 Azure Functions 功能：

* [FunctionName 屬性](#functionname-attribute)
* [HTTP 觸發程序](#http-trigger)
* [Durable Functions HTTP 管理 API](#http-management-api)

### <a name="functionname-attribute"></a>FunctionName 屬性

在 WebJobs SDK 專案中，函式的方法名稱即為函式名稱。 `FunctionName` 屬性只會在 Azure Functions 中使用。

### <a name="http-trigger"></a>HTTP 觸發程序

WebJobs SDK 沒有 HTTP 觸發程序。 範例專案的協調流程用戶端會使用計時器觸發程序：

```cs
public static async Task CronJob(
    [TimerTrigger("0 */2 * * * *")] TimerInfo timer,
    [OrchestrationClient] DurableOrchestrationClient client,
    ILogger logger)
{
  ...
}
```

### <a name="http-management-api"></a>HTTP 管理 API

由於沒有 HTTP 觸發程序，WebJobs SDK 沒有 [HTTP 管理 API](durable-functions-http-api.md)。

在 WebJobs SDK 專案中，您可以在協調流程用戶端物件上呼叫方法，而不是傳送 HTTP 要求。 下列方法對應至三個可使用 HTTP 管理 API 來進行的工作：

* `GetStatusAsync`
* `RaiseEventAsync`
* `TerminateAsync`

範例專案中的協調流程用戶端函式啟動協調器函式，然後進入每 2 秒呼叫 `GetStatusAsync` 一次的迴圈：

```cs
string instanceId = await client.StartNewAsync(nameof(HelloSequence), input: null);
logger.LogInformation($"Started new instance with ID = {instanceId}.");

DurableOrchestrationStatus status;
while (true)
{
    status = await client.GetStatusAsync(instanceId);
    logger.LogInformation($"Status: {status.RuntimeStatus}, Last update: {status.LastUpdatedTime}.");

    if (status.RuntimeStatus == OrchestrationRuntimeStatus.Completed ||
        status.RuntimeStatus == OrchestrationRuntimeStatus.Failed ||
        status.RuntimeStatus == OrchestrationRuntimeStatus.Terminated)
    {
        break;
    }

    await Task.Delay(TimeSpan.FromSeconds(2));
}
```

## <a name="run-the-sample"></a>執行範例

本節概述如何執行[範例專案](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/webjobssdk/chaining)。 如需會說明如何在本機執行 WebJobs SDK 專案並將其部署至 Azure WebJob 的詳細指示，請參閱[開始使用 WebJobs SDK](../app-service/webjobs-sdk-get-started.md#deploy-as-a-webjob)。

### <a name="run-locally"></a>在本機執行

1. 確定儲存體模擬器正在執行 (請參閱[先決條件](#prerequisites))。

1. 在本機執行時，如果您想要在 Application Insights 中看到記錄：

  a. 建立 Application Insights 資源，應用程式輸入 [一般]。

  b. 將檢測金鑰儲存在 App.config 檔案。

1. 執行專案。

### <a name="run-in-azure"></a>在 Azure 中執行

1. 建立 Web 應用程式和儲存體帳戶。

1. 在 Web 應用程式中，將儲存體連接字串儲存在名為 AzureWebJobsStorage 的應用程式設定中。

1. 建立 Application Insights 資源，應用程式輸入 [一般]。

1. 將檢測金鑰儲存在名為 APPINSIGHTS_INSTRUMENTATIONKEY 的應用程式設定中。

1. 部署為 WebJob。

## <a name="webjobs-sdk-3x"></a>WebJobs SDK 3.x

3.x 版所帶來的主要變更，就是使用 .NET Core，而不是使用 .NET Framework。 若要建立 3.x 專案，指示都相同，但有下列例外：

1. 建立 .NET Core 主控台應用程式。 在 Visual Studio 的 [新增專案] 對話方塊中，選取 [.NET Core] > [主控台應用程式] \(.NET Core\)。 專案檔會將 `TargetFramework` 指定為 `netcoreapp2.0`。

1. 選擇下列套件的發行前版本 3.x：

  * `Microsoft.Azure.WebJobs.Extensions`
  * `Microsoft.Azure.WebJobs.Logging.ApplicationInsights`

1. 變更 `Main` 方法程式碼，以使用 .NET Core 組態架構從 appsettings.json 檔案取得儲存體連接字串和 Application Insights 檢測金鑰。  以下是範例：

   ```cs
   static void Main(string[] args)
   {
       var builder = new ConfigurationBuilder()
           .SetBasePath(Directory.GetCurrentDirectory())
           .AddJsonFile("appsettings.json");

       var appSettingsConfig = builder.Build();

       using (var loggerFactory = new LoggerFactory())
       {
           var config = new JobHostConfiguration();

           config.DashboardConnectionString = "";
           config.StorageConnectionString = 
               appSettingsConfig.GetConnectionString("AzureWebJobsStorage");
           var instrumentationKey =
               appSettingsConfig["APPINSIGHTS_INSTRUMENTATIONKEY"];

           config.LoggerFactory = loggerFactory
               .AddApplicationInsights(instrumentationKey, null)
               .AddConsole();

           config.UseTimers();
           config.UseDurableTask(new DurableTaskExtension
           {
               HubName = "MyTaskHub",
           });
           var host = new JobHost(config);
           host.RunAndBlock();
       }
   }
   ```

## <a name="next-steps"></a>後續步驟

若要深入了解 WebJobs SDK，請參閱[如何使用 WebJobs SDK](../app-service/webjobs-sdk-how-to.md)。

