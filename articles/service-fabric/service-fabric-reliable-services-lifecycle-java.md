---
title: Azure Service Fabric Reliable Services 生命週期 | Microsoft Docs
description: 了解 Service Fabric Reliable Services 中的生命週期事件。
services: service-fabric
documentationcenter: java
author: PavanKunapareddyMSFT
manager: timlt
ms.assetid: ''
ms.service: service-fabric
ms.devlang: java
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/30/2017
ms.author: pakunapa
ms.openlocfilehash: cc89d174a201b38d79c7993d548c8eac4a47fbcb
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34210676"
---
# <a name="reliable-services-lifecycle"></a>Reliable Services 生命週期
> [!div class="op_single_selector"]
> * [Windows 上的 C# ](service-fabric-reliable-services-lifecycle.md)
> * [在 Linux 上使用 Java](service-fabric-reliable-services-lifecycle-java.md)
>
>

Reliable Services 是 Azure Service Fabric 提供的一種程式設計模型。 在認識 Reliable Services 的生命週期時，最需要了解的是基本的生命週期事件。 事件的確切順序取決於設定的詳細資料。 

一般而言，Reliable Services 生命週期包含下列事件：

* 在啟動期間：
  * 建構服務。
  * 服務有機會建構和傳回零個以上的接聽程式。
  * 開啟任何傳回的接聽程式，以便與服務進行通訊。
  * 會呼叫服務的 `runAsync` 方法，以便服務能夠進行長時間執行的工作或背景工作。
* 在關閉期間：
  * 會取消傳遞給 `runAsync` 的取消權杖，並關閉接聽程式。
  * 解構服務物件本身。

根據 Reliable Service 是無狀態或具狀態而定，Reliable Service 事件的順序可能稍有變化。 

此外，對於具狀態服務，您必須處理主要交換案例。 在此序列期間，主要角色會轉移至另一個複本 (或回來)，但服務不會關閉。 

最後，您必須考慮錯誤或失敗狀況。

## <a name="stateless-service-startup"></a>無狀態服務啟動
無狀態服務的生命週期相當簡單。 以下是事件的順序︰

1. 建構服務。
2. 以下事件會併行發生：
    - 叫用 `StatelessService.createServiceInstanceListeners()`，並開啟任何傳回的接聽程式。 在每個接聽程式上呼叫 `CommunicationListener.openAsync()`。
    - 呼叫服務的 `runAsync` 方法 (`StatelessService.runAsync()`)。
3. 如果有服務本身的 `onOpenAsync` 方法，會呼叫該方法。 具體來說，會呼叫 `StatelessService.onOpenAsync()`。 這是不常用的覆寫，但可用。

請務必留意，建立及開啟接聽程式的呼叫及 `runAsync` 的呼叫之間，並沒有一定的順序。 接聽程式可能會在開啟 `runAsync` 前啟動。 同樣地，`runAsync` 也可能在通訊接聽程式開啟或建構之前便已經叫用。 如果需要任何同步處理，則必須由實作者完成。 以下是一些常見的解決方案：

* 有時，必須等到建立其他資訊或完成其他工作後，接聽程式才能運作。 對於無狀態服務，該工作通常在服務的建構函式中完成。 可以在 `createServiceInstanceListeners()` 呼叫期間完成，也可以在建構接聽程式本身時完成。
* 有時候，`runAsync` 中的程式碼需要等到接聽程式開啟之後才會啟動。 在此情況下，額外的協調是必要的。 常見的解決方案是在接聽程式中加入旗標。 旗標會在接聽程式完成時指示。 `runAsync` 方法會檢查此項目，再繼續進行實際的工作。

## <a name="stateless-service-shutdown"></a>無狀態服務關閉
關閉無狀態服務時，也依循相同的模式，只是順序相反：

1. 以下事件會併行發生：
    - 任何開啟的接聽程式皆為關閉。 在每個接聽程式上呼叫 `CommunicationListener.closeAsync()`。
    - 傳遞至 `runAsync()` 的取消權杖已取消。 檢查取消權杖的 `isCancelled` 屬性傳回 `true`，還有呼叫權杖的 `throwIfCancellationRequested` 方法是否擲回 `CancellationException`。
2. 每個接聽程式上完成 `closeAsync()` 時，且 `runAsync()` 也完成之後，就呼叫服務的 `StatelessService.onCloseAsync()` 方法 (若有)。 這同樣不是常見的覆寫，不過可以用來安全地關閉資源、停止背景處理、完成外部狀態儲存或關閉現有的連線。
3. `StatelessService.onCloseAsync()` 完成之後，就解構服務物件。

## <a name="stateful-service-startup"></a>具狀態服務啟動
具狀態服務的模式類似於無狀態服務，只有一些不同。  針對啟動具狀態服務，事件的順序如下︰

1. 建構服務。
2. 呼叫 `StatefulServiceBase.onOpenAsync()`。 這個呼叫在服務中不常覆寫。
3. 以下事件會併行發生：
    - 叫用 `StatefulServiceBase.createServiceReplicaListeners()`。 
      - 如果服務是主要服務，則傳回的所有接聽程式為已開啟。 在每個接聽程式上呼叫 `CommunicationListener.openAsync()`。
      - 如果服務是次要服務，只有標示為 `listenOnSecondary = true` 的接聽程式才會開啟。 在次要上開啟的接聽程式較不常見。
    - 如果服務目前是主要，會呼叫此服務的 `StatefulServiceBase.runAsync()` 方法。
4. 當所有複本接聽程式的 `openAsync()` 呼叫完成，而且呼叫 `runAsync()` 之後，會呼叫 `StatefulServiceBase.onChangeRoleAsync()`。 這個呼叫在服務中不常覆寫。

類似於無狀態服務，在具狀態服務中，建立和開啟接聽程式及呼叫 `runAsync` 時，不會協調先後順序。 如果您需要協調，解決方案大致相同。 但是，具狀態服務還有另外一種情況。 假設抵達通訊接聽程式的呼叫需要[可靠的集合](service-fabric-reliable-services-reliable-collections.md)內保留的資訊。 因為在還無法讀取或寫入可靠的集合，且 `runAsync` 也還無法啟動之前，通訊接聽程式可能先開啟，因此，需要一些額外的協調。 最簡單且最常見的解決方法是由通訊接聽程式傳回錯誤碼。 用戶端透過該錯誤碼即知道要重試要求。

## <a name="stateful-service-shutdown"></a>具狀態服務關閉
類似於無狀態服務，關閉期間的生命週期事件與啟動期間相同，但順序相反。 具狀態服務關閉時會發生下列事件︰

1. 以下事件會併行發生：
    - 任何開啟的接聽程式皆為關閉。 在每個接聽程式上呼叫 `CommunicationListener.closeAsync()`。
    - 傳遞至 `runAsync()` 的取消權杖已取消。 對於取消權杖的 `isCancelled()` 方法進行的呼叫傳回 `true`，還有呼叫權杖的 `throwIfCancellationRequested()` 方法是否擲回 `OperationCanceledException`。
2. 當每個接聽程式上完成 `closeAsync()`，且 `runAsync()` 也完成之後，就呼叫服務的 `StatefulServiceBase.onChangeRoleAsync()`。 這個呼叫在服務中不常覆寫。

   > [!NOTE]  
   > 只有在此複本是主要複本時，才需要等候 `runAsync` 完成。

3. 完成 `StatefulServiceBase.onChangeRoleAsync()` 方法之後，會呼叫 `StatefulServiceBase.onCloseAsync()` 方法。 此呼叫是不常用的覆寫，但可用。
3. `StatefulServiceBase.onCloseAsync()` 完成之後，就解構服務物件。

## <a name="stateful-service-primary-swaps"></a>具狀態服務主要複本互換
具狀態服務執行時，只有對於該具狀態服務的主要複本，才會開啟通訊接聽程式並呼叫 `runAsync` 方法。 建構次要複本，但看不到進一步的呼叫。 當具狀態服務執行時，哪個複本目前是主要可能會改變。 具狀態複本可看見的生命週期事件，取決於它在互換期間是降級或升級的複本。

### <a name="for-the-demoted-primary"></a>針對降級的主要複本
Service Fabric 需要降級的主要複本停止處理訊息，並停止任何背景工作。 此步驟類似於關閉服務的情況。 差別在於服務仍為次要，並不會解構或關閉。 隨即會發生下列事件︰

1. 以下事件會併行發生：
    - 任何開啟的接聽程式皆為關閉。 在每個接聽程式上呼叫 `CommunicationListener.closeAsync()`。
    - 傳遞至 `runAsync()` 的取消權杖已取消。 取消權杖的 `isCancelled()` 方法之檢查會傳回 `true`。 如果呼叫，權杖的 `throwIfCancellationRequested()` 方法會擲回 `OperationCanceledException`。
2. 當每個接聽程式上完成 `closeAsync()`，且 `runAsync()` 也完成之後，就呼叫服務的 `StatefulServiceBase.onChangeRoleAsync()`。 這個呼叫在服務中不常覆寫。

### <a name="for-the-promoted-secondary"></a>針對升級的次要複本
同樣地，Service Fabric 需要升級的次要複本開始接聽網路上的訊息，並啟動其需要完成的任何背景工作。 此流程類似於建立服務的情況。 差別在於複本本身已經存在。 隨即會發生下列事件︰

1. 以下事件會併行發生：
    - 叫用 `StatefulServiceBase.createServiceReplicaListeners()`，並開啟任何傳回的接聽程式。 在每個接聽程式上呼叫 `CommunicationListener.openAsync()`。
    - 呼叫服務的 `StatefulServiceBase.runAsync()` 方法。
2. 當所有複本接聽程式的 `openAsync()` 呼叫完成，而且呼叫 `runAsync()` 之後，會呼叫 `StatefulServiceBase.onChangeRoleAsync()`。 這個呼叫在服務中不常覆寫。

### <a name="common-issues-during-stateful-service-shutdown-and-primary-demotion"></a>具狀態服務關閉和主要降級期間的常見問題
Service Fabric 基於多種原因變更具狀態服務的主要。 最常見的原因是[叢集重新平衡](service-fabric-cluster-resource-manager-balancing.md)和[應用程式升級](service-fabric-application-upgrade.md)。 在這些作業期間，服務必須遵循 `cancellationToken`。 這也適用於一般服務關閉的期間，例如刪除服務的情況。

未完全處理取消作業的服務將會遇到幾個問題。 這些作業可能會變慢，因為 Service Fabric 會以正常程序等待服務停止。 這可能最終會導致逾時的失敗升級和復原。 無法採用取消權杖也可能造成不平衡的叢集。 叢集會因為過於經常存取節點而失去平衡。 不過，無法重新平衡服務，因為將服務移至其他地方耗費太多時間。 

因為這些服務是具有狀態的，所以服務會使用[可靠集合](service-fabric-reliable-services-reliable-collections.md)。 在 Service Fabric 中，當主要降級時，首先發生的事情是撤銷基礎狀態的寫入存取權。 這會導致可能影響服務生命週期的第二組問題。 集合會根據時間和複本是否正在移動或關機而傳回例外狀況。 務必正確處理這些例外狀況。 

Service Fabric 擲回的例外狀況都是永久 [(`FabricException`)](https://docs.microsoft.com/java/api/system.fabric.exception) 或暫時性 [(`FabricTransientException`)](https://docs.microsoft.com/java/api/system.fabric.exception._fabric_transient_exception)。 應該記錄並擲回永久例外狀況。 可以根據重試邏輯重試暫時性例外狀況。

測試和驗證 Reliable Services 時，請記得處理因將 `ReliableCollections` 而與服務生命週期事件搭配使用所導致的例外狀況，這是相當重要的一環。 建議您一律在負載下執行您的服務。 也應該先執行升級和[混亂測試](service-fabric-controlled-chaos.md)，再部署至生產環境。 下列基本步驟協助確保您的服務正確地實作，並正確地處理生命週期事件。

## <a name="notes-on-service-lifecycle"></a>服務生命週期的注意事項
* `runAsync()` 方法和 `createServiceInstanceListeners/createServiceReplicaListeners` 呼叫都是選擇性。 服務可能會有其中一個或兩個，也可能完全沒有。 例如，若服務本身自行負責回應使用者呼叫，則不需要實作 `runAsync()`。 只需要通訊接聽程式及其相關聯的程式碼。  同樣地，建立和傳回通訊接聽程式是選擇性的。 該服務可能只有背景工作要進行，因此只需要實作 `runAsync()`。
* 服務可以成功完成 `runAsync()` 並返回。 這不被視為失敗狀況。 這代表服務的背景工作正要完成。 對於具狀態 Reliable Services，如果服務從主要降級，然後又升級回到主要，則會再次呼叫 `runAsync()`。
* 如果服務藉由擲回某些非預期的例外狀況退出 `runAsync()`，即是失敗。 服務物件會關閉，而且會報告健康情況錯誤。
* 雖然從這些方法進行傳回並沒有時間限制，您會立即喪失寫入的能力。 因此，您無法完成任何實際工作。 建議您盡快在收到取消要求後傳回。 如果您的服務未於合理的時間內回應這些 API 呼叫，Service Fabric 可能會強制終止服務。 這通常只發生在應用程式升級期間或刪除服務時。 此逾時預設為 15 分鐘。
* `onCloseAsync()` 路徑的失敗會導致呼叫 `onAbort()`。 此呼叫是服務清除並釋放已宣告之任何資源的最後努力機會。 這個一般會在於節點上偵測到永久錯誤，或因內部失敗而 Service Fabric 無法可靠地管理服務執行個體生命週期時呼叫。
* 具狀態服務複本變更角色 (例如變更為主要或次要角色) 時，會呼叫 `OnChangeRoleAsync()`。 主要複本會獲得寫入狀態 (可建立並寫入可靠的集合)。 次要複本則會獲得讀取狀態 (只能從現有的可靠集合讀取)。 具狀態服務中的大部分工作會在主要複本執行。 次要複本可以執行唯讀驗證、產生報表、資料採礦或其他唯讀作業。

## <a name="next-steps"></a>後續步驟
* [Reliable Services 簡介](service-fabric-reliable-services-introduction.md)
* [Reliable Services 快速入門](service-fabric-reliable-services-quick-start-java.md)

