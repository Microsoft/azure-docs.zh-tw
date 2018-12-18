---
title: Service Fabric Reliable Services 程式設計模型概觀 | Microsoft Docs
description: 深入了解 Service Fabric 可靠的服務的程式設計模型，並開始撰寫自己的服務。
services: Service-Fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: vturecek; mani-ramaswamy
ms.assetid: 0c88a533-73f8-4ae1-a939-67d17456ac06
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 3/9/2018
ms.author: masnider
ms.openlocfilehash: 474cc78a4ceb872742ca7eb10837eeb89dcc1bdb
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
---
# <a name="reliable-services-overview"></a>Reliable Services 概觀
Azure Service Fabric 可簡化撰寫和管理無狀態與具狀態的 Reliable Services。 本主題涵蓋︰

* 無狀態和具狀態之服務的 Reliable Services 程式設計模型。
* 撰寫 Reliable Services 時必須進行的選擇。
* Reliable Services 使用時機及撰寫方式的一些案例和範例。

可靠的服務是 Service Fabric 上可用的其中一種程式設計模型。 另一個是 Reliable Actor 程式設計模型，提供一個以 Reliable Services 模型為基礎的虛擬「執行者」程式設計模型。 如需 Reliable Actors 程式設計模型的詳細資訊，請參閱 [Service Fabric Reliable Actors 簡介](service-fabric-reliable-actors-introduction.md)。

Service Fabric 透過 [Service Fabric 應用程式管理](service-fabric-deploy-remove-applications.md)管理服務的存留期間，從佈建和部署一直到升級和刪除。

## <a name="what-are-reliable-services"></a>什麼是 Reliable Services？
Reliable Service 提供您簡易、功能強大、最高階的程式設計模型，以協助您針對應用程式的重要項目進行表達。 使用 Reliable Services 程式設計模型，您會得到：

* 存取其餘的 Service Fabric 程式設計 API。 不同於模型化為[來賓可執行檔](service-fabric-guest-executables-introduction.md)的 Service Fabric 服務，Reliable Services 會直接使用其餘的 Service Fabric API。 如此可讓服務︰
  * 查詢系統
  * 報告有關叢集中之實體的健全狀況
  * 接收有關設定和程式碼變更的通知
  * 尋找其他服務而與之通訊，
  * (選擇性) 使用[可靠的集合](service-fabric-reliable-services-reliable-collections.md)
  * ...讓它們存取許多其他功能，全都來自一流的程式設計模型，且支援多種程式設計語言。
* 類似您所習慣使用之程式設計模型的簡單模型，以便執行您自己的程式碼。 您的程式碼會具有定義完善的切入點和容易管理的生命週期。
* 隨插即用的通訊模型。 使用您選擇的傳輸方式，例如 HTTP 搭配 [Web API](service-fabric-reliable-services-communication-webapi.md)、WebSockets、自訂 TCP 通訊協定等。 Reliable Services 提供一些很棒的現成選項供您使用，或者您可以提供您自己的選項。
* 對於具狀態服務，Reliable Services 程式設計模型可讓您使用[可靠的集合](service-fabric-reliable-services-reliable-collections.md)，直接在服務內以一致且可靠地方式儲存狀態。 可靠的集合是一組簡單的高可用性和可靠的集合類別，用過 C# 集合的任何人都會很熟悉。 傳統上，服務需要外部系統來進行可靠狀態管理。 有了可靠的集合，您可以將狀態儲存在您的計算旁邊，並且具有您預期高度可用的外部存放區會具備的相同高可用性和可靠性。 此模型也可以改善延遲，因為計算及其運作所需的狀態都在一起。

觀看此 Microsoft Virtual Academy 影片以取得 Reliable Services 概觀︰<center>
<a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=HhD9566yC_4106218965">
<img src="./media/service-fabric-reliable-services-introduction/ReliableServicesVid.png" WIDTH="360" HEIGHT="244" />
</a>
</center>

## <a name="what-makes-reliable-services-different"></a>Reliable Services 有什麼不同之處？
Service Fabric 中的可靠的服務與您之前撰寫的服務不同。 Service Fabric 提供可靠性、可用性、一致性和延展性。

