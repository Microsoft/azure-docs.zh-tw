---
title: 使用 Azure Webhook 監視 .NET 的媒體服務作業通知 | Microsoft Docs
description: 了解如何使用 Azure Webhook 來監視媒體服務作業通知。 程式碼範例是以 C# 撰寫，並使用 Media Services SDK for .NET。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: a61fe157-81b1-45c1-89f2-224b7ef55869
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 12/09/2017
ms.author: juliako
ms.openlocfilehash: 564fc25699c3ae627804d49bfdc40ae9dd559269
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33784487"
---
# <a name="use-azure-webhooks-to-monitor-media-services-job-notifications-with-net"></a>使用 Azure Webhook 監視 .NET 的媒體服務作業通知
執行作業時，您通常需要設法追蹤作業進度。 您可以使用 Azure Webhook 或 [Azure 佇列儲存體](media-services-dotnet-check-job-progress-with-queues.md)來監視媒體服務作業通知。 本文說明如何使用 Webhook。

本文將說明如何

*  定義自訂來回應 Webhook 的 Azure Function。 
    
    在此情況下，當您的編碼作業變更狀態時，媒體服務會觸發 Webhook。 此函式會接聽來自媒體服務通知的 Webhook 回呼，並在作業完成之後發佈輸出資產。 
    
    >[!NOTE]
    >繼續之前，請確定您了解 [Azure Functions HTTP 和 Webhook 繫結](../../azure-functions/functions-bindings-http-webhook.md)的運作方式。
    >
    
* 將 Webhook 新增至您的編碼工作，並指定這個 Webhook 所回應的 Webhook URL 和秘密金鑰。 您將在本文結尾處找到有關將 Webhook 新增至編碼工作的範例。  

