---
title: Reliable Services 通訊概觀 | Microsoft Docs
description: Reliable Services 通訊模型概觀，其中包括開啟服務的接聽程式、解析端點和服務間通訊。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: BharatNarasimman
ms.assetid: 36217988-420e-409d-b0a4-e0e875b6eac8
ms.service: service-fabric
ms.devlang: multiple
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 11/01/2017
ms.author: vturecek
ms.openlocfilehash: 62c81368b8a3129b42262cb99cf23a5021744c1b
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34210754"
---
# <a name="how-to-use-the-reliable-services-communication-apis"></a>如何使用 Reliable Services 通訊 API
「Azure Service Fabric 即平台」完全不受服務間的通訊影響。 所有通訊協定和堆疊 (從 UDP 到 HTTP) 都可接受。 它是由服務開發人員選擇服務應有的通訊方式。 Reliable Services 應用程式架構會提供內建的通訊堆疊以及 API，讓您可用來建置自訂通訊元件。

## <a name="set-up-service-communication"></a>設定服務通訊
Reliable Services API 使用一個簡單的服務通訊介面。 若要開啟服務的端點，只要實作此介面即可：

```csharp

public interface ICommunicationListener
{
    Task<string> OpenAsync(CancellationToken cancellationToken);

    Task CloseAsync(CancellationToken cancellationToken);

    void Abort();
}

```

```java
public interface CommunicationListener {
    CompletableFuture<String> openAsync(CancellationToken cancellationToken);

    CompletableFuture<?> closeAsync(CancellationToken cancellationToken);

    void abort();
}
```

然後，您可以在服務基底類別方法覆寫項中傳回您的通訊接聽程式實作來新增該實作。

對於無狀態服務：

```csharp
public class MyStatelessService : StatelessService
{
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        ...
    }
    ...
}
```
```java
public class MyStatelessService extends StatelessService {

    @Override
    protected List<ServiceInstanceListener> createServiceInstanceListeners() {
        ...
    }
    ...
}
```

對於具狀態服務：

```java
    @Override
    protected List<ServiceReplicaListener> createServiceReplicaListeners() {
        ...
    }
    ...
```

```csharp
public class MyStatefulService : StatefulService
{
    protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
    {
        ...
    }
    ...
}
```

在這兩種情況下，您會傳回接聽程式的集合。 這可讓您的服務透過多個接聽程式，可能使使用不同的通訊協定，在多個端點上接聽。 例如，您可能有 HTTP 接聽程式和個別的 WebSocket 接聽程式。 每個接聽程式都會獲得一個名稱及所產生的名稱集合：當用戶端要求服務執行個體或資料分割的接聽位址時，系統會以 JSON 物件的形式呈現位址配對。

在無狀態服務中，覆寫項會傳回 ServiceInstanceListeners 的集合。 `ServiceInstanceListener` 會包含可建立 `ICommunicationListener(C#) / CommunicationListener(Java)` 的函式，並會為它命名。 就具狀態服務而言，覆寫項則會傳回 ServiceReplicaListeners 集合。 這與其無狀態的對應項稍有不同，因為 `ServiceReplicaListener` 可以選擇在次要複本上將 `ICommunicationListener` 開啟。 您不僅可以在服務中使用多個通訊接聽程式，也可以指定哪些接聽程式要在次要複本上接受要求，以及哪些接聽程式只在主要複本上進行接聽。

例如，您可以有一個只在主要複本上接受 RPC 呼叫的 ServiceRemotingListener，以及一個透過 HTTP 在次要複本上接受讀取要求的第二、自訂接聽程式：

```csharp
protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new[]
    {
        new ServiceReplicaListener(context =>
            new MyCustomHttpListener(context),
            "HTTPReadonlyEndpoint",
            true),

        new ServiceReplicaListener(context =>
            this.CreateServiceRemotingListener(context),
            "rpcPrimaryEndpoint",
            false)
    };
}
```

