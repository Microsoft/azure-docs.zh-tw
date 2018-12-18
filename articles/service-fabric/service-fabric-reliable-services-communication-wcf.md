---
title: Reliable Services WCF 通訊堆疊 | Microsoft Docs
description: Service Fabric 內建的 WCF 通訊堆疊提供 Reliable Services 專用的用戶端服務 WCF 通訊。
services: service-fabric
documentationcenter: .net
author: BharatNarasimman
manager: timlt
editor: vturecek
ms.assetid: 75516e1e-ee57-4bc7-95fe-71ec42d452b2
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 06/07/2017
ms.author: bharatn
ms.openlocfilehash: 4c45bc76c176ce9f2476f6a666afda1daf4cd9c5
ms.sourcegitcommit: 6eb14a2c7ffb1afa4d502f5162f7283d4aceb9e2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/25/2018
ms.locfileid: "36749924"
---
# <a name="wcf-based-communication-stack-for-reliable-services"></a>適用於 Reliable Services 的 WCF 式通訊堆疊
Reliable Services 架構允許服務作者選擇其想要針對服務使用的通訊堆疊。 他們可以透過從 **CreateServiceReplicaListeners 或 CreateServiceInstanceListeners** 方法傳回的 [ICommunicationListener](service-fabric-reliable-services-communication.md) ，來外掛所選擇的通訊堆疊。 服務作者如果想要使用 Windows Communication Foundation (WCF) 式通訊，架構可提供以 WCF 式實作的通訊堆疊。

## <a name="wcf-communication-listener"></a>WCF 通訊接聽程式
**ICommunicationListener** 的 ECF 特定實作係由 **Microsoft.ServiceFabric.Services.Communication.Wcf.Runtime.WcfCommunicationListener** 類別提供。

假設我們有類型 `ICalculator`

```csharp
[ServiceContract]
public interface ICalculator
{
    [OperationContract]
    Task<int> Add(int value1, int value2);
}
```

我們可以透過下列方式在服務中建立 WCF 通訊接聽程式。

```csharp

protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new[] { new ServiceReplicaListener((context) =>
        new WcfCommunicationListener<ICalculator>(
            wcfServiceObject:this,
            serviceContext:context,
            //
            // The name of the endpoint configured in the ServiceManifest under the Endpoints section
            // that identifies the endpoint that the WCF ServiceHost should listen on.
            //
            endpointResourceName: "WcfServiceEndpoint",

            //
            // Populate the binding information that you want the service to use.
            //
            listenerBinding: WcfUtility.CreateTcpListenerBinding()
        )
    )};
}

```

## <a name="writing-clients-for-the-wcf-communication-stack"></a>撰寫適用於 WCF 通訊堆疊的用戶端
為了撰寫能夠使用 WCF 來與服務進行通訊的用戶端，架構提供了 **WcfClientCommunicationFactory**，也就是 [ClientCommunicationFactoryBase](service-fabric-reliable-services-communication.md)的 WCF 特定實作。

```csharp

public WcfCommunicationClientFactory(
    Binding clientBinding = null,
    IEnumerable<IExceptionHandler> exceptionHandlers = null,
    IServicePartitionResolver servicePartitionResolver = null,
    string traceId = null,
    object callback = null);
```

WCF 通訊通道可以從 **WcfCommunicationClientFactory** 建立的 **WcfCommunicationClient** 來存取。

```csharp

public class WcfCommunicationClient : ServicePartitionClient<WcfCommunicationClient<ICalculator>>
   {
       public WcfCommunicationClient(ICommunicationClientFactory<WcfCommunicationClient<ICalculator>> communicationClientFactory, Uri serviceUri, ServicePartitionKey partitionKey = null, TargetReplicaSelector targetReplicaSelector = TargetReplicaSelector.Default, string listenerName = null, OperationRetrySettings retrySettings = null)
           : base(communicationClientFactory, serviceUri, partitionKey, targetReplicaSelector, listenerName, retrySettings)
       {
       }
   }

```

用戶端程式碼可以使用 **WcfCommunicationClientFactory** 連同實作 **ServicePartitionClient** 的 **WcfCommunicationClient** 來決定服務端點，並與服務通訊。

```csharp
// Create binding
Binding binding = WcfUtility.CreateTcpClientBinding();
// Create a partition resolver
IServicePartitionResolver partitionResolver = ServicePartitionResolver.GetDefault();
// create a  WcfCommunicationClientFactory object.
var wcfClientFactory = new WcfCommunicationClientFactory<ICalculator>
    (clientBinding: binding, servicePartitionResolver: partitionResolver);

//
// Create a client for communicating with the ICalculator service that has been created with the
// Singleton partition scheme.
//
var calculatorServiceCommunicationClient =  new WcfCommunicationClient(
                wcfClientFactory,
                ServiceUri,
                ServicePartitionKey.Singleton);

//
// Call the service to perform the operation.
//
var result = calculatorServiceCommunicationClient.InvokeWithRetryAsync(
                client => client.Channel.Add(2, 3)).Result;

```
> [!NOTE]
> 預設 ServicePartitionResolver 假設用戶端正在與服務相同的叢集中執行。 如果不是這樣，請建立 ServicePartitionResolver 物件，並傳入叢集連接端點。
> 
> 

## <a name="next-steps"></a>後續步驟
* [使用 Reliable Services 遠端服務進行遠端程序呼叫](service-fabric-reliable-services-communication-remoting.md)
* [在 Reliable Services 中搭配 OWIN 使用 Web API](service-fabric-reliable-services-communication-webapi.md)
* [Reliable Services 的安全通訊](service-fabric-reliable-services-secure-communication-wcf.md)

