---
title: Service Fabric Reliable Actors 概觀 | Microsoft Docs
description: Service Fabric Reliable Actors 程式設計模型簡介。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: 7fdad07f-f2d6-4c74-804d-e0d56131f060
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/01/2017
ms.author: vturecek
ms.openlocfilehash: d0de7e7596fb7fabbc4061709816111a7a4ad827
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34213039"
---
# <a name="introduction-to-service-fabric-reliable-actors"></a>Service Fabric Reliable Actor 簡介
Reliable Actors 是以 [Virtual Actor](http://research.microsoft.com/en-us/projects/orleans/) 模式為基礎的 Service Fabric 應用程式架構。 Reliable Actors API 提供單一執行緒的程式設計模型，此模型立基於 Service Fabric 所提供的延展性和可靠性保證上。

## <a name="what-are-actors"></a>什麼是動作項目？
動作項目是隔離且獨立的計算與狀態單位，且具備單一執行緒執行。 [動作項目模式](https://en.wikipedia.org/wiki/Actor_model) 是適用於並行或分散式系統的運算模型，其中有大量的這類動作項目可以同時且各自獨立的方式來執行。 動作項目可以彼此通訊，而且可以建立多個動作項目。

### <a name="when-to-use-reliable-actors"></a>使用 Reliable Actors 的時機
Service Fabric Reliable Actors 是動作項目設計模式的實作。 如同任何軟體設計模式，是否要使用特定模式的決策是根據軟體設計問題是否符合模式來決定。

雖然動作項目設計模式適用於許多分散式系統問題和案例，但還是必須謹慎考量實作它的模式與架構的限制。 做為一般指導方針，請在符合下列情況時，考慮使用動作項目模式來建立您的問題或案例模型︰

* 您的問題領域涉及大量 (數千個或更多) 小型、獨立且隔離的狀態和邏輯單位。
* 您想要使用單一執行緒的物件，這類物件不需要與外部元件進行顯著的互動，包括跨一組動作項目查詢狀態。
* 您的動作項目執行個體將不會藉由發出 I/O 作業，使用無法預期的延遲來封鎖呼叫端。

## <a name="actors-in-service-fabric"></a>Service Fabric 中的動作項目
在 Service Fabric 中，動作項目是在 Reliable Actors 架構中實作的︰以動作項目模式為基礎的應用程式架構是建置於 [Service Fabric Reliable Services](service-fabric-reliable-services-introduction.md)的頂端。 您撰寫的每個 Reliable Actor 服務實際上都是已資料分割的具狀態可靠服務。

每個動作項目都會定義為動作項目類型的執行個體，與 .NET 物件是 .NET 類型的執行個體的方式完全相同。 例如，有的動作項目類型會實作計算機的功能，而該類型會有許多動作項目分散到叢集的各種節點上。 每一個這類的動作項目都有唯一的動作項目識別碼識別。

## <a name="actor-lifetime"></a>動作項目生命週期
Service Fabric 動作項目是虛擬的，也就是說，其生命週期不會繫結至其記憶體內部表示法。 因此，不需要明確地進行建立或終結。 Reliable Actors 執行階段會在第一次接收到動作項目識別碼的要求時自動啟動該動作項目。 如果動作項目有一段時間未使用，則 Reliable Actors 執行階段會對記憶體內部物件進行記憶體回收。 它也會維護稍後重新啟動動作項目所需的存在知識。 如需詳細資訊，請參閱 [動作項目生命週期與記憶體回收](service-fabric-reliable-actors-lifecycle.md)。

此虛擬動作項目存留期抽象概念會傳達一些警告做為虛擬動作項目模型的結果，而事實上，Reliable Actors 有時會偏離此模型。

* 動作項目會在訊息第一次傳送到它的動作項目識別碼時自動啟動 (因而造成動作項目物件的建構)。 在一段時間之後，就會對該動作項目物件進行記憶體回收。 未來再次使用動作項目識別碼時，就會導致建構新的動作項目物件。 將動作項目的狀態儲存於狀態管理員時，其存留時間會超過物件的存留期。
* 針對動作項目識別碼呼叫任何動作項目方法都會啟動該動作項目。 基於這個理由，動作項目類型會透過執行階段隱含地呼叫其建構函式。 因此，用戶端程式碼無法將參數傳遞給動作項目類型的建構函式，雖然可能會由服務本身將參數傳遞給動作項目的建構函式。 結果就是，如果動作項目需要來自用戶端的初始化參數，動作項目可能就會依照在其上呼叫其他方法的時間，以部分初始化的狀態來建構。 沒有任何單一進入點可從用戶端啟動動作項目。
* 雖然 Reliable Actors 會以隱含方式建立動作項目物件，但您還是無法明確地刪除動作項目及其狀態。

## <a name="distribution-and-failover"></a>散佈和容錯移轉
為了提供延展性和可靠性，Service Fabric 將動作項目散佈於整個叢集，並視需要讓動作項目從失敗的節點自動移轉到狀況良好的節點。 這是 [已資料分割的具狀態可靠服務](service-fabric-concepts-partitioning.md)上的一個抽象概念。 散佈、延展性、可靠性及自動容錯移轉全都是憑藉動作項目正在名為「動作項目服務」 的具狀態可靠服務內執行的事實來提供。

動作項目會散佈在動作項目服務的分割區上，而這些分割區會散佈到 Service Fabric 叢集中的節點上。 每個服務分割區都會包含一組動作項目。 Service Fabric 會管理服務分割區的散佈和容錯移轉。

例如，若動作項目服務具備九個使用預設動作項目分割區配置部署到三個節點的分割區，則會據此進行散佈：

![Reliable Actors 散佈][2]

動作項目架構會為您管理分割區配置和索引鍵範圍設定。 這會簡化一些選項，但也會帶來一些考量︰

* Reliable Services 可讓您選擇資料分割配置、索引鍵範圍 (使用定界分割配置)，以及分割區計數。 Reliable Actors 會限制為定界分割配置 (平均的 Int64 配置)，而且要求您使用完整的 Int64 索引鍵範圍。
* 根據預設，動作項目會隨機放入分割區，形成平均散佈的結果。
* 由於動作項目是隨機放置的，所以應該預期動作項目作業一律需要網路通訊，包括方法呼叫資料的序列化和還原序列化，因而導致延遲和額外負荷。
* 在進階案例中，使用對應到特定分割區的 Int64 動作項目識別碼，就能控制動作項目分割區放置。 不過，這樣做會導致動作項目以不對稱方式散佈於分割區上。

如需如何分割動作項目服務的詳細資訊，請參閱 [動作項目的分割概念](service-fabric-reliable-actors-platform.md#service-fabric-partition-concepts-for-actors)。

## <a name="actor-communication"></a>動作項目通訊
定義動作項目互動的介面是實作該介面的動作項目所共用的介面，而用戶端會透過同一個介面取得動作項目的 Proxy。 因為此介面是用來以非同步方式叫用動作項目方法，所以介面上的每個方法都必須傳回工作。

方法引動過程及其回應最終會導致叢集中的網路要求，如此一來，引數與其所傳回之工作的結果類型必須可由平台序列化。 特別是，它們必須是 [資料合約序列化](service-fabric-reliable-actors-notes-on-actor-type-serialization.md)。

### <a name="the-actor-proxy"></a>動作項目 Proxy
Reliable Actors 用戶端 API 能夠在動作項目執行個體與動作項目用戶端之間提供通訊。 為了與動作項目通訊，用戶端會建立一個動作項目 Proxy 物件來實作動作項目介面。 用戶端會叫用 Proxy 物件上的方法來與動作項目互動。 動作項目 Proxy 可用於用戶端對動作項目以及動作項目對動作項目的通訊。

```csharp
// Create a randomly distributed actor ID
ActorId actorId = ActorId.CreateRandom();

// This only creates a proxy object, it does not activate an actor or invoke any methods yet.
IMyActor myActor = ActorProxy.Create<IMyActor>(actorId, new Uri("fabric:/MyApp/MyActorService"));

// This will invoke a method on the actor. If an actor with the given ID does not exist, it will be activated by this method call.
await myActor.DoWorkAsync();
```

```java
// Create actor ID with some name
ActorId actorId = new ActorId("Actor1");

// This only creates a proxy object, it does not activate an actor or invoke any methods yet.
MyActor myActor = ActorProxyBase.create(actorId, new URI("fabric:/MyApp/MyActorService"), MyActor.class);

// This will invoke a method on the actor. If an actor with the given ID does not exist, it will be activated by this method call.
myActor.DoWorkAsync().get();
```


請注意，有兩段資訊用於建立動作項目 Proxy 物件：動作項目 ID 與應用程式名稱。 動作項目識別碼可唯一識別動作項目，而應用程式名稱可識別動作項目部署所在的 [Service Fabric 應用程式](service-fabric-reliable-actors-platform.md#application-model) 。

用戶端上的 `ActorProxy`(C#) / `ActorProxyBase`(Java) 類别會執行必要的解決方案，以便依識別碼找到動作項目並開啟與該動作項目通訊的管道。 它也會在通訊失敗和容錯移轉的情況下重新試著尋找動作項目。 因此，訊息傳遞具有下列特性︰

* 最好進行訊息傳遞。
* 動作項目可能收到來自相同用戶端的重複訊息。

## <a name="concurrency"></a>並行
Reliable Actors 執行階段會提供簡單的回合式存取模型來存取動作項目方法。 這表示動作項目物件代碼內永遠只能有一個執行緒為使用中。 回合式存取會大幅簡化並行系統，因為不需要使用同步機制來進行資料存取。 這也表示系統必須針對每個動作項目執行個體的單一執行緒存取本質的特殊考量來設計。

* 單一動作項目執行個體無法一次處理一個以上的要求。 如果預期動作項目執行個體要處理並行要求，可能就會遇到輸送量瓶頸。
* 如果兩個動作項目之間有一個循環要求，同時還會對其中一個動作項目提出外部要求，則動作項目會在彼此間產生死結。 動作項目執行階段將會在動作項目呼叫時自動逾時，並將例外狀況擲回呼叫端，以中斷可能發生的死結狀況。

![Reliable Actors 通訊][3]

### <a name="turn-based-access"></a>回合式存取
一個回合包含完整執行動作項目方法以回應其他動作項目或用戶端的要求，或完整執行 [計時器/提醒](service-fabric-reliable-actors-timers-reminders.md) 回呼。 即使這些方法和回呼非同步，但動作項目執行階段並不使其交錯。 必須完整完成一個回合，才能允許進行下一回合。 換句話說，計時器/提醒回呼目前正在執行的動作項目方法或計時器/提醒回呼必須完整完成，才能對方法或回呼進行新的呼叫。 如果已從方法或回呼傳回執行，且方法或回呼傳回的工作已完成，則會將方法或回呼視為已完成。 值得強調的是，即使是在不同的方法、計時器與回呼之間，仍會遵守回合式並行。

動作項目執行階段會藉由在某回合的開頭取得一個各動作項目鎖定，然後在回合結束時釋放該鎖定，來強制回合式並行。 因此，回合式並行會依各個動作項目強制執行，不會在動作項目之間強制執行。 動作項目方法和計時器/提醒回撥可代表不同的動作項目同時執行。

以下範例說明上述概念。 如果有一個動作項目類型實作兩個非同步方法 (假設為 Method1 與 Method2)，也就是計時器與提醒。 下圖顯示代表這兩個屬於此動作項目類型的動作項目 (ActorId1 與 ActorId2) 執行這些方法與回呼的時間軸範例。

![Reliable Actors 執行階段回合式並行和存取][1]

此圖遵循下列慣例：

* 每一個垂直線顯示代表特定動作項目執行方法或回撥時的邏輯流程。
* 每個垂直線上標示的事件依時間順序發生，新發生的事件排在舊事件下方。
* 時間軸使用不同的顏色來對應不同的動作項目。
* 反白顯示用於指出代表方法或回撥保留各動作項目鎖定的持續期間。

有一些需要考慮的重要事項︰

* 當 Method1 代表 ActorId2 執行以回應用戶端要求 xyz789 時，另一個抵達的用戶端要求 (abc123) 也需要 Method1 由 ActorId2 執行。 不過， *Method1* 的第二次執行會在前一次執行完成後才開始。 同樣的，由 ActorId2 註冊的提醒會在 Method1 正在執行時引發，以回應用戶端要求 xyz789。 提醒回撥只會在 *Method1* 的這兩個執行都完成後才執行。 所有的一切都是因為對 *ActorId2*強制執行回合式並行。
* 同樣地，也會對 ActorId1 強制執行回合式並行，如代表依序發生的 ActorId1 執行 Method1、Method2 和計時器回呼所示。
* 代表 ActorId1 執行 Method1 與代表 ActorId2 執行重疊。 這是因為回合式並行只會在動作項目內強制執行，不會在動作項目間強制執行。
* 在某些方法/回呼執行中，方法/回呼傳回的 `Task`(C#)/`CompletableFuture`(Java) 會在方法傳回後完成。 在某些其他執行中，已在方法/回呼傳回的時間前完成非同步作業。 在這兩種情況下，只有在方法/回呼傳回且完成非同步作業後，才會釋放各個動作項目鎖定。

### <a name="reentrancy"></a>重新進入
動作項目執行階段依預設允許重新進入。 這表示如果 Actor A 的動作項目方法在 Actor B 上呼叫某一個方法，然後反過來在 Actor A 上呼叫另一個方法，則允許執行該方法。 這是因為該方法是同一個邏輯呼叫鏈結內容的一部分。 所有的計時器與提醒呼叫的開頭都是新的邏輯呼叫內容。 如需詳細資訊，請參閱 [Reliable Actors 重新進入](service-fabric-reliable-actors-reentrancy.md) 。

### <a name="scope-of-concurrency-guarantees"></a>並行保證的範圍
動作項目執行階段在控制叫用這些方法的狀況下，提供這些並行保證。 例如，動作項目執行階段會對為回應用戶端要求而進行的方法叫用，以及對計時器與提醒回呼提供這些保證。 然而，如果動作項目程式碼在動作項目執行階段提供的機制之外直接叫用這些方法，則執行階段無法提供任何並行保證。 例如，如果在某個與動作項目方法所傳回的工作不相關聯的工作內容中叫用方法，則執行階段無法提供並行保證。 如果從動作項目自行建立的執行緒叫用方法，則執行階段也無法提供並行保證。 因此，若要執行背景作業，動作項目應使用遵守回合式並行的 [動作項目計時器和動作項目提醒](service-fabric-reliable-actors-timers-reminders.md) 。

## <a name="next-steps"></a>後續步驟
從建置您的第一個 Reliable Actors 服務開始著手：
   * [開始在 .NET 上使用 Reliable Actors](service-fabric-reliable-actors-get-started.md)
   * [開始在 Java 上使用 Reliable Actors](service-fabric-reliable-actors-get-started-java.md)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-introduction/concurrency.png
[2]: ./media/service-fabric-reliable-actors-introduction/distribution.png
[3]: ./media/service-fabric-reliable-actors-introduction/actor-communication.png