* **可靠性** - 即使在不可靠的環境中，您的電腦故障或發生網路問題，或在服務本身遇到錯誤及損毀或失敗的情況下，您的服務仍然可以運作。 對於具狀態服務，即使發生網路或其他失敗，也會保留您的狀態。
* **可用性** - 您的服務可供存取且有回應。 Service Fabric 會維護您所需的執行中複本數目。
* **延展性** - 服務與特定硬體分離，可以視需要透過加入或移除硬體或其他資源而成長或壓縮。 服務可以輕鬆分割 (特別是在具狀態的情況下)，以確保服務可以調整和處理局部失敗。 服務可以透過程式碼來動態建立和刪除，以依需要啟動更多執行個體，例如為了回應客戶要求。 最後，Service Fabric 建議使用輕量型的服務。 Service Fabric 允許在單一處理序內佈建數千個服務，而不需要或讓整個作業系統的執行個體或處理序專用於服務的單一執行個體。
* **一致性** - 此服務中儲存任何資訊保證一致。 即使在服務內跨越多個可靠的集合也一樣。 利用交易不可部分完成的方式，即可在服務內跨集合進行變更。

## <a name="service-lifecycle"></a>服務生命週期
無論您的服務是具狀態還是無狀態，Reliable Services 會提供簡單的生命週期，可讓您快速插入您的程式碼，並開始著手。  您只需要實作一個或兩個方法，即可讓您的服務啟動並執行。

* **CreateServiceReplicaListeners/CreateServiceInstanceListeners** - 服務在此方法中定義它想要使用的通訊堆疊。 通訊堆疊 (例如 [Web API](service-fabric-reliable-services-communication-webapi.md)) 定義服務的一或多個接聽端點 (用戶端如何存取服務)。 它也定義出現的訊息如何與服務程式碼的其餘部分互動。
* **RunAsync** - 服務在此方法中執行其商務邏輯，也在此處啟動應該在服務存留期執行的任何背景工作。 所提供的取消語彙基元是針對該工作何時應該停止的訊號。 比方說，如果服務需要從可靠的佇列提取訊息並加以處理，這就是執行這項工作的地方。

如果您是第一次來了解 Reliable Services，請繼續閱讀！ 如果想要尋找 Reliable Services 留存期的詳細逐步解說，您可以閱讀[這篇文章](service-fabric-reliable-services-lifecycle.md)。

## <a name="example-services"></a>範例服務
了解這個程式設計模型之後，讓我們來快速看看兩個不同的服務，了解這些部分如何彼此搭配運作。

### <a name="stateless-reliable-services"></a>無狀態的可靠的服務
無狀態服務是指在服務內不維護呼叫之間的狀態。 任何已存在的狀態完全可丟棄，不需要同步處理、複寫、持續性或高可用性。

例如，想想沒有記憶體且會接收要一次執行之所有項和作業的計算機。

