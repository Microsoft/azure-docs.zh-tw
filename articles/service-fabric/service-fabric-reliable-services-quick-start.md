---
title: 在 C# 中建立第一個 Service Fabric 應用程式 | Microsoft Docs
description: 概述使用無狀態與具狀態服務來建立 Microsoft Azure Service Fabric 應用程式。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: d9b44d75-e905-468e-b867-2190ce97379a
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/16/2018
ms.author: vturecek
ms.openlocfilehash: 6977fa0a62767cebbd1000335c6c3a33a5991c2c
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34208160"
---
# <a name="get-started-with-reliable-services"></a>開始使用 Reliable Service
> [!div class="op_single_selector"]
> * [Windows 上的 C# ](service-fabric-reliable-services-quick-start.md)
> * [Linux 上的 Java](service-fabric-reliable-services-quick-start-java.md)
> 
> 

Azure Service Fabric 應用程式包含一個或多個執行您的程式碼的服務。 本指南說明如何透過 [Reliable Services](service-fabric-reliable-services-introduction.md)同時建立無狀態與具狀態的 Service Fabric 應用程式。  此 Microsoft Virtual Academy 影片也示範如何建立無狀態的 Reliable Service：<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=s39AO76yC_7206218965">  
<img src="./media/service-fabric-reliable-services-quick-start/ReliableServicesVid.png" WIDTH="360" HEIGHT="244">  
</a></center>

## <a name="basic-concepts"></a>基本概念
若要開始使用 Reliable Services，您只需要了解幾個基本概念：

* **服務類型**：這是您的服務實作。 它是由您撰寫的類別定義，它會擴充 `StatelessService` 與其中使用的任何其他程式碼或相依性，以及名稱與版本號碼。
* **具名服務執行個體**：若要執行您的服務，可以建立您服務類型的具名執行個體，很像您建立類別類型的物件執行個體一樣。 服務執行個體的名稱格式是使用「fabric:/」配置的 URI，例如「fabric:/MyApp/MyService」。
* **服務主機**：您建立的具名服務執行個體需要在主機處理序內部執行。 服務主機只是您服務的執行個體可執行所在的程序。
* **服務註冊**：註冊可將所有項目結合在一起。 服務類型必須在服務主機的 Service Fabric 執行階段中註冊，以允許 Service Fabric 建立其執行個體來執行。  

## <a name="create-a-stateless-service"></a>建立無狀態服務
無狀態服務是目前在雲端應用程式中做為基準的服務類型。 服務會視為無狀態，因為服務本身不包含需要可靠地儲存或設為高度可用的資料。 如果無狀態服務的執行個體關閉，其所有內部狀態都會遺失。 在此類型的服務中，狀態必須保存到外部存放區，例如 Azure 資料表或 SQL 資料庫中，才能成為高度可用且可靠。

以管理員身分啟動 Visual Studio 2015 或 Visual Studio 2017，並建立新的 Service Fabric 應用程式專案，命名為 HelloWorld：

![使用新增專案對話方塊來建立新的 Service Fabric 應用程式](media/service-fabric-reliable-services-quick-start/hello-stateless-NewProject.png)

然後使用 **.Net Core 2.0** 建立名為 HelloWorldStateless 的無狀態服務專案：

![在第二個對話方塊中，建立無狀態服務專案](media/service-fabric-reliable-services-quick-start/hello-stateless-NewProject2.png)

您的方案現在包含兩個專案：

* *HelloWorld*。 這是包含您*服務*的*應用程式*專案。 它也包含描述應用程式的應用程式資訊清單，以及一些幫助您部署應用程式的 PowerShell 指令碼。
* *HelloWorldStateless*。 這是服務專案。 它包含無狀態服務實作。

## <a name="implement-the-service"></a>實作服務
在服務專案中開啟 **HelloWorldStateless.cs** 檔案。 在 Service Fabric 中，服務可以執行任何商務邏輯。 服務 API 為您的程式碼提供兩個進入點：

* 開放式的進入點方法，稱為 *RunAsync*，您可以在這裡開始執行任何工作負載，包括長時間執行的計算工作負載。

```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    ...
}
```

* 通訊進入點，您可以在這裡插入選擇的通訊堆疊，例如 ASP.NET Core。 這就是您可以開始接收來自使用者和其他服務要求的地方。

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    ...
}
```

在本教學課程中，我們將著重在 `RunAsync()` 進入點方法。 這就是您可以立即開始執行程式碼的地方。
專案範本包括 `RunAsync()` 的實作範例，它會遞增一個循環計數。

> [!NOTE]
> 如需有關如何使用通訊堆疊的詳細資訊，請參閱 [Service Fabric Web API 服務與 OWIN 自我裝載](service-fabric-reliable-services-communication-webapi.md)
> 
> 

### <a name="runasync"></a>RunAsync
```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following sample code with your own logic
    //       or remove this RunAsync override if it's not needed in your service.

    long iterations = 0;

    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        ServiceEventSource.Current.ServiceMessage(this.Context, "Working-{0}", ++iterations);

        await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    }
}
```

當服務的執行個體已放置並且可以執行時，平台會呼叫這個方法。 針對無狀態服務，這僅表示開啟服務執行個體的時間。 會提供取消語彙基元以協調服務執行個體何時需要關閉。 在 Service Fabric 中，服務執行個體的開啟/關閉循環可能會在整個服務存留期中發生許多次。 發生這種情形的原因有很多，包括：

* 系統可能為了資源平衡移動您的服務執行個體。
* 在您的程式碼中發生錯誤。
* 應用程式或系統已升級。
* 基礎硬體發生中斷。

此協調流程是由系統管理，可讓您的服務維持高度可用且正確平衡。

`RunAsync()` 不應該同步封鎖。 RunAsync 的實作應該傳回工作或等待任何長時間執行或封鎖作業，以允許執行階段繼續執行。 請注意，在上一個範例的 `while(true)` 迴圈中，會使用 Task-returning `await Task.Delay()`。 如果您的工作負載必須同步封鎖，您應該使用 `Task.Run()` 在您的 `RunAsync` 實作中排定一個新的工作。

取消您的工作負載是由所提供的取消語彙基元協調的協同努力。 系統會先等待您工作結束 (成功完成、取消或錯誤) 再繼續。 系統要求取消時，務必接受取消權杖、完成任何工作，並儘快結束 `RunAsync()`。

在這個無狀態服務範例中，計數會儲存在本機變數。 但是，因為這是無狀態服務，所儲存的值只針對其服務執行個體的目前生命週期而存在。 當服務移動或重新啟動時，值將會遺失。

## <a name="create-a-stateful-service"></a>建立具狀態服務
Service Fabric 導入了一種可設定狀態的新服務。 具狀態服務能夠可靠地在服務本身維持狀態，和使用它的程式碼共置。 狀態是由 Service Fabric 設為高度可用，而不需要將狀態保存到外部存放區。

若要將計數器值從無狀態轉換成高度可用且持續，即使服務移動或重新啟動亦然，您需要具狀態服務。

在同一個 *HelloWorld* 應用程式中，於應用程式專案中的服務參考上按一下滑鼠右鍵並選取 [新增] -> [新增 Service Fabric 服務]，以加入新的服務。

![將服務加入 Service Fabric 應用程式](media/service-fabric-reliable-services-quick-start/hello-stateful-NewService.png)

選取 [.Net Core 2.0] -> [具狀態服務] 並將它命名為 HelloWorldStateful。 按一下 [確定]。

![使用新增專案對話方塊來建立新的 Service Fabric 具狀態服務](media/service-fabric-reliable-services-quick-start/hello-stateful-NewProject.png)

您的應用程式現在應該有兩個服務：無狀態服務 *HelloWorldStateless*和具狀態服務 *HelloWorldStateful*。

具狀態服務與無狀態服務具有相同的進入點。 主要差異在於可以可靠地儲存狀態之狀態供應器  的可用性。 Service Fabric 隨附稱為 [可靠的集合](service-fabric-reliable-services-reliable-collections.md)的狀態供應器實作，它可讓您透過可靠狀態管理員建立複寫的資料結構。 具狀態可靠服務預設會使用此狀態供應器。

在 HelloWorldStateful  中開啟 *HelloWorldStateful.cs*，其中包含下列 RunAsync 方法：

```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following sample code with your own logic
    //       or remove this RunAsync override if it's not needed in your service.

    var myDictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<string, long>>("myDictionary");

    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        using (var tx = this.StateManager.CreateTransaction())
        {
            var result = await myDictionary.TryGetValueAsync(tx, "Counter");

            ServiceEventSource.Current.ServiceMessage(this.Context, "Current Counter Value: {0}",
                result.HasValue ? result.Value.ToString() : "Value does not exist.");

            await myDictionary.AddOrUpdateAsync(tx, "Counter", 0, (key, value) => ++value);

            // If an exception is thrown before calling CommitAsync, the transaction aborts, all changes are
            // discarded, and nothing is saved to the secondary replicas.
            await tx.CommitAsync();
        }

        await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    }
