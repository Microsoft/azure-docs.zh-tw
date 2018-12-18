---
title: 在 Azure Service Fabric 上建立動作項目型服務 | Microsoft Docs
description: 了解如何使用 Service Fabric Reliable Actors 在 C# 中建立、偵錯及部署第一個動作項目型服務。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: d4aebe72-1551-4062-b1eb-54d83297f139
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/16/2018
ms.author: vturecek
ms.openlocfilehash: 4ff896902c1a92c244c3bcd147c3daeeb9e49c77
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44054741"
---
# <a name="getting-started-with-reliable-actors"></a>開始使用 Reliable Actors
> [!div class="op_single_selector"]
> * [Windows 上的 C# ](service-fabric-reliable-actors-get-started.md)
> * [在 Linux 上使用 Java](service-fabric-reliable-actors-get-started-java.md)

本文將逐步說明如何在 Visual Studio 中建立一個簡單的 Reliable Actor 應用程式並進行偵錯。 如需 Reliable Actors 的相關詳細資訊，請參閱 [Service Fabric Reliable Actors 簡介](service-fabric-reliable-actors-introduction.md)。

## <a name="prerequisites"></a>必要條件

在開始之前，請確保您的電腦上已設定 Service Fabric 開發環境 (包括 Visual Studio)。 如需詳細資訊，請參閱[如何設定開發環境](service-fabric-get-started.md)。

## <a name="create-a-new-project-in-visual-studio"></a>在 Visual Studio 中建立新專案

以系統管理員身分啟動 Visual Studio 2015 或更新版本，然後建立一個新的 [Service Fabric 應用程式] 專案：

![適用於 Visual Studio 的 Service Fabric 工具 - 新專案][1]

在下一個對話方塊中，選擇 [.Net Core 2.0] 底下的 [動作項目服務] 並輸入服務的名稱。

![Service Fabric 專案範本][5]

建立的專案會顯示下列結構：

![Service Fabric 專案結構][2]

## <a name="examine-the-solution"></a>檢查方案

方案包含三個專案：

* **應用程式專案 (MyApplication)**。 此專案會封裝所有服務以進行部署。 其包含了用於管理應用程式的 ApplicationManifest.xml  與 PowerShell 指令碼。

* **介面專案 (HelloWorld.Interfaces)**。 此專案包含動作項目的介面定義。 動作項目介面可以在任何名稱的任何專案中定義。  介面會定義由動作項目實作和呼叫動作項目之用戶端共用的動作項目合約。  因為用戶端專案可能依存於此，所以在與動作項目實作不同的組件中定義它，通常是有意義的。

* **動作項目服務專案 (HelloWorld)**。 這個專案會定義即將裝載動作項目的 Service Fabric 服務。 它包含動作項目 *HellowWorld.cs* 的實作。 動作項目實作是衍生自基底類型 `Actor` 的類別，它會實作 *MyActor.Interfaces* 專案中所定義的介面。 動作項目類別也必須實作建構函式來接受 `ActorService` 執行個體和 `ActorId`，並將它們傳遞至基底 `Actor` 類別。
    
    此專案也包含 *Program.cs*，它使用 `ActorRuntime.RegisterActorAsync<T>()` 向 Service Fabric 執行階段註冊動作項目類別。 `HelloWorld` 類別已註冊。 加入至專案的任何其他動作項目實作也必須在 `Main()` 方法中註冊。

## <a name="customize-the-helloworld-actor"></a>自訂 HelloWorld 動作項目

專案範本定義 `IHelloWorld` 介面中的某些方法，並在 `HelloWorld` 動作項目實作中實作這些方法。  取代那些方法，讓動作項目服務傳回簡單的 "Hello World" 字串。

在 *HelloWorld.Interfaces* 專案的 *IHelloWorld.cs* 檔案中，取代介面定義，如下所示：

```csharp
public interface IHelloWorld : IActor
{
    Task<string> GetHelloWorldAsync();
}
```

在 **HelloWorld** 專案的 **HelloWorld.cs** 中，取代整個類別定義，如下所示：

```csharp
[StatePersistence(StatePersistence.Persisted)]
internal class HelloWorld : Actor, IHelloWorld
{
    public HelloWorld(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<string> GetHelloWorldAsync()
    {
        return Task.FromResult("Hello from my reliable actor!");
    }
}
```

按 **Ctrl-Shift-B** 以建置專案，並確定所有項目都會進行編譯。

## <a name="add-a-client"></a>新增用戶端

建立一個簡單的主控台應用程式來呼叫動作項目服務。

1. 以滑鼠右鍵按一下 [方案總管] > [新增] > [新增專案] 中的方案。

2. 在 [.NET Core] 專案類型下，選擇 [主控台應用程式 (.NET Core)]。  將專案命名為 *ActorClient*。
    
    ![[新增專案] 對話方塊][6]    
    
    > [!NOTE]
    > 主控台應用程式不是您通常作為 Service Fabric 中之用戶端使用的應用程式類型，但是它是使用本機 Service Fabric 叢集進行偵錯和測試的便捷範例。

3. 主控台應用程式必須是 64 位元應用程式，才能維持與介面專案和其他相依性的相容性。  在 [方案總管] 中，以滑鼠右鍵按一下 [ActorClient] 專案，然後按一下 [屬性]。  在 [組建] 索引標籤上，將 [平台目標] 設定為 [x64]。
    
    ![組建屬性][8]

4. 用戶端專案需要可靠的動作項目 NuGet 套件。  按一下 [工具] > [NuGet 套件管理員] > [套件管理員主控台]。  在 [套件管理器主控台] 中，輸入下列命令：
    
    ```powershell
    Install-Package Microsoft.ServiceFabric.Actors -IncludePrerelease -ProjectName ActorClient
    ```

    NuGet 套件及其所有相依性都安裝在 ActorClient 專案中。

5. 用戶端專案也需要介面專案的參考。  在 ActorClient 專案中，以滑鼠右鍵按一下 [相依性]，然後按一下 [加入參考]。選取 [專案] > [方案] (如果尚未選取)，然後選取 **HelloWorld.Interfaces** 旁的核取方塊。  按一下 [確定]。
    
    ![[加入參考] 對話方塊][7]

6. 在 ActorClient 專案中，使用下列程式碼取代 *Program.cs* 的全部內容：
    
    ```csharp
    using System;
    using System.Threading.Tasks;
    using Microsoft.ServiceFabric.Actors;
    using Microsoft.ServiceFabric.Actors.Client;
    using HelloWorld.Interfaces;
    
    namespace ActorClient
    {
        class Program
        {
            static void Main(string[] args)
            {
                IHelloWorld actor = ActorProxy.Create<IHelloWorld>(ActorId.CreateRandom(), new Uri("fabric:/MyApplication/HelloWorldActorService"));
                Task<string> retval = actor.GetHelloWorldAsync();
                Console.Write(retval.Result);
                Console.ReadLine();
            }
        }
    }
    ```

## <a name="running-and-debugging"></a>執行和偵錯

按 **F5** 在本機建置、部署和執行 Service Fabric 開發叢集中的應用程式。  在部署的過程中，您可在 [輸出]  視窗中查看進度。

![Service Fabric 偵錯輸出視窗][3]

當輸出包含文字「應用程式已準備好」時，便可使用 ActorClient 應用程式測試此服務。  在 [方案總管] 中，以滑鼠右鍵按一下 **ActorClient** 專案，然後按一下 [偵錯] > [開始新執行個體]。  命令列應用程式應該會顯示動作項目服務的輸出。

![應用程式輸出][9]

> [!TIP]
> Service Fabric Actors 執行階段會發出某些 [事件和與動作項目方法相關的效能計數器](service-fabric-reliable-actors-diagnostics.md#actor-method-events-and-performance-counters)。 這些項目對於診斷與效能監視很有幫助。

## <a name="next-steps"></a>後續步驟
深入了解 [Reliable Actor 如何使用 Service Fabric 平台](service-fabric-reliable-actors-platform.md)。


[1]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject.PNG
[2]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-projectstructure.PNG
[3]: ./media/service-fabric-reliable-actors-get-started/debugging-output.PNG
[4]: ./media/service-fabric-reliable-actors-get-started/vs-context-menu.png
[5]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject1.PNG
[6]: ./media/service-fabric-reliable-actors-get-started/new-console-app.png
[7]: ./media/service-fabric-reliable-actors-get-started/add-reference.png
[8]: ./media/service-fabric-reliable-actors-get-started/build-props.png
[9]: ./media/service-fabric-reliable-actors-get-started/app-output.png