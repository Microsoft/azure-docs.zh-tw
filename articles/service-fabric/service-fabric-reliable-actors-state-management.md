---
title: Reliable Actors 狀態管理 | Microsoft Docs
description: 說明如何管理、保存及且複寫 Reliable Actors 狀態以提供高可用性。
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
ms.date: 11/02/2017
ms.author: vturecek
ms.openlocfilehash: 3cab4d87eacc7bce17da64cda213086c262179a8
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34206193"
---
# <a name="reliable-actors-state-management"></a>Reliable Actors 狀態管理
Reliable Actors 是可封裝邏輯和狀態的單一執行緒物件。 由於動作項目會在 Reliable Services 上執行，因此，它們可以利用相同的持續性和複寫機制，以可靠的方式維護狀態。 如此一來，動作項目就不會在失敗之後、在記憶體回收之後重新啟動，或者因為資源平衡和升級的緣故而在叢集中的節點之間移動時，遺失它們的狀態。

## <a name="state-persistence-and-replication"></a>狀態持續性和複寫
所有的 Reliable Actors 會被視為「具狀態」  ，因為每個動作項目執行個體都會對應到唯一的識別碼。 這表示，對同一個動作項目識別碼所進行的重複呼叫會路由傳送到同一個動作項目執行個體。 相較之下，在無狀態系統中，用戶端呼叫不一定每次都會路由傳送到同一部伺服器。 基於這個理由，動作項目服務永遠都是具狀態服務。

即使動作項目會被視為具狀態，但並不表示它們必須以可靠的方式儲存狀態。 動作項目可以根據其資料儲存體需求來選擇狀態持續性和複寫的層級︰

* **保存的狀態**︰狀態會保存於磁碟，並複寫至 3 個以上的複本。 保存的狀態是最持久的狀態儲存選項，可透過完整的叢集中斷來保存狀態。
* **變動性狀態**︰狀態會複寫至 3 個以上的複本，而且只會保存於記憶體中。 變動性狀態可針對節點失敗和動作項目失敗，以及在升級和資源平衡期間提供恢復能力。 不過，狀態不會保存到磁碟。 因此，如果同時遺失所有複本，狀態也會遺失。
* **沒有保存的狀態**：狀態並未複寫或寫入至磁碟，僅用於不需要確實維護狀態的動作項目。

每個層級的持續性只是您服務的不同「狀態供應器」和「複寫」組態。 是否要將狀態寫入磁碟取決於「狀態供應器」(Reliable Service 中儲存狀態的元件)。 複寫取決於要使用多少個複本來部署服務。 就如同 Reliable Services，您可以輕鬆地手動設定狀態供應器和複本計數。 動作項目架構提供屬性，在動作項目上使用時，會自動選取預設的狀態供應器，並自動產生複本計數的設定，以達到這三個持續性設定的其中一個。 衍生的類別不會繼承 StatePersistence 屬性，每個 Actor 類型必須提供其 StatePersistence 層級。

### <a name="persisted-state"></a>保存的狀態
```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl  extends FabricActor implements MyActor
{
}
```  
此設定會使用狀態供應器，在磁碟上儲存資料，並自動將服務複本計數設定為 3。

### <a name="volatile-state"></a>變動性狀態
```csharp
[StatePersistence(StatePersistence.Volatile)]
class MyActor : Actor, IMyActor
{
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Volatile)
class MyActorImpl extends FabricActor implements MyActor
{
}
```
此設定會使用僅在記憶體中的狀態供應器，並將複本計數設定為 3。

### <a name="no-persisted-state"></a>沒有保存的狀態
```csharp
[StatePersistence(StatePersistence.None)]
class MyActor : Actor, IMyActor
{
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.None)
class MyActorImpl extends FabricActor implements MyActor
{
}
```
此設定會使用僅在記憶體中的狀態供應器，並將複本計數設定為 1。

### <a name="defaults-and-generated-settings"></a>預設值和產生的設定
您使用 `StatePersistence` 屬性時，在動作項目服務啟動時，會在執行階段自動為您選取狀態供應器。 不過，複本計數是在編譯時期由 Visual Studio 動作項目建置工具所設定。 建置工具會在 ApplicationManifest.xml 中自動為動作項目服務產生「預設服務」。 參數是針對「複本集大小下限」和「目標複本集大小」建立。

您可以手動變更這些參數。 不過，每當 `StatePersistence` 屬性變更時，參數會設定為所選 `StatePersistence` 屬性的預設複本集大小值，並覆寫所有舊值。 換句話說，您在 ServiceManifest.xml 中設定的值將*只*會在您變更 `StatePersistence` 屬性值時，於建置階段覆寫。

