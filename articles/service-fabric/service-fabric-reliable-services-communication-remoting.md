---
title: 在 Service Fabric 中使用 C# 進行服務遠端處理 | Microsoft Docs
description: Service Fabric 遠端處理可讓用戶端和服務使用遠端程序呼叫與 C# 服務進行通訊。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: BharatNarasimman
ms.assetid: abfaf430-fea0-4974-afba-cfc9f9f2354b
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 09/20/2017
ms.author: vturecek
ms.openlocfilehash: ddd78e2fad401add35bc246a64236e2679c33cbc
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44023540"
---
# <a name="service-remoting-in-c-with-reliable-services"></a>使用 Reliable Services 在 C# 中進行服務遠端處理

> [!div class="op_single_selector"]
> * [Windows 上的 C# ](service-fabric-reliable-services-communication-remoting.md)
> * [在 Linux 上使用 Java](service-fabric-reliable-services-communication-remoting-java.md)
>
>

對於未繫結至特定通訊協定或堆疊 (例如 Web API、Windows Communication Foundation 或其他項目) 的服務，Reliable Services 架構會提供遠端機制，以便快速且輕鬆設定服務遠端程序呼叫。 本文會討論如何為以 C# 撰寫的服務設定遠端程序呼叫。

## <a name="set-up-remoting-on-a-service"></a>設定在服務上的遠端處理

只要兩個簡單步驟，就能設定服務的遠端處理：

1. 建立服務實作的介面。 這個介面會定義可在您的服務上用於遠端程序呼叫的方法。 方法也必須是傳回工作的非同步方法。 此介面必須實作 `Microsoft.ServiceFabric.Services.Remoting.IService` ，表示服務具有遠端處理介面。
2. 在您的服務中使用遠端接聽程式。 遠端接聽程式是提供遠端功能的 `ICommunicationListener` 實作。 `Microsoft.ServiceFabric.Services.Remoting.Runtime` 命名空間包含適用於無狀態與具狀態服務的擴充方法 `CreateServiceRemotingListener`，可用於建立使用預設遠端傳輸通訊協定的遠端接聽程式。

>[!NOTE]
>`Remoting` 命名空間是以名為 `Microsoft.ServiceFabric.Services.Remoting` 的個別 NuGet 套件形式來提供使用。

例如，下列無狀態服務服務會公開單一方法，透過遠端程序呼叫取得 "Hello World"。

```csharp
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Microsoft.ServiceFabric.Services.Remoting;
using Microsoft.ServiceFabric.Services.Remoting.Runtime;
using Microsoft.ServiceFabric.Services.Runtime;

public interface IMyService : IService
{
    Task<string> HelloWorldAsync();
}

class MyService : StatelessService, IMyService
{
    public MyService(StatelessServiceContext context)
        : base (context)
    {
    }

    public Task<string> HelloWorldAsync()
    {
        return Task.FromResult("Hello!");
    }

    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
     return this.CreateServiceRemotingInstanceListeners();
    }
}
```

> [!NOTE]
> 服務介面中的引數和傳回類型可以是任何簡單、複雜或自訂的類型，但必須能夠由 .NET [DataContractSerializer](https://msdn.microsoft.com/library/ms731923.aspx) 序列化。
>
>

## <a name="call-remote-service-methods"></a>呼叫遠端服務方法

透過 `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` 類別使用連至服務的本機 Proxy，可以在服務上使用遠端堆疊呼叫方法。 `ServiceProxy` 方法會使用該服務所實作的相同介面，建立本機 Proxy。 您可以使用該 Proxy 在介面上遠端呼叫方法。

```csharp

IMyService helloWorldClient = ServiceProxy.Create<IMyService>(new Uri("fabric:/MyApplication/MyHelloWorldService"));

string message = await helloWorldClient.HelloWorldAsync();

```

遠端架構會將服務擲回的例外狀況傳播給用戶端。 因此，使用 `ServiceProxy` 時，用戶端會負責處理由服務擲回的例外狀況。

## <a name="service-proxy-lifetime"></a>服務 Proxy 存留期

建立服務 Proxy 是輕量型作業，因此您可以建立的數目沒有限制。 只要有需要，可以重複使用服務 Proxy 執行個體。 如果遠端程序呼叫擲回例外狀況，您仍然可以重複使用相同的 Proxy 執行個體。 每個服務 Proxy 都包含用來透過網路傳送訊息的通訊用戶端。 叫用遠端呼叫時，系統會執行內部檢查來判斷通訊用戶端是否有效。 根據這些檢查的結果，系統會在必要情況下重建通訊用戶端。 因此，如果發生例外狀況，您無須重建 `ServiceProxy`。

### <a name="service-proxy-factory-lifetime"></a>服務 Proxy 處理站存留期

[ServiceProxyFactory](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.services.remoting.client.serviceproxyfactory) 是建立不同遠端介面 Proxy 執行個體的處理站。 如果您使用 API `ServiceProxy.Create` 來建立 Proxy，則架構會建立單一服務 Proxy。
當您需要覆寫 [IServiceRemotingClientFactory](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.services.remoting.v1.client.iserviceremotingclientfactory) 屬性時，最實用的方式是手動建立一個。
處理站建立是一個成本高昂的作業。 服務 Proxy 處理站會維護通訊用戶端的內部快取。
最佳做法是快取服務 Proxy 處理站的時間愈長愈好。

## <a name="remoting-exception-handling"></a>遠端例外狀況處理

服務 API 擲回的所有遠端例外狀況都會以 AggregateException 的形式傳送回用戶端。 遠端例外狀況應可由 DataContract 序列化。 否則，Proxy API 會擲回包含序列化錯誤的 [ServiceException](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.services.communication.serviceexception)。

服務 Proxy 會處理它為其建立之服務分割區的所有容錯移轉列外狀況。 它會在有容錯移轉例外狀況 (非暫時性例外狀況) 時重新解析端點，然後以正確的端點再次嘗試呼叫。 容錯移轉例外狀況的重試次數並無限制。
如果發生暫時性例外狀況，Proxy 會重試呼叫。

預設的重試參數是由 [OperationRetrySettings](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.services.communication.client.operationretrysettings) 提供。

使用者可以將 OperationRetrySettings 物件傳遞至 ServiceProxyFactory 建構函式，來設定這些值。

## <a name="use-the-remoting-v2-stack"></a>使用遠端 V2 堆疊

從 NuGet 遠端套件 2.8 版開始，您可以選擇使用遠端 V2 堆疊。 遠端 V2 堆疊的效能更好。 也會提供像是自訂序列化和更容易插入的 API 等功能。
程式碼範本會繼續使用遠端 V1 堆疊。
遠端 V2 與 V1 (先前的遠端堆疊) 不相容。 請遵循[從 V1 升級為 V2](#upgrade-from-remoting-v1-to-remoting-v2)一文中的指示，以免影響服務可用性。

下列方法可用來啟用 V2 堆疊。

### <a name="use-an-assembly-attribute-to-use-the-v2-stack"></a>使用組件屬性以使用 V2 堆疊

下列步驟會使用組件屬性將程式碼範本變更為使用 V2 堆疊。

1. 將服務資訊清單中的端點資源從 `"ServiceEndpoint"` 變更為 `"ServiceEndpointV2"`。

  ```xml
  <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2" />
    </Endpoints>
  </Resources>
  ```

2. 使用 `Microsoft.ServiceFabric.Services.Remoting.Runtime.CreateServiceRemotingInstanceListeners` 擴充方法建立遠端接聽程式 (V1 和 V2 皆相同)。

  ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
  ```

3. `FabricTransportServiceRemotingProvider` 屬性標記包含遠端處理介面的組件。

  ```csharp
  [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2, RemotingClientVersion = RemotingClientVersion.V2)]
  ```

用戶端專案中不需要變更任何程式碼。
使用介面組件建置用戶端組件，以確保使用先前所示的組件屬性。

### <a name="use-explicit-v2-classes-to-use-the-v2-stack"></a>使用明確的 V2 類別以使用 V2 堆疊

除了使用組件屬性，V2 堆疊也可以透過使用明確的 V2 類別來啟用。

下列步驟會使用明確的 V2 類別，將程式碼範本變更為使用 V2 堆疊。

1. 將服務資訊清單中的端點資源從 `"ServiceEndpoint"` 變更為 `"ServiceEndpointV2"`。

  ```xml
  <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2" />
    </Endpoints>
  </Resources>
  ```

2. 從 `Microsoft.ServiceFabric.Services.Remoting.V2.FabricTransport.Runtime` 命名空間使用 [FabricTransportServiceRemotingListener](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.runtime.fabrictransportserviceremotingListener?view=azure-dotnet)。

  ```csharp
  protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[]
        {
            new ServiceInstanceListener((c) =>
            {
                return new FabricTransportServiceRemotingListener(c, this);

            })
        };
    }
  ```

3. 從 `Microsoft.ServiceFabric.Services.Remoting.V2.FabricTransport.Client` 命名空間使用 [FabricTransportServiceRemotingClientFactory](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.client.fabrictransportserviceremotingclientfactory?view=azure-dotnet) 來建立用戶端。

  ```csharp
  var proxyFactory = new ServiceProxyFactory((c) =>
          {
              return new FabricTransportServiceRemotingClientFactory();
          });
  ```

## <a name="upgrade-from-remoting-v1-to-remoting-v2"></a>從遠端 V1 升級為遠端 V2

若要從 V1 升級為 V2，必須執行 2 個步驟。 遵循此序列中的步驟。

1. 使用此屬性將 V1 服務升級為 V2 服務。
此變更可確保服務接聽 V1 和 V2 接聽程式。

    a. 在服務資訊清單中新增名稱為 "ServiceEndpointV2" 的端點資源。
      ```xml
      <Resources>
        <Endpoints>
          <Endpoint Name="ServiceEndpointV2" />  
        </Endpoints>
      </Resources>
      ```

    b. 使用下列擴充方法來建立遠端接聽程式。

    ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
    ```

    c. 在遠端介面上新增組件屬性，以使用 V1 和 V2 接聽程式以及 V2 用戶端。
    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2|RemotingListenerVersion.V1, RemotingClientVersion = RemotingClientVersion.V2)]

      ```
2. 使用 V2 用戶端屬性，將 V1 用戶端升級為 V2 用戶端。
此步驟可確保用戶端使用 V2 堆疊。
用戶端專案/服務不需要任何變更。 您可以使用更新的介面組件建置用戶端專案。

3. 此為選用步驟。 使用 V2 接聽程式屬性，然後升級 V2 服務。
此步驟可確保服務只接聽 V2 接聽程式。

    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2, RemotingClientVersion = RemotingClientVersion.V2)]
    ```


