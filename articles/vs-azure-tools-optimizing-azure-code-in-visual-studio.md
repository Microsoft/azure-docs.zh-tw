---
title: 最佳化 Visual Studio 中的 Azure 程式碼 | Microsoft Docs
description: 了解 Visual Studio 中的 Azure 程式碼最佳化工具如何協助讓程式碼更穩定且有更好的效能。
services: visual-studio-online
documentationcenter: na
author: cawa
manager: paulyuk
editor: ''
ms.assetid: ed48ee06-e2d2-4322-af22-07200fb16987
ms.service: multiple
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: vs-azure
ms.workload: azure-vs
ms.date: 11/11/2016
ms.author: cawa
ms.openlocfilehash: 0497ac628d7882a0b722796493c10c0d8b04e759
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42443663"
---
# <a name="optimizing-your-azure-code"></a>最佳化您的 Azure 程式碼
當您在撰寫使用 Microsoft Azure 的應用程式時，請遵循某些程式碼撰寫實務，以避免應用程式在雲端環境中發生延展性、行為和效能方面的問題。 Microsoft 有提供 Azure Code Analysis 工具，可辨識並找出許多常見問題，並幫助您解決這些問題。 您可以在 Visual Studio 中透過 NuGet 下載此工具。

## <a name="azure-code-analysis-rules"></a>Azure Code Analysis 規則
Azure Code Analysis 工具會在找到已知會影響效能的問題時，使用下列規則自動為 Azure 程式碼加上旗標。 偵測到的問題會顯示為警告或編譯器錯誤。 系統常會透過燈泡圖示提供用來解決警告或錯誤的程式碼修正或建議。

## <a name="avoid-using-default-in-process-session-state-mode"></a>避免使用預設 (同處理序) 工作階段狀態模式
### <a name="id"></a>ID
AP0000

### <a name="description"></a>說明
如果您對雲端應用程式使用預設 (同處理序) 工作階段狀態模式，您可能會遺失工作階段狀態。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
根據預設，web.config 檔案中所指定的是同處理序工作階段狀態模式。 此外，如果組態檔中沒有指定項目，工作階段狀態模式會預設為同處理序。 同處理序模式會將工作階段狀態儲存在 Web 伺服器的記憶體中。 當原有執行個體重新啟動或使用新執行個體以支援負載平衡或容錯移轉時，Web 伺服器的記憶體中儲存的工作階段狀態並不會儲存起來。 這種情況會讓應用程式無法在雲端上進行調整。

