---
title: Azure Service Fabric 主控模型 | Microsoft Docs
description: 說明已部署之 Service Fabric 的複本 (或執行個體) 與服務主機處理序之間的關聯性。
services: service-fabric
documentationcenter: .net
author: harahma
manager: timlt
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/15/2017
ms.author: harahma
ms.openlocfilehash: 367f21c63eac3969fb19eada91eae9a8577921de
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44348475"
---
# <a name="azure-service-fabric-hosting-model"></a>Azure Service Fabric 主控模型
本文提供 Azure Service Fabric 所提供之應用程式主控模型的概觀，並說明**共用處理序**與**專屬處理序**模型之間的差異。 本文說明已部署之應用程式在 Service Fabric 節點上看起來的樣子，以及服務的複本 (或執行個體) 與服務主機處理序之間的關聯性。

在進一步進行之前，請確定您已了解[在 Service Fabric 中模型化應用程式][a1]中所說明的各種概念與關聯性。 

> [!NOTE]
> 在本文中，除非明確指出是不同意義，否則：
>
> - 「複本」是指具狀態服務的複本和無狀態服務的執行個體。
> - CodePackage 可視為等同於註冊 ServiceType 並裝載該 ServiceType 服務複本的 ServiceHost 處理序。
>

為了了解主控模型，我們將逐步解說範例。 假設我們有 ApplicationType 'MyAppType'，其具有 ServiceType 'MyServiceType'。 'MyServiceType' 是由 ServicePackage 'MyServicePackage' 提供，它具有 CodePackage 'MyCodePackage'。 'MyCodePackage' 會在執行時註冊 ServiceType 'MyServiceType'。

假設我們有一個含有 3 個節點的叢集，並且建立一個類型為 'MyAppType' 的「應用程式」**fabric:/App1**。 在這個「應用程式」**fabric:/App1** 中，我們會建立一個類型為 'MyServiceType' 的服務 **fabric:/App1/ServiceA**。 此服務具有 2 個分割區 (例如 **P1** 和 **P2**) 且每一分割區有 3 個複本。 下圖顯示此應用程式最後部署在節點上時的檢視。


![所部署應用程式的節點檢視圖表][node-view-one]


Service Fabric 啟動了 'MyServicePackage'，而 'MyServicePackage' 則啟動了裝載來自兩個分割區之複本的 'MyCodePackage'。 此叢集內的所有節點都具有相同檢視，因為我們所選擇的每一分割區複本數目等於此叢集內的節點數目。 讓我們在應用程式 **fabric:/App1** 中建立另一個服務 **fabric:/App1/ServiceB**。 此服務具有 1 個分割區 (例如 **P3**) 且每一分割區有 3 個複本。 下圖顯示節點上的新檢視：


![所部署應用程式的節點檢視圖表][node-view-two]


Service Fabric 在現有的 'MyServicePackage' 啟用項中，放置了服務 **fabric:/App1/ServiceB** 之分割區 **P3** 的新複本。 現在。 建立另一個 'MyAppType' 類型的「應用程式」**fabric:/App2**。 在 **fabric:/App2** 中建立服務 **fabric:/App2/ServiceA**。 此服務具有 2 個分割區 (**P4** 和 **P5**) 且每一分割區有 3 個複本。 下圖顯示新的節點檢視：


![所部署應用程式的節點檢視圖表][node-view-three]


Service Fabric 會啟動 'MyServicePackage' 的新複本，而這會啟動 'MyCodePackage' 的新複本。 **fabric:/App2/ServiceA** 服務的兩個分割區複本 (**P4** 和 **P5**)，會位於這個新複本 'MyCodePackage'。

## <a name="shared-process-model"></a>共用處理序模型
上一節說明了 Service Fabric 所提供的預設主控模型，其稱為「共用處理序」模型。 在此模型中，針對指定的「應用程式」，在一個「節點」上只會啟用一份指定的 ServicePackage (這會啟動其包含的所有 CodePackage)。 指定 ServiceType 的所有服務複本則會放在註冊該 ServiceType 的 CodePackage 中。 換句話說，指定的 *ServiceType* 之節點上所有服務的所有複本都會共用相同的處理序。

## <a name="exclusive-process-model"></a>專屬處理序模型
Service Fabric 所提供的另一個主控模型是「專屬處理序」模型。 在此模型中，指定節點上的每個複本都存在於自己的專屬處理序中。 Service Fabric 會啟動 ServicePackage 的新複本 (這會啟動其內含的所有 CodePackage)。 複本會放在註冊該複本所屬服務 ServiceType 的 CodePackage 中。 

