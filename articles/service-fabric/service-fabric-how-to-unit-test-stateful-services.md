---
title: 為 Azure Service Fabric 中的具狀態服務開發單元測試 | Microsoft Docs
description: 說明如何為 Service Fabric 具狀態服務開發單元測試。
services: service-fabric
documentationcenter: .net
author: charleszipp
manager: timlt
editor: vturecek
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/04/2018
ms.author: ryanwi
ms.openlocfilehash: 945cdf63a178a09f121f355aaa7635537e46e5ff
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43703315"
---
# <a name="create-unit-tests-for-stateful-services"></a>建立具狀態服務的單元測試
對 Service Fabric 具狀態服務進行單元測試，可找出傳統應用程式或特定領域單元測試不一定能找到的常見錯誤。 為具狀態服務開發單元測試時，必須留意某些特殊考量。

1. 每個複本都會執行應用程式的程式碼，但會依據不同的內容。 如果服務使用三個複本，則服務程式碼會在不同的內容/角色下以平行方式對三個節點執行。
2. 在所有複本間，儲存於具狀態服務內的狀態應是一致的。 狀態管理員和可靠的集合依預設會提供此一致性。 不過，記憶體內部狀態將必須由應用程式的程式碼管理。
3. 每個複本在叢集上執行時，將會於某個時間點變更角色。 當裝載主要複本的節點無法使用或超過負載時，次要複本將成為主要複本。 這是 Service Fabric 的正常行為，因此服務必須要有最終會以不同角色執行的計劃。

本文假設您已閱讀[對 Service Fabric 中的具狀態服務進行單元測試](service-fabric-concepts-unit-testing.md)。

