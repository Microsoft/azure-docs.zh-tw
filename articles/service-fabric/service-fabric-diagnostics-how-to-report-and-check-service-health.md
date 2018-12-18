---
title: 使用 Service Fabric 回報和檢查健全狀況 | Microsoft Docs
description: 了解如何從您的服務程式碼傳送健全狀況報告，以及如何使用 Azure Service Fabric 所提供的健全狀況監視工具檢查您服務的健全狀況。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: mfussell
editor: ''
ms.assetid: 7c712c22-d333-44bc-b837-d0b3603d9da8
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/2/2017
ms.author: dekapur
ms.openlocfilehash: d374886efb708797db1dd6352aa063a56aff4f44
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39427303"
---
# <a name="report-and-check-service-health"></a>回報和檢查服務健康情況
當您的服務發生問題時，您回應並修正事件和中斷的能力，取決於您快速偵測問題的能力。 如果您從服務程式碼向 Azure Service Fabric 健全狀況管理員回報問題和失敗，您便可以使用 Service Fabric 提供的標準健全狀況監視工具來檢查健全狀況。

有三種方式可讓您回報服務的健全狀況：

* 使用 [Partition](https://docs.microsoft.com/dotnet/api/system.fabric.istatefulservicepartition) 或 [CodePackageActivationContext](https://docs.microsoft.com/dotnet/api/system.fabric.codepackageactivationcontext) 物件。  
  您可以使用 `Partition` 和 `CodePackageActivationContext` 物件在屬於目前內容一部分的項目中回報健全狀況。 比方說，做為複本一部分執行的程式碼只能回報該複本、其所屬的分割區，以及其所屬應用程式的健全狀況。
* 使用 `FabricClient`。   
  當叢集不是[安全的](service-fabric-cluster-security.md)或者服務是以系統管理員權限執行時，您才能使用 `FabricClient`，從服務程式碼回報健全狀況。 大部分的實際案例不會使用不安全的叢集，或提供系統管理員權限。 您可以使用 `FabricClient`，回報任何屬於叢集一部分之實體的健全狀況。 然而在理想的情況下，服務程式碼應該只會傳送與其本身健全狀況相關的報告。
* 使用位於叢集、應用程式、已部署應用程式、服務、服務套件、磁碟分割、複本或節點層級的 REST API。 這可用來從容器內部回報健全狀況。

這篇文章會引導您完成從服務程式碼回報健全狀況的範例。 此範例也會示範如何使用 Service Fabric 提供的工具來檢查健康情況。 本文旨在快速介紹 Service Fabric 的健全狀況監視功能。 如需更詳細的資訊，您可以閱讀一系列有關健全狀況的深入文章，從本文結尾的連結開始。

## <a name="prerequisites"></a>必要條件
您必須安裝下列項目：

* Visual Studio 2015 或 Visual Studio 2017
* Service Fabric SDK

## <a name="to-create-a-local-secure-dev-cluster"></a>建立本機的安全開發人員叢集
* 以系統管理員權限開啟 PowerShell 並執行下列命令︰

![示範如何建立安全開發人員叢集的命令](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/create-secure-dev-cluster.png)

## <a name="to-deploy-an-application-and-check-its-health"></a>部署應用程式並檢查其健康情況
1. 以系統管理員身分開啟 Visual Studio。
1. 使用 **具狀態服務** 範本建立專案。
   
    ![建立與具狀態服務搭配使用的 Service Fabric 應用程式](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/create-stateful-service-application-dialog.png)
1. 按 **F5** 以在偵錯模式中執行應用程式。 應用程式會部署至本機叢集。
1. 應用程式執行之後，在通知區域中的本機叢集管理員圖示上按一下滑鼠右鍵，然後從捷徑功能表選取 管理本機叢集  ，以開啟 Service Fabric 總管。
   
    ![從通知區域開啟 Service Fabric 總管](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/LaunchSFX.png)
1. 應用程式健全狀況應該會如此圖所示。 此時，應用程式應該狀況良好而沒有任何錯誤。
   
    ![Service Fabric 總管中狀況良好的應用程式](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/sfx-healthy-app.png)
1. 您也可以使用 PowerShell 來檢查健康情況。 您可以使用 ```Get-ServiceFabricApplicationHealth``` 檢查應用程式的健全狀況，而且您可以使用 ```Get-ServiceFabricServiceHealth``` 來檢查服務的健全狀況。 在 PowerShell 中適用於相同應用程式的健全狀況報告位於此圖中。
   
    ![PowerShell 中狀況良好的應用程式](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/ps-healthy-app-report.png)

## <a name="to-add-custom-health-events-to-your-service-code"></a>將自訂健康情況事件新增至您的服務程式碼
Visual Studio 中的 Service Fabric 專案範本包含範例程式碼。 以下步驟說明如何從您的服務程式碼回報自訂健全狀況事件。 這類報告會自動顯示在 Service Fabric 所提供之健康情況監視的標準工具中，例如 Service Fabric Explorer、Azure 入口網站健康情況檢視以及 PowerShell。

1. 在 Visual Studio 中重新開啟您先前建立的應用程式，或使用 **具狀態服務** Visual Studio 範本建立新應用程式。
1. 開啟 Stateful1.cs 檔案，並尋找 `RunAsync` 方法中的 `myDictionary.TryGetValueAsync` 呼叫。 您可以看到這個方法傳回存放目前計數器值的 `result` ，因為此應用程式的主要邏輯是要讓計數能夠執行。 如果這是一個實際的應用程式，且缺乏結果代表失敗，您會想要標示該事件。
1. 若要在缺乏結果代表失敗時回報健全狀況事件，請新增下列步驟。
   
    a. 請將 `System.Fabric.Health` 命名空間新增至 Stateful1.cs 檔案。
   
    ```csharp
    using System.Fabric.Health;
    ```
   
    b. 在 `myDictionary.TryGetValueAsync` 呼叫之後新增下列程式碼
   
    ```csharp
    if (!result.HasValue)
    {
        HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
        this.Partition.ReportReplicaHealth(healthInformation);
    }
    ```
    我們會回報複本健全狀況，因為它是從具狀態服務回報的。 `HealthInformation` 參數會儲存所要回報之健全狀況問題的相關資訊。
   
    如果您已建立無狀態服務，請使用下列程式碼
   
    ```csharp
    if (!result.HasValue)
    {
        HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
        this.Partition.ReportInstanceHealth(healthInformation);
    }
    ```
1. 如果您的服務是以系統管理員權限執行，或者叢集不是[安全的](service-fabric-cluster-security.md)，您也可以使用 `FabricClient` 來回報健全狀況，如下列步驟所示。  
   
    a. 在 `var myDictionary` 宣告之後建立 `FabricClient`。
   
    ```csharp
    var fabricClient = new FabricClient(new FabricClientSettings() { HealthReportSendInterval = TimeSpan.FromSeconds(0) });
    ```
   
    b. 在 `myDictionary.TryGetValueAsync` 呼叫之後新增下列程式碼。
   
    ```csharp
    if (!result.HasValue)
    {
       var replicaHealthReport = new StatefulServiceReplicaHealthReport(
            this.Context.PartitionId,
            this.Context.ReplicaId,
            new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error));
        fabricClient.HealthManager.ReportHealth(replicaHealthReport);
    }
    ```
1. 讓我們來模擬此失敗並看它顯示在健康情況監視工具中。 若要模擬此失敗，請將您稍早新增的健全狀況回報程式碼中的第一行標記成註解。 將第一行標記成註解之後，程式碼會看起來如下列範例所示。
   
    ```csharp
    //if(!result.HasValue)
    {
        HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
        this.Partition.ReportReplicaHealth(healthInformation);
    }
    ```
   此程式碼會在每次 `RunAsync` 執行時引發健康情況報告。 在您進行變更之後，請按 **F5** 執行應用程式。
1. 在應用程式執行之後，開啟 [Service Fabric 總管] 來檢查應用程式的健全狀況。 這次，[Service Fabric Explorer] 會將應用程式顯示為健康情況不良。 這是因為我們先前新增的程式碼所回報的錯誤所致。
   
    ![Service Fabric 總管中狀況不良的應用程式](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/sfx-unhealthy-app.png)
1. 如果您在 [Service Fabric 總管] 的樹狀結構檢視中選取主要複本，您將會看到 **健全狀態** 也顯示為發生錯誤。 [Service Fabric 總管] 也會顯示已新增至程式碼中 `HealthInformation` 參數的健全狀況報告詳細資料。 您可以在 PowerShell 和 Azure 入口網站中看到相同的健全狀況報告。
   
    ![Service Fabric 總管中的複本健康情況](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/replica-health-error-report-sfx.png)

此報告會保留在健康情況管理員中，直到被另一份報告取代或直到此複本被刪除為止。 因為我們並未在 `HealthInformation` 物件中設定此健康情況報告的 `TimeToLive`，所以報告永遠不會到期。

建議您應該在最細微的層級上回報健全狀況，在此案例中為複本。 您也可以回報 `Partition`的健全狀況。

```csharp
HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
this.Partition.ReportPartitionHealth(healthInformation);
```

若要回報 `Application`、`DeployedApplication`、`DeployedServicePackage` 的健全狀況，請使用 `CodePackageActivationContext`。

```csharp
HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
var activationContext = FabricRuntime.GetActivationContext();
activationContext.ReportApplicationHealth(healthInformation);
```

## <a name="next-steps"></a>後續步驟
* [深入了解 Service Fabric 健康情況](service-fabric-health-introduction.md)
* [回報服務健全狀況的 REST API](https://docs.microsoft.com/rest/api/servicefabric/report-the-health-of-a-service)
* [回報應用程式健全狀況的 REST API](https://docs.microsoft.com/rest/api/servicefabric/report-the-health-of-an-application)