如果您使用的是 Service Fabric 5.6 版或更新版本，您可以在建立服務時選擇專屬處理序模型 (藉由使用 [PowerShell][p1]、[REST][r1] 或 [FabricClient][c1])。 將 **ServicePackageActivationMode** 指定為 'ExclusiveProcess'。

```powershell
PS C:\>New-ServiceFabricService -ApplicationName "fabric:/App1" -ServiceName "fabric:/App1/ServiceA" -ServiceTypeName "MyServiceType" -Stateless -PartitionSchemeSingleton -InstanceCount -1 -ServicePackageActivationMode "ExclusiveProcess"
```

```csharp
var serviceDescription = new StatelessServiceDescription
{
    ApplicationName = new Uri("fabric:/App1"),
    ServiceName = new Uri("fabric:/App1/ServiceA"),
    ServiceTypeName = "MyServiceType",
    PartitionSchemeDescription = new SingletonPartitionSchemeDescription(),
    InstanceCount = -1,
    ServicePackageActivationMode = ServicePackageActivationMode.ExclusiveProcess
};

var fabricClient = new FabricClient(clusterEndpoints);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

如果您的應用程式資訊清單中有預設的服務，您可以指定 **ServicePackageActivationMode** 屬性來選擇「專屬處理序」模型：

```xml
<DefaultServices>
  <Service Name="MyService" ServicePackageActivationMode="ExclusiveProcess">
    <StatelessService ServiceTypeName="MyServiceType" InstanceCount="1">
      <SingletonPartition/>
    </StatelessService>
  </Service>
