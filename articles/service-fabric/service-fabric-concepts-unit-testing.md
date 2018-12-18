---
title: 對 Azure Service Fabric 中的具狀態服務進行單元測試 | Microsoft Docs
description: 了解對 Service Fabric 具狀態服務進行單元測試的概念和做法。
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
ms.openlocfilehash: c2d98316b81b3d908ebbe6147fe40f231e94c142
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43703292"
---
# <a name="unit-testing-stateful-services-in-service-fabric"></a>對 Service Fabric 中的具狀態服務進行單元測試

本文內容涵蓋對 Service Fabric 具狀態服務進行單元測試的概念和做法。 由於應用程式程式碼可有效地在多個不同內容下執行，在 Service Fabric 內進行單元測試時理應有其本身的考量。 本文說明可用做法，以確保應用程式程式碼會涵蓋在其可執行所在的每個不同內容下。

## <a name="unit-testing-and-mocking"></a>單元測試和模擬
在本文內容中所進行的單元測試為自動化測試，可於測試執行器 (例如 MSTest 或 NUnit) 的內容中執行。 本文內的單元測試不會針對遠端資源 (例如，資料庫或 RESTFul API) 執行作業。 請模擬這些遠端資源。 本文內容中的模擬會偽造、記錄和控制遠端資源的傳回值。

### <a name="service-fabric-considerations"></a>Service Fabric 考量
對 Service Fabric 具狀態服務進行單元測試時，有幾個考量事項。 首先，服務程式碼會在多個節點上執行，但所扮演的角色不同。 單元測試應該評估每個角色下的程式碼，才能實現完整涵蓋。 這些不同角色分別是主要、作用中次要、閒置次要和未知。 「無」角色通常不需要任何特殊涵蓋，原因是 Service Fabric 會將此角色視為無效或 Null 的服務。 其次，每個節點都會在任何指定時間點變更其角色。 若要實現完整涵蓋，請在發生角色變更的情況下測試程式碼執行路徑。

## <a name="why-unit-test-stateful-services"></a>為何要對具狀態服務進行單元測試？ 
對具狀態服務進行單元測試有助於發現一些傳統應用程式或特定領域單元測試不一定能抓到的常犯錯誤。 例如，如果具狀態服務有任何記憶體內部狀態，這種測試可以確認這個記憶體內部狀態在每個複本之間有保持同步。 這種測試也可以確認具狀態服務會回應 Service Fabric 協調流程所適當傳入的取消權杖。 取消一經觸發，服務就應該停止任何長時間執行和/或非同步的作業。  

## <a name="common-practices"></a>常見做法