> [!NOTE]
> 建立服務的多個接聽程式時， **必須** 為每個接聽程式提供唯一的名稱。
>
>

最後，在 [服務資訊清單](service-fabric-application-and-service-manifests.md) 中有關端點的區段下方說明服務所需的端點。

```xml
<Resources>
    <Endpoints>
      <Endpoint Name="WebServiceEndpoint" Protocol="http" Port="80" />
      <Endpoint Name="OtherServiceEndpoint" Protocol="tcp" Port="8505" />
    <Endpoints>
</Resources>

```

通訊接聽程式可以從 `ServiceContext` 中的 `CodePackageActivationContext` 存取配置給它的端點資源。 然後接聽程式會在開啟時開始接聽要求。

```csharp
var codePackageActivationContext = serviceContext.CodePackageActivationContext;
var port = codePackageActivationContext.GetEndpoint("ServiceEndpoint").Port;

```
```java
CodePackageActivationContext codePackageActivationContext = serviceContext.getCodePackageActivationContext();
int port = codePackageActivationContext.getEndpoint("ServiceEndpoint").getPort();

```

> [!NOTE]
> 端點資源通用於整個服務封裝，並在服務封裝啟動時由 Service Fabric 配置。 裝載於相同 ServiceHost 的多個服務複本可能會共用相同的連接埠。 這表示通訊接聽程式應該支援連接埠共用。 建議做法是讓通訊接聽程式在產生接聽位址時，使用資料分割識別碼和複本/執行個體識別碼。
>
>

### <a name="service-address-registration"></a>服務位址註冊
名為「命名服務」  的系統服務會在 Service Fabric 叢集上執行。 命名服務是適用於服務及其位址的註冊機構，而服務的每個執行個體或複本正在其上接聽。 當 `ICommunicationListener(C#) / CommunicationListener(Java)` 的 `OpenAsync(C#) / openAsync(Java)` 方法完成時，它的傳回值會在命名服務中註冊。 這個在命名服務中發佈的傳回值是一個字串，其值完全可以是任何項目。 這個字串值是用戶端向命名服務要求服務的位址時將會看見的內容。

```csharp
public Task<string> OpenAsync(CancellationToken cancellationToken)
{
    EndpointResourceDescription serviceEndpoint = serviceContext.CodePackageActivationContext.GetEndpoint("ServiceEndpoint");
    int port = serviceEndpoint.Port;

    this.listeningAddress = string.Format(
                CultureInfo.InvariantCulture,
                "http://+:{0}/",
                port);

    this.publishAddress = this.listeningAddress.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);

    this.webApp = WebApp.Start(this.listeningAddress, appBuilder => this.startup.Invoke(appBuilder));

    // the string returned here will be published in the Naming Service.
    return Task.FromResult(this.publishAddress);
}
```
```java
public CompletableFuture<String> openAsync(CancellationToken cancellationToken)
{
    EndpointResourceDescription serviceEndpoint = serviceContext.getCodePackageActivationContext.getEndpoint("ServiceEndpoint");
    int port = serviceEndpoint.getPort();

    this.publishAddress = String.format("http://%s:%d/", FabricRuntime.getNodeContext().getIpAddressOrFQDN(), port);

    this.webApp = new WebApp(port);
    this.webApp.start();

    /* the string returned here will be published in the Naming Service.
     */
    return CompletableFuture.completedFuture(this.publishAddress);
}
```

Service Fabric 提供 API，讓用戶端和其他服務之後能夠依服務名稱來要求這個位址。 這一點很重要，因為服務位址不是靜態的。 服務會為了資源平衡和可用性目的在叢集中移動。 這是可讓用戶端解析服務接聽位址的機制。