</DefaultServices>
```
現在，讓我們在應用程式 **fabric:/App1** 中建立另一個服務 **fabric:/App1/ServiceC**。 此服務具有 2 個分割區 (例如 **P6** 和 **P7**) 且每一分割區有 3 個複本。 您將 **ServicePackageActivationMode** 設定為 'ExclusiveProcess'。 下圖顯示節點上的新檢視：


![所部署應用程式的節點檢視圖表][node-view-four]


如您所見，Service Fabric 啟用了兩份新的 'MyServicePackage' (針對來自分割區 **P6** 和 **P7** 的每個複本各啟用一份)。 Service Fabric 將每個複本放在其專用的一份 CodePackage 中。 當您使用「專屬處理序」模型時，就指定的「應用程式」而言，一個「節點」上可以有多份作用中的指定 ServicePackage。 在上述範例中**fabric:/App1** 有三份作用中的 'MyServicePackage'。 這每一份作用中的 'MyServicePackage' 都有關聯的 **ServicePackageActivationId**。 此識別碼可識別「應用程式」**fabric:/App1** 中的該複本。

當您只對應用程式使用共用處理序模型時，一個節點上只能有一份作用中的 ServicePackage。 這個 ServicePackage 啟用的 **ServicePackageActivationId** 為空字串。 例如，**fabric:/App2** 便是這種情況。

> [!NOTE]
>- 共用處理序主控模型會與等於 **SharedProcess** 的 **ServicePackageActivationMode** 對應。 這是預設的主控模型，在建立服務時不須指定 **ServicePackageActivationMode**。
>
>- 專屬處理序主控模型會與等於 **ExclusiveProcess** 的 **ServicePackageActivationMode** 對應。 若要使用此設定，您應該在建立服務時予以明確指定。 
>
>- 若要檢視服務的主控模型，請查詢[服務描述][p2]並查看 **ServicePackageActivationMode** 的值。
>
>

## <a name="work-with-a-deployed-service-package"></a>使用已部署的服務套件
節點上一份作用中的 ServicePackage 稱為[已部署的服務套件][p3]。 當您使用「專屬處理序」模型來建立服務時，就指定的「應用程式」而言，同一個 ServicePackage 可能會有多個已部署的服務套件。 如果您要執行所部署服務套件特有的作業，請提供 **ServicePackageActivationId** 來識別所部署的特定服務套件。 例如，如果您要[報告所部署服務套件的健康情況][ p4]或[重新啟動所部署服務套件的程式碼套件][p5]，請提供識別碼。

您可以查詢節點上[已部署的服務套件][p3]清單，來找到所部署服務套件的 **ServicePackageActivationId**。 查詢節點上的[已部署的服務類型][p6]、[已部署的複本][p7]及[已部署的程式碼套件][p8]時，查詢結果也會包含父代已部署服務套件的 **ServicePackageActivationId**。

> [!NOTE]
>- 在「共用處理序」主控模型下，指定節點上的指定應用程式只會啟用一份 ServicePackage。 其 **ServicePackageActivationId** 會等於「空字串」，在執行與所部署服務套件相關的作業時不須指定。 
>
> - 在「專屬處理序」主控模型下，指定節點上的指定應用程式會有一或多份作用中的 ServicePackage。 每個啟動項都有一個「非空白」的 **ServicePackageActivationId**，在執行與所部署服務套件相關的作業時已指定。 
>
> - 如果省略 **ServicePackageActivationId**，它會預設為「空字串」。 如果有在「共用處理序」模型下啟用的已部署服務套件存在，系統就會在該套件上執行作業。 否則作業會失敗。
>
> - 請勿查詢一次並快取 **ServicePackageActivationId**。 識別碼是動態產生的，會因為各種原因而改變。 在執行需要 **ServicePackageActivationId** 的作業之前，您應該先查詢節點上[已部署的服務套件][p3]清單。 然後使用來自查詢結果的 **ServicePackageActivationId** 來執行原先的作業。
>
>

## <a name="guest-executable-and-container-applications"></a>客體可執行檔和容器應用程式
Service Fabric 會將[客體可執行檔][a2]和[容器][a3]應用程式視為獨立的無狀態服務。 ServiceHost (一個處理序或容器) 中沒有任何 Service Fabric 執行階段。 由於這些服務是獨立的，因此每一 ServiceHost 的複本數並不適用於這些服務。 與這些服務搭配使用的最常見組態是 [InstanceCount][c2] 等於 -1 的單一分割區 (在叢集的每個節點上都會執行一份服務程式碼)。 

這些服務的預設 **ServicePackageActivationMode** 是 **SharedProcess**，在此情況下，Service Fabric 會針對指定的「應用程式」在一個「節點」上啟用一份 ServicePackage。  這意謂著一個節點上只會執行一份服務程式碼。 如果您想要在一個節點上執行多份服務程式碼，請在建立服務時將 **ServicePackageActivationMode** 指定為 **ExclusiveProcess**。 例如，您可以在建立 ServiceType (指定於 ServiceManifest 中) 的多個服務 (Service1 至 ServiceN) 時這麼做，也可以在您的服務是多重分割時這麼做。 

## <a name="change-the-hosting-model-of-an-existing-service"></a>變更現有服務的主控模型
現階段，您無法將現有服務的主控模型從共用處理序變更為專屬處理序 (反之亦然)。

## <a name="choose-between-the-hosting-models"></a>在主控模型之間做出選擇
您應該評估哪種主控模型最適合您的需求。 共用處理序模型有較佳的作業系統資源使用率，因為產生的處理序較少，且同一個處理序中的多個複本可以共用連接埠。 不過，如果其中一個複本發生錯誤而必須關閉服務主機，就會影響到該相同處理序中的所有其他複本。

 專屬處理序模型會讓每個複本處於自己的處理序中，而可提供較佳的隔離。 如果某個複本發生問題，該複本不會影響到其他複本。 此模型適用於通訊協定不支援連接埠共用的情況。 它可協助在複本層級套用資源管理。 不過，專屬處理序會耗用較多的作業系統資源，因為它會為節點上的每個複本都產生一個處理序。

## <a name="exclusive-process-model-and-application-model-considerations"></a>專屬處理序模型和應用程式模型考量
對於大部分的應用程式，您可以藉由讓每一 ServicePackage 保有一個 ServiceType，以在 Service Fabric 中建立應用程式模型。 

在某些案例中，Service Fabric 也允許每個 ServicePackage 具有多個 ServiceType (而且一個 CodePackage 可以註冊多個 ServiceType)。 以下是一些適用這些設定的案例：

- 您想要藉由減少產生處理序並提高每一處理序的複本密度，將資源使用率最佳化。
- 來自不同 ServiceType 的複本需要共用一些具有高初始化或記憶體成本的通用資料。
- 您有免費的服務供應項目，而您想要透過將服務的所有複本都放在同一個處理序中，為資源使用量設定限制。

專屬處理序主控模型與每一 ServicePackage 有多個 ServiceType 的應用程式模型並不相同。 這是因為每一 ServicePackage 多個 ServiceType 的設計是用來提高複本間的資源共用，以及提高每一處理序的複本密度。 專屬處理序模型的設計目的是要實現不同結果。

思考一下每一 ServicePackage 有多個 ServiceType 的案例，其中是由不同的 CodePackage 註冊每個 ServiceType。 假設我們有一個 ServicePackage 'MultiTypeServicePackge'，其中包含兩個 CodePackage：

- 'MyCodePackageA'，它會註冊 ServiceType 'MyServiceTypeA'。
- 'MyCodePackageB'，它會註冊 ServiceType 'MyServiceTypeB'。

現在，假設我們建立一個「應用程式」**fabric:/SpecialApp**。 在 **fabric:/SpecialApp** 中，我們使用專屬處理序模型建立下列兩個服務：

- 'MyServiceTypeA' 類型的服務 **fabric:/SpecialApp/ServiceA**，此服務有 2 個分割區 (例如 **P1** 和 **P2**) 且每一分割區有 3 個複本。
- 'MyServiceTypeB' 類型的服務 **fabric:/SpecialApp/ServiceB**，此服務有 2 個分割區 (**P3** 和 **P4**) 且每一分割區有 3 個複本。

在指定的節點上，這兩個服務各有兩個複本。 由於我們已使用專屬處理序模型來建立這些服務，因此 Service Fabric 會為每個複本建立一份新的 'MyServicePackage'。 每個 'MultiTypeServicePackge' 啟用項都會啟動一份 'MyCodePackageA' 和 'MyCodePackageB'。 不過，'MyCodePackageA' 或 'MyCodePackageB' 只有其中之一會裝載作為 'MultiTypeServicePackge' 啟用對象的複本。 下圖顯示節點檢視：


![所部署應用程式的節點檢視圖表][node-view-five]


在服務 **fabric:/SpecialApp/ServiceA** 分割區 **P1** 之複本的 'MultiTypeServicePackge' 啟用項中，是由 'MyCodePackageA' 裝載複本。 'MyCodePackageB' 則正在執行。 同樣地，在服務 **fabric:/SpecialApp/ServiceB** 分割區 **P3** 之複本的 'MultiTypeServicePackge' 啟用項中，是由 'MyCodePackageB' 裝載複本。 'MyCodePackageB' 則正在執行。 因此，每一 ServicePackage 的 CodePackage (註冊不同的 ServiceType) 數目越多，備援資源的使用量就越高。 
 
 不過，如果我們使用「共用處理序」模型來建立服務 **fabric:/SpecialApp/ServiceA** 和 **fabric:/SpecialApp/ServiceB**，則 Service Fabric 將只會為「應用程式」**fabric:/SpecialApp** 啟用一份 'MultiTypeServicePackge'。 'MyCodePackageA' 裝載服務 **fabric:/SpecialApp/ServiceA** 的所有複本。 'MyCodePackageB' 裝載服務 **fabric:/SpecialApp/ServiceB** 的所有複本。 下圖顯示此設定中的節點檢視： 


![所部署應用程式的節點檢視圖表][node-view-six]


在上述範例中，您可能會認為如果 'MyCodePackageA' 既註冊 'MyServiceTypeA' 也註冊 'MyServiceTypeB'，而沒有 'MyCodePackageB' 存在，就不會執行備援的 CodePackage。 雖然這是正確的，但此應用程式模型與專屬處理序主控模型並不相同。 如果目標是要將每個複本放在其自己的專用處理序中，就不需要從同一個 CodePackage 註冊兩個 ServiceType。 相反地，您只需要將每個 ServiceType 放在它自己的 ServicePackage 中。

## <a name="next-steps"></a>後續步驟
[對應用程式進行封裝][a4]並使它準備好進行部署。

[部署與移除應用程式][a5]。 此文章說明如何使用 PowerShell 來管理應用程式執行個體。

<!--Image references-->
[node-view-one]: ./media/service-fabric-hosting-model/node-view-one.png
[node-view-two]: ./media/service-fabric-hosting-model/node-view-two.png
[node-view-three]: ./media/service-fabric-hosting-model/node-view-three.png
[node-view-four]: ./media/service-fabric-hosting-model/node-view-four.png
[node-view-five]: ./media/service-fabric-hosting-model/node-view-five.png
[node-view-six]: ./media/service-fabric-hosting-model/node-view-six.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[a1]: service-fabric-application-model.md
[a2]: service-fabric-guest-executables-introduction.md
[a3]: service-fabric-containers-overview.md
[a4]: service-fabric-package-apps.md
[a5]: service-fabric-deploy-remove-applications.md

[r1]: https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-createservice

[c1]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync
[c2]: https://docs.microsoft.com/dotnet/api/system.fabric.description.statelessservicedescription.instancecount

[p1]: https://docs.microsoft.com/powershell/module/servicefabric/new-servicefabricservice
[p2]: https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricservicedescription
[p3]: https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricdeployedservicePackage
[p4]: https://docs.microsoft.com/powershell/module/servicefabric/send-servicefabricdeployedservicepackagehealthreport
[p5]: https://docs.microsoft.com/powershell/module/servicefabric/restart-servicefabricdeployedcodepackage
[p6]: https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricdeployedservicetype
[p7]: https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricdeployedreplica
[p8]: https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricdeployedcodepackage
