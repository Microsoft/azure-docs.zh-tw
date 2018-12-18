---
title: Azure 服務匯流排 WCF 轉送教學課程 | Microsoft Docs
description: 使用 WCF 轉送建立用戶端和服務應用程式。
services: service-bus-relay
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: 53dfd236-97f1-4778-b376-be91aa14b842
ms.service: service-bus-relay
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/02/2017
ms.author: spelluru
ms.openlocfilehash: 0833a7ec71a0aea66f8ebfdfff81d88925019309
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43701860"
---
# <a name="azure-wcf-relay-tutorial"></a>Azure WCF 轉送教學課程

本教學課程說明如何使用 Azure 轉送，來建置簡單的 WCF 轉送用戶端應用程式和服務。 如需使用[服務匯流排通訊](../service-bus-messaging/service-bus-messaging-overview.md)的類似教學課程，請參閱[開始使用服務匯流排佇列](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md)。

循序完成本教學課程，可讓您了解建立 WCF 轉送用戶端和服務應用程式所需的步驟。 如同他們原始的 WCF 對應項目，服務是可公開一或多個端點的結構，而每個端點都會公開一或多個服務作業。 服務的端點指定可找到服務的位址、內含用戶端與服務通訊所需資訊的繫結，以及可定義服務提供給其用戶端之功能的合約。 WCF 與 WCF 轉送的主要差異在於端點是在雲端公開，而不是在您的電腦本機上公開。

在您逐步完成本教學課程中的各個主題後，您會有一項執行中的服務，以及可叫用服務作業的用戶端。 第一個主題說明如何設定帳戶。 後續步驟說明如何定義使用合約的服務、如何實作服務，以及如何在程式碼中設定服務。 此外，也會說明如何主控和執行服務。 建立的服務會自我裝載，而用戶端和服務會在相同的電腦上執行。 您可以使用程式碼或組態檔來設定服務。

最後三個步驟描述如何建立用戶端應用程式、設定用戶端應用程式，以及建立和使用可存取主機功能的用戶端。

## <a name="prerequisites"></a>必要條件

若要完成此教學課程，您需要下列項目：

