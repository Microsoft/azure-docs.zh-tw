---
title: Azure Service Fabric 動作項目生命週期的概觀 | Microsoft Docs
description: 說明 Service Fabric Reliable Actor 生命週期、記憶體回收，以及手動刪除動作項目與其狀態
services: service-fabric
documentationcenter: .net
author: amanbha
manager: timlt
editor: vturecek
ms.assetid: b91384cc-804c-49d6-a6cb-f3f3d7d65a8e
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/06/2017
ms.author: amanbha
ms.openlocfilehash: dbd9551027744d443613e32e0a082c10d4f357d5
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44052041"
---
# <a name="actor-lifecycle-automatic-garbage-collection-and-manual-delete"></a>動作項目生命週期、自動記憶體回收，以及手動刪除
第一次呼叫動作項目的方法時就會啟動動作項目。 如果有一段可設定的時間未使用動作項目，動作項目就會停用 (由動作項目執行階段進行記憶體回收)。 動作項目與其狀態也可以隨時進行手動刪除。

## <a name="actor-activation"></a>啟用動作項目
啟用動作項目後，會發生下列情況︰

* 當動作項目有呼叫卻尚未為使用中時，會建立新的動作項目。
* 載入動作項目狀態 (如果正在維護狀態)
* 會呼叫 `OnActivateAsync` (C#) 或 `onActivateAsync` (Java) 方法 (可在動作項目實作中被覆寫)。
* 動作項目現在被視為作用中。

## <a name="actor-deactivation"></a>停用動作項目
停用動作項目後，會發生下列情況︰

* 動作項目若一段時間未使用，便會從使用中動作項目資料表移除。
* 會呼叫 `OnDeactivateAsync` (C#) 或 `onDeactivateAsync` (Java) 方法 (可在動作項目實作中被覆寫)。 這會清除動作項目的所有計時器。 您不應該從此方法呼叫動作項目作業 (例如狀態變更)。

> [!TIP]
> 網狀架構動作項目的執行階段會發出某些[與動作項目啟用和停用相關的事件](service-fabric-reliable-actors-diagnostics.md#list-of-events-and-performance-counters)。 這些項目對於診斷與效能監視很有幫助。
>
>

### <a name="actor-garbage-collection"></a>動作項目記憶體回收
停用動作項目後，動作項目物件的參考會釋出，而且通常可由 Common Language Runtime (CLR) 或 Java 虛擬機器 (JVM) 記憶體回收行程進行記憶體回收。 記憶體回收只會清除動作項目物件；它 **不會** 移除動作項目的狀態管理員中儲存的狀態。 下次啟用動作項目時，會建立新的動作項目物件並還原其狀態。

就停用和記憶體回收的用途而言，什麼算是「一直在使用中」？

* 一直收到呼叫
* `IRemindable.ReceiveReminderAsync` 方法 (僅適用於動作項目使用提醒時)。

> [!NOTE]
> 如果動作項目使用計時器，且已叫用其計時器回撥，則 **不** 算是「一直使用中」。
>
>

在進入停用的細節前，最重要的是定義下列詞彙：

* *掃描間隔*。 這是動作項目執行階段掃描其作用中動作項目資料表中，是否有動作項目可予以停用和進行記憶體回收的間隔。 預設值為 1 分鐘。
* *閒置逾時*。 這是動作項目維持未使用 (閒置) 所需的時間長度，過此時間後即可停用和進行記憶體回收。 預設值為 60 分鐘。

通常不需要變更這些預設值。 不過，如有必要，可以在註冊[動作項目服務](service-fabric-reliable-actors-platform.md)時透過 `ActorServiceSettings` 變更這些間隔：

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        ActorRuntime.RegisterActorAsync<MyActor>((context, actorType) =>
                new ActorService(context, actorType,
                    settings:
                        new ActorServiceSettings()
                        {
                            ActorGarbageCollectionSettings =
                                new ActorGarbageCollectionSettings(10, 2)
                        }))
            .GetAwaiter()
            .GetResult();
    }
}
```

```Java
public class Program
{
    public static void main(String[] args)
    {
        ActorRuntime.registerActorAsync(
                MyActor.class,
                (context, actorTypeInfo) -> new FabricActorService(context, actorTypeInfo),
                timeout);
    }
}
```
對於每個作用中動作項目，動作項目執行階段會持續追蹤動作項目已閒置 (亦即未使用) 的時間。 動作項目執行階段每隔 `ScanIntervalInSeconds` 就會檢查每個動作項目，查看其是否可進行記憶體回收，如果已閒置 `IdleTimeoutInSeconds`，就會將其回收。

只要使用動作項目，其閒置時間就會重設為 0。 在此之後，只有當動作項目再次閒置達 `IdleTimeoutInSeconds`時，才會將動作項目作為記憶體回收。 請回想一下，當動作項目介面方法或動作項目提醒回撥執行時，動作項目會視為已使用。 如果動作項目的計時器回撥執行時， **不會** 將動作項目視為已使用。

下圖顯示單一動作項目的生命週期來說明下列概念。

![閒置時間的範例][1]

範例將說明動作項目方法呼叫、提醒，以及此動作項目存留期之計時器的影響。 範例中有幾下幾點值得注意：

* ScanInterval 及 IdleTimeout 分別設為 5 和 10。 (單位並不重要，因為我們的目的只為了說明概念)。
* 系統會依照掃描間隔為 5 的定義，在 T=0、5、10、15、20、25 時掃描是否有可作為記憶體回收的動作項目。
* 定期計時器會在 T=4、8、12、16、20、24 時引發，並執行其回呼。 這不會影響動作項目的閒置時間。
* 在 T=7 的動作項目方法呼叫會將閒置時間重設為 0，並延遲動作項目的記憶體回收。
* 動作項目提醒回撥會在 T=14 執行，並進一步延遲動作項目的廢棄項目收集。
* 在 T=25 的記憶體回收期間，動作項目的閒置時間最後會超過為 10 的閒置逾時，並會將動作項目作為記憶體回收。

動作項目正在執行其中一個方法時，無論在執行該方法時花費了多久的時間，動作項目絕對不會進行記憶體回收。 如先前所述，執行動作項目介面方法和提醒回撥會將動作項目的閒置時間重設為 0，來防止廢棄項目收集。 執行計時器回撥不會將閒置時間重設為 0。 不過，計時器回撥完成執行之前，會延遲動作項目的廢棄項目收集。

## <a name="manually-deleting-actors-and-their-state"></a>手動刪除動作項目及其狀態
已停用動作項目的記憶體回收只會清除動作項目物件；但不會移除動作項目的狀態管理員中儲存的資料。 重新啟用動作項目後，會再次透過狀態管理員提供其資料。 在動作項目將資料儲存於狀態管理員後停用，而永遠不會重新啟用的情況下，可能需要清除其資料。  如需如何刪除執行者的範例，請參閱[刪除執行者及其狀態](service-fabric-reliable-actors-delete-actors.md)。

## <a name="next-steps"></a>後續步驟
* [動作項目計時器和提醒](service-fabric-reliable-actors-timers-reminders.md)
* [動作項目事件](service-fabric-reliable-actors-events.md)
* [動作項目重新進入](service-fabric-reliable-actors-reentrancy.md)
* [動作項目診斷與效能監視](service-fabric-reliable-actors-diagnostics.md)
* [動作項目 API 參考文件](https://msdn.microsoft.com/library/azure/dn971626.aspx)
* [C# 範例程式碼 (英文)](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Java 範例程式碼 (英文)](http://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-lifecycle/garbage-collection.png
