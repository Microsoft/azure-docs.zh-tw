---
title: Service Fabric 服務的延展性 | Microsoft Docs
description: 描述如何調整 Service Fabric 服務
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: ed324f23-242f-47b7-af1a-e55c839e7d5d
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 88c563876940da14f75d7ab30aa3f79a8f6dd870
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34209367"
---
# <a name="scaling-in-service-fabric"></a>Service Fabric 中的縮放比例
Azure Service Fabric 可管理叢集節點上的服務、分割區和複本，讓您輕鬆建置可調整的應用程式。 在相同硬體上執行許多工作負載時，會獲得最大資源使用量，還可以針對您選擇調整工作負載的方式提供彈性。 這個 Channel 9 影片說明如何建置可調整的微服務應用程式：

> [!VIDEO https://channel9.msdn.com/Events/Connect/2017/T116/player]

Service Fabric 的縮放比例會以數種不同的方式來完成：

1. 透過建立或移除無狀態服務執行個體進行縮放比例
2. 透過建立或移除新的具名服務進行縮放比例
3. 透過建立或移除新的具名應用程式執行個體進行縮放比例
4. 使用分割區的服務進行縮放比例
5. 透過新增和移除叢集中的節點進行縮放比例 
6. 使用叢集 Resource Manager 計量進行縮放比例

## <a name="scaling-by-creating-or-removing-stateless-service-instances"></a>透過建立或移除無狀態服務執行個體進行縮放比例
在 Service Fabric 內進行縮放的其中一種最簡單方式可適用於無狀態服務。 當您建立無狀態服務時，有機會可定義 `InstanceCount`。 `InstanceCount` 會定義服務啟動時要建立幾份該服務程式碼的執行中複本。 比方說，叢集中有 100 個節點。 另外再假設建立服務時，`InstanceCount` 為 10。 在執行階段期間，程式碼的 10 個執行中複本可能全部會變得太忙碌 (或可能不是足夠忙碌)。 調整該工作負載的其中一種方式是變更執行個體數目。 例如，監視或管理的程式碼有某些部分可以將現有的執行個體數目變更為 50 (或變更為 5)，取決於是否需要以負載作為基礎，將工作負載相應縮小或相應放大。 

C#：

```csharp
StatelessServiceUpdateDescription updateDescription = new StatelessServiceUpdateDescription(); 
updateDescription.InstanceCount = 50;
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/app/service"), updateDescription);
```

PowerShell：

```posh
Update-ServiceFabricService -Stateless -ServiceName $serviceName -InstanceCount 50
```
### <a name="using-dynamic-instance-count"></a>使用動態執行個體計數
特別針對無狀態服務，Service Fabric 會提供變更執行個體計數的自動方式。 這可讓服務使用可用的節點數目進行動態縮放。 選擇此行為的方式，是設定執行個體計數 = -1。 InstanceCount = -1 是 Service Fabric 的指示，顯示「在每個節點上執行此無狀態服務」。 如果節點的數目有所變更，Service Fabric 就會自動變更要比對的執行個體計數，從而確保服務是在所有的有效節點上執行。 

C#：

```csharp
StatelessServiceDescription serviceDescription = new StatelessServiceDescription();
//Set other service properties necessary for creation....
serviceDescription.InstanceCount = -1;
await fc.ServiceManager.CreateServiceAsync(serviceDescription);
```

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName -Stateless -PartitionSchemeSingleton -InstanceCount "-1"
```

## <a name="scaling-by-creating-or-removing-new-named-services"></a>透過建立或移除新的具名服務進行縮放比例
具名服務執行個體是一些叢集中具名應用程式執行個體內的服務類型之特定執行個體 (請參閱 [Service Fabric 應用程式生命週期](service-fabric-application-lifecycle.md))。 

可以在服務的忙碌程度提高或降低時，建立 (或移除) 新的具名服務執行個體。 這樣可讓要求散佈到多個服務執行個體，通常會使現有服務上的負載降低。 在建立服務時，Service Fabric 叢集 Resource Manager 會以分散式方式將服務置於叢集中。 確切決策是依叢集和其他放置規則中的[計量](service-fabric-cluster-resource-manager-metrics.md)來控管。 可以數個不同的方式來建立服務，但最常見的方式是透過系統管理動作 (如某人呼叫[`New-ServiceFabricService`](https://docs.microsoft.com/powershell/module/servicefabric/new-servicefabricservice?view=azureservicefabricps)，或藉由程式碼呼叫[`CreateServiceAsync`](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync?view=azure-dotnet))。 甚至可以從叢集中執行的其他服務內呼叫 `CreateServiceAsync`。

各種情節中皆可使用動態建立服務，且是常見的模式。 例如，假設有一個可代表特定工作流程的具狀態服務。 會向這項服務顯示代表工作的呼叫，且此服務會對該工作流程和資料列進度執行這些步驟。 

您會如何進行此特定的服務規模？ 服務可能是某種形式的多租用戶，且會接受呼叫並可一次開始進行相同工作流程中許多不同執行個體的所有步驟。 不過，這樣會讓程式碼更複雜，因為現在必須擔心相同工作流程的許多不同執行個體，全都是不同階段及來自不同客戶。 此外，同時處理多個工作流程並無法解決縮放問題。 這是因為在某個時間點，此服務需耗用太多資源才能符合特定的電腦。 起初未建立此模式的許多服務，也會因程式碼中的某些固有瓶頸或速度變慢而遇到困難。 這類問題還會在它所追蹤的同時工作流程數目變大時，造成服務無法運作。  

解決方案是針對您需要追蹤之工作流程的每個不同執行個體，建立此服務的執行個體。這是絕佳的模式，而無論服務是無狀態或具狀態皆可運作。 若要運作此模式，通常有另一項服務可作為「工作負載管理員服務」。 這項服務的作業會接收要求，並將這些要求路由至其他服務。 管理員可以在其接收訊息時動態建立工作負載服務的執行個體，然後再將要求傳遞至這些服務。 當指定的工作流程服務完成其作業時，管理員服務可能也會收到回呼。 管理員收到這些回呼時，它可以刪除工作流程服務的執行個體，或是如果有多個呼叫，就會將它保留。 

此類型管理員的進階版本甚至可以建立其所管理的服務集區。 集區可協助確保在新的要求進入時，不必等待服務就能加速。 反之，管理員可以只從集區或路由中隨機挑選目前非忙碌中的工作流程服務。 將服務集區保持可用能夠加速處理新的要求，因為要求比較不需要等候新服務就能加速。 建立新的服務很快速，但並非免費或可瞬間完成。 集區有助於降低要求在接受服務之前所要等待的時間。 在回應時間至關重要的情況下，您將會經常看到這個管理員和集區模式。 將要求佇列並在背景中建立服務_然後_加以傳遞，也是常用的管理員模式，如同以該服務目前已暫止的某些追蹤工作量作為基礎，將服務建立和刪除。 

## <a name="scaling-by-creating-or-removing-new-named-application-instances"></a>透過建立或移除新的具名應用程式執行個體進行縮放比例
建立和刪除整個應用程式執行個體類似於建立和刪除服務的模式。 對於此模式，有一些管理員服務會以它所查看的要求，以及它從叢集內其他服務接收的資訊作為基礎來制定決策。 

何時應該使用建立新的具名應用程式執行個體，而不是在某些現有的應用程式中建立新的具名服務執行個體？ 還有一些案例：

  * 新的應用程式執行個體適用於程式碼必須在某些特定身分識別或安全性設定下才能執行的客戶。
    * Service Fabric 可讓您定義不同的程式碼套件，以在特定的身分識別下執行。 若要以不同的身分識別啟動相同的程式碼套件，必須在不同的應用程式執行個體中進行啟用。 試想一個案例，您已部署現有客戶的工作負載。 這些可能會以特定的身分識別執行，所以您可以監視及控制它們存取其他資源，例如遠端資料庫或其他系統。 在此情況下，當新客戶註冊時，您可能不需要在相同的內容 (流程空間) 中啟動其程式碼。 雖然您可以這樣做，但這會讓您的服務程式碼更難以在特定身分識別的內容中行動。 您通常必須有更高的安全性、隔離及身分識別管理程式碼。 您並非在相同的應用程式執行個體而從此在相同的流程空間內使用不同的具名服務執行個體，而是可以使用不同的具名 Service Fabric 應用程式執行個體。 這可讓您更輕鬆地定義不同的身分識別內容。
  * 新的應用程式執行個體也可作為一種組態
    * 根據預設，應用程式執行個體內所有特定服務類型的具名服務執行個體，都會在指定節點的相同流程中執行。 這表示，儘管您可以使用不同的方式設定每個服務執行個體，但這麼做會很複雜。 服務必須有一些權杖，以供他們用來查閱其組態套件內的組態。 通常這只是服務的名稱。 這可正常運作，但它會將組態與該應用程式執行個體內的個別具名服務執行個體的名稱結合。 這可能會造成混淆且難以管理，因為組態通常是包含應用程式執行個體之特定值的設計階段成品。 建立更多服務一律表示多個應用程式升級，可變更組態套件內的資訊，或可部署新的讓新的組態套件，讓新的服務可以查閱其特定資訊。 要建立全新的具名應用程式執行個體通常較為容易。 然後您可以使用應用程式參數來設定服務所需的任何必要設定。 如此一來，該具名應用程式執行個體內建立的所有服務都可以繼承特定的組態設定。 比方說，您並非擁有包含每位客戶之設定和自訂的單一組態檔，例如祕密或每個客戶的資源限制，而是在覆寫這些設定的情況下針對每個客戶擁有不同的應用程式執行個體。 
  * 新的應用程式會作為升級的界限
    * 在 Service Fabric 內，不同的具名應用程式執行個體會作為升級的界限。 一個具名應用程式執行個體的升級不會影響另一個具名應用程式執行個體正在執行的程式碼。 不同的應用程式最終會在相同節點上執行相同程式碼的不同版本。 當您需要進行縮放比例的決策時，這可能會成為一個因素，因為您可以選擇新的程式碼是否應遵循與另一個服務相同的升級。 例如，假設呼叫抵達負責縮放特定客戶工作負載比例的管理員服務，方法是以動態方式將服務建立或刪除。 不過，在此情況下，呼叫是針對與_新_客戶相關聯的工作負載。 大部分的客戶偏好彼此隔離，不只是基於先前所列出的安全性和設定理由，還因為它針對執行特定版本的軟體，以及在它們升級時進行選擇方面，能提供更多的彈性。 您也可以建立新的應用程式執行個體並就地建立服務，從而進一步將任何一個升級會觸及的服務數量進行分割。 個別的應用程式執行個體能在進行應用程式升級時提供更大的細微度，還可進行 A/B 測試和藍色/綠色部署。 
  * 現有的應用程式執行個體已滿
    * 在 Service Fabric 中，[應用程式容量](service-fabric-cluster-resource-manager-application-groups.md)這個概念，可用來控制特定應用程式執行個體可用的資源數量。 例如，您可能會決定指定的服務必須建立另一個執行個體才能進行縮放比例。 不過，此應用程式執行個體已超出特定計量的容量。 如果仍需授與更多資源給此特定客戶或工作負載，您可以增加該應用程式的現有容量或建立新的應用程式。 

## <a name="scaling-at-the-partition-level"></a>調整資料分割層級
Service Fabric 支援資料分割。 分割區會將服務分割成數個邏輯與實體區段，其中每一個都是獨立運作。 使用具狀態服務時這會很有用，因為沒有任何一組複本必須處理所有的呼叫，以及一次操作所有的狀態。 [資料分割概觀](service-fabric-concepts-partitioning.md) 提供所支援資料分割配置類型的相關資訊。 每個分割區的複本會分散在叢集中的節點、發佈該服務的負載，並確定服務整體或任何分割區都有單一個失敗點。 

假設有一個使用範圍資料分割配置的服務，其中低索引鍵為 0、高索引鍵為 99、資料分割計數為 4。 在三個節點叢集中，服務可能使用四個複本進行配置，並共用每個節點上的資源，如下所示：

<center>
![使用三個節點分割配置](./media/service-fabric-concepts-scalability/layout-three-nodes.png)
</center>

如果您增加節點數目，Service Fabric 會將一些現有的複本移至那裡。 例如，假設節點的數目增加到四個，且重新發佈複本。 現在，服務有三個複本在每個節點上執行，每個都屬於不同的分割區。 這可提升資源使用率，因為新的節點並不陌生。 通常，它也會改善效能，因為每個服務都有較多資源可供其使用。

<center>
![使用四個節點分割配置](./media/service-fabric-concepts-scalability/layout-four-nodes.png)
</center>

## <a name="scaling-by-using-the-service-fabric-cluster-resource-manager-and-metrics"></a>使用 Service Fabric 叢集 Resource Manager 和計量進行縮放比例
[計量](service-fabric-cluster-resource-manager-metrics.md)是服務表示其對 Service Fabric 資源耗用量的方式。 使用計量可讓叢集 Resource Manager 有機會將叢集的配置重新組織並最佳化。 例如，在叢集中可能有充分的資源，但可能不會配置於目前正在工作的服務。 使用計量可讓叢集 Resource Manager 重新組織叢集，從而確認服務可存取可用的資源。 


## <a name="scaling-by-adding-and-removing-nodes-from-the-cluster"></a>透過新增和移除叢集中的節點進行縮放比例 
使用 Service Fabric 縮放比例的另一個選項是變更叢集的大小。 變更叢集大小表示新增或移除叢集中一個或多個節點類型的節點。 例如，試想一個情況，叢集中所有的節點都會作用。 這表示叢集的資源幾乎全部耗用。 在此情況下，縮放比例的最佳方式就是將多個節點新增到叢集。 一旦新的節點加入叢集，Service Fabric 叢集 Service Fabric 會將服務移動至這些節點，使現有節點上的總負載較低。 針對執行個體計數 = -1 的無狀態服務，會自動建立多個服務執行個體。 這可讓某些呼叫從現有的節點移至新的節點。 

如需詳細資訊，請參閱[叢集調整](service-fabric-cluster-scaling.md)。

## <a name="putting-it-all-together"></a>總整理
讓我們根據這裡討論的所有概念來探討範例。 假設下列服務︰您嘗試建置一個服務作為通訊錄，以儲存名稱和連絡人資訊。 

就在最前面位置有許多關於縮放比例的問題：您將有多少使用者？ 每個使用者會儲存多少連絡人？ 在第一次建立服務時，很難完全釐清這些資料。 例如，假設您要搭配單一靜態服務與特定分割區計數。 挑選錯誤的資料分割計數會導致以後調整時發生問題。 同樣地，即使您挑選適當的計數，可能還是無法取得您所需要的所有資訊。 比方說，您也必須領先決定叢集的大小，包括節點的數目和其大小等方面。 通常很難預測服務在其存留期內會耗用多少資源。 可能也很難事先了解服務實際上看到的流量模式。 比方說，或許人員只是在早上先新增和移除他們的連絡人，或它可能在過去一天內平均地發佈。 您可能需要以這個作為基礎來動態相應放大或相應縮小。 您可能要學習預測何時需要相應放大或相應縮小，但無論如何，您可能必須回應變更您的服務所耗用的資源。 這可能會涉及到變更叢集大小，以在重新組織使用現有的資源不足時能提供更多資源。 

但為什麼要嘗試為所有使用者挑選單一資料分割配置？ 為何要將自己限制於一個服務以及一個靜態叢集？ 實際的狀況通常更為動態。 

在針對縮放進行建置時，請考慮下列動態模式。 您可能需要縮放來適應您的情況：

1. 建置「服務管理員」，而不要嘗試事先為每個人挑選資料分割配置。
2. 當客戶註冊您的服務時，管理員服務就負責查看他們的資訊。 然後，根據這項資訊而定，管理員服務會建立「只針對該客戶」建立「實際」連絡人儲存體服務的執行個體。 如果他們需要特定組態、隔離或升級，您也可以決定針對此客戶加速應用程式執行個體。 

此動態建立模式的多項優點：

  - 您不必事先為所有使用者猜測正確的分割區計數，或建置它自己全都要無限縮放的單一服務。 
  - 不同的使用者不需要有相同的分割區計數、複本計數、放置限制、計量、預設負載、服務名稱、DNS 設定，或是服務或應用程式層級中指定的任何其他屬性。 
  - 您可以取得其他的資料區段。 每個客戶都有自己的服務複本
    - 每個客戶服務可以有不同的設定方式並授與較多或較少資源，以預期的縮放作為基礎，視需要使用較多或較少的分割區或複本。
      - 例如，假設客戶付費使用「黃金」層次 - 他們可以取得更多複本或更高的分割區計數，且可能會透過計量和應用程式容量取得其服務專用的資源。
      - 或者，假設他們提供資訊指出所需的連絡人數目「很少」- 他們只會取得少數分割區，或甚至可能與其他客戶一起放在共用服務集區內。
  - 在等待客戶出現時，您沒有執行一堆服務執行個體或複本
  - 一旦客戶離開，就如要求管理員刪除它所建立的服務，即可從您的服務或應用程式中移除客戶的資訊，如此簡單。

## <a name="next-steps"></a>後續步驟
如需有關 Service Fabric 概念的詳細資訊，請參閱下列文章：

* [Service Fabric 服務的可用性](service-fabric-availability-services.md)
* [分割 Service Fabric 服務](service-fabric-concepts-partitioning.md)
