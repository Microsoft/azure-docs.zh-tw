---
title: 在 .NET 中開始使用 Azure 轉送 WCF 轉送 | Microsoft Docs
description: 了解如何使用 Azure 轉送 WCF 轉送連接主控於相異位置的兩個應用程式。
services: service-bus-relay
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: 5493281a-c2e5-49f2-87ee-9d3ffb782c75
ms.service: service-bus-relay
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 12/20/2017
ms.author: spelluru
ms.openlocfilehash: 6c8498a43b127fecc02473177ac955ae51a647d6
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48854111"
---
# <a name="how-to-use-azure-relay-wcf-relays-with-net"></a>如何使用 Azure 轉送 WCF 轉送搭配 .NET
本文說明如何使用 Azure 轉送服務。 這些範例均以 C# 撰寫，並使用 Windows Communication Foundation (WCF) API 以及包含在服務匯流排組件中的擴充功能。 如需 Azure 轉送的詳細資訊，請參閱 [Azure 轉送概觀](relay-what-is-it.md)。

[!INCLUDE [create-account-note](../../includes/create-account-note.md)]

## <a name="what-is-wcf-relay"></a>什麼是 WCF 轉送？

Azure [WCF 轉送服務](relay-what-is-it.md)可讓您建立一個可在 Azure 資料中心和您自己的內部部署企業環境中執行的混合式應用程式。 轉送服務可幫助達成此目標，方法是讓您以安全的方式，向公用雲端公開位於企業網路內部的 Windows Communication Foundation (WCF) 服務，而無需開啟防火牆連線或要求對企業網路基礎結構的進行侵入式變更。

![WCF 轉送概念](./media/service-bus-dotnet-how-to-use-relay/sb-relay-01.png)

Azure 轉送可讓您代管位於現有企業環境內的 WCF 服務。 您可以接著將接聽這些 WCF 服務的傳入工作階段和要求，委派給在 Azure 內部執行的轉送服務。 這可讓您將這些服務公開給在 Azure 中執行的應用程式程式碼，或是給行動工作者或外部網路合作夥伴環境。 轉送可讓您以安全的方式，在精細的層次控制可存取這些服務的使用者。 它提供了功能強大及安全的方式，來公開應用程式功能及現有企業解決方案的資料，並從雲端加以利用。