## <a name="the-servicefabricmocks-library"></a>ServiceFabric.Mocks 程式庫
從 3.3.0 版開始，[ServiceFabric.Mocks](https://www.nuget.org/packages/ServiceFabric.Mocks/) 將提供可模擬複本和狀態管理協調流程的 API。 我們將在範例中使用此 API。

[Nuget](https://www.nuget.org/packages/ServiceFabric.Mocks/)
[Github](https://github.com/loekd/ServiceFabric.Mocks)

*ServiceFabric.Mocks 並非由 Microsoft 所擁有或維護。不過，這是 Microsoft 目前針對具狀態服務的單元測試建議使用的程式庫。*

## <a name="set-up-the-mock-orchestration-and-state"></a>設定模擬協調流程和狀態
在編排測試的過程中，將會建立模擬複本集和狀態管理員。 其後，複本集將自行為每個複本建立受測服務的執行個體。 它也將自行執行生命週期事件，例如 `OnChangeRole` 和 `RunAsync`。 模擬狀態管理員會確保對狀態管理員執行的任何作業，都會比照實際狀態管理員的方式執行和保存。

1. 建立會將正在測試的服務具現化的服務處理站委派。 此委派通常會與 Service Fabric 服務或動作項目的 `Program.cs` 中所存放的服務處理站回呼相類似或相同。 它應遵循下列簽章：
```csharp
MyStatefulService CreateMyStatefulService(StatefulServiceContext context, IReliableStateManagerReplica2 stateManager)
```
2. 建立 `MockReliableStateManager` 類別的執行個體。 這樣會模擬所有與狀態管理員的互動。
3. 建立 `MockStatefulServiceReplicaSet<TStatefulService>` 的執行個體，其中，`TStatefulService` 是正在測試的服務類型。 這將需要在步驟 1 中建立的委派，和步驟 2 中具現化的狀態管理員
4. 將複本新增至複本集。 指定角色 (例如 Primary、ActiveSecondary、IdleSecondary) 和複本的識別碼
> 請妥善保存複本識別碼！ 單元測試的執行和判斷提示部分將可能用到這些識別碼。

```csharp
//service factory to instruct how to create the service instance
var serviceFactory = (StatefulServiceContext context, IReliableStateManagerReplica2 stateManager) => new MyStatefulService(context, stateManager);
//instantiate a new mock state manager
var stateManager = new MockReliableStateManager();
//instantiate a new replica set with the service factory and state manager
var replicaSet = new MockStatefulServiceReplicaSet<MyStatefulService>(CreateStatefulService, stateManager);
//add a new Primary replica with id 1
await replicaSet.AddReplicaAsync(ReplicaRole.Primary, 1);
//add a new ActiveSecondary replica with id 2
await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 2);
//add a second ActiveSecondary replica with id 3
await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 3);
```

## <a name="execute-service-requests"></a>執行服務要求
您可以使用方便的屬性和查閱對特定複本執行服務要求。
```csharp
const string stateName = "test";
var payload = new Payload(StatePayload);

//execute a request on the primary replica using
await replicaSet.Primary.ServiceInstance.InsertAsync(stateName, payload);

//execute a request against replica with id 2
await replicaSet[2].ServiceInstance.InsertAsync(stateName, payload);

//execute a request against one of the active secondary replicas
await replicaSet.FirstActiveSecondary.InsertAsync(stateName, payload);
```

## <a name="execute-a-service-move"></a>執行服務移動
模擬複本集中會公開數個便利的方法，用來觸發不同類型的服務移動。
```csharp
//promote the first active secondary to primary
replicaSet.PromoteNewReplicaToPrimaryAsync();
//promote the secondary with replica id 4 to primary
replicaSet.PromoteNewReplicaToPrimaryAsync(4);

//promote the first idle secondary to an active secondary
PromoteIdleSecondaryToActiveSecondaryAsync();
//promote idle secodary with replica id 4 to active secondary 
PromoteIdleSecondaryToActiveSecondaryAsync(4);

//add a new replica with randomly assigned replica id and promote it to primary
PromoteNewReplicaToPrimaryAsync()
//add a new replica with replica id 4 and promote it to primary
PromoteNewReplicaToPrimaryAsync(4)
```

## <a name="putting-it-all-together"></a>總整理
下列測試示範如何設定三個節點的複本集，並驗證在角色變更後可從次要節點取得資料。 這項測試可找出的常見問題之一，是在 `InsertAsync` 期間新增的資料是儲存到記憶體內部的項目，還是儲存到可靠的集合而未執行 `CommitAsync`。 無論是何者，次要節點都會與主要節點不同步。 這會在服務移動後導致回應不一致的狀況。

```csharp
[TestMethod]
public async Task TestServiceState_InMemoryState_PromoteActiveSecondary()
{
    var stateManager = new MockReliableStateManager();
    var replicaSet = new MockStatefulServiceReplicaSet<MyStatefulService>(CreateStatefulService, stateManager);
    await replicaSet.AddReplicaAsync(ReplicaRole.Primary, 1);
    await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 2);
    await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 3);

    const string stateName = "test";
    var payload = new Payload(StatePayload);

    //insert data
    await replicaSet.Primary.ServiceInstance.InsertAsync(stateName, payload);
    //promote one of the secondaries to primary
    await replicaSet.PromoteActiveSecondaryToPrimaryAsync(2);
    //get data
    var payloads = (await replicaSet.Primary.ServiceInstance.GetPayloadsAsync()).ToList();

    //data should match what was inserted against the primary
    Assert.IsTrue(payloads.Count == 1);
    Assert.IsTrue(payloads[0].Content == payload.Content);

    //verify the data was saved against the reliable dictionary
    var dictionary = await StateManager.GetOrAddAsync<IReliableDictionary<string, Payload>>(MyStatefulService.StateManagerDictionaryKey);
    using(var tx = StateManager.CreateTransaction())
    {
        var payload = await dictionary.TryGetValue(stateName);
        Assert.IsTrue(payload.HasValue);
        Assert.IsTrue(payload.Value.Content == payload.Content);
    }
}
```

## <a name="next-steps"></a>後續步驟
了解如何測試[服務對服務通訊](service-fabric-testability-scenarios-service-communication.md)和[使用受控的混亂模擬失敗](service-fabric-controlled-chaos.md)。