下一節會針對進行具狀態服務單元測試時最常見的做法提出建議。 其中也會建議模擬層應該具有怎樣的環境，才能緊密配合 Service Fabric 協調流程和狀態管理。 所模擬的程式庫必須是確實提供這項功能的程式庫。 [ServiceFabric.Mocks](https://www.nuget.org/packages/ServiceFabric.Mocks/) 3.3.0 以上的版本便是這樣的程式庫，其可提供所建議的模擬功能，並遵循下面所述的做法。

### <a name="arrangement"></a>安排

#### <a name="use-multiple-service-instances"></a>使用多個服務執行個體
單元測試應針對具狀態服務執行多個執行個體。 在叢集中，若 Service Fabric 會佈建多個跨不同節點執行服務的複本，這麼做可模擬叢集的實際運作情形。 不過，這些執行個體各自會在不同的內容下執行。 在執行測試時，每個執行個體應該事先備有叢集上預期會有的角色設定。 例如，如果服務預期的目標複本大小為 3，Service Fabric 就會在不同節點上佈建三個複本。 一個為主要複本，另外兩個為作用中次要複本。

在大部分情況下，服務的執行路徑會隨著每個角色而略有不同。 例如，如果服務不應該接受來自作用中次要複本的要求，則服務可能會有針對此情況的檢查，其會擲回具有資訊的例外狀況，以指出有人對次要複本嘗試提出要求。 擁有多個執行個體便可測試這種情況。

此外，擁有多個執行個體可讓測試切換各個執行個體的角色，以確認不論角色如何變更，回應都保持一致。

#### <a name="mock-the-state-manager"></a>模擬狀態管理員
狀態管理員應該視為是遠端資源，並因此加以模擬。 在模擬狀態管理員時，您必須有一些基礎記憶體內部儲存體，以便追蹤儲存到狀態管理員的項目，從而能夠讀取並驗證狀態管理員。 針對可靠集合的每種類型建立模擬執行個體即可輕鬆達到此目的。 在這些模擬中，所使用的資料類型必須密切配合針對該集合所執行的作業。 以下是針對每個可靠集合所建議的一些資料類型

- IReliableDictionary<TKey, TValue> -> System.Collections.Concurrent.ConcurrentDictionary<TKey, TValue>
- IReliableQueue<T> -> System.Collections.Generic.Queue<T>
- IReliableConcurrentQueue<T> -> System.Collections.Concurrent.ConcurrentQueue<T>

#### <a name="many-state-manager-instances-single-storage"></a>多個狀態管理員執行個體搭配單一儲存體
之前提到，狀態管理員和可靠集合應該視為是遠端資源。 因此，您應該且將會在單元測試中模擬這些資源。 不過，在為具狀態服務執行多個執行個體時，要讓每個模擬的狀態管理員在不同的具狀態服務執行個體中保持同步並不容易。 當具狀態服務在叢集上執行時，Service Fabric 會負責讓每個次要複本的狀態管理員與主要複本保持一致。 因此，測試的行為應該相同，才能模擬角色變更。

想要達成此一同步有個簡單的方法，那就是針對會將寫入至每個可靠集合的資料儲存起來的基礎物件，使用單一資料庫模式。 例如，如果具狀態服務使用 `IReliableDictionary<string, string>`。 模擬的狀態管理員應該傳回 `IReliableDictionary<string, string>` 的模擬。 該模擬可能會使用 `ConcurrentDictionary<string, string>` 來追蹤寫入的索引鍵/值組。 `ConcurrentDictionary<string, string>` 應該是傳遞至服務的狀態管理員所有執行個體所使用的單一資料庫。

#### <a name="keep-track-of-cancellation-tokens"></a>追蹤取消權杖
取消權杖是重要、但常常未能注意到的具狀態服務層面。 Service Fabric 在啟動具狀態服務的主要複本時，會提供取消權杖。 此取消權杖的目的，是要在遭到移除或降級為不同角色時，對服務發出訊號。 具狀態服務應該停止任何長時間執行或非同步的作業，讓 Service Fabric 可以完成角色變更工作流程。

在執行單元測試時，提供給 RunAsync、ChangeRoleAsync、OpenAsync 和 CloseAsync 的取消權杖，應在測試執行期間保留下來。 保留這些權杖可讓測試作業模擬服務關閉或降級，並適當地驗證服務回應。

#### <a name="test-end-to-end-with-mocked-remote-resources"></a>使用模擬的遠端資源進行端對端測試
單元測試應該盡可能地執行可修改具狀態服務狀態的應用程式程式碼。 建議您進行更具端對端本質的測試。 唯一存在的模擬是為了記錄、模擬及/或驗證遠端資源的互動。 其中包括與狀態管理員和可靠集合的互動。 下列程式碼片段是 gherkin 的範例，以便示範端對端測試：

```
    Given stateful service named "fabric:/MyApp/MyService" is created
    And a new replica is created as "Primary" with id "111"
    And a new replica is created as "IdleSecondary" with id "222"
    And a new replica is created as "IdleSecondary" with id "333"
    And all idle secondary replicas are promoted to active secondary
    When a request is made to add the an employee "John Smith"
    And the active secondary replica "222" is promoted to primary
    And a request is made to get all employees
    Then the request should should return the "John Smith" employee
```

這項測試可判斷，在某個複本所擷取的資料可在升階為主要複本的次要複本上使用。 假設可靠集合是員工資料的備份存放區，且如果應用程式程式碼未在交易上執行 `CommitAsync` 以儲存新的員工，則我們可能會在這項測試中捕捉到失敗。 在此情況下，目的是要取得員工的第二個要求就不會傳回第一個要求所新增的員工。

### <a name="acting"></a>行動
#### <a name="mimic-service-fabric-replica-orchestration"></a>模擬 Service Fabric 複本協調流程
在管理多個服務執行個體時，測試應該採用和 Service Fabric 協調流程相同的方式，來將這些服務初始化並卸除。 例如，在新的主要複本上建立服務時，Service Fabric 會叫用 CreateServiceReplicaListener、OpenAsync、ChangeRoleAsync 和 RunAsync。 生命週期事件記載於下列文章：

- [具狀態服務啟動](service-fabric-reliable-services-lifecycle.md#stateful-service-startup)
- [具狀態服務關閉](service-fabric-reliable-services-lifecycle.md#stateful-service-shutdown)
- [具狀態服務主要複本互換](service-fabric-reliable-services-lifecycle.md#stateful-service-primary-swaps)

#### <a name="run-replica-role-changes"></a>執行複本角色變更
單元測試應該採用和 Service Fabric 協調流程相同的方式，來變更服務執行個體的角色。 角色狀態機器記載於下列文章：

[複本角色狀態機器](service-fabric-concepts-replica-lifecycle.md#replica-role)

模擬角色變更是測試中較為重要的層面之一，並可發現複本狀態彼此不一致的問題。 複本狀態不一致的發生原因，可能是由於將記憶體內部狀態儲存在靜態或類別層級的執行個體變數中。 這方面的範例可能是取消權杖、列舉和設定物件/值。 這也會確保服務會採用 RunAsync 期間所提供的取消權杖，來允許角色變更發生。 模擬角色變更也可發現未撰寫程式碼來允許叫用 RunAsync 多次時所可能引發的問題。

#### <a name="cancel-cancellation-tokens"></a>將取消權杖取消
應該會有某些單元測試已取消提供給 RunAsync 的取消權杖。 這會讓測試得以確認該服務是否正常關閉。 在關閉期間，任何長時間執行或非同步的作業都應該停下。 可能存在於服務上的長時間執行程序範例，就是在可靠的佇列上接聽訊息的程序。 這可能會直接存在於 RunAsync 或背景執行緒內。 實作中應該包含此取消權杖已取消時，用於結束作業的邏輯。

如果具狀態服務所利用的快取或記憶體內部狀態只應存在於主要複本，則應於此時處置該狀態。 這是為了確保此狀態會在節點稍後重新變為主要複本時保持一致。 取消測試可讓測試確認此狀態是否有適當處置。

#### <a name="execute-requests-against-multiple-replicas"></a>針對多個複本執行要求
判斷測試應針對不同複本執行相同的要求。 若與角色變更搭配，則可以發現一致性問題。 測試範例可能會執行下列步驟：
1. 針對目前的主要複本執行寫入要求
2. 針對目前的主要複本，執行會傳回步驟 1 所寫入資料的讀取要求
3. 將次要複本升階為主要複本。 這也應該將目前的主要複本降級為次要複本
4. 針對新的次要複本，執行步驟 2 的相同讀取要求。

在最後一個步驟中，此測試可以判斷所傳回的資料一致。 此測試可能發現的潛在問題是，服務所傳回的資料可能在記憶體內，但最終卻由可靠的集合支援。 該記憶體內部資料可能不會正確地與存在於可靠集合中的資料保持同步。

記憶體內部資料通常會用來針對存在於可靠集合中的資料建立次要索引或彙總。

### <a name="asserting"></a>判斷
#### <a name="ensure-responses-match-across-replicas"></a>確定複本之間的回應相符
單元測試應該判斷，在多個複本轉換為主要複本後，指定要求的回應在這些複本之間是否一致。 這可能會浮現潛在問題，也就是回應中所提供的資料不是未由可靠的集合支援，就是保留在記憶體內部，但卻沒有機制可在複本之間同步處理該資料。 這會確保在 Service Fabric 重新平衡或容錯移轉至新的主要複本之後，服務會傳回一致的回應。

#### <a name="verify-service-respects-cancellation"></a>確認服務有採用取消
應該在取消權杖取消時加以終止的長時間執行或非同步的程序，應該要確認其確實已在取消後遭到終止。 這可確保儘管複本變更了角色，不適合在非主要複本上繼續執行的程序會在轉換完成前停下。 這也可發現這種程序會讓來自 Service Fabric 的角色變更或關閉要求無法完成的問題。

#### <a name="verify-which-replicas-should-serve-requests"></a>確認哪個複本應該處理要求
此測試應該判斷當要求路由傳送至非主要複本時的預期行為。 Service Fabric 確實能夠讓次要複本處理要求。 不過，對可靠集合的寫入則只能從主要複本進行。 如果應用程式只想要讓主要複本處理要求，或只有部分要求可由次要複本處理，則測試應該判斷正面和負面情況下的預期行為。 負面情況是要求路由傳送至不應處理要求的複本，反之則為正面情況。

## <a name="next-steps"></a>後續步驟
了解如何[對具狀態服務進行單元測試](service-fabric-how-to-unit-test-stateful-services.md)。