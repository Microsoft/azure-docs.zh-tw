---
title: 以程式設計方式監視和管理 Azure 串流分析作業
description: 本文說明如何以程式設計方式監視透過 REST API、Azure SDK 或 PowerShell 建立的串流分析作業。
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/20/2017
ms.openlocfilehash: 2688f148185b1c1523178d190a7a2a76e6ceabef
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30908780"
---
# <a name="programmatically-create-a-stream-analytics-job-monitor"></a>以程式設計方式來建立串流分析工作監視

本文示範如何為串流分析工作啟用監視。 透過 REST API、Azure SDK 或 PowerShell 建立的串流分析作業預設不會啟用監視。 您可以在 Azure 入口網站中前往該作業的 [監視] 頁面，然後按一下 [啟用] 按鈕來手動啟用，或是按照本文中的步驟執行，將此程序自動化。 串流分析工作的監視資料將會顯示在 Azure 入口網站的 [計量] 區域中。

## <a name="prerequisites"></a>先決條件

開始此程序之前，您必須有下列項目：

* Visual Studio 2017 或 2015
* 已下載並安裝 [Azure .NET SDK](https://azure.microsoft.com/downloads/)
* 一項已啟用監視的現有串流分析作業

## <a name="create-a-project"></a>建立專案

1. 建立 Visual Studio C# .NET 主控台應用程式。
2. 在 Package Manager Console 中，執行下列命令以安裝 NuGet 封裝。 第一個是 Azure 串流分析管理 .NET SDK。 第二個是將用來啟用監視功能的 Azure 監視器 SDK。 最後一個是驗證要使用的 Azure Active Directory 用戶端。
   
   ```
   Install-Package Microsoft.Azure.Management.StreamAnalytics
   Install-Package Microsoft.Azure.Insights -Pre
   Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
   ```
3. 將下列 appSettings 區段加入至 App.config 檔案。
   
   ```
   <appSettings>
     <!--CSM Prod related values-->
     <add key="ResourceGroupName" value="RESOURCE GROUP NAME" />
     <add key="JobName" value="YOUR JOB NAME" />
     <add key="StorageAccountName" value="YOUR STORAGE ACCOUNT"/>
     <add key="ActiveDirectoryEndpoint" value="https://login.microsoftonline.com/" />
     <add key="ResourceManagerEndpoint" value="https://management.azure.com/" />
     <add key="WindowsManagementUri" value="https://management.core.windows.net/" />
     <add key="AsaClientId" value="1950a258-227b-4e31-a9cf-717495945fc2" />
     <add key="RedirectUri" value="urn:ietf:wg:oauth:2.0:oob" />
     <add key="SubscriptionId" value="YOUR AZURE SUBSCRIPTION ID" />
     <add key="ActiveDirectoryTenantId" value="YOUR TENANT ID" />
   </appSettings>
   ```
   以您的 Azure 訂用帳戶 ID 和租用戶識別碼取代 *SubscriptionId* 和 *ActiveDirectoryTenantId* 的值。 您可以藉由執行下列 PowerShell Cmdlet 來取得這些值：
   
   ```
   Get-AzureAccount
   ```
4. 將下列 using 陳述式加入至專案的原始程式檔 (Program.cs) 中。
   
   ```
     using System;
     using System.Configuration;
     using System.Threading;
     using Microsoft.Azure;
     using Microsoft.Azure.Management.Insights;
     using Microsoft.Azure.Management.Insights.Models;
     using Microsoft.Azure.Management.StreamAnalytics;
     using Microsoft.Azure.Management.StreamAnalytics.Models;
     using Microsoft.IdentityModel.Clients.ActiveDirectory;
   ```
5. 新增驗證協助程式方法。
   
     public static string GetAuthorizationHeader()
   
         {
             AuthenticationResult result = null;
             var thread = new Thread(() =>
             {
                 try
                 {
                     var context = new AuthenticationContext(
                         ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] +
                         ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);
   
                     result = context.AcquireToken(
                         resource: ConfigurationManager.AppSettings["WindowsManagementUri"],
                         clientId: ConfigurationManager.AppSettings["AsaClientId"],
                         redirectUri: new Uri(ConfigurationManager.AppSettings["RedirectUri"]),
                         promptBehavior: PromptBehavior.Always);
                 }
                 catch (Exception threadEx)
                 {
                     Console.WriteLine(threadEx.Message);
                 }
             });
   
             thread.SetApartmentState(ApartmentState.STA);
             thread.Name = "AcquireTokenThread";
             thread.Start();
             thread.Join();
   
             if (result != null)
             {
                 return result.AccessToken;
             }
   
             throw new InvalidOperationException("Failed to acquire token");
     }

