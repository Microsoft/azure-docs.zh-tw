---
title: 在 Linux 中針對 Azure Service Fabric 應用程式進行偵錯 | Microsoft Docs
description: 了解如何在本機 Linux 開發電腦上監視和診斷 Service Fabric 服務。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: 4eebe937-ab42-4429-93db-f35c26424321
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: 5aeb87538968304d3eaf73873d4c4c762c07329c
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44051369"
---
# <a name="monitor-and-diagnose-services-in-a-local-machine-development-setup"></a>監視和診斷本機開發設定中的服務


> [!div class="op_single_selector"]
> * [Windows](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)
> * [Linux](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md)
>
>

監視、偵測、診斷和疑難排解可讓服務繼續順利執行，盡可能減少服務中斷的使用者經驗。 在實際部署的生產環境中，監視和診斷非常重要。 若在開發服務期間採用類似的模型，當您移到生產環境時，將可確保診斷管線能夠運作。 Service Fabric 可讓服務開發人員輕鬆實作診斷，可以在單一電腦本機開發設定和實際生產叢集設定上順暢地工作。


## <a name="debugging-service-fabric-java-applications"></a>針對 Service Fabric Java 應用程式進行偵錯

對於 Java 應用程式，有 [多個記錄架構](http://en.wikipedia.org/wiki/Java_logging_framework) 可用。 由於 `java.util.logging` 是 JRE 的預設選項，它也會用於 [github 中的程式碼範例](http://github.com/Azure-Samples/service-fabric-java-getting-started)。  下列討論說明如何設定 `java.util.logging` 架構。

您可以使用 java.util.logging 將應用程式記錄檔重新導向至記憶體、輸出串流、主控台檔案或通訊端。 對於其中每個選項，架構中已經提供預設處理常式。 您可以建立 `app.properties` 檔案來設定應用程式的檔案處理常式，將所有記錄檔重新導向至本機檔案。

下列程式碼片段包含範例組態︰

```java
handlers = java.util.logging.FileHandler

java.util.logging.FileHandler.level = ALL
java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter
java.util.logging.FileHandler.limit = 1024000
java.util.logging.FileHandler.count = 10
java.util.logging.FileHandler.pattern = /tmp/servicefabric/logs/mysfapp%u.%g.log             
```

`app.properties` 檔案所指向的資料夾必須存在。 建立 `app.properties` 檔案之後，您還必須修改 `<applicationfolder>/<servicePkg>/Code/` 資料夾中的進入點指令碼 `entrypoint.sh`，以將屬性 `java.util.logging.config.file` 設定為 `app.propertes` 檔案。 輸入如下列片段所示：

```sh
java -Djava.library.path=$LD_LIBRARY_PATH -Djava.util.logging.config.file=<path to app.properties> -jar <service name>.jar
```


此設定會導致在 `/tmp/servicefabric/logs/`中以輪替方式收集記錄檔。 在此情況下，記錄檔的名稱為 mysfapp%u.%g.log，其中︰
* **%u** 是解決同時 Java 處理序之間衝突的唯一號碼。
* **%g** 是區分輪替記錄的產生號碼。

依預設，如果未明確設定任何處理常式，則會註冊主控台處理常式。 使用者可以在 /var/log/syslog 下檢視 syslog 中的記錄檔。

如需詳細資訊，請參閱 [github 中的程式碼範例](http://github.com/Azure-Samples/service-fabric-java-getting-started)。  


## <a name="debugging-service-fabric-c-applications"></a>針對 Service Fabric C# 應用程式進行偵錯


有多個架構適用於追蹤 Linux 上的 CoreCLR 應用程式。 如需詳細資訊，請參閱 [GitHub：logging](http:/github.com/aspnet/logging)。  因為 C# 開發人員熟悉 EventSource，本文使用 EventSource 來追蹤 Linux 上的 CoreCLR 範例。

第一個步驟是加入 System.Diagnostics.Tracing，使您可以將您的記錄檔寫入記憶體中、輸出串流或主控台檔案。  針對使用 EventSource 進行記錄，請將下列專案加入您的 project.json︰

```
    "System.Diagnostics.StackTrace": "4.0.1"
```

您可以使用自訂 EventListener 接聽服務事件，然後適當地將其重新導向至追蹤檔案。 下列程式碼片段顯示使用 EventSource 和自訂 EventListener 來進行記錄的範例實作︰


```csharp

 public class ServiceEventSource : EventSource
 {
        public static ServiceEventSource Current = new ServiceEventSource();

        [NonEvent]
        public void Message(string message, params object[] args)
        {
            if (this.IsEnabled())
            {
                var finalMessage = string.Format(message, args);
                this.Message(finalMessage);
            }
        }

        // TBD: Need to add method for sample event.

}

```


```csharp
   internal class ServiceEventListener : EventListener
   {

        protected override void OnEventSourceCreated(EventSource eventSource)
        {
            EnableEvents(eventSource, EventLevel.LogAlways, EventKeywords.All);
        }
        protected override void OnEventWritten(EventWrittenEventArgs eventData)
        {
            using (StreamWriter Out = new StreamWriter( new FileStream("/tmp/MyServiceLog.txt", FileMode.Append)))           
        { 
                 // report all event information               
         Out.Write(" {0} ",  Write(eventData.Task.ToString(), eventData.EventName, eventData.EventId.ToString(), eventData.Level,""));
                if (eventData.Message != null)              
            Out.WriteLine(eventData.Message, eventData.Payload.ToArray());              
            else             
        { 
                    string[] sargs = eventData.Payload != null ? eventData.Payload.Select(o => o.ToString()).ToArray() : null; 
                    Out.WriteLine("({0}).", sargs != null ? string.Join(", ", sargs) : "");             
        }
           }
        }
    }
```


上述程式碼片段會將記錄輸出到 `/tmp/MyServiceLog.txt` 中的檔案。 此檔案名稱必須適當地更新。 如果您想要將記錄重新導向至主控台，在您的自訂 EventListener 類別中使用下列程式碼片︰

```csharp
public static TextWriter Out = Console.Out;
```

[C# Samples](https://github.com/Azure-Samples/service-fabric-dotnet-core-getting-started) 中的範例使用 EventSource 和自訂 EventListener 事件將事件記錄到檔案中。



## <a name="next-steps"></a>後續步驟
新增至應用程式的相同追蹤程式碼，也可以用來配合診斷 Azure 叢集上的應用程式。 請參閱下列文章，其中討論各種適用於工具的選項，並說明如何設定它們。
* [如何利用 Azure 診斷收集記錄檔](service-fabric-diagnostics-how-to-setup-lad.md)
