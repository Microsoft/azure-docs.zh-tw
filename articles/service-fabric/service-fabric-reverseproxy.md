---
title: Azure Service Fabric 反向 Proxy | Microsoft Docs
description: 使用 Service Fabric 反向 Proxy 從叢集內部和外部與微服務進行通訊。
services: service-fabric
documentationcenter: .net
author: BharatNarasimman
manager: timlt
editor: vturecek
ms.assetid: 47f5c1c1-8fc8-4b80-a081-bc308f3655d3
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 11/03/2017
ms.author: bharatn
ms.openlocfilehash: 521a7b90b971ff3ba867945a4713b1f6dc8dbebc
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39503514"
---
# <a name="reverse-proxy-in-azure-service-fabric"></a>Azure Service Fabric 中的反向 Proxy
Azure Service Fabric 內建的反向 Proxy 可協助在 Service Fabric 叢集中執行的微服務進行探索，並與其他擁有 http 端點的服務通訊。

## <a name="microservices-communication-model"></a>微服務通訊模型
Service Fabric 中的微服務在叢集中的節點子集上執行，而且可以因為各種原因在不同的節點之間轉移。 因此，微服務的端點可以動態變更。 若要探索以及與叢集中的其他服務通訊，微服務必須完成下列步驟：

1. 透過命名服務解析服務位置。
2. 連線至服務。
3. 將上述步驟包裝在實作服務解析的迴圈中，然後重試原則以在連線失敗時套用

如需詳細資訊，請參閱[連線至服務並與其進行通訊](service-fabric-connect-and-communicate-with-services.md)。

### <a name="communicating-by-using-the-reverse-proxy"></a>使用反向 Proxy 進行通訊
反向 Proxy 是一項服務，它會在每個節點上代表用戶端服務執行及處理端點解析、自動重試和其他連線失敗。 經過設定後，當反向 Proxy 處理來自用戶端服務的要求時，可以套用不同的原則。 透過反向 Proxy，用戶端服務可以使用任何用戶端 HTTP 通訊程式庫，而且服務中不需要特殊的解析和重試邏輯。 

反向 Proxy 會公開本機節點上的一個或多個端點，供用戶端服務用來將要求傳送給其他服務。

![內部通訊][1]

> [!NOTE]
> **支援的平台**
>
> Service Fabric 中的反向 Proxy 目前支援下列平台
> * *Windows 叢集*：Windows 8 和更新版本或 Windows Server 2012 和更新版本
> * *Linux 叢集*：反向 Proxy 目前不適用於 Linux 叢集
>

## <a name="reaching-microservices-from-outside-the-cluster"></a>從叢集外部連線到微服務
微服務的預設外部通訊模型是選擇加入模型，每項服務都無法從外部用戶端直接存取。 [Azure Load Balancer](../load-balancer/load-balancer-overview.md) 是微服務與外部用戶端之間的網路界限，負責執行網路位址轉譯，並將外部要求轉送給內部的 IP:port 端點。 若要讓微服務的端點能直接從外部用戶端來存取，您必須先將 Load Balancer 設為轉送流量到服務在叢集中使用的每個連接埠。 此外，大部分的微服務 (特別是可設定狀態的微服務) 不存在於叢集的所有節點上。 微服務可以在容錯移轉時於節點之間移動。 在這種情況下，Load Balancer 無法有效地判斷它應該將流量轉送到之複本的目標節點位置。

### <a name="reaching-microservices-via-the-reverse-proxy-from-outside-the-cluster"></a>從叢集外部透過反向 Proxy 連線到微服務
您可以在 Load Balancer 中直接設定反向 Proxy 的連接埠，而不需要在 Load Balancer 中設定個別服務的連接埠。 此組態可讓叢集外部的用戶端透過反向 Proxy 連線到叢集內部的服務，而不需要額外設定。

![外部通訊][0]

> [!WARNING]
> 當您在 Load Balancer 中設定反向 Proxy 的連接埠時，叢集中公開 HTTP 端點的所有微服務就能從叢集外部尋址。 這表示已判定的惡意使用者可以探索必定是內部的微服務。 這可能代表可以利用的嚴重弱點，例如：
>
> * 惡意使用者可重複呼叫沒有充分強化攻擊面的內部服務，以啟動拒絕服務的攻擊。
> * 惡意使用者可以將格式不正確的封包，傳遞至導致非預期行為的內部服務。
> * 必定是內部的服務可能會傳回不打算對叢集外部服務公開的私人或敏感性資訊，因而對惡意使用者公開此敏感性資訊。 
>
> 在公開反向 Proxy 連接埠之前，確定您完全了解您的叢集以及在其上執行的應用程式，並減輕其潛在安全性分歧。 
>