> [!NOTE]
> 如需如何撰寫通訊接聽程式的完整逐步解說，請參閱 [Service Fabric Web API 服務與 OWIN 自我裝載](service-fabric-reliable-services-communication-webapi.md) (若為 C#)，而您可以撰寫自己的 HTTP 伺服器實作 (若為 Java)，請參閱 https://github.com/Azure-Samples/service-fabric-java-getting-started 中的 EchoServer 應用程式範例。
>
>

## <a name="communicating-with-a-service"></a>與服務通訊
Reliable Services API 提供下列程式庫來撰寫與服務通訊的用戶端。

### <a name="service-endpoint-resolution"></a>服務端點解析
與服務通訊的第一個步驟是，解析您想要通訊之服務的分割區或執行個體的端點位址。 `ServicePartitionResolver(C#) / FabricServicePartitionResolver(Java)` 公用程式類別是一個基本類型，可協助用戶端在執行階段判斷服務的端點。 在 Service Fabric 術語中，判斷服務端點的程序稱為「服務端點解析」 。

若要連線到叢集內的服務，可以使用預設設定建立 ServicePartitionResolver。 這是大多數情況的建議用法︰

```csharp
ServicePartitionResolver resolver = ServicePartitionResolver.GetDefault();
```
```java
FabricServicePartitionResolver resolver = FabricServicePartitionResolver.getDefault();
```

若要連線到不同叢集中的服務，可利用一組叢集閘道端點來建立 ServicePartitionResolver。 請注意，閘道端點就只是可用來連接到相同叢集的不同端點。 例如︰

```csharp
ServicePartitionResolver resolver = new  ServicePartitionResolver("mycluster.cloudapp.azure.com:19000", "mycluster.cloudapp.azure.com:19001");
```
```java
FabricServicePartitionResolver resolver = new  FabricServicePartitionResolver("mycluster.cloudapp.azure.com:19000", "mycluster.cloudapp.azure.com:19001");
```

另外，可為 `ServicePartitionResolver` 指定一個函式來建立 `FabricClient`，以便在內部使用：

```csharp
public delegate FabricClient CreateFabricClientDelegate();
```
```java
public FabricServicePartitionResolver(CreateFabricClient createFabricClient) {
...
}

public interface CreateFabricClient {
    public FabricClient getFabricClient();
}
```

`FabricClient` 是為了叢集上各種管理作業而用來與 Service Fabric 叢集通訊的物件。 當您想要更充分掌控服務分割解析程式與叢集互動的方式時，這非常實用。 `FabricClient` 會在內部執行快取但建立的成本通常很高，因此一定要儘可能重複使用 `FabricClient` 執行個體。

```csharp
ServicePartitionResolver resolver = new  ServicePartitionResolver(() => CreateMyFabricClient());
```
```java
FabricServicePartitionResolver resolver = new  FabricServicePartitionResolver(() -> new CreateFabricClientImpl());
```

解析方法接著可用於擷取服務或已資料分割之服務的服務分割區的位址。

```csharp
ServicePartitionResolver resolver = ServicePartitionResolver.GetDefault();

ResolvedServicePartition partition =
    await resolver.ResolveAsync(new Uri("fabric:/MyApp/MyService"), new ServicePartitionKey(), cancellationToken);
```
```java
FabricServicePartitionResolver resolver = FabricServicePartitionResolver.getDefault();

CompletableFuture<ResolvedServicePartition> partition =
    resolver.resolveAsync(new URI("fabric:/MyApp/MyService"), new ServicePartitionKey());
```

服務位址可以使用 ServicePartitionResolver 輕鬆地加以解析，但需要執行更多工作，才能確保解析的位址可正確使用。 您的用戶端必須偵測連線嘗試是否因為暫時性錯誤而失敗且可重試 (例如，服務已移動或暫時無法使用)，或因永久錯誤而失敗 (例如，已刪除服務，或要求的資源不存在)。 服務執行個體或複本隨時都可基於多重因素在節點間移動。 透過 ServicePartitionResolver 解析的服務位址，可能會在您的用戶端程式碼嘗試連線之前過時。 再回到該情況，用戶端必須重新解析位址。 提供先前的 `ResolvedServicePartition` ，表示解析程式需要再試一次，而不只是擷取快取的位址。

通常用戶端程式碼不需要直接搭配 ServicePartitionResolver 使用。 它已建立並傳遞給 Reliable Services API 中的通訊用戶端 Factory。 Factory 會在內部使用解析程式來產生可用來與服務通訊的用戶端物件。

### <a name="communication-clients-and-factories"></a>通訊用戶端和 Factory
通訊 Factory 程式庫會實作典型的錯誤處理重試模式，更容易重試與已解析服務端點的連接。 儘管您提供錯誤處理常式，Factory 程式庫還是會提供重試機制。

`ICommunicationClientFactory(C#) / CommunicationClientFactory(Java)` 定義通訊用戶端 Factory 所實作的基底介面，並產生可以與 Service Fabric 服務通訊的用戶端。 CommunicationClientFactory 的實作取決於用戶端想要通訊的 Service Fabric 服務所使用的通訊堆疊。 Reliable Services API 提供 `CommunicationClientFactoryBase<TCommunicationClient>`。 這樣可以提供 CommunicationClientFactory 介面的基底實作，並執行所有通訊堆疊都通用的工作。 (這些工作包括使用 ServicePartitionResolver 來判斷服務端點)。 用戶端通常會實作 CommunicationClientFactoryBase 抽象類別來處理通訊堆疊專用的邏輯。

通訊用戶端只會接收位址，並使用它來連接到服務。 用戶端可以使用它想要的任何通訊協定。

```csharp
public class MyCommunicationClient : ICommunicationClient
{
    public ResolvedServiceEndpoint Endpoint { get; set; }

    public string ListenerName { get; set; }

    public ResolvedServicePartition ResolvedServicePartition { get; set; }
}
```
```java
public class MyCommunicationClient implements CommunicationClient {

    private ResolvedServicePartition resolvedServicePartition;
    private String listenerName;
    private ResolvedServiceEndpoint endPoint;

    /*
     * Getters and Setters
     */
}
```

用戶端 Factory 主要是負責建立通訊用戶端。 對於不會維持持續連線的用戶端 (例如 HTTP 用戶端)，用戶端 Factory 只需建立並傳回用戶端。 其他會維持持續連線的通訊協定 (例如某些二進位通訊協定) 也應該由 Factory 驗證，以判斷是否需要重新建立連線。  

```csharp
public class MyCommunicationClientFactory : CommunicationClientFactoryBase<MyCommunicationClient>
{
    protected override void AbortClient(MyCommunicationClient client)
    {
    }

    protected override Task<MyCommunicationClient> CreateClientAsync(string endpoint, CancellationToken cancellationToken)
    {
    }

    protected override bool ValidateClient(MyCommunicationClient clientChannel)
    {
    }

    protected override bool ValidateClient(string endpoint, MyCommunicationClient client)
    {
    }
}
```
```java
public class MyCommunicationClientFactory extends CommunicationClientFactoryBase<MyCommunicationClient> {

    @Override
    protected boolean validateClient(MyCommunicationClient clientChannel) {
    }

    @Override
    protected boolean validateClient(String endpoint, MyCommunicationClient client) {
    }

    @Override
    protected CompletableFuture<MyCommunicationClient> createClientAsync(String endpoint) {
    }

    @Override
    protected void abortClient(MyCommunicationClient client) {
    }
}
```

最後，例外狀況處理常式會負責判斷發生例外狀況時所要採取的動作。 例外狀況會分類為**可重試**和**不可重試**。

* **不可重試**的例外狀況只會重新擲回給呼叫端。
* **不可重試**的例外狀況會進一步分類為**暫時性**和**非暫時性**。
  * **暫時性** 例外狀況是只會重試而不會重新解析服務端點位址的例外狀況。 這類例外狀況包括暫時性網路問題或服務錯誤回應，但不包括指出服務端點位址不存在的錯誤回應。
  * **非暫時性** 例外狀況是需要重新解析服務端點位址的例外狀況。 這類例外狀況包括指出無法連上服務端點 (表示服務已移至其他節點) 的例外狀況。

`TryHandleException` 會做出有關特定例外狀況的決定。 如果它**不知道**要對例外狀況做出哪些決定，則應傳回 **false**。 如果它**知道**如何做決定，則應該據以設定結果並傳回 **true**。

```csharp
class MyExceptionHandler : IExceptionHandler
{
    public bool TryHandleException(ExceptionInformation exceptionInformation, OperationRetrySettings retrySettings, out ExceptionHandlingResult result)
    {
        // if exceptionInformation.Exception is known and is transient (can be retried without re-resolving)
        result = new ExceptionHandlingRetryResult(exceptionInformation.Exception, true, retrySettings, retrySettings.DefaultMaxRetryCount);
        return true;


        // if exceptionInformation.Exception is known and is not transient (indicates a new service endpoint address must be resolved)
        result = new ExceptionHandlingRetryResult(exceptionInformation.Exception, false, retrySettings, retrySettings.DefaultMaxRetryCount);
        return true;

        // if exceptionInformation.Exception is unknown (let the next IExceptionHandler attempt to handle it)
        result = null;
        return false;
    }
}
```
```java
public class MyExceptionHandler implements ExceptionHandler {

    @Override
    public ExceptionHandlingResult handleException(ExceptionInformation exceptionInformation, OperationRetrySettings retrySettings) {        

        /* if exceptionInformation.getException() is known and is transient (can be retried without re-resolving)
         */
        result = new ExceptionHandlingRetryResult(exceptionInformation.getException(), true, retrySettings, retrySettings.getDefaultMaxRetryCount());
        return true;


        /* if exceptionInformation.getException() is known and is not transient (indicates a new service endpoint address must be resolved)
         */
        result = new ExceptionHandlingRetryResult(exceptionInformation.getException(), false, retrySettings, retrySettings.getDefaultMaxRetryCount());
        return true;

        /* if exceptionInformation.getException() is unknown (let the next ExceptionHandler attempt to handle it)
         */
        result = null;
        return false;

    }
}
```
### <a name="putting-it-all-together"></a>總整理
使用以通訊協定建構的 `ICommunicationClient(C#) / CommunicationClient(Java)`、`ICommunicationClientFactory(C#) / CommunicationClientFactory(Java)` 和 `IExceptionHandler(C#) / ExceptionHandler(Java)`，`ServicePartitionClient(C#) / FabricServicePartitionClient(Java)` 會將它全部包裝在一起，並為這些元件提供錯誤處理和服務分割區位址解析迴圈。

```csharp
private MyCommunicationClientFactory myCommunicationClientFactory;
private Uri myServiceUri;

var myServicePartitionClient = new ServicePartitionClient<MyCommunicationClient>(
    this.myCommunicationClientFactory,
    this.myServiceUri,
    myPartitionKey);

var result = await myServicePartitionClient.InvokeWithRetryAsync(async (client) =>
   {
      // Communicate with the service using the client.
   },
   CancellationToken.None);

```
```java
private MyCommunicationClientFactory myCommunicationClientFactory;
private URI myServiceUri;

FabricServicePartitionClient myServicePartitionClient = new FabricServicePartitionClient<MyCommunicationClient>(
    this.myCommunicationClientFactory,
    this.myServiceUri,
    myPartitionKey);

CompletableFuture<?> result = myServicePartitionClient.invokeWithRetryAsync(client -> {
      /* Communicate with the service using the client.
       */
   });

```

## <a name="next-steps"></a>後續步驟
* [有可靠服務的 ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md)
* [使用 Reliable Services 遠端服務進行遠端程序呼叫](service-fabric-reliable-services-communication-remoting.md)
* [使用 Reliable Services 的 WCF 通訊](service-fabric-reliable-services-communication-wcf.md)
