---
title: 管理 Azure Service Fabric 狀態 | Microsoft Docs
description: 瞭解如何存取、儲存、移除 Service Fabric Reliable Actors 狀態。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: 37cf466a-5293-44c0-a4e0-037e5d292214
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/19/2018
ms.author: vturecek
ms.openlocfilehash: ac3afe144b9cf9e2fb307087edb175a603ffe4e9
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
---
# <a name="access-save-and-remove-reliable-actors-state"></a>存取、儲存、移除 Reliable Actors 狀態
[Reliable Actors](service-fabric-reliable-actors-introduction.md) 是可封裝邏輯和狀態並穩定維持狀態的單一執行緒物件。 每個動作項目執行個體都有自己的[狀態管理員](service-fabric-reliable-actors-state-management.md)︰以可靠方式儲存索引鍵/值組的字典式資料結構。 狀態管理員是包住狀態供應器的包裝函式。 可以用來儲存資料，無論所用的是何種[持續性設定](service-fabric-reliable-actors-state-management.md#state-persistence-and-replication)。

狀態管理員索引鍵必須是字串。 值是泛型值，而且可以是任何類型，包括自訂類型。 儲存在狀態管理員中的值必須是可進行資料合約序列化的，因為根據動作項目的狀態持續性設定，它們可能會在複寫期間透過網路傳輸至其他節點，而且可能會寫入磁碟。

狀態管理員會公開一般字典方法來管理狀態，類似於在可靠的字典中找到的項目。

如需相關資訊，請參閱[管理動作項目狀態的最佳做法](service-fabric-reliable-actors-state-management.md#best-practices)。

## <a name="access-state"></a>存取狀態
狀態可以透過狀態管理員依索引鍵來存取。 狀態管理員方法全都是非同步的，因為當動作項目具有保存的狀態時，它們可能需要磁碟 I/O。 第一次存取時，會將狀態物件快取於記憶體中。 重複存取作業會從記憶體中直接存取物件並以同步方式傳回，而不會造成磁碟 I/O 或非同步內容切換的負擔。 狀態物件會在下列情況中從快取移除︰

* 從狀態管理員中擷取物件之後，動作項目方法會擲回未處理的例外狀況。
* 動作項目會在已停用之後或因為失敗而重新啟動。
* 狀態供應器會將狀態分頁到磁碟。 這個行為取決於狀態供應器實作。 `Persisted` 設定的預設狀態供應器具有這個行為。

如果索引鍵的項目不存在，您可以使用會擲回 `KeyNotFoundException`(C#) 或 `NoSuchElementException`(Java) 的標準 *Get* 作業來擷取狀態：

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<int> GetCountAsync()
    {
        return this.StateManager.GetStateAsync<int>("MyState");
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture<Integer> getCountAsync()
    {
        return this.stateManager().getStateAsync("MyState");
    }
}
```

如果索引鍵的項目不存在，您也可以使用不會擲回任何項目的 *TryGet* 方法來擷取狀態：

```csharp
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task<int> GetCountAsync()
    {
        ConditionalValue<int> result = await this.StateManager.TryGetStateAsync<int>("MyState");
        if (result.HasValue)
        {
            return result.Value;
        }

        return 0;
    }
}
```
```Java
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture<Integer> getCountAsync()
    {
        return this.stateManager().<Integer>tryGetStateAsync("MyState").thenApply(result -> {
            if (result.hasValue()) {
                return result.getValue();
            } else {
                return 0;
            });
    }
}
```

## <a name="save-state"></a>儲存狀態
狀態管理員擷取方法會傳回本機記憶體中物件的參考。 在本機記憶體中單獨修改此物件並不會永久儲存該物件。 從狀態管理員中擷取並修改物件時，必須將它重新插入狀態管理員，才能永久儲存。

您可以使用無條件的 *Set* 來插入狀態，這相當於 `dictionary["key"] = value` 語法︰

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task SetCountAsync(int value)
    {
        return this.StateManager.SetStateAsync<int>("MyState", value);
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture setCountAsync(int value)
    {
        return this.stateManager().setStateAsync("MyState", value);
    }
}
```

您可以使用 *Add* 方法來新增狀態。 這個方法會在它新增已存在的索引鍵時擲回 `InvalidOperationException`(C#) 或 `IllegalStateException`(Java)。

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task AddCountAsync(int value)
    {
        return this.StateManager.AddStateAsync<int>("MyState", value);
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture addCountAsync(int value)
    {
        return this.stateManager().addOrUpdateStateAsync("MyState", value, (key, old_value) -> old_value + value);
    }
}
```

您也可以使用 *TryAdd* 方法來新增狀態。 此方法不會在它嘗試新增已存在的索引鍵時擲回任何項目。

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task AddCountAsync(int value)
    {
        bool result = await this.StateManager.TryAddStateAsync<int>("MyState", value);

        if (result)
        {
            // Added successfully!
        }
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture addCountAsync(int value)
    {
        return this.stateManager().tryAddStateAsync("MyState", value).thenApply((result)->{
            if(result)
            {
                // Added successfully!
            }
        });
    }
}
```

在動作項目方法結束時，狀態管理員會自動儲存透過插入或更新作業所新增或修改的任何值。 根據所使用的設定，「儲存」可以包括保存到磁碟與複本。 尚未經過修改的值不會保存或複寫。 如果未修改任何值，儲存作業就不會有任何動作。 如果儲存失敗，將會捨棄已修改的狀態並重新載入原始的狀態。

您也可以呼叫動作項目基底上的 `SaveStateAsync` 方法來手動儲存狀態︰

```csharp
async Task IMyActor.SetCountAsync(int count)
{
    await this.StateManager.AddOrUpdateStateAsync("count", count, (key, value) => count > value ? count : value);

    await this.SaveStateAsync();
}
```
```Java
interface MyActor {
    CompletableFuture setCountAsync(int count)
    {
        this.stateManager().addOrUpdateStateAsync("count", count, (key, value) -> count > value ? count : value).thenApply();

        this.stateManager().saveStateAsync().thenApply();
    }
}
```

## <a name="remove-state"></a>移除狀態
您可以藉由呼叫 *Remove* 方法，從動作項目的狀態管理員中永久移除狀態。 這個方法會在它嘗試移除不存在的索引鍵時擲回 `KeyNotFoundException`(C#) 或 `NoSuchElementException`(Java)。

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task RemoveCountAsync()
    {
        return this.StateManager.RemoveStateAsync("MyState");
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture removeCountAsync()
    {
        return this.stateManager().removeStateAsync("MyState");
    }
}
```

您也可以使用 *TryRemove* 方法永久移除狀態。 此方法不會在它嘗試移除不存在的索引鍵時擲回任何項目。

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task RemoveCountAsync()
    {
        bool result = await this.StateManager.TryRemoveStateAsync("MyState");

        if (result)
        {
            // State removed!
        }
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture removeCountAsync()
    {
        return this.stateManager().tryRemoveStateAsync("MyState").thenApply((result)->{
            if(result)
            {
                // State removed!
            }
        });
    }
}
```

## <a name="next-steps"></a>後續步驟

儲存在 Reliable Actors 中的狀態必須先經過序列化，才能寫入到磁碟中並進行複寫來提供高可用性。 深入了解[動作項目類型序列化](service-fabric-reliable-actors-notes-on-actor-type-serialization.md)。

接著，深入了解[動作項目診斷與效能監視](service-fabric-reliable-actors-diagnostics.md)。
