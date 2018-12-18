---
title: 保護與 Azure Service Fabric 中的 C# 進行服務遠端通訊時的安全 | Microsoft Docs
description: 了解如何針對在 Azure Service Fabric 叢集中執行的 C# 可靠服務，保護以服務遠端為基礎的通訊。
services: service-fabric
documentationcenter: .net
author: suchiagicha
manager: timlt
editor: vturecek
ms.assetid: fc129c1a-fbe4-4339-83ae-0e69a41654e0
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 04/20/2017
ms.author: suchiagicha
ms.openlocfilehash: be5dab7b9714f13a4bd30e6ab33a5a0e2016212d
ms.sourcegitcommit: 0fa8b4622322b3d3003e760f364992f7f7e5d6a9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37020014"
---
# <a name="secure-service-remoting-communications-in-a-c-service"></a>保護 C# 服務中的服務遠端通訊
> [!div class="op_single_selector"]
> * [Windows 上的 C# ](service-fabric-reliable-services-secure-communication.md)
> * [在 Linux 上使用 Java](service-fabric-reliable-services-secure-communication-java.md)
>
>

安全性是通訊最為重視的其中一個部分。 Reliable Services 應用程式架構會提供可用來改善安全性的一些預先建置通訊堆疊和工具。 本文探討如何改善在 C# 服務中使用服務遠端時的安全性。 它建置於現有[範例](service-fabric-reliable-services-communication-remoting.md)上，其會說明如何針對以 C# 撰寫的可靠服務設定遠端。 

若要在搭配 C# 服務使用服務遠端時協助保護服務安全，請遵循下列步驟：

1. 建立 `IHelloWorldStateful`介面，這個介面會定義將在您的服務上用於遠端程序呼叫的方法。 您的服務將使用在 `Microsoft.ServiceFabric.Services.Remoting.FabricTransport.Runtime` 命名空間中宣告的 `FabricTransportServiceRemotingListener`。 這是提供遠端功能的 `ICommunicationListener` 實作。

    ```csharp
    public interface IHelloWorldStateful : IService
    {
        Task<string> GetHelloWorld();
    }

    internal class HelloWorldStateful : StatefulService, IHelloWorldStateful
    {
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]{
                    new ServiceReplicaListener(
                        (context) => new FabricTransportServiceRemotingListener(context,this))};
        }

        public Task<string> GetHelloWorld()
        {
            return Task.FromResult("Hello World!");
        }
    }
    ```
2. 新增接聽程式設定和安全性認證。

    確定您想要用來協助保護服務通訊安全的憑證已安裝於叢集的所有節點上。 
    
    > [!NOTE]
    > 在 Linux 節點上，憑證必須以 PEM 格式的檔案形式存在於 */var/lib/sfcerts* 目錄中。 若要深入了解，請參閱 [Linux 節點上 X.509 憑證的位置與格式](./service-fabric-configure-certificates-linux.md#location-and-format-of-x509-certificates-on-linux-nodes)。 

    有兩種方式可提供接聽程式設定和安全性認證：

   1. 直接在服務程式碼中提供它們：

       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           FabricTransportRemotingListenerSettings  listenerSettings = new FabricTransportRemotingListenerSettings
           {
               MaxMessageSize = 10000000,
               SecurityCredentials = GetSecurityCredentials()
           };
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(context,this,listenerSettings))
           };
       }

       private static SecurityCredentials GetSecurityCredentials()
       {
           // Provide certificate details.
           var x509Credentials = new X509Credentials
           {
               FindType = X509FindType.FindByThumbprint,
               FindValue = "4FEF3950642138446CC364A396E1E881DB76B48C",
               StoreLocation = StoreLocation.LocalMachine,
               StoreName = "My",
               ProtectionLevel = ProtectionLevel.EncryptAndSign
           };
           x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
           x509Credentials.RemoteCertThumbprints.Add("9FEF3950642138446CC364A396E1E881DB76B483");
           return x509Credentials;
       }
       ```
   2. 使用 [組態封裝](service-fabric-application-and-service-manifests.md)提供它們：

       在 settings.xml 檔案中新增具名的 `TransportSettings` 區段。

       ```xml
       <Section Name="HelloWorldStatefulTransportSettings">
           <Parameter Name="MaxMessageSize" Value="10000000" />
           <Parameter Name="SecurityCredentialsType" Value="X509" />
           <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
           <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
           <Parameter Name="CertificateRemoteThumbprints" Value="9FEF3950642138446CC364A396E1E881DB76B483" />
           <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
           <Parameter Name="CertificateStoreName" Value="My" />
           <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
           <Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
       </Section>
       ```

       在此情況下， `CreateServiceReplicaListeners` 方法看起來像這樣：

       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(
                       context,this,FabricTransportRemotingListenerSettings .LoadFrom("HelloWorldStatefulTransportSettings")))
           };
       }
       ```

        如果您在 settings.xml 檔案中新增 `TransportSettings` 區段，則 `FabricTransportRemotingListenerSettings ` 預設會載入此區段中的所有設定。

        ```xml
        <!--"TransportSettings" section .-->
        <Section Name="TransportSettings">
            ...
        </Section>
        ```
        在此情況下， `CreateServiceReplicaListeners` 方法看起來像這樣：

        ```csharp
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]
            {
                return new[]{
                        new ServiceReplicaListener(
                            (context) => new FabricTransportServiceRemotingListener(context,this))};
            };
        }
        ```
3. 如果在安全服務上使用遠端堆疊來呼叫方法，而不是使用 `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` 類別來建立服務 Proxy，請使用 `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxyFactory`。 傳入包含 `SecurityCredentials` 的 `FabricTransportRemotingSettings`。

    ```csharp

    var x509Credentials = new X509Credentials
    {
        FindType = X509FindType.FindByThumbprint,
        FindValue = "9FEF3950642138446CC364A396E1E881DB76B483",
        StoreLocation = StoreLocation.LocalMachine,
        StoreName = "My",
        ProtectionLevel = ProtectionLevel.EncryptAndSign
    };
    x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
    x509Credentials.RemoteCertThumbprints.Add("4FEF3950642138446CC364A396E1E881DB76B48C");

    FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
    {
        SecurityCredentials = x509Credentials,
    };

    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(transportSettings));

    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

    如果用戶端程式碼正在當作服務一部分執行，則可以從 settings.xml 檔案中載入 `FabricTransportRemotingSettings` 。 建立與服務程式碼類似的 HelloWorldClientTransportSettings 區段，如前所示。 對用戶端程式碼進行下列變更：

    ```csharp
    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(FabricTransportRemotingSettings.LoadFrom("HelloWorldClientTransportSettings")));

    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

    如果用戶端未當作服務一部分執行，則您可以在 client_name.exe 所在的同一位置中建立 client_name.settings.xml 檔案。 然後在該檔案中建立 TransportSettings 區段。

    與此服務類似，如果您在用戶端 settings.xml/client_name.settings.xml 中新增 `TransportSettings` 區段，則 `FabricTransportRemotingSettings` 預設會載入此區段中的所有設定。

    在該情況下，先前的程式碼甚至會更進一步地簡化：  

    ```csharp

    IHelloWorldStateful client = ServiceProxy.Create<IHelloWorldStateful>(
                 new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```


接著請參閱[在 Reliable Services 中搭配 OWIN 使用 Web API](service-fabric-reliable-services-communication-webapi.md)。
