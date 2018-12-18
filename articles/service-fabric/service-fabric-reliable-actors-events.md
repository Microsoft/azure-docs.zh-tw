---
title: 動作項目型 Azure Service Fabric 動作項目中的事件 | Microsoft Docs
description: Service Fabric Reliable Actor 事件簡介。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: aa01b0f7-8f88-403a-bfe1-5aba00312c24
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/06/2017
ms.author: amanbha
ms.openlocfilehash: c228821383a1bfedf380f97e3411fdacc322a6f9
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44054473"
---
# <a name="actor-events"></a>動作項目事件
動作項目事件會將最佳效果通知從動作項目傳送到用戶端。 動作項目事件是為了動作項目與用戶端之間的通訊而設計，不應用於動作項目與動作項目之間的通訊。

下列程式碼片段顯示如何在應用程式中使用動作項目事件。

定義描述動作項目所發佈事件的介面。 此介面必須衍生自 `IActorEvents` 介面。 方法的引數必須是 [資料合約序列化](service-fabric-reliable-actors-notes-on-actor-type-serialization.md)。 當事件通知是單向且為最佳效果時，方法必須傳回無效。

```csharp
public interface IGameEvents : IActorEvents
{
    void GameScoreUpdated(Guid gameId, string currentScore);
}
```
```Java
public interface GameEvents implements ActorEvents
{
    void gameScoreUpdated(UUID gameId, String currentScore);
}
```
宣告由動作項目介面中動作項目發佈的事件。

```csharp
public interface IGameActor : IActor, IActorEventPublisher<IGameEvents>
{
    Task UpdateGameStatus(GameStatus status);

    Task<string> GetGameScore();
}
```
```Java
public interface GameActor extends Actor, ActorEventPublisherE<GameEvents>
{
    CompletableFuture<?> updateGameStatus(GameStatus status);

    CompletableFuture<String> getGameScore();
}
```
在用戶端上，實作事件處理常式。

```csharp
class GameEventsHandler : IGameEvents
{
    public void GameScoreUpdated(Guid gameId, string currentScore)
    {
        Console.WriteLine(@"Updates: Game: {0}, Score: {1}", gameId, currentScore);
    }
}
```

```Java
class GameEventsHandler implements GameEvents {
    public void gameScoreUpdated(UUID gameId, String currentScore)
    {
        System.out.println("Updates: Game: "+gameId+" ,Score: "+currentScore);
    }
}
```

在用戶端上，對發佈事件的動作項目建立 Proxy，並訂閱其事件。

```csharp
var proxy = ActorProxy.Create<IGameActor>(
                    new ActorId(Guid.Parse(arg)), ApplicationName);

await proxy.SubscribeAsync<IGameEvents>(new GameEventsHandler());
```

```Java
GameActor actorProxy = ActorProxyBase.create<GameActor>(GameActor.class, new ActorId(UUID.fromString(args)));

return ActorProxyEventUtility.subscribeAsync(actorProxy, new GameEventsHandler());
```

發生容錯移轉時，動作項目會容錯移轉至不同的程序或節點。 動作項目 Proxy 會管理使用中的訂用帳戶，並自動重新訂閱。 您可以透過 `ActorProxyEventExtensions.SubscribeAsync<TEvent>` API 控制重新訂閱間隔。 若要取消訂閱，請使用 `ActorProxyEventExtensions.UnsubscribeAsync<TEvent>` API。

在動作項目上，當事件發生時發佈事件。 如果有訂閱者訂閱事件，動作項目執行階段會將事件傳送至通知。

```csharp
var ev = GetEvent<IGameEvents>();
ev.GameScoreUpdated(Id.GetGuidId(), score);
```
```Java
GameEvents event = getEvent<GameEvents>(GameEvents.class);
event.gameScoreUpdated(Id.getUUIDId(), score);
```


## <a name="next-steps"></a>後續步驟
* [動作項目重新進入](service-fabric-reliable-actors-reentrancy.md)
* [動作項目診斷與效能監視](service-fabric-reliable-actors-diagnostics.md)
* [動作項目 API 參考文件](https://msdn.microsoft.com/library/azure/dn971626.aspx)
* [C# 範例程式碼 (英文)](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [C# .NET Core 範例程式碼](https://github.com/Azure-Samples/service-fabric-dotnet-core-getting-started)
* [Java 範例程式碼 (英文)](http://github.com/Azure-Samples/service-fabric-java-getting-started)