## <a name="use-the-remoting-v2-interface-compatible-stack"></a>使用遠端 V2(介面相容) 堆疊

 遠端 V2 (介面相容，也稱為 V2_1) 堆疊具有 V2 遠端堆疊的所有功能。 它的介面堆疊與遠端 V1 堆疊相容，但是 V2 與 V1 不具回溯相容性。 若要從 V1 升級到 V2_1 而不會影響服務可用性，請遵循[從 V1 升級為 V2 (介面相容)](#upgrade-from-remoting-v1-to-remoting-v2interfacecompatible) 一文中的步驟。


### <a name="use-an-assembly-attribute-to-use-the-remoting-v2-interface-compatible-stack"></a>透過組件屬性使用遠端 V2 (介面相容) 堆疊

請遵循下列步驟以變更為 V2_1 堆疊。

1. 在服務資訊清單中新增名稱為 "ServiceEndpointV2_1" 的端點資源。

  ```xml
  <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2_1" />  
    </Endpoints>
  </Resources>
  ```

2. 使用此擴充方法來建立遠端接聽程式。

  ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
  ```

3. 在遠端介面上新增[組件屬性](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.services.remoting.fabrictransport.fabrictransportserviceremotingproviderattribute?view=azure-dotnet)。

  ```csharp
    [assembly:  FabricTransportServiceRemotingProvider(RemotingListenerVersion=  RemotingListenerVersion.V2_1, RemotingClientVersion= RemotingClientVersion.V2_1)]

  ```

用戶端專案不需要任何變更。
使用介面組件建置用戶端組件，以確保使用先前的組件屬性。

### <a name="use-explicit-remoting-classes-to-create-a-listenerclient-factory-for-the-v2-interface-compatible-version"></a>使用明確的遠端類別來建立 V2 (介面相容) 版本的接聽程式/用戶端處理站。

請遵循下列步驟：

1. 在服務資訊清單中新增名稱為 "ServiceEndpointV2_1" 的端點資源。

  ```xml
  <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpointV2_1" />  
    </Endpoints>
  </Resources>
  ```

2. 使用[遠端 V2 接聽程式](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.runtime.fabrictransportserviceremotinglistener?view=azure-dotnet)。 預設使用的服務端點資源名稱是 "ServiceEndpointV2_1"。 該名稱必須定義在服務資訊清單中。

  ```csharp
  protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[]
        {
            new ServiceInstanceListener((c) =>
            {
                var settings = new FabricTransportRemotingListenerSettings();
                settings.UseWrappedMessage = true;
                return new FabricTransportServiceRemotingListener(c, this,settings);

            })
        };
    }
  ```

3. 使用 V2 [用戶端處理站](https://docs.microsoft.com/dotnet/api/microsoft.servicefabric.services.remoting.v2.fabrictransport.client.fabrictransportserviceremotingclientfactory?view=azure-dotnet)。
  ```csharp
  var proxyFactory = new ServiceProxyFactory((c) =>
          {
            var settings = new FabricTransportRemotingSettings();
            settings.UseWrappedMessage = true;
            return new FabricTransportServiceRemotingClientFactory(settings);
          });
  ```

## <a name="upgrade-from-remoting-v1-to-remoting-v2-interface-compatible"></a>從遠端 V1 升級為遠端 V2 (介面相容)

若要從 V1 升級為 V2 (介面相容，也稱為 V2_1)，需要 2 個步驟。 遵循此序列中的步驟。

1. 使用下列屬性將 V1 服務升級為 V2_1 服務。
此變更可確保服務正在接聽 V1 和 V2_1 接聽程式。

    a. 在服務資訊清單中新增名稱為 "ServiceEndpointV2_1" 的端點資源。
      ```xml
      <Resources>
        <Endpoints>
          <Endpoint Name="ServiceEndpointV2_1" />  
        </Endpoints>
      </Resources>
      ```

    b. 使用下列擴充方法來建立遠端接聽程式。

    ```csharp
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return this.CreateServiceRemotingInstanceListeners();
    }
    ```

    c. 在遠端介面上新增組件屬性，以使用 V1、V2_1 接聽程式和 V2_1 用戶端。
    ```csharp
   [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1 | RemotingListenerVersion.V1, RemotingClientVersion = RemotingClientVersion.V2_1)]

      ```
2. 使用 V2_1 用戶端屬性，將 V1 用戶端升級為 V2_1 用戶端。
此步驟可確保用戶端使用 V2_1 堆疊。
用戶端專案/服務不需要任何變更。 您可以使用更新的介面組件建置用戶端專案。

3. 此為選用步驟。 從屬性中移除 V1 接聽程式版本，然後升級 V2 服務。
此步驟可確保服務只接聽 V2 接聽程式。

    ```csharp
    [assembly: FabricTransportServiceRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1, RemotingClientVersion = RemotingClientVersion.V2_1)]
    ```
  
### <a name="use-custom-serialization-with-a-remoting-wrapped-message"></a>使用自訂序列化搭配遠端包裝訊息

對於遠端包裝訊息，我們會建立含有所有參數的單一包裝物件作為其中的欄位。
請遵循下列步驟：

1. 實作 `IServiceRemotingMessageSerializationProvider` 介面，以提供自訂序列化的實作。
    此程式碼片段會顯示實作情況。

      ```csharp
      public class ServiceRemotingJsonSerializationProvider : IServiceRemotingMessageSerializationProvider
      {
        public IServiceRemotingMessageBodyFactory CreateMessageBodyFactory()
        {
          return new JsonMessageFactory();
        }

        public IServiceRemotingRequestMessageBodySerializer CreateRequestMessageSerializer(Type serviceInterfaceType, IEnumerable<Type> requestWrappedType, IEnumerable<Type> requestBodyTypes = null)
        {
          return new ServiceRemotingRequestJsonMessageBodySerializer();
        }

        public IServiceRemotingResponseMessageBodySerializer CreateResponseMessageSerializer(Type serviceInterfaceType, IEnumerable<Type> responseWrappedType, IEnumerable<Type> responseBodyTypes = null)
        {
          return new ServiceRemotingResponseJsonMessageBodySerializer();
        }
      }
      ```
      ```csharp
        class JsonMessageFactory : IServiceRemotingMessageBodyFactory
            {

              public IServiceRemotingRequestMessageBody CreateRequest(string interfaceName, string methodName, int numberOfParameters, object wrappedRequestObject)
              {
                return new JsonBody(wrappedRequestObject);
              }

              public IServiceRemotingResponseMessageBody CreateResponse(string interfaceName, string methodName, object wrappedRequestObject)
              {
                return new JsonBody(wrappedRequestObject);
              }
            }
      ```
      ```csharp
      class ServiceRemotingRequestJsonMessageBodySerializer : IServiceRemotingRequestMessageBodySerializer
        {
            private JsonSerializer serializer;

            public ServiceRemotingRequestJsonMessageBodySerializer()
            {
              serializer = JsonSerializer.Create(new JsonSerializerSettings()
              {
                TypeNameHandling = TypeNameHandling.All
                });
              }

              public IOutgoingMessageBody Serialize(IServiceRemotingRequestMessageBody serviceRemotingRequestMessageBody)
             {
               if (serviceRemotingRequestMessageBody == null)
               {
                 return null;
               }          
               using (var writeStream = new MemoryStream())
               {
                 using (var jsonWriter = new JsonTextWriter(new StreamWriter(writeStream)))
                 {
                   serializer.Serialize(jsonWriter, serviceRemotingRequestMessageBody);
                   jsonWriter.Flush();
                   var bytes = writeStream.ToArray();
                   var segment = new ArraySegment<byte>(bytes);
                   var segments = new List<ArraySegment<byte>> { segment };
                   return new OutgoingMessageBody(segments);
                 }
               }
              }

              public IServiceRemotingRequestMessageBody Deserialize(IIncomingMessageBody messageBody)
             {
               using (var sr = new StreamReader(messageBody.GetReceivedBuffer()))
               {
                 using (JsonReader reader = new JsonTextReader(sr))
                 {
                   var ob = serializer.Deserialize<JsonBody>(reader);
                   return ob;
                 }
               }
             }
            }
      ```
      ```csharp
      class ServiceRemotingResponseJsonMessageBodySerializer : IServiceRemotingResponseMessageBodySerializer
       {
         private JsonSerializer serializer;

        public ServiceRemotingResponseJsonMessageBodySerializer()
        {
          serializer = JsonSerializer.Create(new JsonSerializerSettings()
          {
              TypeNameHandling = TypeNameHandling.All
            });
          }

          public IOutgoingMessageBody Serialize(IServiceRemotingResponseMessageBody responseMessageBody)
          {
            if (responseMessageBody == null)
            {
              return null;
            }

            using (var writeStream = new MemoryStream())
            {
              using (var jsonWriter = new JsonTextWriter(new StreamWriter(writeStream)))
              {
                serializer.Serialize(jsonWriter, responseMessageBody);
                jsonWriter.Flush();
                var bytes = writeStream.ToArray();
                var segment = new ArraySegment<byte>(bytes);
                var segments = new List<ArraySegment<byte>> { segment };
                return new OutgoingMessageBody(segments);
              }
            }
          }

          public IServiceRemotingResponseMessageBody Deserialize(IIncomingMessageBody messageBody)
          {

             using (var sr = new StreamReader(messageBody.GetReceivedBuffer()))
             {
               using (var reader = new JsonTextReader(sr))
               {
                 var obj = serializer.Deserialize<JsonBody>(reader);
                 return obj;
               }
             }
           }
       }
    ```
    ```csharp
    class JsonBody : WrappedMessage, IServiceRemotingRequestMessageBody, IServiceRemotingResponseMessageBody
    {
          public JsonBody(object wrapped)
          {
            this.Value = wrapped;
          }

          public void SetParameter(int position, string parameName, object parameter)
          {  //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public object GetParameter(int position, string parameName, Type paramType)
          {
            //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public void Set(object response)
          { //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }

          public object Get(Type paramType)
          {  //Not Needed if you are using WrappedMessage
            throw new NotImplementedException();
          }
    }
    ```

2. 使用遠端接聽程式的 `JsonSerializationProvider` 覆寫預設序列化提供者。

  ```csharp
  protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
   {
       return new[]
       {
           new ServiceInstanceListener((c) =>
           {
               return new FabricTransportServiceRemotingListener(c, this,
                   new ServiceRemotingJsonSerializationProvider());
           })
       };
   }
  ```

3. 使用遠端用戶端處理站的 `JsonSerializationProvider` 覆寫預設序列化提供者。

    ```csharp
    var proxyFactory = new ServiceProxyFactory((c) =>
    {
        return new FabricTransportServiceRemotingClientFactory(
        serializationProvider: new ServiceRemotingJsonSerializationProvider());
      });
      ```

## <a name="next-steps"></a>後續步驟

* [在 Reliable Services 中搭配 OWIN 使用 Web API](service-fabric-reliable-services-communication-webapi.md)
* [使用 Reliable Services 的 Windows Communication Foundation 通訊](service-fabric-reliable-services-communication-wcf.md)
* [Reliable Services 的安全通訊](service-fabric-reliable-services-secure-communication.md)
