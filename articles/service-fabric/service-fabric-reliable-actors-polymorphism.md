---
title: Reliable Actors 架構中的多型 | Microsoft Docs
description: 在 Reliable Actors 架構中建置 .NET 介面和類型的階層，以重複使用功能和 API 定義。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: vturecek
ms.assetid: ef0eeff6-32b7-410d-ac69-87cba8b8fd46
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/02/2017
ms.author: vturecek
ms.openlocfilehash: f03330bbbf2c1c7f92310211058f7ef8858f1941
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34209333"
---
# <a name="polymorphism-in-the-reliable-actors-framework"></a>Reliable Actors 架構中的多型
Reliable Actors 架構可讓您使用許多您會在物件導向設計中使用的相同技巧來建置動作項目。 這些技巧的其中之一就是多型，此技巧允許從更一般化的父系繼承類型和介面。 Reliable Actors 架構中的繼承通常會遵循 .NET 模型，但有幾個額外的條件約束。 針對 Java/Linux，它會遵循 Java 模型。

## <a name="interfaces"></a>介面
Reliable Actors 架構會要求您至少定義一個要由動作項目類型實作的介面。 此介面會用來產生可供用戶端用來與您的動作項目進行通訊的 Proxy 類別。 只要動作項目類型及其所有父系所實作的每個介面最終都是衍生自 IActor(C#) 或 Actor(Java)，這些介面便可以從其他介面繼承。 IActor(C#) 和 Actor(Java) 分別是針對 .NET 和 Java 架構中動作項目的平台定義基底介面。 因此，使用圖形的傳統多型範例可能會看起來像這樣：

![圖形動作項目的介面階層][shapes-interface-hierarchy]

## <a name="types"></a>類型
您也可以建立衍生自平台所提供之基底「動作項目」類別的動作項目類型階層。 如果是圖形，您可能會有一個基底 `Shape`(C#) 或 `ShapeImpl`(Java) 類型：

```csharp
public abstract class Shape : Actor, IShape
{
    public abstract Task<int> GetVerticeCount();

    public abstract Task<double> GetAreaAsync();
}
```
```Java
public abstract class ShapeImpl extends FabricActor implements Shape
{
    public abstract CompletableFuture<int> getVerticeCount();

    public abstract CompletableFuture<double> getAreaAsync();
}
```

`Shape`(C#) 或 `ShapeImpl`(Java) 的子類型可以從基底覆寫方法。

```csharp
[ActorService(Name = "Circle")]
[StatePersistence(StatePersistence.Persisted)]
public class Circle : Shape, ICircle
{
    public override Task<int> GetVerticeCount()
    {
        return Task.FromResult(0);
    }

    public override async Task<double> GetAreaAsync()
    {
        CircleState state = await this.StateManager.GetStateAsync<CircleState>("circle");

        return Math.PI *
            state.Radius *
            state.Radius;
    }
}
```
```Java
@ActorServiceAttribute(name = "Circle")
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
public class Circle extends ShapeImpl implements Circle
{
    @Override
    public CompletableFuture<Integer> getVerticeCount()
    {
        return CompletableFuture.completedFuture(0);
    }

    @Override
    public CompletableFuture<Double> getAreaAsync()
    {
        return (this.stateManager().getStateAsync<CircleState>("circle").thenApply(state->{
          return Math.PI * state.radius * state.radius;
        }));
    }
}
```

請注意動作項目類型上的 `ActorService` 屬性。 此屬性會告知 Reliable Actor 架構應該自動建立一個服務來裝載此類型的動作項目。 在某些情況下，您可能會想要建立僅供與子類型共用功能而永遠不用來將具體動作項目具現化的基底類型。 在這些情況下，您應該使用 `abstract` 關鍵字來指出您永遠不會根據該類型建立動作項目。

## <a name="next-steps"></a>後續步驟
* 請參閱 [Reliable Actors 架構如何利用 Service Fabric 平台](service-fabric-reliable-actors-platform.md) 以提供可靠性、延展性及一致的狀態。
* 了解 [動作項目生命週期](service-fabric-reliable-actors-lifecycle.md)。

<!-- Image references -->

[shapes-interface-hierarchy]: ./media/service-fabric-reliable-actors-polymorphism/Shapes-Interface-Hierarchy.png