## <a name="uri-format-for-addressing-services-by-using-the-reverse-proxy"></a>透過反向 Proxy 定址服務的 URI 格式
反向 Proxy 使用特定的統一資源識別項 (URI) 格式來識別連入要求應該轉送到哪個服務資料分割︰

```
http(s)://<Cluster FQDN | internal IP>:Port/<ServiceInstanceName>/<Suffix path>?PartitionKey=<key>&PartitionKind=<partitionkind>&ListenerName=<listenerName>&TargetReplicaSelector=<targetReplicaSelector>&Timeout=<timeout_in_seconds>
```

* **http(s)：** 反向 Proxy 可設定為接受 HTTP 或 HTTPS 流量。 對於 HTTPS 轉送，在您設定反向 Proxy 於 HTTPS 接聽後，請參閱[使用反向 Proxy 連線安全的服務](service-fabric-reverseproxy-configure-secure-communication.md)。
* **叢集完整網域名稱 (FQDN) | 內部 IP：** 如果是外部用戶端，您可以設定反向 Proxy 讓其可透過叢集網域 (例如 mycluster.eastus.cloudapp.azure.com) 來聯繫到。 根據預設，反向 Proxy 會在每個節點上執行。 如果是內部流量，反向 Proxy 可在 localhost 或任何內部節點 IP (例如 10.0.0.1) 上聯繫到。
* **連接埠︰** 這是為反向 Proxy 指定的連接埠，例如 19081。
* **ServiceInstanceName：** 這是您嘗試不使用「fabric:/」配置來連線到之已部署服務執行個體的完整名稱。 例如，若要連線到 fabric:/myapp/myservice/ 服務，可以使用 myapp/myservice。

    服務執行個體名稱區分大小寫。 對於 URL 中的服務執行個體名稱使用不同的大小寫，會導致要求失敗，並顯示「404 (找不到)」。
* **尾碼路徑︰** 這是要連線到之服務的實際 URL 路徑，例如 myapi/values/add/3。
* **PartitionKey：** 若為資料分割服務，這是您想要連線的資料分割計算資料分割金鑰。 請注意，這不是  資料分割識別碼 GUID。 使用單一資料分割配置的服務不需要這個參數。
* **PartitionKind：** 這是服務資料分割配置。 這可以是 'Int64Range' 或 'Named'。 使用單一資料分割配置的服務不需要這個參數。
* **ListenerName** 服務傳回的端點格式為 {"Endpoints":{"Listener1":"Endpoint1","Listener2":"Endpoint2" ...}}。 當服務公開多個端點時，這可識別用戶端要求應該轉送至哪個端點。 如果服務只有一個接聽程式，這可省略。
* **TargetReplicaSelector** 這指定應該如何選取目標複本或執行個體。
  * 當目標服務具狀態時，TargetReplicaSelector 可以是下列其中一個：'PrimaryReplica'、'RandomSecondaryReplica' 或 'RandomReplica'。 未指定此參數時，預設值是 'PrimaryReplica'。
  * 當目標服務無狀態時，反向 Proxy 會挑選服務資料分割的隨機執行個體，將要求轉送至此執行個體。
* **逾時︰** 指定由反向 Proxy 建立的 HTTP 要求代替用戶端要求傳送到服務的逾時。 預設值為 60 秒。 這是選擇性參數。

### <a name="example-usage"></a>使用方式範例
例如，讓我們採用在下列 URL 上開啟 HTTP 接聽程式的 fabric:/MyApp/MyService 服務：

```
http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/
```

以下是服務的資源︰

* `/index.html`
* `/api/users/<userId>`

如果服務使用單一資料分割配置，則不需要 PartitionKey 和 PartitionKind 查詢字串參數，可透過閘道以下列方式連線到服務︰

* 外部： `http://mycluster.eastus.cloudapp.azure.com:19081/MyApp/MyService`
* 內部： `http://localhost:19081/MyApp/MyService`

如果服務使用統一 Int64 資料分割配置，則必須使用 PartitionKey 和 PartitionKind 查詢字串參數來連線到服務的資料分割︰

* 外部： `http://mycluster.eastus.cloudapp.azure.com:19081/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`
* 內部： `http://localhost:19081/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`

若要連線到服務公開的資源，只要將資源路徑放在 URL 的服務名稱之後︰