ASP.NET 工作階段狀態支援數種不同的工作階段狀態資料儲存選項：InProc、StateServer、SQLServer、自訂和關閉。 建議您使用自訂模式在外部工作階段狀態存放區裝載資料，例如 [適用於 Redis 的 Azure 工作階段狀態提供者](http://go.microsoft.com/fwlink/?LinkId=401521)。

### <a name="solution"></a>解決方法
有一個建議的解決方案是在受控快取服務儲存工作階段狀態。 了解如何使用 [適用於 Redis 的 Azure 工作階段狀態提供者](http://go.microsoft.com/fwlink/?LinkId=401521) 來儲存工作階段狀態。 您也可以在其他位置儲存工作階段狀態，以確保應用程式可在雲端上進行調整。 若要深入了解替代解決方案，請參閱 [工作階段狀態模式](https://msdn.microsoft.com/library/ms178586)。

## <a name="run-method-should-not-be-async"></a>執行方法必須同步
### <a name="id"></a>ID
AP1000

### <a name="description"></a>說明
在 [Run()](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 方法外部建立非同步方法 (例如 [await](https://msdn.microsoft.com/library/hh156528.aspx))，然後從 [Run()](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 呼叫非同步方法。 將 [[Run()](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx)](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 宣告為非同步方法會導致背景工作角色輸入重新啟動迴圈。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
在 [Run()](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 方法內部呼叫非同步方法會導致雲端服務執行階段回收背景工作角色。 當背景工作角色啟動時，便會在 [Run ()](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 方法內執行所有程式。 結束 [Run ()](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 方法會導致背景工作角色重新啟動。 當背景工作角色執行階段叫用非同步方法時，它會在非同步方法之後分派所有作業，然後返回。 這會使背景工作角色結束 [[[[Run()](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx)](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx)](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx)](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 方法並重新啟動。 在下一輪執行時，背景工作角色會再次叫用非同步方法並重新啟動，導致背景工作角色又再次回收。

### <a name="solution"></a>解決方法
將所有非同步作業放在 [Run()](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 方法外部。 然後從 [[Run()](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx)](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.run.aspx) 方法內部呼叫重新建構後的非同步方法，例如 RunAsync().wait。 Azure Code Analysis 工具可協助您修正此問題。

下列程式碼片段示範此問題的程式碼修正程式：

```
public override void Run()
{
    RunAsync().Wait();
}

public async Task RunAsync()
{
    //Asynchronous operations code logic
    // This is a sample worker implementation. Replace with your logic.

    Trace.TraceInformation("WorkerRole1 entry point called");

    HttpClient client = new HttpClient();

    Task<string> urlString = client.GetStringAsync("http://msdn.microsoft.com");

    while (true)
    {
        Thread.Sleep(10000);
        Trace.TraceInformation("Working");

        string stream = await urlString;
    }

}
```

## <a name="use-service-bus-shared-access-signature-authentication"></a>使用服務匯流排共用存取簽章驗證
### <a name="id"></a>ID
AP2000

### <a name="description"></a>說明
使用共用存取簽章 (SAS) 進行驗證。 存取控制服務 (ACS) 將不可再用於進行服務匯流排驗證。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
為加強安全性，Azure Active Directory 將以 SAS 驗證取代 ACS 驗證。 請參閱 [Azure Active Directory 是 ACS 的未來](https://cloudblogs.microsoft.com/enterprisemobility/2013/06/22/azure-active-directory-is-the-future-of-acs/) ，以取得轉換計畫的相關資訊。

### <a name="solution"></a>解決方法
在應用程式中使用 SAS 驗證。 下列範例顯示如何使用現有 SAS 權杖存取服務匯流排命名空間或實體。

```
MessagingFactory listenMF = MessagingFactory.Create(endpoints, new StaticSASTokenProvider(subscriptionToken));
SubscriptionClient sc = listenMF.CreateSubscriptionClient(topicPath, subscriptionName);
BrokeredMessage receivedMessage = sc.Receive();
```

如需詳細資訊，請參閱下列主題。

* 如需概觀，請參閱 [使用服務匯流排的共用存取簽章驗證](https://msdn.microsoft.com/library/dn170477.aspx)
* [如何搭配使用共用存取簽章驗證與服務匯流排](https://msdn.microsoft.com/library/dn205161.aspx)
* 如須專案範例，請參閱 [搭配使用共用存取簽章 (SAS) 驗證與服務匯流排訂用帳戶](http://code.msdn.microsoft.com/windowsazure/Using-Shared-Access-e605b37c)

## <a name="consider-using-onmessage-method-to-avoid-receive-loop"></a>考慮使用 OnMessage 方法來避免「接收迴圈」
### <a name="id"></a>ID
AP2002

### <a name="description"></a>說明
若要避免陷入「接收迴圈」，呼叫 **OnMessage** 方法會比呼叫 **Receive** 方法更適合用來接收訊息。 不過，如果您必須使用 **Receive** 方法，而且您指定了非預設的伺服器等待時間，請確定伺服器等待時間超過一分鐘。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
在呼叫 **OnMessage**時，用戶端會啟動持續輪詢佇列或訂用帳戶的內部訊息幫浦。 此訊息幫浦包含會發出訊息接收呼叫的無限迴圈。 如果呼叫逾時，它就會發出新的呼叫。 逾時間隔是由所使用的 [MessagingFactory](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.messagingfactory.aspx) 的 [OperationTimeout](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.messagingfactorysettings.operationtimeout.aspx) 屬性值所決定。

相較於 **Receive**，使用 **OnMessage** 的優點是使用者不必手動輪詢訊息、處理例外狀況、平行處理多個訊息，以及完成訊息。

如果您呼叫的 **Receive** 不是使用其預設值，請確定 *ServerWaitTime* 值有超過一分鐘。 將 *ServerWaitTime* 設定為超過一分鐘可防止伺服器沒接收完訊息就逾時。

### <a name="solution"></a>解決方法
請參閱下列程式碼範例以了解建議用法。 如需詳細資訊，請參閱 [QueueClient.OnMessage 方法 (Microsoft.ServiceBus.Messaging)](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.queueclient.onmessage.aspx) 和 [QueueClient.Receive 方法 (Microsoft.ServiceBus.Messaging)](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.queueclient.receive.aspx)。

若要改善 Azure 傳訊基礎結構的效能，請參閱設計模式 [非同步傳訊入門](https://msdn.microsoft.com/library/dn589781.aspx)。

以下是使用 **OnMessage** 來接收訊息的範例。

```
void ReceiveMessages()
{
    // Initialize message pump options.
    OnMessageOptions options = new OnMessageOptions();
    options.AutoComplete = true; // Indicates if the message-pump should call complete on messages after the callback has completed processing.
    options.MaxConcurrentCalls = 1; // Indicates the maximum number of concurrent calls to the callback the pump should initiate.
    options.ExceptionReceived += LogErrors; // Enables you to get notified of any errors encountered by the message pump.

    // Start receiving messages.
    QueueClient client = QueueClient.Create("myQueue");
    client.OnMessage((receivedMessage) => // Initiates the message pump and callback is invoked for each message that is recieved, calling close on the client will stop the pump.
    {
        // Process the message.
    }, options);
    Console.WriteLine("Press any key to exit.");
    Console.ReadKey();
```

以下是使用 **Receive** 與預設伺服器等待時間的範例。

```
string connectionString =  
CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

QueueClient Client =  
    QueueClient.CreateFromConnectionString(connectionString, "TestQueue");

while (true)  
{   
   BrokeredMessage message = Client.Receive();
   if (message != null)
   {
      try  
      {
         Console.WriteLine("Body: " + message.GetBody<string>());
         Console.WriteLine("MessageID: " + message.MessageId);
         Console.WriteLine("Test Property: " +  
            message.Properties["TestProperty"]);

         // Remove message from queue
         message.Complete();
      }

      catch (Exception)
      {
         // Indicate a problem, unlock message in queue
         message.Abandon();
      }
   }
```

以下是使用 **Receive** 與非預設伺服器等待時間的範例。

```
while (true)  
{   
   BrokeredMessage message = Client.Receive(new TimeSpan(0,1,0));

   if (message != null)
   {
      try  
      {
         Console.WriteLine("Body: " + message.GetBody<string>());
         Console.WriteLine("MessageID: " + message.MessageId);
         Console.WriteLine("Test Property: " +  
            message.Properties["TestProperty"]);

         // Remove message from queue
         message.Complete();
      }

      catch (Exception)
      {
         // Indicate a problem, unlock message in queue
         message.Abandon();
      }
   }
}
```
## <a name="consider-using-asynchronous-service-bus-methods"></a>考慮使用非同步服務匯流排方法
### <a name="id"></a>ID
AP2003

### <a name="description"></a>說明
使用非同步服務匯流排方法可改善代理傳訊的效能。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
使用非同步方法可實現應用程式並行效果，因為在執行每個呼叫時並不會封鎖主要執行緒。 使用服務匯流排傳訊方法時，需要花時間執行各項作業 (傳送、接收、刪除等)。 這個時間包括服務匯流排服務處理作業的時間加上要求和回覆的延遲時間。 若要增加每次的作業數目，就必須並行執行作業。 如需詳細資訊，請參閱[使用服務匯流排代理傳訊的效能改進最佳作法](https://msdn.microsoft.com/library/azure/hh528527.aspx)。

### <a name="solution"></a>解決方法
請參閱 [QueueClient 類別 (Microsoft.ServiceBus.Messaging)](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.queueclient.aspx) ，以取得如何使用建議的非同步方法的相關資訊。

若要改善 Azure 傳訊基礎結構的效能，請參閱設計模式 [非同步傳訊入門](https://msdn.microsoft.com/library/dn589781.aspx)。

## <a name="consider-partitioning-service-bus-queues-and-topics"></a>考慮分割服務匯流排佇列和主題
### <a name="id"></a>ID
AP2004

### <a name="description"></a>說明
分割服務匯流排佇列和主題以獲得更好的服務匯流排傳訊效能。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
分割服務匯流排佇列和主題可增加效能輸送量和服務可用性，因為分割後的佇列或主題的整體輸送量不會再受限於單一訊息代理程式或訊息存放區的效能。 此外，即使訊息存放區暫時中斷也不會讓分割後的佇列或主題無法使用。 如需詳細資訊，請參閱 [分割訊息實體](https://msdn.microsoft.com/library/azure/dn520246.aspx)。

### <a name="solution"></a>解決方法
下列程式碼片段顯示如何分割訊息實體。

```
// Create partitioned topic.
NamespaceManager ns = NamespaceManager.CreateFromConnectionString(myConnectionString);
TopicDescription td = new TopicDescription(TopicName);
td.EnablePartitioning = true;
ns.CreateTopic(td);
```

如需詳細資訊，請參閱[分割的服務匯流排佇列和主題 | Microsoft Azure 部落格](https://azure.microsoft.com/blog/2013/10/29/partitioned-service-bus-queues-and-topics/)和 [Microsoft Azure 服務匯流排分割的佇列範例](https://code.msdn.microsoft.com/windowsazure/Service-Bus-Partitioned-7dfd3f1f)。

## <a name="do-not-set-sharedaccessstarttime"></a>不要設定 SharedAccessStartTime
### <a name="id"></a>ID
AP3001

### <a name="description"></a>說明
您應該避免使用設定為目前時間的 SharedAccessStartTime，以立即啟動共用存取原則。 除非您想要稍後再啟動共用存取原則，才需要設定這個屬性。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
時鐘同步處理會讓資料中心彼此間產生些微時差。 例如，您會理所當然地認為，使用 DateTime.Now 或類似方法將儲存體 SAS 原則的開始時間設定為目前時間，會讓 SAS 原則立即生效。 不過，資料中心彼此間的些微時差會讓此一假設產生問題，因為某些資料中心的時間可能稍晚於開始時間，有些則是稍早。 如此一來，如果原則的存留期設得太短，SAS 原則可能會快速 (甚至立即) 到期。

如需在 Azure 儲存體上使用共用存取簽章的詳細指引，請參閱 [簡介資料表 SAS (共用存取簽章)、佇列 SAS 和 Blob SAS 的更新 - Microsoft Azure 儲存體團隊部落格 - 網站首頁 - MSDN 部落格](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx)。

### <a name="solution"></a>解決方法
移除用來設定共用存取原則開始時間的陳述式。 Azure Code Analysis 工具有提供此問題的修正程式。 如需安全性管理的詳細資訊，請參閱設計模式 [貼身金鑰模式](https://msdn.microsoft.com/library/dn568102.aspx)。

下列程式碼片段示範此問題的程式碼修正程式。

```
// The shared access policy provides  
// read/write access to the container for 10 hours.
blobPermissions.SharedAccessPolicies.Add("mypolicy", new SharedAccessBlobPolicy()
{
   // To ensure SAS is valid immediately, don’t set start time.
   // This way, you can avoid failures caused by small clock differences.
   SharedAccessExpiryTime = DateTime.UtcNow.AddHours(10),
   Permissions = SharedAccessBlobPermissions.Write |
      SharedAccessBlobPermissions.Read
});
```

## <a name="shared-access-policy-expiry-time-must-be-more-than-five-minutes"></a>共用存取原則的到期時間必須超過五分鐘
### <a name="id"></a>ID
AP3002

### <a name="description"></a>說明
由於有稱為「時鐘誤差」的狀況，不同位置的資料中心之間，最多會差上五分鐘。 為了防止 SAS 原則權杖在預計時間前提早到期，請將到期時間設定為超過五分鐘。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
分處世界各地不同位置的資料中心是以時脈訊號來同步。 時脈訊號需要時間來傳遞到不同位置，因此雖然理論上各地的時間皆同步，但不同地理位置的資料中心之間還是會有時間差。 此一時間差可能會影響共用存取原則的開始時間和到期間隔。 因此，為了確保共用存取原則能立即生效，請勿指定開始時間。 此外，請確定到期時間超過 5 分鐘，以防止提早逾時。

如需在 Azure 儲存體上使用共用存取簽章的詳細資訊，請參閱 [簡介資料表 SAS (共用存取簽章)、佇列 SAS 和 Blob SAS 的更新 - Microsoft Azure 儲存體團隊部落格 - 網站首頁 - MSDN 部落格](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx)。

### <a name="solution"></a>解決方法
如需安全性管理的詳細資訊，請參閱設計模式 [貼身金鑰模式](https://msdn.microsoft.com/library/dn568102.aspx)。

以下是未指定共用存取原則開始時間的範例。

```
// The shared access policy provides  
// read/write access to the container for 10 hours.
blobPermissions.SharedAccessPolicies.Add("mypolicy", new SharedAccessBlobPolicy()
{
   // To ensure SAS is valid immediately, don’t set start time.
   // This way, you can avoid failures caused by small clock differences.
   SharedAccessExpiryTime = DateTime.UtcNow.AddHours(10),
   Permissions = SharedAccessBlobPermissions.Write |
      SharedAccessBlobPermissions.Read
});
```

以下是指定共用存取原則開始時間，且原則到期期間超過五分鐘的範例。

```
// The shared access policy provides  
// read/write access to the container for 10 hours.
blobPermissions.SharedAccessPolicies.Add("mypolicy", new SharedAccessBlobPolicy()
{
   // To ensure SAS is valid immediately, don’t set start time.
   // This way, you can avoid failures caused by small clock differences.
  SharedAccessStartTime = new DateTime(2014,1,20),   
 SharedAccessExpiryTime = new DateTime(2014, 1, 21),
   Permissions = SharedAccessBlobPermissions.Write |
      SharedAccessBlobPermissions.Read
});
```

如需詳細資訊，請參閱 [建立和使用共用存取簽章](https://msdn.microsoft.com/library/azure/jj721951.aspx)。

## <a name="use-cloudconfigurationmanager"></a>使用 CloudConfigurationManager
### <a name="id"></a>ID
AP4000

### <a name="description"></a>說明
對 Azure 網站和 Azure 行動服務等專案使用 [ConfigurationManager](https://msdn.microsoft.com/library/system.configuration.configurationmanager\(v=vs.110\).aspx) 類別不會產生執行階段問題。 不過，最佳做法是使用雲端 [ConfigurationManager](https://msdn.microsoft.com/library/system.configuration.configurationmanager\(v=vs.110\).aspx) 做為所有 Azure 雲端應用程式組態的統一管理方式。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
CloudConfigurationManager 會讀取適合應用程式環境使用的組態檔。

[CloudConfigurationManager](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.cloudconfigurationmanager.aspx)

### <a name="solution"></a>解決方法
重新建構程式碼以使用 [CloudConfigurationManager 類別](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.cloudconfigurationmanager.aspx)。 Azure Code Analysis 工具有提供此問題的程式碼修正。

下列程式碼片段示範此問題的程式碼修正程式。 將

`var settings = ConfigurationManager.AppSettings["mySettings"];`

取代為

`var settings = CloudConfigurationManager.GetSetting("mySettings");`

以下是如何在 App.config 或 Web.config 檔案中儲存組態設定的範例。 請將設定加入至組態檔的 appSettings 區段。 以下是上一個程式碼範例的 Web.config 檔案。

```
<appSettings>
    <add key="webpages:Version" value="3.0.0.0" />
    <add key="webpages:Enabled" value="false" />
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
    <add key="mySettings" value="[put_your_setting_here]"/>
  </appSettings>  
```

## <a name="avoid-using-hard-coded-connection-strings"></a>避免使用硬式編碼的連接字串
### <a name="id"></a>ID
AP4001

### <a name="description"></a>說明
如果您使用硬式編碼的連接字串，並且需要在稍後加以更新，您必須對原始程式碼進行變更並重新編譯應用程式。 不過，如果您將連接字串儲存在組態檔中，之後只要更新組態檔就能變更連接字串。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
將連接字串硬式編碼不是個好辦法，因為這種方式會在需要快速變更連接字串時引發問題。 此外，如果需要將專案簽入至原始檔控制，硬式編碼的連接字串會引發安全性漏洞，因為在原始程式碼中就能檢視字串。

### <a name="solution"></a>解決方法
將連接字串儲存在組態檔或 Azure 環境中。

* 若是獨立應用程式，請使用 app.config 來儲存連接字串設定。
* 若是 IIS 裝載的應用程式，請使用 web.config 來儲存連接字串。
* 若是 ASP.NET vNext 應用程式，請使用 configuration.json 來儲存連接字串。

如需使用 web.config 或 app.config 等組態檔的相關資訊，請參閱 [ASP.NET Web 組態指導方針](https://msdn.microsoft.com/library/vstudio/ff400235\(v=vs.100\).aspx)。 如需 Azure 環境變數運作方式的相關資訊，請參閱 [Azure 網站：應用程式字串與連接字串的運作方式](https://azure.microsoft.com/blog/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work/)。 如需在原始檔控制中儲存連接字串的相關資訊，請參閱 [避免將敏感資訊 (例如連接字串) 放在儲存於原始程式碼儲存機制的檔案](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control)。

## <a name="use-diagnostics-configuration-file"></a>使用診斷組態檔
### <a name="id"></a>ID
AP5000

### <a name="description"></a>說明
與其在程式碼中設定診斷設定 (例如使用 Microsoft.WindowsAzure.Diagnostics 程式設計 API)，不如在 diagnostics.wadcfg 檔案中設定診斷設定。 (或者，如果您使用 Azure SDK 2.5，則在 diagnostics.wadcfgx 中設定)。 如此一來，您就可以變更診斷設定而不必重新編譯程式碼。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
在 Azure SDK 2.5 (使用 Azure 診斷 1.3) 之前，可使用幾種不同的方法設定 Azure 診斷 (WAD)：將它加入至儲存體中的組態 Blob、使用命令式程式碼、宣告式組態或預設組態。 不過，設定診斷功能時最好是使用應用程式專案中的 XML 組態檔 (若是 SDK 2.5 和更新版本，則是 diagnostics.wadcfg 或 diagnositcs.wadcfgx)。 透過這種方法，diagnostics.wadcfg 檔案可完整定義組態，並可隨意加以更新和重新部署。 將 diagnostics.wadcfg 組態檔和可使用 [DiagnosticMonitor](https://msdn.microsoft.com/library/microsoft.windowsazure.diagnostics.diagnosticmonitor.aspx)或 [RoleInstanceDiagnosticManager](https://msdn.microsoft.com/library/microsoft.windowsazure.diagnostics.management.roleinstancediagnosticmanager.aspx) 類別設定組態的程式設計方法混合使用會造成混淆。 如需詳細資訊，請參閱 [初始化或變更 Azure 診斷組態](https://msdn.microsoft.com/library/azure/hh411537.aspx) 。

從 WAD 1.3 (隨附於 Azure SDK 2.5) 開始，就無法再使用程式碼來設定診斷功能。 因此，您只能在套用或更新診斷延伸模組時提供組態。

### <a name="solution"></a>解決方法
使用診斷組態設計工具將診斷設定移至診斷組態檔 (若是 SDK 2.5 和更新版本，則是 diagnositcs.wadcfg 或 diagnositcs.wadcfgx)。 也建議您安裝 [Azure SDK 2.5](http://go.microsoft.com/fwlink/?LinkId=513188) 並使用最新的診斷功能。

1. 在您要設定之角色的捷徑功能表上，選擇 [屬性]，然後選擇 [組態] 索引標籤。
2. 在 [診斷] 區段中，確定已選取 [啟用診斷] 核取方塊。
3. 選擇 [設定]  按鈕。

   ![存取啟用診斷選項](./media/vs-azure-tools-optimizing-azure-code-in-visual-studio/IC796660.png)

   如需詳細資訊，請參閱 [為 Azure 雲端服務和虛擬機器設定診斷功能](vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines.md) 。

## <a name="avoid-declaring-dbcontext-objects-as-static"></a>避免將 DbContext 物件宣告為靜態
### <a name="id"></a>ID
AP6000

### <a name="description"></a>說明
為了節省記憶體，請避免將 DBContext 物件宣告為靜態。

請在 [Azure Code Analysis 意見反應](http://go.microsoft.com/fwlink/?LinkId=403771)分享您的想法和意見。

### <a name="reason"></a>原因
DBContext 物件會保存每個呼叫的查詢結果。 在卸載應用程式網域後，才會處置靜態 DBContext 物件。 因此，靜態 DBContext 物件可能會耗用大量記憶體。

### <a name="solution"></a>解決方法
將 DBContext 宣告為區域變數或非靜態執行個體欄位、將它用於工作，然後讓它在使用後受到處置。

下列 MVC 控制器類別範例顯示如何使用 DBContext 物件。

```
public class BlogsController : Controller
    {
        //BloggingContext is a subclass to DbContext        
        private BloggingContext db = new BloggingContext();
        // GET: Blogs
        public ActionResult Index()
        {
            //business logics…
            return View();
        }
        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                db.Dispose();
            }
            base.Dispose(disposing);
        }
    }
```

## <a name="next-steps"></a>後續步驟
若要深入了解如何針對 Azure 應用程式進行最佳化和疑難排解，請參閱[使用 Visual Studio 針對 Azure App Service 中的 Web 應用程式進行疑難排解](app-service/web-sites-dotnet-troubleshoot-visual-studio.md)。