在此情況下，服務的 `RunAsync()` (C#) 或 `runAsync()` (Java) 可以是空的，因為沒有服務需要進行的背景工作處理。 建立計算機服務時，它會傳回 `ICommunicationListener` (C#) 或 `CommunicationListener` (Java) (例如 [Web API](service-fabric-reliable-services-communication-webapi.md))，而這會在某個連接埠上開啟接聽端點。 此接聽端點會連結到不同的計算方法 (範例："Add(n1, n2)")，這些方法定義計算機的公用 API。

從用戶端進行呼叫時，會叫用適當的方法，且計算機服務會在所提供的資料上執行作業，並傳回結果。 它不會儲存任何狀態。

不儲存任何內部狀態會讓此範例計算機變得較簡單。 不過大多數服務並不是真正無狀態。 相反地，它們是將狀態外部化到其他某些存放區  (例如，任何依賴在備份存放區或快取中保留工作階段狀態的 Web 應用程式便不是無狀態)。

Service Fabric 中常見的無狀態服務使用範例是做為前端，其公開 Web 應用程式的公用 API。 前端服務接著和具狀態服務交談，以完成使用者的要求。 在此情況下，用戶端的呼叫會導向至無狀態服務接聽的已知連接埠 (例如 80)。 這個無狀態服務收到呼叫，並判斷呼叫是否來自受信任的合作對象，以及其預定為哪些服務。  然後，無狀態服務將呼叫轉送至正確的具狀態服務分割，並等候回應。 當無狀態服務收到回應時，它會回覆原始用戶端。 我們的 [C#](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started) / [Java](https://github.com/Azure-Samples/service-fabric-java-getting-started/tree/master/Actors/VisualObjectActor/VisualObjectWebService) 範例中有這類服務的例子。 這只是範例中這種模式的其中一個例子，其他範例中還有其他例子。

### <a name="stateful-reliable-services"></a>具狀態可靠的服務
具狀態服務必須有某部分的狀態保持一致且存在，服務才能運作。 假設有一個服務不斷地根據某個值收到的更新，計算它的滾動平均。 若要這樣做，它必須擁有目前需要處理的連入要求集，以及目前的平均。 擷取、處理並將資訊儲存在外部存放區 (例如現在的 Azure Blob 或資料表存放區) 的任何服務都是具狀態。 它只將狀態保留在外部狀態存放區中。

現在的大部分服務會在外部儲存其狀態，因為外部存放區為該狀態提供可靠性、可用性、延展性和一致性。 在 Service Fabric 中，服務不需要將狀態儲存在外部。 Service Fabric 會為服務程式碼和服務狀態來處理這些需求。

假設我們想要撰寫一個處理映像的服務。 在做法上，這個服務需要取得映像和要在該映像上執行的一系列轉換。 此服務會傳回一個通訊接聽程式 (假設是 WebAPI)，其中公開一個像 `ConvertImage(Image i, IList<Conversion> conversions)` 的 API。 當它收到要求時，服務會將它儲存在 `IReliableQueue` 中，並將某個識別碼傳回給用戶端，讓它能夠追蹤要求。

在這個服務中，`RunAsync()` 可能較為複雜。 服務的 `RunAsync()` 內有一個迴圈會從 `IReliableQueue` 提取要求，並執行所要求的轉換。 結果會儲存在 `IReliableDictionary` 中，當用戶端回來時，便可取得轉換後的映像。 為了確保即使發生失敗，也不會遺失映像，這個 Reliable Services 會從佇列提取、執行轉換，並將整個結果儲存在單一交易中。 在此情況下，只有在轉換完成時，才會從佇列移除訊息並將結果儲存在結果字典中。 或者，服務也可能從佇列提取映像，並立即將它儲存在遠端存放區。 這樣可以減少服務必須管理的狀態，但會增加複雜性，因為服務必須保留必要的中繼資料，以管理遠端存放區。 不論採取何種方法，如果中途發生失敗，要求會保留在佇列中等候處理。

關於這個服務，請注意一件事，它聽起來像是一般的 .NET 服務！ 唯一的差別在於使用的資料結構 (`IReliableQueue` 和 `IReliableDictionary`) 是由 Service Fabric 提供，可靠性、可用性和一致性都很高。

## <a name="when-to-use-reliable-services-apis"></a>使用 Reliable Services API 的時機
如果下列任何一項描述出您的應用程式服務需求，那麼便應該考慮 Reliable Services API：

* 您希望服務的程式碼 (也許還有狀態) 具有高可用性和可靠性
* 您需要跨多個狀態單位 (例如，訂單和訂單明細項目) 的交易式保證。
* 您的應用程式狀態可以自然地模型化，做為可靠的字典和佇列。
* 應用程式的程式碼或狀態需要具有高可用性，以及較低的讀取和寫入延遲性。
* 您的應用程式必須跨越一個或多個可靠的集合，控制交易作業的並行處理或資料粒度。
* 您想要管理通訊或控制服務的資料分割配置。
* 您的程式碼需要無限制執行緒的執行階段環境。
* 您的應用程式需要在執行階段動態建立或終結可靠的字典或佇列，或整個服務。
* 您需要以程式設計方式為服務的狀態控制 Service Fabric 提供的備份和還原功能。
* 您的應用程式需要維護其狀態單位的變更歷程記錄。
* 您想要開發或使用協力廠商開發的自訂狀態提供器。

## <a name="next-steps"></a>後續步驟
* [Reliable Services 快速入門](service-fabric-reliable-services-quick-start.md)
* [Reliable Collections](service-fabric-reliable-services-reliable-collections.md)
* [Reliable Actors 程式設計模型](service-fabric-reliable-actors-introduction.md)