您可以在[這裡](https://github.com/Azure-Samples/media-services-dotnet-functions-integration) \(英文\) 找到各種媒體服務 .NET Azure Functions 的定義 (包括本文所示的定義)。

## <a name="prerequisites"></a>先決條件

需要有下列項目，才能完成教學課程：

* 一個 Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* 媒體服務帳戶。 若要建立媒體服務帳戶，請參閱[如何建立媒體服務帳戶](media-services-portal-create-account.md)。
* 了解[如何使用 Azure Functions](../../azure-functions/functions-overview.md)。 另請檢閱 [Azure Functions HTTP 和 Webhook 繫結](../../azure-functions/functions-bindings-http-webhook.md)。

## <a name="create-a-function-app"></a>建立函數應用程式

1. 移至 [Azure 入口網站](http://portal.azure.com) ，然後以您的 Azure 帳戶登入。
2. 如[這裡](../../azure-functions/functions-create-function-app-portal.md)所述建立函式應用程式。

## <a name="configure-function-app-settings"></a>設定函式應用程式設定

開發媒體服務函式時，您可以很方便地新增要在整個函式中使用的環境變數。 若要設定應用程式設定，請按一下 [設定應用程式設定] 連結。 

[應用程式設定](media-services-dotnet-how-to-use-azure-functions.md#configure-function-app-settings)區段會定義用於本文所定義之 Webhook 的參數。 另外，請將下列參數新增至應用程式設定。 

|Name|定義|範例| 
|---|---|---|
|SigningKey |簽署金鑰。| j0txf1f8msjytzvpe40nxbpxdcxtqcgxy0nt|
|WebHookEndpoint | 一個 webhook 端點位址。 建立您的 Webhook 函式之後，您就可以從 [取得函式 URL] 連結複製 URL。 | https://juliakofuncapp.azurewebsites.net/api/Notification_Webhook_Function?code=iN2phdrTnCxmvaKExFWOTulfnm4C71mMLIy8tzLr7Zvf6Z22HHIK5g==。|

## <a name="create-a-function"></a>建立函式

部署函式應用程式之後，您可以在**應用程式服務** Azure Functions 中找到它。

1. 選取您的函式應用程式，然後按一下 [新增函式]。
2. 選取 [C#] 程式碼以及 [API 與 Webhook] 案例。 
3. 選取 [Generic Webhook - C#] (泛型 Webhook - C#)。
4. 命名您的 Webhook，然後按 [建立]。

### <a name="files"></a>檔案

您的 Azure 函式會與本節所述的程式碼檔案和其他檔案建立關聯。 根據預設，函式會與 **function.json** 和 **run.csx** (C#) 檔案相關聯。 您必須新增 **project.json** 檔案。 本節其餘部分會說明這些檔案的定義。

![檔案](./media/media-services-azure-functions/media-services-azure-functions003.png)

#### <a name="functionjson"></a>function.json

function.json 檔案會定義函式繫結和其他組態設定。 執行階段使用此檔案來判斷要監視的事件，以及如何傳入資料並從函式執行傳回資料。 

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "webHookType": "genericJson",
      "name": "req"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ],
  "disabled": false
}
```

#### <a name="projectjson"></a>project.json

project.json 檔案包含相依性。 

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "windowsazure.mediaservices": "4.0.0.4",
        "windowsazure.mediaservices.extensions": "4.0.0.4",
        "Microsoft.IdentityModel.Clients.ActiveDirectory": "3.13.1",
        "Microsoft.IdentityModel.Protocol.Extensions": "1.0.2.206221351"
      }
    }
   }
}
```
    
#### <a name="runcsx"></a>run.csx

本節的程式碼示範如何實作作為 Webhook 的 Azure 函式。 在此範例中，此函式會接聽來自媒體服務通知的 Webhook 回呼，並在作業完成之後發佈輸出資產。

Webhook 預期簽署金鑰 (認證) 會符合您在設定通知端點時所傳遞的金鑰。 簽署金鑰是 64 位元組的 Base64 編碼值，可用來保護來自 Azure 媒體服務之 WebHook 回呼的安全。 

在下列 Webhook 定義程式碼中，**VerifyWebHookRequestSignature** 方法會進行通知訊息的驗證。 此驗證的目的是確定訊息是由 Azure 媒體服務所傳送，並且未遭到竄改。 簽章是 Azure Functions 的選擇性選項，因為它具有**程式碼**值，可透過傳輸層安全性 (TLS) 作為查詢參數。 

>[!NOTE]
>對於不同的 AMS 原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的日期 / 存取權限，例如，要長時間維持就地 (非上載原則) 的定位器原則，您應該使用相同的原則識別碼。 如需詳細資訊，請參閱[此主題](media-services-dotnet-manage-entities.md#limit-access-policies)。

```csharp
///////////////////////////////////////////////////
#r "Newtonsoft.Json"

using System;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.IO;
using System.Globalization;
using Newtonsoft.Json;
using Microsoft.Azure;
using System.Net;
using System.Security.Cryptography;
using Microsoft.Azure.WebJobs;
using Microsoft.IdentityModel.Clients.ActiveDirectory;

internal const string SignatureHeaderKey = "sha256";
internal const string SignatureHeaderValueTemplate = SignatureHeaderKey + "={0}";
static string _webHookEndpoint = Environment.GetEnvironmentVariable("WebHookEndpoint");
static string _signingKey = Environment.GetEnvironmentVariable("SigningKey");

static readonly string _AADTenantDomain = Environment.GetEnvironmentVariable("AMSAADTenantDomain");
static readonly string _RESTAPIEndpoint = Environment.GetEnvironmentVariable("AMSRESTAPIEndpoint");

static readonly string _AMSClientId = Environment.GetEnvironmentVariable("AMSClientId");
static readonly string _AMSClientSecret = Environment.GetEnvironmentVariable("AMSClientSecret");

static CloudMediaContext _context = null;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    log.Info($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");

    Task<byte[]> taskForRequestBody = req.Content.ReadAsByteArrayAsync();
    byte[] requestBody = await taskForRequestBody;

    string jsonContent = await req.Content.ReadAsStringAsync();
    log.Info($"Request Body = {jsonContent}");

    IEnumerable<string> values = null;
    if (req.Headers.TryGetValues("ms-signature", out values))
    {
        byte[] signingKey = Convert.FromBase64String(_signingKey);
        string signatureFromHeader = values.FirstOrDefault();

        if (VerifyWebHookRequestSignature(requestBody, signatureFromHeader, signingKey))
        {
            string requestMessageContents = Encoding.UTF8.GetString(requestBody);

            NotificationMessage msg = JsonConvert.DeserializeObject<NotificationMessage>(requestMessageContents);

            if (VerifyHeaders(req, msg, log))
            { 
                string newJobStateStr = (string)msg.Properties.Where(j => j.Key == "NewState").FirstOrDefault().Value;
                if (newJobStateStr == "Finished")
                {
                    AzureAdTokenCredentials tokenCredentials = new AzureAdTokenCredentials(_AADTenantDomain,
                                new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                                AzureEnvironments.AzureCloudEnvironment);

                    AzureAdTokenProvider tokenProvider = new AzureAdTokenProvider(tokenCredentials);

                    _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

                    if(_context!=null)   
                    {                        
                        string urlForClientStreaming = PublishAndBuildStreamingURLs(msg.Properties["JobId"]);
                        log.Info($"URL to the manifest for client streaming using HLS protocol: {urlForClientStreaming}");
                    }
                }

                return req.CreateResponse(HttpStatusCode.OK, string.Empty);
            }
            else
            {
                log.Info($"VerifyHeaders failed.");
                return req.CreateResponse(HttpStatusCode.BadRequest, "VerifyHeaders failed.");
            }
        }
        else
        {
            log.Info($"VerifyWebHookRequestSignature failed.");
            return req.CreateResponse(HttpStatusCode.BadRequest, "VerifyWebHookRequestSignature failed.");
        }
    }

    return req.CreateResponse(HttpStatusCode.BadRequest, "Generic Error.");
}

private static string PublishAndBuildStreamingURLs(String jobID)
{
    IJob job = _context.Jobs.Where(j => j.Id == jobID).FirstOrDefault();
    IAsset asset = job.OutputMediaAssets.FirstOrDefault();

    // Create a 30-day readonly access policy. 
    // You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.
    IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
    TimeSpan.FromDays(30),
    AccessPermissions.Read);

    // Create a locator to the streaming content on an origin. 
    ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
    policy,
    DateTime.UtcNow.AddMinutes(-5));

    // Get a reference to the streaming manifest file from the  
    // collection of files in the asset. 
    var manifestFile = asset.AssetFiles.Where(f => f.Name.ToLower().
                EndsWith(".ism")).
                FirstOrDefault();

    // Create a full URL to the manifest file. Use this for playback
    // in streaming media clients. 
    string urlForClientStreaming = originLocator.Path + manifestFile.Name + "/manifest" +  "(format=m3u8-aapl)";
    return urlForClientStreaming;

}

private static bool VerifyWebHookRequestSignature(byte[] data, string actualValue, byte[] verificationKey)
{
    using (var hasher = new HMACSHA256(verificationKey))
    {
        byte[] sha256 = hasher.ComputeHash(data);
        string expectedValue = string.Format(CultureInfo.InvariantCulture, SignatureHeaderValueTemplate, ToHex(sha256));

        return (0 == String.Compare(actualValue, expectedValue, System.StringComparison.Ordinal));
    }
}

private static bool VerifyHeaders(HttpRequestMessage req, NotificationMessage msg, TraceWriter log)
{
    bool headersVerified = false;

    try
    {
        IEnumerable<string> values = null;
        if (req.Headers.TryGetValues("ms-mediaservices-accountid", out values))
        {
            string accountIdHeader = values.FirstOrDefault();
            string accountIdFromMessage = msg.Properties["AccountId"];

            if (0 == string.Compare(accountIdHeader, accountIdFromMessage, StringComparison.OrdinalIgnoreCase))
            {
                headersVerified = true;
            }
            else
            {
                log.Info($"accountIdHeader={accountIdHeader} does not match accountIdFromMessage={accountIdFromMessage}");
            }
        }
        else
        {
            log.Info($"Header ms-mediaservices-accountid not found.");
        }
    }
    catch (Exception e)
    {
        log.Info($"VerifyHeaders hit exception {e}");
        headersVerified = false;
    }

    return headersVerified;
}

private static readonly char[] HexLookup = new char[] { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F' };

/// <summary>
/// Converts a <see cref="T:byte[]"/> to a hex-encoded string.
/// </summary>
private static string ToHex(byte[] data)
{
    if (data == null)
    {
        return string.Empty;
    }

    char[] content = new char[data.Length * 2];
    int output = 0;
    byte d;

    for (int input = 0; input < data.Length; input++)
    {
        d = data[input];
        content[output++] = HexLookup[d / 0x10];
        content[output++] = HexLookup[d % 0x10];
    }

    return new string(content);
}

internal enum NotificationEventType
{
    None = 0,
    JobStateChange = 1,
    NotificationEndPointRegistration = 2,
    NotificationEndPointUnregistration = 3,
    TaskStateChange = 4,
    TaskProgress = 5
}

internal sealed class NotificationMessage
{
    public string MessageVersion { get; set; }
    public string ETag { get; set; }
    public NotificationEventType EventType { get; set; }
    public DateTime TimeStamp { get; set; }
    public IDictionary<string, string> Properties { get; set; }
}
```

儲存並執行您的函式。

### <a name="function-output"></a>函式輸出

觸發 Webhook 之後，上述範例會產生下列輸出，而您的值可能會不同。

    C# HTTP trigger function processed a request. RequestUri=https://juliako001-functions.azurewebsites.net/api/Notification_Webhook_Function?code=9376d69kygoy49oft81nel8frty5cme8hb9xsjslxjhalwhfrqd79awz8ic4ieku74dvkdfgvi
    Request Body = 
    {
      "MessageVersion": "1.1",
      "ETag": "b8977308f48858a8f224708bc963e1a09ff917ce730316b4e7ae9137f78f3b20",
      "EventType": 4,
      "TimeStamp": "2017-02-16T03:59:53.3041122Z",
      "Properties": {
        "JobId": "nb:jid:UUID:badd996c-8d7c-4ae0-9bc1-bd7f1902dbdd",
        "TaskId": "nb:tid:UUID:80e26fb9-ee04-4739-abd8-2555dc24639f",
        "NewState": "Finished",
        "OldState": "Processing",
        "AccountName": "mediapkeewmg5c3peq",
        "AccountId": "301912b0-659e-47e0-9bc4-6973f2be3424",
        "NotificationEndPointId": "nb:nepid:UUID:cb5d707b-4db8-45fe-a558-19f8d3306093"
      }
    }
    
    URL to the manifest for client streaming using HLS protocol: http://mediapkeewmg5c3peq.streaming.mediaservices.windows.net/0ac98077-2b58-4db7-a8da-789a13ac6167/BigBuckBunny.ism/manifest(format=m3u8-aapl)

## <a name="add-a-webhook-to-your-encoding-task"></a>將 Webhook 新增至您的編碼工作

本節會顯示將 Webhook 通知新增至工作的程式碼。 您也可以新增作業層級通知，這更適用於具有鏈結工作的作業。  

1. 在 Visual Studio 中，建立新的 C# 主控台應用程式。 依序輸入 [名稱]、[位置] 和 [方案名稱]，然後按一下 [確定]。
2. 使用 [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices) 來安裝 Azure 媒體服務。
3. 使用適當的值來更新 App.config 檔案： 
    
    * Azure 媒體服務連線資訊， 
    * 預期會收到通知的 Webhook URL， 
    * 符合您 Webhook 所預期金鑰的簽署金鑰。 簽署金鑰是 64 位元組的 Base64 編碼值，可用來保護來自 Azure 媒體服務之 Webhook 回呼的安全。 

    ```xml
            <appSettings>
                <add key="AMSAADTenantDomain" value="domain" />
                <add key="AMSRESTAPIEndpoint" value="endpoint" />

                <add key="AMSClientId" value="clinet id" />
                <add key="AMSClientSecret" value="client secret" />

                <add key="WebhookURL" value="https://yourapp.azurewebsites.net/api/functionname?code=ApiKey" />
                <add key="WebhookSigningKey" value="j0txf1f8msjytzvpe40nxbpxdcxtqcgxy0nt" />
            </appSettings>
    ```

4. 使用下列程式碼來更新 Program.cs 檔案：

    ```csharp
            using System;
            using System.Configuration;
            using System.Linq;
            using Microsoft.WindowsAzure.MediaServices.Client;

            namespace NotificationWebHook
            {
                class Program
                {
                // Read values from the App.config file.
                private static readonly string _AMSAADTenantDomain =
                    ConfigurationManager.AppSettings["AMSAADTenantDomain"];
                private static readonly string _AMSRESTAPIEndpoint =
                    ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];

                private static readonly string _AMSClientId =
                    ConfigurationManager.AppSettings["AMSClientId"];
                private static readonly string _AMSClientSecret =
                    ConfigurationManager.AppSettings["AMSClientSecret"];

                private static readonly string _webHookEndpoint =
                    ConfigurationManager.AppSettings["WebhookURL"];
                private static readonly string _signingKey =
                    ConfigurationManager.AppSettings["WebhookSigningKey"];

                // Field for service context.
                private static CloudMediaContext _context = null;

                static void Main(string[] args)
                {
                    AzureAdTokenCredentials tokenCredentials = new AzureAdTokenCredentials(_AMSAADTenantDomain,
                        new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                        AzureEnvironments.AzureCloudEnvironment);

                    AzureAdTokenProvider tokenProvider = new AzureAdTokenProvider(tokenCredentials);

                    _context = new CloudMediaContext(new Uri(_AMSRESTAPIEndpoint), tokenProvider);

                    byte[] keyBytes = Convert.FromBase64String(_signingKey);

                    IAsset newAsset = _context.Assets.FirstOrDefault();

                    // Check for existing Notification Endpoint with the name "FunctionWebHook"

                    var existingEndpoint = _context.NotificationEndPoints.Where(e => e.Name == "FunctionWebHook").FirstOrDefault();
                    INotificationEndPoint endpoint = null;

                    if (existingEndpoint != null)
                    {
                    Console.WriteLine("webhook endpoint already exists");
                    endpoint = (INotificationEndPoint)existingEndpoint;
                    }
                    else
                    {
                    endpoint = _context.NotificationEndPoints.Create("FunctionWebHook",
                        NotificationEndPointType.WebHook, _webHookEndpoint, keyBytes);
                    Console.WriteLine("Notification Endpoint Created with Key : {0}", keyBytes.ToString());
                    }

                    // Declare a new encoding job with the Standard encoder
                    IJob job = _context.Jobs.Create("MES Job");

                    // Get a media processor reference, and pass to it the name of the 
                    // processor to use for the specific task.
                    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

                    ITask task = job.Tasks.AddNew("My encoding task",
                    processor,
                    "Adaptive Streaming",
                    TaskOptions.None);

                    // Specify the input asset to be encoded.
                    task.InputAssets.Add(newAsset);

                    // Add an output asset to contain the results of the job. 
                    // This output is specified as AssetCreationOptions.None, which 
                    // means the output asset is not encrypted. 
                    task.OutputAssets.AddNew(newAsset.Name, AssetCreationOptions.None);

                    // Add the WebHook notification to this Task and request all notification state changes.
                    // Note that you can also add a job level notification
                    // which would be more useful for a job with chained tasks.  
                    if (endpoint != null)
                    {
                    task.TaskNotificationSubscriptions.AddNew(NotificationJobState.All, endpoint, true);
                    Console.WriteLine("Created Notification Subscription for endpoint: {0}", _webHookEndpoint);
                    }
                    else
                    {
                    Console.WriteLine("No Notification Endpoint is being used");
                    }

                    job.Submit();

                    Console.WriteLine("Expect WebHook to be triggered for the Job ID: {0}", job.Id);
                    Console.WriteLine("Expect WebHook to be triggered for the Task ID: {0}", task.Id);

                    Console.WriteLine("Job Submitted");

                }
                private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
                {
                    var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
                    ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

                    if (processor == null)
                    throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

                    return processor;
                }
                }
            }
    ```

## <a name="next-steps"></a>後續步驟

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
