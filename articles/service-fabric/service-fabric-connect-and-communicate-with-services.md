---
title: 連接至 Azure Service Fabric 中的服務並與其進行通訊 | Microsoft Docs
description: 了解如何解析、連接至 Service Fabric 應用程式中的服務並與其進行通訊。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: msfussell
ms.assetid: 7d1052ec-2c9f-443d-8b99-b75c97266e6c
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/01/2017
ms.author: vturecek
ms.openlocfilehash: 2b6fd2373a9cd0b376a6c8729d5952c5fc48ddf8
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34205581"
---
# <a name="connect-and-communicate-with-services-in-service-fabric"></a>連接至 Service Fabric 中的服務並與其進行通訊
在 Service Fabric 中，服務會在 Service Fabric 叢集中的某處執行，通常是分散到多個 VM。 它可以由服務擁有者或是 Service Fabric 自動從某個位置移到其他位置。 服務無法以靜態方式繫結至特定電腦或位址。

Service Fabric 應用程式通常是由許多不同的服務組成，每一個服務用來執行特定的工作。 這些服務可能會彼此進行通訊以形成完整的函式，例如轉譯 Web 應用程式的不同部分。 有些用戶端應用程式會連接到服務並與其通訊。 本文件討論如何設定您在 Service Fabric 中的服務之間的通訊。

這段 Microsoft Virtual Academy 影片也會討論服務通訊︰<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=iYFCk76yC_6706218965">  
<img src="./media/service-fabric-connect-and-communicate-with-services/CommunicationVid.png" WIDTH="360" HEIGHT="244">  
</a></center>

## <a name="bring-your-own-protocol"></a>引進您自己的通訊協定
Service Fabric 可以幫助管理您的服務的生命週期，但是它不會決定您的服務要執行的動作。 這包含通訊： 當您的服務是由 Service Fabric 開啟時，就是您的服務設定連入要求端點的機會，使用您想要的任何通訊協定或通訊堆疊。 您的服務會接聽一般 **IP:port** 位址，使用任何定址配置，例如 URI。 多個服務執行個體或複本可能共用主機處理序，在此情況下，它們必須使用不同的連接埠，或使用連接埠共用機制，如 Windows 的 http.sys 核心驅動程式。 在任一情況下，主機處理序中的每個服務執行個體或複本必須可以唯一定址。

![服務端點][1]

## <a name="service-discovery-and-resolution"></a>服務探索和解析
在分散式系統中，服務可能會隨著時間從一部電腦移動到另一部電腦。 發生這種情形的原因有很多，包括資源平衡、升級、容錯移轉或向外延展。這表示服務端點位址會在服務移至具有不同 IP 位址的節點時變更，而且如果服務使用動態選取的連接埠，可能會在不同的連接埠上開啟。

![服務的分佈][7]

Service Fabric 提供稱為「命名服務」的探索和解析服務。 「命名服務」會維護資料表，該資料表將已命名服務執行個體對應至它們接聽的端點位址。 Service Fabric 中的所有已命名服務執行個體，皆如 URI 擁有唯一的名稱，例如， `"fabric:/MyApplication/MyService"`。 服務的名稱不會隨著服務的存留期變更，當服務移動時，只有端點位址可以變更。 這類似於具有常數 URL 的網站，但 IP 位址可能會變更。 並且類似於 Web 上的 DNS，可解析連接至 IP 位址的網站 URL，Service Fabric 具有註冊機構，會將服務名稱對應至其端點位址。

![服務端點][2]

解析和連接到服務牽涉到在迴圈中執行的下列步驟︰

* **解析**︰取得端點，該端點是由服務從「命名服務」所發佈。
* **連接**︰透過在該端點上所使用的任何通訊協定連接至服務。
* **重試**︰連接嘗試可能會因為各種原因而失敗，例如，如果服務自從上次端點位址解析之後已經移動。 在此情況下，則必須重試先前的解析和連接步驟，且此循環會重複執行直到連接成功為止。

## <a name="connecting-to-other-services"></a>連接到其他服務
叢集內彼此連接的服務通常可以直接存取其他服務的端點，因為叢集中的節點位於相同的本機網路上。 為了能夠更輕鬆在服務之間連接，Service Fabric 提供使用「命名服務」的額外服務。 DNS 服務和反向 proxy 服務。


### <a name="dns-service"></a>DNS 服務
由於許多服務 (特別是容器化服務) 都可以有現有的 URL 名稱，因此能夠使用標準 DNS 通訊協定 (而不是「命名服務」通訊協定) 來解析這些名稱就非常便利，特別是在應用程式的「隨即轉移」案例中。 這就是 DNS 服務的功能所在。 它可讓您將 DNS 服務對應到某個服務名稱，再由此解析端點 IP 位址。 