```xml
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application12Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Parameters>
      <Parameter Name="MyActorService_PartitionCount" DefaultValue="10" />
      <Parameter Name="MyActorService_MinReplicaSetSize" DefaultValue="3" />
      <Parameter Name="MyActorService_TargetReplicaSetSize" DefaultValue="3" />
   </Parameters>
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="MyActorPkg" ServiceManifestVersion="1.0.0" />
   </ServiceManifestImport>
   <DefaultServices>
      <Service Name="MyActorService" GeneratedIdRef="77d965dc-85fb-488c-bd06-c6c1fe29d593|Persisted">
         <StatefulService ServiceTypeName="MyActorServiceType" TargetReplicaSetSize="[MyActorService_TargetReplicaSetSize]" MinReplicaSetSize="[MyActorService_MinReplicaSetSize]">
            <UniformInt64Partition PartitionCount="[MyActorService_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
         </StatefulService>
      </Service>
   </DefaultServices>
</ApplicationManifest>
```

## <a name="state-manager"></a>狀態管理員
每個動作項目執行個體都有它自己的狀態管理員︰以可靠方式儲存金鑰-值組的字典式資料結構。 狀態管理員是包住狀態供應器的包裝函式。 您可以使用它來儲存的資料，不論哪一個會使用持續性設定。 它不保證執行中的動作項目服務可以透過輪流升級從變動性 (僅在記憶體中) 狀態設定變更保存的狀態設定，同時保留資料。 但是，針對執行中的服務變更複本計數是可行的。

狀態管理員索引鍵必須是字串。 值是泛型值，而且可以是任何類型，包括自訂類型。 儲存在狀態管理員中的值必須是可進行資料合約序列化的，因為根據動作項目的狀態持續性設定，它們可能會在複寫期間透過網路傳輸至其他節點，而且可能會寫入磁碟。

狀態管理員會公開一般字典方法來管理狀態，類似於在可靠的字典中找到的項目。

如需管理動作項目狀態的範例，請參閱[存取、儲存和移除 Reliable Actors](service-fabric-reliable-actors-access-save-remove-state.md) (英文)。

## <a name="best-practices"></a>最佳作法
以下是管理動作項目狀態的一些建議做法和疑難排解祕訣。

### <a name="make-the-actor-state-as-granular-as-possible"></a>儘可能細分動作項目狀態
這對於應用程式的效能和資源使用量而言很重要。 當動作項目的「具名狀態」有任何寫入/更新時，對應至該「具名狀態」的整個值會序列化，並透過網路傳送到次要複本。  次要複本會將它寫入至本機磁碟，並回覆回到主要複本。 當主要複本收到來自次要複本仲裁的通知時，它會將狀態寫入到其本機磁碟。 例如，假設此值是有 20 個成員且大小為 1 MB 的類別。 即使您只修改其中一個大小為 1 KB 的類別成員，您最終會在序列化和網路及磁碟寫入上付出整個 1 MB 的代價。 同樣地，如果此值為集合 (例如清單、陣列或字典)，即使您只修改它的其中一個成員，您仍需付出完整集合的成本。 動作項目類別的 StateManager 介面就像字典。 您一律應在此字典上建立資料結構的模型，以代表動作項目的狀態。
 
### <a name="correctly-manage-the-actors-life-cycle"></a>正確管理動作項目的生命週期
您應該有關於管理動作項目服務的每個資料分割中狀態大小的清除原則。 動作項目服務應該有固定的動作項目數目，且儘可能重複使用它們。 如果您持續建立新的動作項目，則必須在它們完成其工作後加以刪除。 動作項目架構會儲存有關每個現有動作項目的一些中繼資料。 刪除動作項目的所有狀態，並不會移除有關該動作項目的中繼資料。 您必須刪除動作項目 (請參閱[刪除動作項目及其狀態](service-fabric-reliable-actors-lifecycle.md#manually-deleting-actors-and-their-state))，以移除儲存在系統中有關它的所有資訊。 有一項額外檢查：您應該偶爾查詢動作項目服務 (請參閱[列舉動作項目](service-fabric-reliable-actors-platform.md))，以確保動作項目數目在預期的範圍內。
 
如果您看到動作項目服務的資料庫檔案大小增加超過預期的大小，請確定您遵循上述的指導方針。 如果您遵循這些指導方針，但仍有資料庫檔案大小問題，您應該[開啟支援票證](service-fabric-support.md)，向產品小組尋求協助。

## <a name="next-steps"></a>後續步驟

儲存在 Reliable Actors 中的狀態必須先經過序列化，才能寫入到磁碟中並進行複寫來提供高可用性。 深入了解[動作項目類型序列化](service-fabric-reliable-actors-notes-on-actor-type-serialization.md)。

接著，深入了解[動作項目診斷與效能監視](service-fabric-reliable-actors-diagnostics.md)。