* [Microsoft Visual Studio 2015 或更高版本](http://visualstudio.com)。 本教學課程使用 Visual Studio 2017。
* 使用中的 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費帳戶。 如需詳細資料，請參閱 [Azure 免費試用](https://azure.microsoft.com/free/)。

## <a name="create-a-service-namespace"></a>建立服務命名空間

第一個步驟是建立命名空間，並取得[共用存取簽章 (SAS)](../service-bus-messaging/service-bus-sas.md) 金鑰。 命名空間會為每個透過轉送服務公開的應用程式提供應用程式界限。 建立服務命名空間時，系統會自動產生 SAS 金鑰。 服務命名空間與 SAS 金鑰的組合會提供一個認證，供 Azure 用來驗證對應用程式的存取權。 請依照[這裡的指示](relay-create-namespace-portal.md)來建立轉送命名空間。

## <a name="define-a-wcf-service-contract"></a>定義 WCF 服務合約

服務合約會指定服務可支援哪些作業 (方法或函式的 Web 服務術語)。 合約可以透過定義 C++、C# 或 Visual Basic 介面建立。 介面中的每個方法會對應一個特定服務作業。 每個介面都必須已套用 [ServiceContractAttribute](https://msdn.microsoft.com/library/system.servicemodel.servicecontractattribute.aspx) 屬性，而且每個作業都必須已套用 [OperationContractAttribute](https://msdn.microsoft.com/library/system.servicemodel.operationcontractattribute.aspx) 屬性。 如果介面中的方法有 [ServiceContractAttribute](https://msdn.microsoft.com/library/system.servicemodel.servicecontractattribute.aspx) 屬性，但沒有 [OperationContractAttribute](https://msdn.microsoft.com/library/system.servicemodel.operationcontractattribute.aspx) 屬性，則不會公開該方法。 程序後面的範例會提供這些工作的程式碼。 如需合約與服務的詳細討論，請參閱 WCF 文件中的[設計和實作服務](https://msdn.microsoft.com/library/ms729746.aspx)。

### <a name="create-a-relay-contract-with-an-interface"></a>使用介面建立轉送合約

1. 以系統管理員身分開啟 Visual Studio：以滑鼠右鍵按一下 [開始] 功能表中的程式，然後選取 [以系統管理員身分執行]。
2. 這會建立新的主控台應用程式專案。 按一下 [檔案] 功能表，選取 [新增]，然後按一下 [專案]。 在 [新增專案] 對話方塊中，按一下 **Visual C#** (如果 **Visual C#** 未出現，請查看 [其他語言] 下方)。 按一下 [主控台應用程式 (.NET Framework)] 範本，並將它命名為 **EchoService**。 按一下 [確定]  以建立專案。

    ![][2]

3. 安裝服務匯流排 NuGet 套件。 此套件會自動新增服務匯流排程式庫及 WCF **System.ServiceModel** 的參考。 [System.ServiceModel](https://msdn.microsoft.com/library/system.servicemodel.aspx) 是可讓您以程式設計方式存取 WCF 基本功能的命名空間。 服務匯流排會使用 WCF 的許多物件和屬性來定義服務合約。

    在 [方案總管] 中，以滑鼠右鍵按一下專案，然後按一下 [管理 NuGet 套件]。按一下 [瀏覽] 索引標籤，然後搜尋 **WindowsAzure.ServiceBus**。 確定已在 [版本] 方塊中選取此專案名稱。 按一下 [安裝] 並接受使用條款。

    ![][3]
4. 在 [方案總管] 中，按兩下 Program.cs 檔案，以在編輯器中開啟它 (如果尚未開啟的話)。
5. 在檔案頂端加入下列 using 陳述式：

    ```csharp
    using System.ServiceModel;
    using Microsoft.ServiceBus;
    ```
6. 將命名空間名稱從 **EchoService** 的預設名稱變更為 **Microsoft.ServiceBus.Samples**。

   > [!IMPORTANT]
   > 本教學課程使用 C# 命名空間 **Microsoft.ServiceBus.Samples**，也就是在[設定 WCF 用戶端](#configure-the-wcf-client)步驟的設定檔中所使用合約型受控型別的命名空間。 您可以在建置此範例時指定您要的任何命名空間；不過，除非您後來在應用程式組態檔中相應地修改合約的命名空和服務，否則本教學課程將無法運作。 在 App.config 檔案中指定的命名空間必須與在 C# 檔案中指定的命名空間相同。
   >
   >
7. 緊接在 `Microsoft.ServiceBus.Samples` 命名空間宣告後面 (但在命名空間內)，定義名為 `IEchoContract` 的新介面，並將 `ServiceContractAttribute` 屬性套用至命名空間值為 `http://samples.microsoft.com/ServiceModel/Relay/` 的介面。 命名空間值與您的整個程式碼範圍中使用的命名空間不同。 然而，命名空間值會作為此合約的唯一識別碼。 明確指定命名空間可避免將預設命名空間值新增至合約名稱。 將下列程式碼貼上到命名空間宣告之後：

    ```csharp
    [ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
    }
    ```

   > [!NOTE]
   > 一般而言，服務合約命名空間包含命名配置 (其中包含版本資訊)。 在服務合約命名空間中包含版本資訊，可讓服務定義具有新命名空間的新服務合約並在新的端點上公開它，藉此區隔重大變更。 如此一來，用戶端可以繼續使用舊服務合約，而不需要更新。 版本資訊可以包含日期或組建編號。 如需詳細資訊，請參閱[服務版本設定](http://go.microsoft.com/fwlink/?LinkID=180498)。 基於本教學課程的目的，服務合約命名空間的命名配置不包含版本資訊。
   >
   >
8. 在 `IEchoContract` 介面中，針對 `IEchoContract` 合約在介面中公開的單一作業宣告方法，並將 `OperationContractAttribute` 屬性套用至您想要公開為公用 WCF 轉送合約一部分的方法，如下所示：

    ```csharp
    [OperationContract]
    string Echo(string text);
    ```
9. 直接在 `IEchoContract` 介面定義之後，宣告同時繼承自 `IEchoContract` 與 `IClientChannel` 介面的通道，如下所示：

    ```csharp
    public interface IEchoChannel : IEchoContract, IClientChannel { }
    ```

    通道是 WCF 物件，主機和用戶端可透過它彼此傳遞資訊。 稍後，您將對此通道撰寫程式碼，以回應兩個應用程式之間的資訊。
10. 從 [建置] 功能表中，按一下 [建置方案] 或按 **Ctrl+Shift+B**，確認到目前為止您的工作正確無誤。

### <a name="example"></a>範例

下列程式碼示範定義 WCF 轉送合約的基本介面。

```csharp
using System;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

現在已建立介面，您可以實作此介面。

## <a name="implement-the-wcf-contract"></a>實作 WCF 合約

建立 Azure 轉送需要先建立合約，您可使用介面定義該合約。 如需建立介面的詳細資訊，請參閱上一個步驟。 下一步是實作介面。 這牽涉到建立名為 `EchoService` 的類別，該類別會實作使用者定義的 `IEchoContract` 介面。 實作合約後，接著可使用 App.config 組態檔設定介面。 組態檔包含應用程式的必要資訊，例如服務名稱、合約名稱，以及用來與轉送服務通訊的通訊協定類型。 程序後面的範例提供用來執行這些工作的程式碼。 如需如何實作服務合約的一般討論，請參閱 WCF 文件中的[實作服務合約](https://msdn.microsoft.com/library/ms733764.aspx)。

1. 緊接在 `IEchoContract` 介面的定義之後，建立名為 `EchoService` 的新類別。 `EchoService` 類別會實作 `IEchoContract` 介面。

    ```csharp
    class EchoService : IEchoContract
    {
    }
    ```

    類似於其他介面實作，您可以在不同的檔案中實作定義。 不過，在此教學課程中，實作會位於與介面定義和 `Main` 方法相同的檔案中。
2. 將 [ServiceBehaviorAttribute](https://msdn.microsoft.com/library/system.servicemodel.servicebehaviorattribute.aspx) 屬性套用至 `IEchoContract` 介面。 此屬性會指定服務名稱和命名空間。 這麼做之後，`EchoService` 類別會如下所示︰

    ```csharp
    [ServiceBehavior(Name = "EchoService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    class EchoService : IEchoContract
    {
    }
    ```
3. 實作在 `EchoService` 類別的 `IEchoContract` 介面中定義的 `Echo` 方法。

    ```csharp
    public string Echo(string text)
    {
        Console.WriteLine("Echoing: {0}", text);
        return text;
    }
    ```
4. 按一下 [建置]，然後按一下 [建置方案]，確認您的工作正確無誤。

### <a name="define-the-configuration-for-the-service-host"></a>定義服務主機的設定

1. 此組態檔非常類似於 WCF 組態檔。 其中包含服務名稱、端點 (也就是 Azure 轉送公開的位置，讓用戶端與主機能夠彼此通訊) 和繫結 (用於通訊的通訊協定類型)。 主要差異在於這個已設定的服務端點會參考不屬於 .NET Framework 的 [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding) 繫結。 [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding) 是服務所定義的其中一個繫結。
2. 在 [方案總管] 中，按兩下 App.config 檔案，以在 Visual Studio 編輯器中開啟它。
3. 在 `<appSettings>` 元素中，以您的服務命名空間名稱以及您在先前步驟中複製的 SAS 金鑰取代預留位置。
4. 在 `<system.serviceModel>` 標記內，加入 `<services>` 元素。 您可以在單一組態檔中定義多個轉送應用程式。 不過，本教學課程只會定義一個。

    ```xml
    <?xmlversion="1.0"encoding="utf-8"?>
    <configuration>
      <system.serviceModel>
        <services>

        </services>
      </system.serviceModel>
    </configuration>
    ```
5. 在 `<services>` 元素內，加入 `<service>` 元素以定義服務的名稱。

    ```xml
    <service name="Microsoft.ServiceBus.Samples.EchoService">
    </service>
    ```
6. 在 `<service>` 元素內，定義端點合約的位置，以及端點的繫結類型。

    ```xml
    <endpoint contract="Microsoft.ServiceBus.Samples.IEchoContract" binding="netTcpRelayBinding"/>
    ```

    端點會定義用戶端將在何處尋找主應用程式。 稍後，本教學課程會使用此步驟來建立 URI，以透過 Azure 轉送完全公開主機。 繫結會宣告我們使用 TCP 作為與轉送服務通訊的通訊協定。
7. 從 [建置] 功能表中，按一下 [建置方案]，確認您的工作正確無誤。

### <a name="example"></a>範例

下列程式碼示範服務合約的實作。

```csharp
[ServiceBehavior(Name = "EchoService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]

    class EchoService : IEchoContract
    {
        public string Echo(string text)
        {
            Console.WriteLine("Echoing: {0}", text);
            return text;
        }
    }
```

下列程式碼顯示與服務相關聯之 App.config 檔案的基本格式。

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.serviceModel>
    <services>
      <service name="Microsoft.ServiceBus.Samples.EchoService">
        <endpoint contract="Microsoft.ServiceBus.Samples.IEchoContract" binding="netTcpRelayBinding" />
      </service>
    </services>
    <extensions>
      <bindingExtensions>
        <add name="netTcpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
      </bindingExtensions>
    </extensions>
  </system.serviceModel>
</configuration>
```

## <a name="host-and-run-a-basic-web-service-to-register-with-the-relay-service"></a>裝載並執行基本 Web 服務以向轉送服務登錄

此步驟描述如何執行 Azure 轉送服務。

### <a name="create-the-relay-credentials"></a>建立轉送認證

1. 在 `Main()` 中，建立兩個變數，以儲存命名空間以及從主控台視窗讀取的 SAS 金鑰。

    ```csharp
    Console.Write("Your Service Namespace: ");
    string serviceNamespace = Console.ReadLine();
    Console.Write("Your SAS key: ");
    string sasKey = Console.ReadLine();
    ```

    SAS 金鑰稍後將用來存取您的專案。 命名空間會當做參數傳遞至 `CreateServiceUri` 以建立服務 URI。
2. 使用 [TransportClientEndpointBehavior](/dotnet/api/microsoft.servicebus.transportclientendpointbehavior) 物件，宣告您將使用 SAS 金鑰作為認證類型。 將下列程式碼直接加在最後一個步驟中新增的程式碼之後。

    ```csharp
    TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
    sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);
    ```

### <a name="create-a-base-address-for-the-service"></a>建立服務的基底位址

在最後一個步驟中加入的程式碼後面，建立服務基底位址的 `Uri` 執行個體。 此 URI 會指定服務匯流排配置、命名空間和服務介面的路徑。

```csharp
Uri address = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");
```

"sb" 是服務匯流排配置的縮寫，並指出我們使用 TCP 作為通訊協定。 當 [NetTcpRelayBinding](https://msdn.microsoft.com/library/microsoft.servicebus.nettcprelaybinding.aspx) 被指定為繫結時，先前也已在組態檔中指出此項。

在本教學課程中，URI 為 `sb://putServiceNamespaceHere.windows.net/EchoService`。

### <a name="create-and-configure-the-service-host"></a>建立並設定服務主機

1. 將連線模式設定為 `AutoDetect`。

    ```csharp
    ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;
    ```

    連線模式可描述服務用來與轉送服務通訊的通訊協定 (HTTP 或 TCP)。 使用預設設定 `AutoDetect`，服務會嘗試透過 TCP 連接至 Azure 轉送，如果無法使用 TCP，則會透過 HTTP 連接。 請注意，這有別於服務針對用戶端通訊指定的通訊協定。 該通訊協定取決於使用的繫結。 例如，服務可以使用 [BasicHttpRelayBinding](https://msdn.microsoft.com/library/microsoft.servicebus.basichttprelaybinding.aspx) 繫結，指定它的端點會透過 HTTP 來與用戶端通訊。 該相同服務可以指定 **ConnectivityMode.AutoDetect**，讓服務能夠透過 TCP 來與 Azure 轉送通訊。
2. 使用本節稍早建立的 URI，建立服務主機。

    ```csharp
    ServiceHost host = new ServiceHost(typeof(EchoService), address);
    ```

    服務主機是將服務具現化的 WCF 物件。 在此，您會將您要建立的服務類型 (`EchoService` 類型)，以及您要公開服務的位址傳給它。
3. 在 Program.cs 檔案頂端，加入 [System.ServiceModel.Description](https://msdn.microsoft.com/library/system.servicemodel.description.aspx) 和 [Microsoft.ServiceBus.Description](/dotnet/api/microsoft.servicebus.description) 的參考。

    ```csharp
    using System.ServiceModel.Description;
    using Microsoft.ServiceBus.Description;
    ```
4. 回到 `Main()`，設定要啟用公開存取的端點。

    ```csharp
    IEndpointBehavior serviceRegistrySettings = new ServiceRegistrySettings(DiscoveryType.Public);
    ```

    此步驟會通知轉送服務，藉由檢查專案的 ATOM 摘要，公開找到您的應用程式。 如果您將 **DiscoveryType** 設定為 [私人]，用戶端仍可存取服務。 不過，服務在搜尋轉送命名空間時並不會出現。 相反地，用戶端必須事先知道端點路徑。
5. 將服務認證套用至在 App.config 檔案中定義的服務端點︰

    ```csharp
    foreach (ServiceEndpoint endpoint in host.Description.Endpoints)
    {
        endpoint.Behaviors.Add(serviceRegistrySettings);
        endpoint.Behaviors.Add(sasCredential);
    }
    ```

    如上一個步驟所述，您可能已在組態檔中宣告多個服務和端點。 若是如此，此程式碼會周遊組態檔及搜尋每個應套用您的認證的端點。 不過，在本教學課程中，組態檔只有一個端點。

### <a name="open-the-service-host"></a>開啟服務主機

1. 開啟服務。

    ```csharp
    host.Open();
    ```
2. 通知使用者此服務正在執行，以及說明如何關閉服務。

    ```csharp
    Console.WriteLine("Service address: " + address);
    Console.WriteLine("Press [Enter] to exit");
    Console.ReadLine();
    ```
3. 完成時，關閉服務主機。

    ```csharp
    host.Close();
    ```
4. 按 **Ctrl+Shift+B** 建置專案。

### <a name="example"></a>範例

完整的服務程式碼應如下所示。 程式碼包括教學課程先前步驟的服務合約和實作，並在主控台應用程式中裝載服務。

```csharp
using System;
using System.ServiceModel;
using System.ServiceModel.Description;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Description;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { };

    [ServiceBehavior(Name = "EchoService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    class EchoService : IEchoContract
    {
        public string Echo(string text)
        {
            Console.WriteLine("Echoing: {0}", text);
            return text;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;         

            Console.Write("Your Service Namespace: ");
            string serviceNamespace = Console.ReadLine();
            Console.Write("Your SAS key: ");
            string sasKey = Console.ReadLine();

           // Create the credentials object for the endpoint.
            TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
            sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);

            // Create the service URI based on the service namespace.
            Uri address = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");

            // Create the service host reading the configuration.
            ServiceHost host = new ServiceHost(typeof(EchoService), address);

            // Create the ServiceRegistrySettings behavior for the endpoint.
            IEndpointBehavior serviceRegistrySettings = new ServiceRegistrySettings(DiscoveryType.Public);

            // Add the Relay credentials to all endpoints specified in configuration.
            foreach (ServiceEndpoint endpoint in host.Description.Endpoints)
            {
                endpoint.Behaviors.Add(serviceRegistrySettings);
                endpoint.Behaviors.Add(sasCredential);
            }

            // Open the service.
            host.Open();

            Console.WriteLine("Service address: " + address);
            Console.WriteLine("Press [Enter] to exit");
            Console.ReadLine();

            // Close the service.
            host.Close();
        }
    }
}
```

## <a name="create-a-wcf-client-for-the-service-contract"></a>建立服務合約的 WCF 用戶端

下一個步驟是建立用戶端應用程式，並定義您將在後續步驟中實作的服務合約。 請注意，這其中有許多步驟類似於用來建立服務的步驟︰定義合約、編輯 App.config 檔案、使用認證來連接至轉送服務等等。 程序後面的範例提供用來執行這些工作的程式碼。

1. 若要在目前的 Visual Studio 方案中為用戶端建立新專案，請執行下列作業︰

   1. 在方案總管中含有服務的相同方案中，以滑鼠右鍵按一下目前的方案 (而非專案)，然後按一下 [加入]。 然後按一下 [新增專案]。
   2. 在 [新增專案] 對話方塊中，按一下 [Visual C#] \(如果 **Visual C#** 未出現，請在 [其他語言] 下尋找)，選取 [主控台應用程式 (.NET Framework)] 範本，並將它命名為 **EchoClient**。
   3. 按一下 [確定]。
      <br />
2. 在 [方案總管] 中，按兩下 **EchoClient** 專案中的 Program.cs 檔案，以在編輯器中開啟它 (如果尚未開啟的話)。
3. 將命名空間名稱從 `EchoClient` 的預設名稱變更為 `Microsoft.ServiceBus.Samples`。
4. 安裝[服務匯流排 NuGet 封裝](https://www.nuget.org/packages/WindowsAzure.ServiceBus)：在 [方案總管] 中，以滑鼠右鍵按一下 **EchoClient** 專案，然後按一下 [管理 NuGet 封裝]。 按一下 [瀏覽] 索引標籤，然後搜尋 `Microsoft Azure Service Bus`。 按一下 [安裝] 並接受使用條款。

    ![][3]
5. 在 Program.cs 檔案中加入 [System.ServiceModel](https://msdn.microsoft.com/library/system.servicemodel.aspx) 命名空間的 `using` 陳述式。

    ```csharp
    using System.ServiceModel;
    ```
6. 下列範例所示，將服務合約定義新增至命名空間。 請注意，此定義與 [服務] 專案中使用的定義相同。 您應該在 `Microsoft.ServiceBus.Samples` 命名空間的頂端加入此程式碼。

    ```csharp
    [ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        string Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }
    ```
7. 按 **Ctrl+Shift+B** 建置用戶端。

### <a name="example"></a>範例

下列程式碼顯示 **EchoClient** 專案中 Program.cs 檔案的目前狀態。

```csharp
using System;
using Microsoft.ServiceBus;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{

    [ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        string Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }


    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

## <a name="configure-the-wcf-client"></a>設定 WCF 用戶端

在此步驟中，您可為基本用戶端應用程式建立 App.config 檔案，以存取先前在本教學課程中建立的服務。 此 App.config 檔案定義端點的合約、繫結和名稱。 程序後面的範例提供用來執行這些工作的程式碼。

1. 在 [方案總管] 的 **EchoClient** 專案中，按兩下 **App.config**，在 Visual Studio 編輯器中開啟該檔案。
2. 在 `<appSettings>` 元素中，以您的服務命名空間名稱以及您在先前步驟中複製的 SAS 金鑰取代預留位置。
3. 在 system.serviceModel 元素中加入 `<client>` 元素。

    ```xml
    <?xmlversion="1.0"encoding="utf-8"?>
    <configuration>
      <system.serviceModel>
        <client>
        </client>
      </system.serviceModel>
    </configuration>
    ```

    此步驟宣告您正在定義 WCF 樣式的用戶端應用程式。
4. 在 `client` 元素內，定義端點的名稱、合約和繫結類型。

    ```xml
    <endpoint name="RelayEndpoint"
                    contract="Microsoft.ServiceBus.Samples.IEchoContract"
                    binding="netTcpRelayBinding"/>
    ```

    此步驟會定義端點的名稱、服務中定義的合約，以及用戶端應用程式使用 TCP 來與 Azure 轉送通訊的細節。 端點名稱在下一步中用來連結此端點組態與服務 URI。
5. 按一下 [檔案]，然後按一下 [全部儲存]。

## <a name="example"></a>範例

下列程式碼會顯示 Echo 用戶端的 App.config 檔案。

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.serviceModel>
    <client>
      <endpoint name="RelayEndpoint"
                      contract="Microsoft.ServiceBus.Samples.IEchoContract"
                      binding="netTcpRelayBinding"/>
    </client>
    <extensions>
      <bindingExtensions>
        <add name="netTcpRelayBinding"
                    type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
      </bindingExtensions>
    </extensions>
  </system.serviceModel>
</configuration>
```

## <a name="implement-the-wcf-client"></a>實作 WCF 用戶端
在此步驟中，您可實作基本用戶端應用程式，以存取您先前在本教學課程中建立的服務。 與此服務類似，用戶端會執行許多相同的作業來存取 Azure 轉送：

1. 設定連線模式。
2. 建立可找出主機服務的 URI。
3. 定義安全性認證。
4. 將認證套用至連線。
5. 開啟連線。
6. 執行應用程式特定的工作。
7. 關閉連線。

不過，其中一個主要差異在於用戶端應用程式會使用通道來連接至轉送服務，而此服務會使用對 **ServiceHost** 的呼叫。 程序後面的範例提供用來執行這些工作的程式碼。

### <a name="implement-a-client-application"></a>實作用戶端應用程式
1. 將連線模式設定為 [自動偵測]。 在 **EchoClient** 應用程式的 `Main()` 方法中新增下列程式碼。

    ```csharp
    ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;
    ```
2. 定義變數，以保留服務命名空間以及從主控台讀取之 SAS 金鑰的值。

    ```csharp
    Console.Write("Your Service Namespace: ");
    string serviceNamespace = Console.ReadLine();
    Console.Write("Your SAS Key: ");
    string sasKey = Console.ReadLine();
    ```
3. 建立可定義轉送專案中主機位置的 URI。

    ```csharp
    Uri serviceUri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");
    ```
4. 建立服務命名空間端點的認證物件。

    ```csharp
    TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
    sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);
    ```
5. 建立通道工廠，以載入 App.config 檔案中所述的組態。

    ```csharp
    ChannelFactory<IEchoChannel> channelFactory = new ChannelFactory<IEchoChannel>("RelayEndpoint", new EndpointAddress(serviceUri));
    ```

    通道工廠是一個 WCF 物件，可建立通道以供服務與用戶端應用程式進行通訊。
6. 套用認證。

    ```csharp
    channelFactory.Endpoint.Behaviors.Add(sasCredential);
    ```
7. 建立並開啟服務的通道。

    ```csharp
    IEchoChannel channel = channelFactory.CreateChannel();
    channel.Open();
    ```
8. 撰寫 Echo 的基本使用者介面和功能。

    ```csharp
    Console.WriteLine("Enter text to echo (or [Enter] to exit):");
    string input = Console.ReadLine();
    while (input != String.Empty)
    {
        try
        {
            Console.WriteLine("Server echoed: {0}", channel.Echo(input));
        }
        catch (Exception e)
        {
            Console.WriteLine("Error: " + e.Message);
        }
        input = Console.ReadLine();
    }
    ```

    請注意，程式碼會使用通道物件的執行個體作為服務的 Proxy。
9. 關閉通道，並關閉工廠。

    ```csharp
    channel.Close();
    channelFactory.Close();
    ```

## <a name="example"></a>範例

完整的程式碼應如下所示，顯示如何建立用戶端應用程式、如何呼叫服務的作業，以及如何在完成作業呼叫之後關閉用戶端。

```csharp
using System;
using Microsoft.ServiceBus;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;


            Console.Write("Your Service Namespace: ");
            string serviceNamespace = Console.ReadLine();
            Console.Write("Your SAS Key: ");
            string sasKey = Console.ReadLine();



            Uri serviceUri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");

            TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
            sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);

            ChannelFactory<IEchoChannel> channelFactory = new ChannelFactory<IEchoChannel>("RelayEndpoint", new EndpointAddress(serviceUri));

            channelFactory.Endpoint.Behaviors.Add(sasCredential);

            IEchoChannel channel = channelFactory.CreateChannel();
            channel.Open();

            Console.WriteLine("Enter text to echo (or [Enter] to exit):");
            string input = Console.ReadLine();
            while (input != String.Empty)
            {
                try
                {
                    Console.WriteLine("Server echoed: {0}", channel.Echo(input));
                }
                catch (Exception e)
                {
                    Console.WriteLine("Error: " + e.Message);
                }
                input = Console.ReadLine();
            }

            channel.Close();
            channelFactory.Close();

        }
    }
}
```

## <a name="run-the-applications"></a>執行應用程式

1. 按 **Ctrl+Shift+B** 以建置方案。 這會建置您在先前步驟中建立的用戶端專案和服務專案。
2. 執行用戶端應用程式之前，您必須確定服務應用程式正在執行。 在 Visual Studio 的 [方案總管] 中，以滑鼠右鍵按一下 **EchoService** 方案，然後按一下 [屬性]。
3. 在 [方案屬性] 對話方塊中，按一下 [啟始專案]，然後按一下 [多個啟始專案] 按鈕。 確定 **EchoService** 顯示在清單中的最前面。
4. 將 **EchoService** 和 **EchoClient** 專案的 [動作] 方塊設定為 [啟動]。

    ![][5]
5. 按一下 [專案相依性]。 在 [專案] 方塊中，選取 **EchoClient**。 在 [取決於] 方塊中，確定已核取 **EchoService**。

    ![][6]
6. 按一下 [確定] 以關閉 [屬性] 對話方塊。
7. 按 **F5** 執行這兩個專案。
8. 兩個主控台視窗隨即開啟並提示您輸入命名空間名稱。 必須先執行服務，因此在 **EchoService** 主控台視窗中輸入命名空間，然後按 **Enter**。
9. 接下來，系統會提示您輸入 SAS 金鑰。 輸入 SAS 金鑰並按 ENTER 鍵。

    以下是主控台視窗的範例輸出。 請注意，此處提供的值僅適用於範例。

    `Your Service Namespace: myNamespace` `Your SAS Key: <SAS key value>`

    服務應用程式會將它所接聽的位址列印到主控台視窗，如下列範例所示。

    `Service address: sb://mynamespace.servicebus.windows.net/EchoService/` `Press [Enter] to exit`
10. 在 **EchoClient** 主控台視窗中，輸入您先前針對此服務應用程式輸入的相同資訊。 遵循上述步驟，為此用戶端應用程式輸入相同的服務命名空間和 SAS 金鑰值。
11. 輸入這些值後，用戶端就會開啟服務通道，並提示您輸入一些文字，如下列主控台輸出範例所示。

    `Enter text to echo (or [Enter] to exit):`

    輸入一些文字以傳送至服務應用程式並按 Enter 鍵。 這段文字會透過 Echo 服務作業傳送至服務，並如下列範例輸出所示出現在服務主控台視窗。

    `Echoing: My sample text`

    用戶端應用程式會接收 `Echo` 作業的傳回值 (此值是原始文字)，並將它列印至主控台視窗。 以下是用戶端主控台視窗的範例輸出。

    `Server echoed: My sample text`
12. 您可以繼續以這種方式將文字訊息從用戶端傳送至服務。 完成後，在用戶端和服務主控台視窗中按 Enter 鍵以結束這兩個應用程式。

## <a name="next-steps"></a>後續步驟

本教學課程示範了如何使用服務匯流排的 WCF 轉送功能，來建置 Azure 轉送用戶端應用程式和服務。 如需使用[服務匯流排通訊](../service-bus-messaging/service-bus-messaging-overview.md)的類似教學課程，請參閱[開始使用服務匯流排佇列](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md)。

若要深入了解 Azure 轉送，請參閱下列主題。

* [Azure 服務匯流排架構概觀](../service-bus-messaging/service-bus-fundamentals-hybrid-solutions.md#relays)
* [Azure 轉送概觀](relay-what-is-it.md)
* [如何使用 WCF 轉送服務搭配 .NET](relay-wcf-dotnet-get-started.md)

[2]: ./media/service-bus-relay-tutorial/create-console-app.png
[3]: ./media/service-bus-relay-tutorial/install-nuget.png
[5]: ./media/service-bus-relay-tutorial/set-projects.png
[6]: ./media/service-bus-relay-tutorial/set-depend.png