如下圖所示，在 Service Fabric 叢集中執行的 DNS 服務會將 DNS 名稱對應到服務名稱，然後由「命名服務」解析後傳回要連接的端點位址。 服務的 DNS 名稱是在建立時提供的。 

![服務端點][9]

如需有關如何使用 DNS 服務的更多詳細資料，請參閱 [Azure Service Fabric 中的 DNS 服務](service-fabric-dnsservice.md)一文。

### <a name="reverse-proxy-service"></a>反向 Proxy服務
反向 Proxy 可處理叢集中公開 HTTP 端點 (包括 HTTPS) 的服務。 反向 Proxy 藉由採用特定的 URI 格式，以及處理一個服務使用「命名服務」與另一個服務進行通訊所需的解析、連接、重試步驟，將呼叫其他服務及其方法大幅簡化。 換句話說，它會在呼叫其他服務時，透過讓此呼叫就像呼叫 URL 一樣簡單，對您隱藏「命名服務」。

![服務端點][10]

如需有關如何使用反向 Proxy 服務的詳細資訊，請參閱 [Azure Service Fabric 中的反向 Proxy](service-fabric-reverseproxy.md) 一文。

## <a name="connections-from-external-clients"></a>從外部用戶端連接
叢集內彼此連接的服務通常可以直接存取其他服務的端點，因為叢集中的節點位於相同的本機網路上。 但是，在相同的環境中，叢集可能會位於負載平衡器後方，該負載平衡器會透過有限制的一組連接埠路由傳送外部輸入流量。 在這些情況下，服務仍然可以使用「命名服務」，彼此進行通訊及解析位址，但是必須採取額外的步驟，讓外部用戶端連接至服務。

## <a name="service-fabric-in-azure"></a>Azure 中的 Service Fabric
Azure 中的 Service Fabric 叢集位於 Azure 負載平衡器後方。 到叢集的所有外部流量必須經過負載平衡器。 負載平衡器會自動將指定連接埠上的輸入流量轉送至具有相同的開啟連接埠的隨機「節點」  。 Azure Load Balancer 只會知道「節點」上開啟的連接埠，它不知道由個別「服務」開啟的連接埠。

![Azure 負載平衡器和 Service Fabric 拓撲][3]

例如，若要在連接埠 **80**上接受外部流量，必須設定下列項目︰

1. 寫入在連接埠 80 上接聽的服務。 在服務的 ServiceManifest.xml 中設定連接埠 80，並且在服務中開啟接聽程式，例如自我裝載的 Web 伺服器。

    ```xml
    <Resources>
        <Endpoints>
            <Endpoint Name="WebEndpoint" Protocol="http" Port="80" />
        </Endpoints>
    </Resources>
    ```
    ```csharp
        class HttpCommunicationListener : ICommunicationListener
        {
            ...

            public Task<string> OpenAsync(CancellationToken cancellationToken)
            {
                EndpointResourceDescription endpoint =
                    serviceContext.CodePackageActivationContext.GetEndpoint("WebEndpoint");

                string uriPrefix = $"{endpoint.Protocol}://+:{endpoint.Port}/myapp/";

                this.httpListener = new HttpListener();
                this.httpListener.Prefixes.Add(uriPrefix);
                this.httpListener.Start();

                string publishUri = uriPrefix.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);
                return Task.FromResult(publishUri);
            }

            ...
        }

        class WebService : StatelessService
        {
            ...

            protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
            {
                return new[] { new ServiceInstanceListener(context => new HttpCommunicationListener(context))};
            }

            ...
        }
    ```
    ```java
        class HttpCommunicationlistener implements CommunicationListener {
            ...

            @Override
            public CompletableFuture<String> openAsync(CancellationToken arg0) {
                EndpointResourceDescription endpoint =
                    this.serviceContext.getCodePackageActivationContext().getEndpoint("WebEndpoint");
                try {
                    HttpServer server = com.sun.net.httpserver.HttpServer.create(new InetSocketAddress(endpoint.getPort()), 0);
                    server.start();

                    String publishUri = String.format("http://%s:%d/",
                        this.serviceContext.getNodeContext().getIpAddressOrFQDN(), endpoint.getPort());
                    return CompletableFuture.completedFuture(publishUri);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }

            ...
        }

        class WebService extends StatelessService {
            ...

            @Override
            protected List<ServiceInstanceListener> createServiceInstanceListeners() {
                <ServiceInstanceListener> listeners = new ArrayList<ServiceInstanceListener>();
                listeners.add(new ServiceInstanceListener((context) -> new HttpCommunicationlistener(context)));
                return listeners;       
            }

            ...
        }
    ```
2. 在 Azure 中建立 Service Fabric 叢集，並指定連接埠 **80** 做為將裝載服務的節點類型的自訂端點連接埠。 如果您有一個以上的節點類型，您可以在服務上設定「放置條件約束」  ，以確保它只會在具有已開啟的自訂端點連接埠的節點類型上執行。

    ![開啟節點類型上的連接埠][4]
