---
title: 保護與 Azure Service Fabric 中的 Java 進行服務遠端通訊時的安全 | Microsoft Docs
description: 了解如何針對在 Azure Service Fabric 叢集中執行的 Java 可靠服務，保護以服務遠端為基礎的通訊。
services: service-fabric
documentationcenter: java
author: PavanKunapareddyMSFT
manager: timlt
ms.assetid: ''
ms.service: service-fabric
ms.devlang: java
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 06/30/2017
ms.author: pakunapa
ms.openlocfilehash: cbefb3ede6d0d1fe21065b49c84db9f4db5dd39c
ms.sourcegitcommit: 0fa8b4622322b3d3003e760f364992f7f7e5d6a9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37020808"
---
# <a name="secure-service-remoting-communications-in-a-java-service"></a>保護 Java 服務中的服務遠端通訊
> [!div class="op_single_selector"]
> * [Windows 上的 C# ](service-fabric-reliable-services-secure-communication.md)
> * [在 Linux 上使用 Java](service-fabric-reliable-services-secure-communication-java.md)
>
>

安全性是通訊最為重視的其中一個部分。 Reliable Services 應用程式架構會提供可用來改善安全性的一些預先建置通訊堆疊和工具。 本文探討如何改善在 Java 服務中使用服務遠端時的安全性。 它建置於現有[範例](service-fabric-reliable-services-communication-remoting-java.md)上，其會說明如何針對以 Java 撰寫的可靠服務設定遠端。 

若要在搭配 Java 服務使用服務遠端時協助保護服務安全，請遵循下列步驟：

1. 建立 `HelloWorldStateless`介面，這個介面會定義將在您的服務上用於遠端程序呼叫的方法。 您的服務將使用在 `microsoft.serviceFabric.services.remoting.fabricTransport.runtime` 封裝中宣告的 `FabricTransportServiceRemotingListener`。 這是提供遠端功能的 `CommunicationListener` 實作。

    ```java
    public interface HelloWorldStateless extends Service {
        CompletableFuture<String> getHelloWorld();
    }

    class HelloWorldStatelessImpl extends StatelessService implements HelloWorldStateless {
        @Override
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this);
            }));
        return listeners;
        }

        public CompletableFuture<String> getHelloWorld() {
            return CompletableFuture.completedFuture("Hello World!");
        }
    }
    ```
2. 新增接聽程式設定和安全性認證。

    確定您想要用來協助保護服務通訊安全的憑證已安裝於叢集的所有節點上。 針對在 Linux 上執行的服務，憑證必須以 PEM 格式的檔案形式來提供；可以是包含憑證和私密金鑰的 `.pem` 檔案，或是包含憑證的 `.crt` 檔案和包含私密金鑰的 `.key` 檔案。 若要深入了解，請參閱 [Linux 節點上 X.509 憑證的位置與格式](./service-fabric-configure-certificates-linux.md#location-and-format-of-x509-certificates-on-linux-nodes)。
    
    有兩種方式可提供接聽程式設定和安全性認證：

   1. 使用 [組態封裝](service-fabric-application-and-service-manifests.md)提供它們：

       在 settings.xml 檔案中新增具名的 `TransportSettings` 區段。

       ```xml
       <!--Section name should always end with "TransportSettings".-->
       <!--Here we are using a prefix "HelloWorldStateless".-->
        <Section Name="HelloWorldStatelessTransportSettings">
            <Parameter Name="MaxMessageSize" Value="10000000" />
            <Parameter Name="SecurityCredentialsType" Value="X509_2" />
            <Parameter Name="CertificatePath" Value="/path/to/cert/BD1C71E248B8C6834C151174DECDBDC02DE1D954.crt" />
            <Parameter Name="CertificateProtectionLevel" Value="EncryptandSign" />
            <Parameter Name="CertificateRemoteThumbprints" Value="BD1C71E248B8C6834C151174DECDBDC02DE1D954" />
        </Section>

       ```

       在此情況下， `createServiceInstanceListeners` 方法看起來像這樣：

       ```java
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this, FabricTransportRemotingListenerSettings.loadFrom(HelloWorldStatelessTransportSettings));
            }));
            return listeners;
        }
       ```

        如果您在 settings.xml 檔案中新增 `TransportSettings` 區段，而沒有任何前置詞，則 `FabricTransportListenerSettings` 預設會載入此區段中的所有設定。

        ```xml
        <!--"TransportSettings" section without any prefix.-->
        <Section Name="TransportSettings">
            ...
        </Section>
        ```
        在此情況下， `CreateServiceInstanceListeners` 方法看起來像這樣：

        ```java
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this);
            }));
            return listeners;
        }
       ```
3. 如果在安全服務上使用遠端堆疊來呼叫方法，而不是使用 `microsoft.serviceFabric.services.remoting.client.ServiceProxyBase` 類別來建立服務 Proxy，請使用 `microsoft.serviceFabric.services.remoting.client.FabricServiceProxyFactory`。

    如果用戶端程式碼正在當作服務一部分執行，則可以從 settings.xml 檔案中載入 `FabricTransportSettings` 。 建立與服務程式碼類似的 TransportSettings 區段，如前所示。 對用戶端程式碼進行下列變更：

    ```java

    FabricServiceProxyFactory serviceProxyFactory = new FabricServiceProxyFactory(c -> {
            return new FabricTransportServiceRemotingClientFactory(FabricTransportRemotingSettings.loadFrom("TransportPrefixTransportSettings"), null, null, null, null);
        }, null)

    HelloWorldStateless client = serviceProxyFactory.createServiceProxy(HelloWorldStateless.class,
        new URI("fabric:/MyApplication/MyHelloWorldService"));

    CompletableFuture<String> message = client.getHelloWorld();

    ```