## <a name="create-management-clients"></a>建立管理用戶端

下列程式碼將設定必要的變數與管理用戶端。

    string resourceGroupName = "<YOUR AZURE RESOURCE GROUP NAME>";
    string streamAnalyticsJobName = "<YOUR STREAM ANALYTICS JOB NAME>";

    // Get authentication token
    TokenCloudCredentials aadTokenCredentials =
        new TokenCloudCredentials(
            ConfigurationManager.AppSettings["SubscriptionId"],
            GetAuthorizationHeader());

    Uri resourceManagerUri = new
    Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    // Create Stream Analytics and Insights management client
    StreamAnalyticsManagementClient streamAnalyticsClient = new
    StreamAnalyticsManagementClient(aadTokenCredentials, resourceManagerUri);
    InsightsManagementClient insightsClient = new
    InsightsManagementClient(aadTokenCredentials, resourceManagerUri);

## <a name="enable-monitoring-for-an-existing-stream-analytics-job"></a>為現有串流分析作業啟用監視

下列程式碼將為「現有」 串流分析作業啟用監視。 程式碼的第一部分會對串流分析服務執行 GET 要求，以擷取特定串流分析工作的相關資訊。 在程式碼的第二部分使用 *Id* 屬性 (擷取自 GET 要求) 當成 Put 方法的參數，將 PUT 要求傳送至 Insights 服務，來為串流分析作業啟用監視。

>[!WARNING]
>如果您先前已經為不同的串流分析作業啟用監視 (不論是透過 Azure 入口網站，還是以程式設計方式透過以下的程式碼)，**建議您提供先前啟用監視時所提供的相同儲存體帳戶名稱。**
> 
> 儲存體帳戶會連結到您建立串流分析作業所在的區域，而不是明確地連結到作業本身。
> 
> 相同區域中的所有串流分析作業 (以及其他所有 Azure 資源) 都共用此儲存體帳戶儲存監視資料。 如果您提供不同的儲存體帳戶，可能會對其他串流分析作業或其他 Azure 資源的監視產生非預期的副作用。
> 
> 用來取代下方程式碼中 `<YOUR STORAGE ACCOUNT NAME>` 的儲存體帳戶名稱應該是與您為其啟用監視功能的「串流分析」作業屬於相同訂用帳戶的儲存體帳戶。
> 
> 

    // Get an existing Stream Analytics job
    JobGetParameters jobGetParameters = new JobGetParameters()
    {
        PropertiesToExpand = "inputs,transformation,outputs"
    };
    JobGetResponse jobGetResponse = streamAnalyticsClient.StreamingJobs.Get(resourceGroupName, streamAnalyticsJobName, jobGetParameters);

    // Enable monitoring
    ServiceDiagnosticSettingsPutParameters insightPutParameters = new ServiceDiagnosticSettingsPutParameters()
    {
            Properties = new ServiceDiagnosticSettings()
            {
                StorageAccountName = "<YOUR STORAGE ACCOUNT NAME>"
            }
    };
    insightsClient.ServiceDiagnosticSettingsOperations.Put(jobGetResponse.Job.Id, insightPutParameters);



## <a name="get-support"></a>取得支援

如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。

## <a name="next-steps"></a>後續步驟

* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