3. 一旦建立叢集，在叢集的資源群組中設定 Azure 負載平衡器，以轉送連接埠 80 上的流量。 透過 Azure 入口網站建立叢集時，會針對每個已設定的自訂端點連接埠設定這個項目。

    ![轉送 Azure 負載平衡器的流量][5]
4. Azure 負載平衡器使用探查，來決定是否要將流量傳送到特定節點。 探查會定期檢查每個節點上的端點，以判斷節點是否有回應。 如果探查在設定的次數之後無法接收到回應，負載平衡器會停止將流量傳送到該節點。 透過 Azure 入口網站建立叢集時，會針對每個已設定的自訂端點連接埠自動設定探查。

    ![轉送 Azure 負載平衡器的流量][8]

請務必記住，Azure Load Balancer 和探查只知道「節點」，不知道在節點上執行的「服務」。 Azure 負載平衡器一律會將流量傳送到回應探查的節點，因此必須小心以確保可以在能夠回應探查的節點上使用服務。

## <a name="reliable-services-built-in-communication-api-options"></a>Reliable Services：內建的通訊 API 選項
Reliable Services 架構隨附數個預先建置的通訊選項。 最適合您選項的決定取決於如何選擇程式設計模型、通訊架構以及用來撰寫您服務的程式語言。

* **沒有特定通訊協定**：如果您沒有特定的通訊架構選擇，但您想要快速啟動並執行，則適合您的理想選項為[遠端服務](service-fabric-reliable-services-communication-remoting.md)，允許 Reliable Services 和 Reliable Actors 的強型別遠端程序呼叫。 若要開始使用服務通訊，這是最簡單且快速的方式。 遠端服務會處理服務位址、連接、重試和錯誤處理的解析。 這同時適用 C# 和 Java 應用程式。
* **HTTP**︰對於無從驗證語言的通訊，HTTP 提供業界標準的選擇，具有工具與許多不同語言的 HTTP 伺服器，都受到 Service Fabric 的支援。 服務可以使用任何可用的 HTTP 堆疊，包括 C# 應用程式適用的 [ASP.NET Web API](service-fabric-reliable-services-communication-webapi.md)。 以 C# 撰寫的用戶端可以利用 `ICommunicationClient` 和 `ServicePartitionClient` 類別，而使用 Java 的用戶端則可以利用 `CommunicationClient` 和 `FabricServicePartitionClient` 進行[服務解析、HTTP 連線和重試迴圈](service-fabric-reliable-services-communication.md)。
* **WCF**：若您有現有的程式碼且使用 WCF 作為通訊架構，則您可以針對伺服器端使用 `WcfCommunicationListener` 類別，並針對用戶端使用 `WcfCommunicationClient` 和 `ServicePartitionClient` 類別。 不過，這只適用以 Windows 為基礎的叢集上的 C# 應用程式。 如需詳細資訊，請參閱本文中 [通訊堆疊的 WCF 式實作](service-fabric-reliable-services-communication-wcf.md)。

## <a name="using-custom-protocols-and-other-communication-frameworks"></a>使用自訂通訊協定以及其他通訊架構
服務可以使用任何通訊協定或架構進行通訊，無論是透過 TCP 通訊端的自訂二進位通訊協定，或透過 [Azure 事件中樞](https://azure.microsoft.com/services/event-hubs/)或 [Azure IoT 中樞](https://azure.microsoft.com/services/iot-hub/)的串流事件。 Service Fabric 提供通訊 API，您可以將通訊堆疊插入其中，同時讓您免於探索和連接的所有工作。 如需詳細資訊，請參閱本文中 [Reliable Service 通訊模型](service-fabric-reliable-services-communication.md) 。

## <a name="next-steps"></a>後續步驟
您可以在 [Reliable Services 通訊模型](service-fabric-reliable-services-communication.md)中深入了解概念和可用的 API，然後快速地開始使用[遠端服務](service-fabric-reliable-services-communication-remoting.md)或深入了解如何使用 [Web API 與 OWIN 自我裝載](service-fabric-reliable-services-communication-webapi.md)撰寫通訊接聽程式。

[1]: ./media/service-fabric-connect-and-communicate-with-services/serviceendpoints.png
[2]: ./media/service-fabric-connect-and-communicate-with-services/namingservice.png
[3]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancertopology.png
[4]: ./media/service-fabric-connect-and-communicate-with-services/nodeport.png
[5]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerport.png
[7]: ./media/service-fabric-connect-and-communicate-with-services/distributedservices.png
[8]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerprobe.png
[9]: ./media/service-fabric-connect-and-communicate-with-services/dns.png
[10]: ./media/service-fabric-reverseproxy/internal-communication.png