```

### <a name="runasync"></a>RunAsync
`RunAsync()` 在具狀態和無狀態服務中的運作方式類似。 只不過在具狀態服務中，平台會代替您執行額外的工作，然後才執行 `RunAsync()`。 這項工作包含確保可靠狀態管理員和可靠的集合可以使用。

### <a name="reliable-collections-and-the-reliable-state-manager"></a>可靠的集合與可靠狀態管理員
```csharp
var myDictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<string, long>>("myDictionary");
```

[IReliableDictionary](https://msdn.microsoft.com/library/dn971511.aspx) 是一個字典實作，您可以在服務中可靠地儲存狀態。 有了 Service Fabric 和可靠的集合，您可以直接在服務中儲存資料，而不需要外部的持續性存放區。 可靠的集合可讓您的資料具備高可用性。 Service Fabric 會藉由為您建立與管理服務的多個複本  來完成此作業。 它也提供 API 讓管理這些複本和其狀態轉換的複雜性抽象化。

可靠的集合可以儲存任何 .NET 類型 (包括您的自訂類型)，不過有幾個需要注意的事項：

* Service Fabric 藉由在節點之間複寫  狀態來使您的狀態高度可用，而可靠的集合會將您的資料儲存到每個複本上的本機磁碟。 這表示所有儲存在可靠的集合中的項目必須可序列化 。 根據預設，可靠的集合使用 [DataContract](https://msdn.microsoft.com/library/system.runtime.serialization.datacontractattribute%28v=vs.110%29.aspx) 進行序列化，因此在您使用預設序列化程式時，請務必確定您的類型受到[資料合約序列化程式支援](https://msdn.microsoft.com/library/ms731923%28v=vs.110%29.aspx)。
* 當您在可靠的集合上認可交易時，物件會複寫以獲得高可用性。 儲存在可靠集合中的物件會保留在服務中的本機記憶體。 這表示您有物件的本機參考。
  
   很重要的一點是，您不要改變那些物件的本機執行個體而不在交易中的可靠集合上執行更新作業。 這是因為不會自動複寫對本機物件執行個體所做的變更。 您必須將物件重新插入到字典中，或在字典上使用其中一個更新  方法。

可靠狀態管理員會為您管理可靠的集合。 在您的服務中隨時隨地，您只需要以名稱向可靠狀態管理員要求可靠的集合。 可靠狀態管理員會確保您取回參考。 不建議您將可靠集合執行個體的參考儲存在類別成員變數或屬性中。 請特別小心以確保在服務生命週期中隨時將參考設定為執行個體。 可靠狀態管理員會為您處理這項工作，並且針對重複造訪最佳化。

### <a name="transactional-and-asynchronous-operations"></a>交易式和非同步作業
```csharp
using (ITransaction tx = this.StateManager.CreateTransaction())
{
    var result = await myDictionary.TryGetValueAsync(tx, "Counter-1");

    await myDictionary.AddOrUpdateAsync(tx, "Counter-1", 0, (k, v) => ++v);

    await tx.CommitAsync();
}
```

可靠的集合與其 `System.Collections.Generic` 和 `System.Collections.Concurrent` 對應項目具有許多相同的作業 (LINQ 除外)。 可靠的集合上的作業是非同步的。 這是因為具備可靠集合的寫入作業執行 I/O 作業以將資料複寫並保存至磁碟。

可靠的集合作業為「交易式」 作業，因此您可以在多個可靠的集合和作業之間維持狀態的一致。 比方說，您可能會從可靠佇列取出一個工作項目、對它執行作業，然後將結果儲存在可靠字典中，全都在單一交易中完成。 這會被視為不可部分完成的作業，而且它可保證整個作業都會成功，或整個作業都會回復。 如果您從佇列取消項目之後，但在您儲存結果之前發生錯誤，那麼會回復整個交易，且項目會保持在佇列中進行處理。

## <a name="run-the-application"></a>執行應用程式
現在我們回到 HelloWorld  應用程式。 您現在可以建置並部署您的服務。 當您按下 **F5**，您的應用程式會建置並部署至本機叢集。

服務開始執行之後，您可以在 [診斷事件]  視窗中檢視產生的 Windows 事件追蹤 (ETW) 事件。 請注意，應用程式中會同時顯示無狀態服務和具狀態服務的事件。 您可以按一下 [暫停]  按鈕來暫停串流。 然後，您可以藉由展開訊息來檢查訊息的詳細資料。

> [!NOTE]
> 在您執行應用程式之前，請確定您有執行中的本機開發叢集。 查看 [入門指南](service-fabric-get-started.md) 以取得設定您的本機環境的資訊。
> 
> 

![在 Visual Studio 中檢視診斷事件](media/service-fabric-reliable-services-quick-start/hello-stateful-Output.png)

## <a name="next-steps"></a>後續步驟
[在 Visual Studio 中偵錯 Service Fabric 應用程式](service-fabric-debugging-your-application.md)

[開始使用：Service Fabric Web API 服務與 OWIN 自我裝載 | Microsoft Azure](service-fabric-reliable-services-communication-webapi.md)

[深入了解可靠的集合](service-fabric-reliable-services-reliable-collections.md)

[部署應用程式](service-fabric-deploy-remove-applications.md)

[應用程式升級](service-fabric-application-upgrade.md)

[Reliable Services 的開發人員參考資料](https://msdn.microsoft.com/library/azure/dn706529.aspx)