本文章將討論如何使用服 Azure 轉送來建立 WCF Web 服務，使用可在兩端之間實作安全交談的 TCP 通道繫結來加以公開。

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## <a name="get-the-service-bus-nuget-package"></a>取得服務匯流排 NuGet 封裝
[服務匯流排 NuGet 套件](https://www.nuget.org/packages/WindowsAzure.ServiceBus) 為取得服務匯流排 API，並設定具有所有服務匯流排相依性的應用程式的最容易方式。 若要在專案中安裝 NuGet 封裝，請執行下列動作：

1. 在 [方案總管] 中，以滑鼠右鍵按一下 [參考]，然後按一下 [管理 NuGet 套件]。
2. 搜尋「服務匯流排」並選取 [Microsoft Azure 服務匯流排]  項目。 按一下 [安裝] 完成安裝作業，然後關閉下列對話方塊：
   
   ![](./media/service-bus-dotnet-how-to-use-relay/getting-started-multi-tier-13.png)

## <a name="expose-and-consume-a-soap-web-service-with-tcp"></a>透過 TCP 公開及取用 SOAP Web 服務
若要將現有的 WCF SOAP Web 服務公開給外部使用，您必須對服務繫結和位址進行變更。 這可能需要變更組態檔，或可能需要變更程式碼，視您如何設定和配置 WCF 服務而定。 請注意，WCF 可讓您對相同的服務有多個網路端點，因此您可以保留現有的內部端點，並同時新增用於外部存取的轉送端點。

在本工作中，您會建立一個簡單的 WCF 服務，並為它新增轉送接聽程式。 本練習假設您對 Visual Studio 有一定程度的了解，因此將不會逐步解說完成建立專案的所有詳細資料。 而是會將重點放在程式碼。

開始這些步驟之前，請完成下列設定環境的程序：

1. 在 Visual Studio 中，建立解決方案中包含兩個專案 ("Client" 和 "Service") 的主控台應用程式。
2. 對兩個專案新增服務匯流排 NuGet 套件。 此封裝會將所有必要組件參考新增至您的專案。

### <a name="how-to-create-the-service"></a>如何建立服務
首先建立服務本身。 任何 WCF 服務都包含至少三個獨特部分：

* 說明會交換哪些訊息以及會叫用哪些作業的合約定義。
* 該合約的實作。
* 裝載 WCF 服務並公開數個端點的主機。

本節中的程式碼範例將逐一說明這些元件。

此合約會定義一個單一作業，即可新增兩個數字並傳回結果的 `AddNumbers`。 `IProblemSolverChannel` 介面可讓用戶端更輕鬆地管理 Proxy 存留期。 建立此類介面被視為是最佳做法。 最好能夠將此合約定義置於個別檔案中，以便您可以分別從 "Client" 和 "Service" 專案中參照該檔案，但您也可以將此程式碼複製到這兩個專案內。

```csharp
using System.ServiceModel;

[ServiceContract(Namespace = "urn:ps")]
interface IProblemSolver
{
    [OperationContract]
    int AddNumbers(int a, int b);
}

interface IProblemSolverChannel : IProblemSolver, IClientChannel {}
```

合約準備好了以後，實作會如下所述：

```csharp
class ProblemSolver : IProblemSolver
{
    public int AddNumbers(int a, int b)
    {
        return a + b;
    }
}
```

### <a name="configure-a-service-host-programmatically"></a>以程式設計方式設定服務主機
在準備好合約和實作之後，您便可以開始代管服務。 代管會發生在 [System.ServiceModel.ServiceHost](https://msdn.microsoft.com/library/system.servicemodel.servicehost.aspx) 物件內，此物件將負責管理服務的執行個體並代管接聽訊息的端點。 下列程式碼將設定包含一般本機端點和轉送端點的服務，以緊密地說明內部和外部端點的外觀。 使用您的命名空間名稱來取代字串 *namespace*，並使用上述設定步驟中所取得的 SAS 金鑰來取代 *yourKey*。

```csharp
ServiceHost sh = new ServiceHost(typeof(ProblemSolver));

sh.AddServiceEndpoint(
   typeof (IProblemSolver), new NetTcpBinding(),
   "net.tcp://localhost:9358/solver");

sh.AddServiceEndpoint(
   typeof(IProblemSolver), new NetTcpRelayBinding(),
   ServiceBusEnvironment.CreateServiceUri("sb", "namespace", "solver"))
    .Behaviors.Add(new TransportClientEndpointBehavior {
          TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", "<yourKey>")});

sh.Open();

Console.WriteLine("Press ENTER to close");
Console.ReadLine();

sh.Close();
```

在此範例中，您將建立相同合約實作的兩個端點。 一個位於本機，一個透過 Azure 轉送投射。 它們之間的主要差異是繫結；[NetTcpBinding](https://msdn.microsoft.com/library/system.servicemodel.nettcpbinding.aspx) 用於本機，而 [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding#microsoft_servicebus_nettcprelaybinding) 用於轉送端點和位址。 本機端點會包含具有獨特連接埠的本機網路位址。 轉送端點會包含一個由字串 `sb`、您的命名空間名稱及路徑 "solver" 組合而成的端點位址。 這會產生 URI `sb://[serviceNamespace].servicebus.windows.net/solver`，指出服務端點為具有完整外部 DNS 名稱的服務匯流排 (轉送) TCP 端點。 如果您將要取代預留位置的程式碼置入 **Service** 應用程式的 `Main` 函式中，您將會有一個功能性服務。 如果您想要服務專門接聽轉送，請移除本機端點宣告。

### <a name="configure-a-service-host-in-the-appconfig-file"></a>在 App.config 檔案中設定服務主機
您也可以使用 App.config 檔案來設定主機。 此案例中的服務裝載程式碼會出現在下一個範例中。

```csharp
ServiceHost sh = new ServiceHost(typeof(ProblemSolver));
sh.Open();
Console.WriteLine("Press ENTER to close");
Console.ReadLine();
sh.Close();
```

端點定義移入 App.config 檔案。 NuGet 套件已在 App.config 檔案中新增許多定義，這些都是 Azure 轉送的必要組態擴充功能。 下列範例 (與上述程式碼完全相同) 應會出現在 **system.serviceModel** 元素的正下方。 這個程式碼範例假設您的專案 C# 命名空間名稱為 **Service**。
使用您的轉送命名空間名稱和金鑰來取代預留位置。

```xml
<services>
    <service name="Service.ProblemSolver">
        <endpoint contract="Service.IProblemSolver"
                  binding="netTcpBinding"
                  address="net.tcp://localhost:9358/solver"/>
        <endpoint contract="Service.IProblemSolver"
                  binding="netTcpRelayBinding"
                  address="sb://<namespaceName>.servicebus.windows.net/solver"
                  behaviorConfiguration="sbTokenProvider"/>
    </service>
</services>
<behaviors>
    <endpointBehaviors>
        <behavior name="sbTokenProvider">
            <transportClientEndpointBehavior>
                <tokenProvider>
                    <sharedAccessSignature keyName="RootManageSharedAccessKey" key="<yourKey>" />
                </tokenProvider>
            </transportClientEndpointBehavior>
        </behavior>
    </endpointBehaviors>
</behaviors>
```

在進行這些變更之後，此服務便會和以前一樣啟動，但會多了兩個即時端點：一個在本機，一個在雲端接聽。

### <a name="create-the-client"></a>建立用戶端
#### <a name="configure-a-client-programmatically"></a>以程式設計方式設定用戶端
若要取用此服務，您可以使用 [ChannelFactory](https://msdn.microsoft.com/library/system.servicemodel.channelfactory.aspx) 物件來建構 WCF 用戶端。 服務匯流排會使用以 SAS 所實作的權杖型安全性模型。 [TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider) 類別代表安全性權杖提供者，其具有內建 Factory 方法，可傳回部分已知的權杖提供者。 以下範例使用 [CreateSharedAccessSignatureTokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider#Microsoft_ServiceBus_TokenProvider_CreateSharedAccessSignatureTokenProvider_System_String_) 方法以處理適當 SAS 權杖的擷取。 如上一節所述，將從入口網站取得這些名稱和金鑰。

首先，將 `IProblemSolver` 合約程式碼從服務中參照或複製到您的用戶端專案。

然後，取代用戶端 `Main` 方法中的程式碼，並再次使用您的轉送命名空間和 SAS 金鑰來取代預留位置文字。

```csharp
var cf = new ChannelFactory<IProblemSolverChannel>(
    new NetTcpRelayBinding(),
    new EndpointAddress(ServiceBusEnvironment.CreateServiceUri("sb", "<namespaceName>", "solver")));

cf.Endpoint.Behaviors.Add(new TransportClientEndpointBehavior
            { TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey","<yourKey>") });

using (var ch = cf.CreateChannel())
{
    Console.WriteLine(ch.AddNumbers(4, 5));
}
```

您現在可以建置用戶端和服務、執行它們 (先執行服務)，然後用戶端會呼叫此服務並列印 **9**。 您可以在不同機器上 (即使是在不同網路上) 執行用戶端和伺服器，通訊仍然可以運作。 您也可以在雲端或在本機上執行用戶端程式碼。

#### <a name="configure-a-client-in-the-appconfig-file"></a>在 App.config 檔案中設定用戶端
下列程式碼示範如何使用 App.config 檔案設定用戶端。

```csharp
var cf = new ChannelFactory<IProblemSolverChannel>("solver");
using (var ch = cf.CreateChannel())
{
    Console.WriteLine(ch.AddNumbers(4, 5));
}
```

端點定義移入 App.config 檔案。 下列範例 (與上述程式碼相同) 應會出現在 `<system.serviceModel>` 元素的正下方。 和以前一樣，您必須在此處使用您的轉送命名空間和 SAS 金鑰來取代預留位置。

```xml
<client>
    <endpoint name="solver" contract="Service.IProblemSolver"
              binding="netTcpRelayBinding"
              address="sb://<namespaceName>.servicebus.windows.net/solver"
              behaviorConfiguration="sbTokenProvider"/>
</client>
<behaviors>
    <endpointBehaviors>
        <behavior name="sbTokenProvider">
            <transportClientEndpointBehavior>
                <tokenProvider>
                    <sharedAccessSignature keyName="RootManageSharedAccessKey" key="<yourKey>" />
                </tokenProvider>
            </transportClientEndpointBehavior>
        </behavior>
    </endpointBehaviors>
</behaviors>
```

## <a name="next-steps"></a>後續步驟
了解基本的 Azure 轉送之後，請參考下列連結以取得更多資訊。

* [什麼是 Azure 轉送？](relay-what-is-it.md)
* 從 [Azure 範例][Azure samples]下載服務匯流排範例，或參閱[服務匯流排範例概觀][overview of Service Bus samples]。

[Shared Access Signature Authentication with Service Bus]: ../service-bus-messaging/service-bus-shared-access-signature-authentication.md
[Azure samples]: https://code.msdn.microsoft.com/site/search?query=service%20bus&f%5B0%5D.Value=service%20bus&f%5B0%5D.Type=SearchText&ac=2
[overview of Service Bus samples]: ../service-bus-messaging/service-bus-samples.md
