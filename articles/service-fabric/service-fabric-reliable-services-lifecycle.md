---
title: Azure Service Fabric Reliable Services 生命週期的概觀 | Microsoft Docs
description: 了解 Service Fabric Reliable Services中不同的生命週期事件
services: Service-Fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: vturecek;
ms.assetid: ''
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: f301c0156265f055f0ebf7cdad8dba7f39f5ba2b
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39044572"
---
# <a name="reliable-services-lifecycle-overview"></a>Reliable Services 生命週期概觀
> [!div class="op_single_selector"]
> * [Windows 上的 C# ](service-fabric-reliable-services-lifecycle.md)
> * [在 Linux 上使用 Java](service-fabric-reliable-services-lifecycle-java.md)
>
>

在考慮 Azure Service Fabric Reliable Services 的生命週期時，生命週期的基礎最重要。 一般情況下，生命週期包括下列各項：

- 在啟動期間：
  - 建構服務。
  - 服務有機會建構和傳回零個以上的接聽程式。
  - 開啟任何傳回的接聽程式，能夠與服務進行通訊。
  - 會呼叫服務的 **RunAsync** 方法，以允許服務進行長時間執行的工作或背景工作。
- 在關閉期間：
  - 會取消傳遞給 **RunAsync** 的取消權杖，並關閉接聽程式。
  - 接聽程式關閉之後，就解構服務物件本身。

這些事件的確切順序還有一些細節可討論。 根據 Reliable Service 是無狀態或具狀態而定，事件的順序可能稍有變化。 此外，對於具狀態服務，我們必須處理「主要」複本互換情況。 在此序列期間，「主要」角色會轉移至另一個複本 (或回來)，但服務不會關閉。 最後，我們必須考慮錯誤或失敗狀況。

## <a name="stateless-service-startup"></a>無狀態服務啟動
無狀態服務的生命週期相當簡單。 以下是事件的順序︰

1. 建構服務。
2. 接著，兩件事平行發生︰
    - 叫用 `StatelessService.CreateServiceInstanceListeners()`，並開啟任何傳回的接聽程式。 在每個接聽程式上呼叫 `ICommunicationListener.OpenAsync()`。
    - 呼叫服務的 `StatelessService.RunAsync()` 方法。
3. 如果有的話，會呼叫服務的 `StatelessService.OnOpenAsync()` 方法。 此呼叫是不常用的覆寫，但可用。 這個階段可以啟動擴充服務的初始化作業。

請記住，建立及開啟接聽程式的呼叫及 **RunAsync** 之間，並沒有一定的順序。 接聽程式可以在 **RunAsync** 啟動之前開啟。 同樣地，您可以在通訊接聽程式開啟或建構之前叫用 **RunAsync**。 如果需要任何同步處理，則留給實作者去負責。 以下是一些常見的解決方案：

  - 有時，必須等到建立其他一些資訊或完成工作後，接聽程式才能運作。 運作的無狀態服務通常可在其他位置完成，如下所示： 
    - 在服務的建構函式中。
    - 在 `CreateServiceInstanceListeners()` 呼叫期間。
    - 作為接聽程式本身建構的一部分。
  - 有時候，**RunAsync** 中的程式碼需要等到接聽程式開啟之後才會啟動。 在此情況下，額外的協調是必要的。 一個常見的解決方案是接聽程式內有旗標，指出完成的時間。 接著，該旗標會先簽入 **RunAsync**，然後再繼續進行實際工作。

## <a name="stateless-service-shutdown"></a>無狀態服務關閉
針對關閉無狀態服務，也依循相同的模式，只是順序相反：

1. 平行：
    - 任何開啟的接聽程式皆為關閉。 在每個接聽程式上呼叫 `ICommunicationListener.CloseAsync()`。
    - 傳遞至 `RunAsync()` 的取消權杖已取消。 檢查取消權杖的 `IsCancellationRequested` 屬性傳回 true，還有呼叫權杖的 `ThrowIfCancellationRequested` 方法是否擲回 `OperationCanceledException`。
2. 當每個接聽程式上完成 `CloseAsync()`，且 `RunAsync()` 也完成之後，就呼叫服務的 `StatelessService.OnCloseAsync()` 方法 (若有的話)。  無狀態服務執行個體即將正常關閉時，會呼叫 OnCloseAsync。 這可能在升級服務程式碼、因負載平衡而移動服務執行個體或偵測到暫時性失敗的時侯發生。 覆寫 `StatelessService.OnCloseAsync()`，不過可以用來安全地關閉資源、停止背景處理、完成外部狀態儲存或關閉現有的連線。
3. `StatelessService.OnCloseAsync()` 完成之後，就解構服務物件。

## <a name="stateful-service-startup"></a>具狀態服務啟動
具狀態服務的模式類似於無狀態服務，只有一些不同。 針對啟動具狀態服務，事件的順序如下︰

1. 建構服務。
2. 呼叫 `StatefulServiceBase.OnOpenAsync()`。 這個呼叫在服務中不常覆寫。
3. 下列事情會同時發生：
    - 叫用 `StatefulServiceBase.CreateServiceReplicaListeners()`。 
      - 如果服務是「主要」服務，則傳回的所有接聽程式為已開啟。 在每個接聽程式上呼叫 `ICommunicationListener.OpenAsync()`。
      - 如果服務是「次要」服務，只有標示為 `ListenOnSecondary = true` 的接聽程式才會開啟。 在次要上開啟的接聽程式較不常見。
    - 如果服務目前是「主要」，會呼叫此服務的 `StatefulServiceBase.RunAsync()` 方法。
4. 當所有複本接聽程式的 `OpenAsync()` 呼叫完成，而且呼叫 `RunAsync()` 之後，會呼叫 `StatefulServiceBase.OnChangeRoleAsync()`。 這個呼叫在服務中不常覆寫。

類似於無狀態服務，在建立和開啟接聽程式及呼叫 **RunAsync** 時，不會協調先後順序。 如果您需要協調，解決方案大致相同。 具狀態服務還有一個額外情況。 假設抵達通訊接聽程式的呼叫需要[可靠的集合](service-fabric-reliable-services-reliable-collections.md)內保留的資訊。

   > [!NOTE]  
   > 因為在還無法讀取或寫入可靠的集合，且 **RunAsync** 也還無法啟動之前，通訊接聽程式可能先開啟，因此，一些額外的協調有必要。 最簡單且最常見的解決方法是由通訊接聽程式傳回錯誤碼，讓用戶端知道要重試要求。

## <a name="stateful-service-shutdown"></a>具狀態服務關閉
類似於無狀態服務，關閉期間的生命週期事件與啟動期間相同，但順序相反。 具狀態服務關閉時會發生下列事件︰

1. 平行：
    - 任何開啟的接聽程式皆為關閉。 在每個接聽程式上呼叫 `ICommunicationListener.CloseAsync()`。
    - 傳遞至 `RunAsync()` 的取消權杖已取消。 檢查取消權杖的 `IsCancellationRequested` 屬性傳回 true，還有呼叫權杖的 `ThrowIfCancellationRequested` 方法是否擲回 `OperationCanceledException`。
2. 當每個接聽程式上完成 `CloseAsync()`，且 `RunAsync()` 也完成之後，就呼叫服務的 `StatefulServiceBase.OnChangeRoleAsync()`。 這個呼叫在服務中不常覆寫。

   > [!NOTE]  
   > 只有當此複本是「主要」複本時，才需要等候 **RunAsync** 完成。

3. 完成 `StatefulServiceBase.OnChangeRoleAsync()` 方法之後，會呼叫 `StatefulServiceBase.OnCloseAsync()` 方法。 此呼叫是不常用的覆寫，但可用。
3. `StatefulServiceBase.OnCloseAsync()` 完成之後，就解構服務物件。

## <a name="stateful-service-primary-swaps"></a>具狀態服務主要複本互換
當具狀態服務執行時，只有此具狀態服務的「主要」複本會開啟通訊接聽程式和呼叫 **RunAsync** 方法。 建構次要複本，但看不到進一步的呼叫。 當具狀態服務執行時，哪個複本目前是「主要」可能會改變。 就複本可見的生命週期事件來說，這代表什麼？ 具狀態複本所看到的行為，取決於它在互換期間是降級或升級的複本。

### <a name="for-the-primary-thats-demoted"></a>降級的主要複本
針對降級的主要複本，Service Fabric 需要此複本停止處理訊息並結束它正在進行的任何背景工作。 因此，這個步驟類似於服務關閉時的情形。 差別在於服務仍為「次要」，並不會解構或關閉。 呼叫下列 API：

1. 平行：
    - 任何開啟的接聽程式皆為關閉。 在每個接聽程式上呼叫 `ICommunicationListener.CloseAsync()`。
    - 傳遞至 `RunAsync()` 的取消權杖已取消。 檢查取消權杖的 `IsCancellationRequested` 屬性傳回 true，還有呼叫權杖的 `ThrowIfCancellationRequested` 方法是否擲回 `OperationCanceledException`。
2. 當每個接聽程式上完成 `CloseAsync()`，且 `RunAsync()` 也完成之後，就呼叫服務的 `StatefulServiceBase.OnChangeRoleAsync()`。 這個呼叫在服務中不常覆寫。

### <a name="for-the-secondary-thats-promoted"></a>升級的次要複本
同樣地，Service Fabric 需要升級的次要複本開始接聽網路上的訊息，並啟動其需要完成的任何背景工作。 因此，這個程序類似於建立服務時的情形，差別在於複本它本身已存在。 呼叫下列 API：

1. 平行：
    - 叫用 `StatefulServiceBase.CreateServiceReplicaListeners()`，並開啟任何傳回的接聽程式。 在每個接聽程式上呼叫 `ICommunicationListener.OpenAsync()`。
    - 呼叫服務的 `StatefulServiceBase.RunAsync()` 方法。
2. 當所有複本接聽程式的 `OpenAsync()` 呼叫完成，而且呼叫 `RunAsync()` 之後，會呼叫 `StatefulServiceBase.OnChangeRoleAsync()`。 這個呼叫在服務中不常覆寫。

### <a name="common-issues-during-stateful-service-shutdown-and-primary-demotion"></a>具狀態服務關閉和主要降級期間的常見問題
Service Fabric 基於各種原因變更具狀態服務的「主要」。 最常見的是[叢集重新平衡](service-fabric-cluster-resource-manager-balancing.md)和[應用程式升級](service-fabric-application-upgrade.md)。 在這些作業期間 (以及在正常服務關機期間，如您在刪除服務時所見)，服務遵守 `CancellationToken` 是相當重要的。 

未完全處理取消作業的服務將會遇到幾個問題。 這些作業可能會變慢，因為 Service Fabric 會以正常程序等待服務停止。 這可能最終會導致逾時的失敗升級和復原。 無法採用取消權杖也可能造成不平衡的叢集。 叢集變成不平衡，因為節點變成經常性存取，但無法重新平衡服務，因為將服務移至其他地方耗費太多時間。 

因為這些服務是具有狀態的，所以它們會使用[可靠集合](service-fabric-reliable-services-reliable-collections.md)。 在 Service Fabric 中，當「主要」降級時，首先發生的事情是撤銷基礎狀態的寫入存取權。 這會導致可能影響服務生命週期的第二組問題。 集合會根據時間和複本是否正在移動或關機而傳回例外狀況。 應該正確處理這些例外狀況。 Service Fabric 擲回的例外狀況可分為永久性 [(`FabricException`)](https://docs.microsoft.com/dotnet/api/system.fabric.fabricexception?view=azure-dotnet) 和暫時性[(`FabricTransientException`)](https://docs.microsoft.com/dotnet/api/system.fabric.fabrictransientexception?view=azure-dotnet) 類別。 永久性例外狀況應該要記錄並擲回，而暫時性例外狀況可能會根據一些重試邏輯來重試。

處理由於使用與服務生命週期事件搭配使用的 `ReliableCollections` 而造成的例外況況，是測試和驗證可靠服務的重要環節。 建議一律先在低於負載的情況下執行服務，同時執行升級和[混亂測試](service-fabric-controlled-chaos.md)，然後再部署到生產。 下列基本步驟協助確保您的服務正確地實作，並正確地處理生命週期事件。


## <a name="notes-on-the-service-lifecycle"></a>服務生命週期的注意事項
  - `RunAsync()` 方法和 `CreateServiceReplicaListeners/CreateServiceInstanceListeners` 呼叫都是選擇性。 一個服務可能具有其中一個、兩者或都沒有。 例如，若服務本身自行負責回應使用者呼叫，則不需要實作 `RunAsync()`。 只需要通訊接聽程式及其相關聯的程式碼。 同樣地，建立和傳回通訊接聽程式是選擇性，因為服務可能只有背景工作要處理，所以只需要實作 `RunAsync()`。
  - 服務可以成功完成 `RunAsync()` 並返回。 完成不是失敗情況。 完成 `RunAsync()` 表示服務的背景工作已完成。 對於具狀態可靠服務，如果複本從「主要」降級到「次要」，然後又升級回到「主要」，則會再次呼叫 `RunAsync()`。
  - 如果服務藉由擲回某些非預期的例外狀況退出 `RunAsync()`，則會構成失敗。 服務物件會關閉，而且會報告健康情況錯誤。
  - 雖然從這些方法傳回沒有時間限制，但您會立即喪失寫入到可靠的集合的能力，並因此無法完成任何實際工作。 建議您盡快在收到取消要求後傳回。 如果您的服務未於合理的時間內回應這些 API 呼叫，Service Fabric 可能會強制終止服務。 這通常只發生在應用程式升級期間或刪除服務時。 此逾時預設為 15 分鐘。
  - `OnCloseAsync()` 路徑失敗會導致呼叫 `OnAbort()`，這是服務清除並釋放已宣告之任何資源的最後努力機會。 這個一般會在於節點上偵測到永久錯誤，或因內部失敗而 Service Fabric 無法可靠地管理服務執行個體生命週期時呼叫。
  - 具狀態服務複本變更角色 (例如變更為主要或次要角色) 時，會呼叫 `OnChangeRoleAsync()`。 主要複本會獲得寫入狀態 (可建立並寫入可靠的集合)。 次要複本則會獲得讀取狀態 (只能從現有的可靠集合讀取)。 具狀態服務中的大部分工作會在主要複本執行。 次要複本可以執行唯讀驗證、產生報表、資料採礦或其他唯讀作業。

## <a name="next-steps"></a>後續步驟
- [Reliable Services 簡介](service-fabric-reliable-services-introduction.md)
- [Reliable Services 快速入門](service-fabric-reliable-services-quick-start.md)
- [複本和執行個體](service-fabric-concepts-replica-lifecycle.md)
