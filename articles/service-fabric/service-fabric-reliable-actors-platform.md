---
title: Service Fabric 上的 Reliable Actors | Microsoft Docs
description: 說明 Reliable Actors 如何在 Reliable Services 上分層，以及如何使用 Service Fabric 平台的功能。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: amanbha
ms.assetid: 45839a7f-0536-46f1-ae2b-8ba3556407fb
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 3/9/2018
ms.author: vturecek
ms.openlocfilehash: f8e6ad4b23eeaf46cccac9c8ff9d41f71511129d
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34642847"
---
# <a name="how-reliable-actors-use-the-service-fabric-platform"></a>Reliable Acto 如何使用 Service Fabric 平台
本文說明 Reliable Actors 在 Azure Service Fabric 平台上的運作方式。 Reliable Actors 會在名為*動作項目服務*的具狀態可靠服務實作上裝載的架構中執行。 動作項目服務包含管理生命週期和您的動作項目用於發送之訊息所需的所有元件︰

* 動作項目執行階段會管理生命週期、記憶體回收，並強制執行單一執行緒存取。
* 動作項目服務遠端處理接聽程式會接受對動作項目進行的遠端存取呼叫，並將它們傳送到發送器，以路由傳送到適當的動作項目執行個體。
* 動作項目狀態供應器會包裝狀態供應器 (例如「可靠的集合」狀態供應器)，並提供配接器來進行動作項目狀態管理。

這些元件一起構成 Reliable Actor 架構。

## <a name="service-layering"></a>將服務分層
由於動作項目服務本身是一個 Reliable Service，因此 Reliable Services 的所有 [應用程式模型](service-fabric-application-model.md)、生命週期、[封裝](service-fabric-package-apps.md)、[部署](service-fabric-deploy-remove-applications.md)、升級及調整概念都會以相同方式套用到動作項目服務。

![將動作項目服務分層][1]

上圖顯示 Service Fabric 應用程式架構和使用者程式碼之間的關聯性。 藍色項目代表 Reliable Services 應用程式架構、橘色代表 Reliable Actor 架構，而綠色代表使用者程式碼。

在 Reliable Services 中，您的服務會繼承 `StatefulService` 類別。 這個類別本身是衍生自 `StatefulServiceBase` (或無狀態服務的 `StatelessService`)。 在 Reliable Actors 中，您會使用動作項目服務。 動作項目服務是 `StatefulServiceBase` 類別的不同實作，此類別會實作動作項目執行所在的動作項目模式。 由於動作項目服務本身只是 `StatefulServiceBase` 的實作，因此，您可以自行撰寫衍生自 `ActorService` 的服務，並以您在繼承 `StatefulService` 時所使用的相同方式來實作服務層級功能，例如︰

* 服務備份與還原。
* 適用於所有動作項目的共用功能，例如斷路器。
* 遠端處理程序會在動作項目服務本身，以及每個個別動作項目上進行呼叫。

如需詳細資訊，請參閱[在動作項目服務中實作服務層級功能](service-fabric-reliable-actors-using.md)。

## <a name="application-model"></a>應用程式模型
動作項目服務是 Reliable Services，因此應用程式模型是一樣的。 不過，動作項目架構建置工具可為您產生某些應用程式模型檔案。

### <a name="service-manifest"></a>服務資訊清單
動作項目架構建置工具會自動產生動作項目服務的 ServiceManifest.xml 檔案內容。 此檔案包括：

* 動作項目服務類型。 類型名稱是根據您的動作項目專案名稱來產生。 根據動作項目上的持續性屬性，也會據以設定 HasPersistedState 旗標。
* 程式碼封裝。
* 組態封裝。
* 資源和端點。

### <a name="application-manifest"></a>應用程式資訊清單
動作項目架構建置工具會自動建立適用於您動作項目服務的預設服務定義。 建置工具會填入預設服務屬性︰

* 複本集數量取決於動作項目上的持續性屬性。 每當動作項目上的持續性屬性變更時，預設服務定義中的複本集數量會據以重設。
* 分割區配置和範圍會利用完整的 Int64 索引鍵範圍設定為「平均的 Int64 」。

## <a name="service-fabric-partition-concepts-for-actors"></a>動作項目的 Service Fabric 資料分割概念
動作項目服務是已資料分割的具狀態服務。 動作項目服務的每個分割區都包含一組動作項目。 服務分割區會自動散佈於 Service Fabric 中的多個節點上。 結果就是散佈動作項目執行個體。

![動作項目的資料分割和散佈][5]

您可以使用不同的分割區配置和分割區索引鍵範圍來建立 Reliable Services。 動作項目服務會搭配完整的 Int64 索引鍵範圍使用 Int64 資料分割配置，來將動作項目對應到分割區。

### <a name="actor-id"></a>動作項目識別碼
服務中建立的每個動作項目都具有與它相關聯的唯一識別碼，由 `ActorId` 類別來表示。 `ActorId` 是不透明的識別碼值，可藉由產生隨機識別碼，在服務分割區上平均散佈動作項目。

```csharp
ActorProxy.Create<IMyActor>(ActorId.CreateRandom());
```
```Java
ActorProxyBase.create<MyActor>(MyActor.class, ActorId.newId());
```


每個 `ActorId` 雜湊為 int64。 這就是為什麼動作項目服務必須搭配完整的 Int64 索引鍵範圍使用 Int64 資料分割配置的原因。 不過，自訂識別碼值可用於 `ActorID`，包括 GUID/UUID、字串和 Int64。

```csharp
ActorProxy.Create<IMyActor>(new ActorId(Guid.NewGuid()));
ActorProxy.Create<IMyActor>(new ActorId("myActorId"));
ActorProxy.Create<IMyActor>(new ActorId(1234));
```
```Java
ActorProxyBase.create(MyActor.class, new ActorId(UUID.randomUUID()));
ActorProxyBase.create(MyActor.class, new ActorId("myActorId"));
ActorProxyBase.create(MyActor.class, new ActorId(1234));
```

您使用 GUID/UUID 和字串時，已將值雜湊為 Int64。 不過，在您將 Int64 明確提供給 `ActorId`之前，Int64 將會直接對應到分割區，而不需進一步雜湊。 您可以使用這項技術控制要將動作項目放置於哪個分割區。


## <a name="next-steps"></a>後續步驟
* [動作項目狀態管理](service-fabric-reliable-actors-state-management.md)
* [動作項目生命週期與記憶體回收](service-fabric-reliable-actors-lifecycle.md)
* [動作項目 API 參考文件](https://msdn.microsoft.com/library/azure/dn971626.aspx)
* [.NET 範例程式碼 (英文)](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Java 範例程式碼 (英文)](http://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-platform/actor-service.png
[2]: ./media/service-fabric-reliable-actors-platform/app-deployment-scripts.png
[3]: ./media/service-fabric-reliable-actors-platform/actor-partition-info.png
[4]: ./media/service-fabric-reliable-actors-platform/actor-replica-role.png
[5]: ./media/service-fabric-reliable-actors-introduction/distribution.png