* 外部： `http://mycluster.eastus.cloudapp.azure.com:19081/MyApp/MyService/index.html?PartitionKey=3&PartitionKind=Int64Range`
* 內部： `http://localhost:19081/MyApp/MyService/api/users/6?PartitionKey=3&PartitionKind=Int64Range`

閘道會將這些要求轉送到服務的 URL：

* `http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/index.html`
* `http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/api/users/6`

## <a name="special-handling-for-port-sharing-services"></a>連接埠共用服務特殊處理
Service Fabric 反向 Proxy 會嘗試重新解析服務位址，並在無法連線到服務時重試要求。 通常，當服務無法連線時，即表示服務執行個體或複本已移至另一個節點，這是其一般生命週期的過程。 發生這種情況時，反向 Proxy 可能會收到指出端點已不在原先解析的位址上開啟的網路連線錯誤。

不過，複本或服務執行個體可以共用主機處理序，在由以 http.sys 為基礎的 Web 伺服器託管時也可以共用連接埠，這些 Web 伺服器包括︰

* [System.Net.HttpListener](https://msdn.microsoft.com/library/system.net.httplistener%28v=vs.110%29.aspx)
* [ASP.NET Core WebListener](https://docs.asp.net/latest/fundamentals/servers.html#weblistener)
* [Katana](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.OwinSelfHost/)

在此情況下，可能是 Web 伺服器可用於主機處理序和回應要求，但已解析的服務執行個體或複本已無法再於主機上提供。 在此情況下，閘道將從 Web 伺服器收到 HTTP 404 回應。 因此，HTTP 404 回應可以有兩個不同的意義：

- 案例 1︰服務位址正確，但使用者所要求的資源不存在。
- 案例 2︰服務位址不正確，而且使用者所要求的資源可能存在於不同節點上。

第一個案例是一般的 HTTP 404，會被視為使用者錯誤。 不過，在第二個案例中，使用者要求了存在的資源。 反向 Proxy 之所以找不到它，是因為服務本身已移動。 反向 Proxy 必須重新解析位址，然後重試要求。

因此反向 Proxy 需要能夠區分這兩種案例的方法。 為了區分，您需要來自伺服器的提示。

* 根據預設，反向 Proxy 會採用案例 2，並嘗試重新解析並發出要求。
* 若要向反向 Proxy 表明是案例 #1，服務應傳回下列 HTTP 回應標頭︰

  `X-ServiceFabric : ResourceNotFound`

此 HTTP 回應標頭表示指出一般的 HTTP 404 情況，即要求的資源不存在，反向 Proxy 將不會嘗試重新解析服務位址。

## <a name="special-handling-for-services-running-in-containers"></a>針對在容器中執行之服務的特殊處理

針對在容器內執行的服務，您可以使用環境變數 `Fabric_NodeIPOrFQDN` 來建構[反向 Proxy URL](#uri-format-for-addressing-services-by-using-the-reverse-proxy)，如下列程式碼所示：

```csharp
    var fqdn = Environment.GetEnvironmentVariable("Fabric_NodeIPOrFQDN");
    var serviceUrl = $"http://{fqdn}:19081/DockerSFApp/UserApiContainer";
```
針對本機叢集，預設會將 `Fabric_NodeIPOrFQDN` 設定為 "localhost"。 使用 `-UseMachineName` 參數啟動本機叢集，以確保容器可連線至在節點上執行的反向 Proxy。 如需詳細資訊，請參閱[設定開發人員環境以對容器進行偵錯](service-fabric-how-to-debug-windows-containers.md#configure-your-developer-environment-to-debug-containers)。

## <a name="next-steps"></a>後續步驟
* [在叢集上安裝及設定反向 Proxy](service-fabric-reverseproxy-setup.md)。
* [設定透過反向 Proxy 轉送到安全的 HTTP 服務](service-fabric-reverseproxy-configure-secure-communication.md)
* 請參閱 [GitHub 上的範例專案](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)中服務之間的 HTTP 通訊範例。
* [使用 Reliable Services 遠端服務進行遠端程序呼叫](service-fabric-reliable-services-communication-remoting.md)
* [在 Reliable Services 中使用 OWIN 的 Web API](service-fabric-reliable-services-communication-webapi.md)
* [使用 Reliable Services 的 WCF 通訊](service-fabric-reliable-services-communication-wcf.md)

[0]: ./media/service-fabric-reverseproxy/external-communication.png
[1]: ./media/service-fabric-reverseproxy/internal-communication.png
