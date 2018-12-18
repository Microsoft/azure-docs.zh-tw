---
title: Reliable Services 通知 | Microsoft Docs
description: Service Fabric Reliable Services 通知的概念文件
services: service-fabric
documentationcenter: .net
author: mcoskun
manager: timlt
editor: masnider,vturecek
ms.assetid: cdc918dd-5e81-49c8-a03d-7ddcd12a9a76
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 6/29/2017
ms.author: mcoskun
ms.openlocfilehash: a13e5d74390b82888f51cfd225c54e29550354e9
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47433509"
---
# <a name="reliable-services-notifications"></a>Reliable Services 通知
通知可讓用戶端追蹤對於他們感興趣物件所進行的變更。 兩種類型的物件支援通知：「可靠的狀態管理員」和「可靠的字典」。

使用通知的常見原因如下：

* 建置具體化檢視 (例如次要索引) 或複本狀態的彙總篩選檢視。 可靠的字典中所有索引鍵的排序索引便是其中一個例子。
* 傳送監視資料，例如過去一小時內新增的使用者數目。

套用作業過程中即會引發通知。 因為如此，應該以最快速度處理通知，且同步事件不應包含任何耗費資源的作業。

## <a name="reliable-state-manager-notifications"></a>可靠的狀態管理員通知
可靠的狀態管理員提供下列事件的通知︰

* 交易
  * 認可
* 狀態管理員
  * 重建
  * 加入可靠的狀態
  * 移除可靠的狀態

可靠狀態管理員會追蹤目前傳遞的交易。 唯一會造成通知引發的交易狀態變更是認可交易。

可靠的狀態管理員會維護可靠狀態的集合，例如可靠的字典和可靠的佇列。 可靠的狀態管理員會在此集合變更時引發通知︰加入或移除可靠的狀態，或者重建整個集合時。
可靠的狀態管理員集合會在下列三種情況下重建：

* 復原︰當複本啟動時，它會從磁碟復原為先前的狀態。 復原結束時，它會利用 **NotifyStateManagerChangedEventArgs** 引發事件，其中包含一組復原的可靠狀態。
* 完整備份︰在複本可加入組態集之前，必須先建置它。 有時，這需要主要複本的可靠狀態管理員的狀態完整複本，以套用到閒置的次要複本。 次要複本上的可靠狀態管理員會使用 **NotifyStateManagerChangedEventArgs** 引發事件，其中包含一組它從主要複本取得的可靠狀態。
* 還原︰在災害復原案例中，複本的狀態可經由 **RestoreAsync**從備份還原。 在這類案例中，主要複本上的可靠狀態管理員會使用 **NotifyStateManagerChangedEventArgs** 引發事件，其中包含一組它從備份還原來的可靠狀態。

若要註冊交易通知及/或狀態管理員通知，您必須向可靠的狀態管理員分別註冊 **TransactionChanged** 或 **StateManagerChanged** 事件。 註冊這些事件處理常式的常見位置是您的具狀態服務的建構函式。 當您註冊建構函式時，您也不會錯過 **IReliableStateManager**存留期間的變更所造成的任何通知。

```csharp
public MyService(StatefulServiceContext context)
    : base(MyService.EndpointName, context, CreateReliableStateManager(context))
{
    this.StateManager.TransactionChanged += this.OnTransactionChangedHandler;
    this.StateManager.StateManagerChanged += this.OnStateManagerChangedHandler;
}
```

**TransactionChanged** 事件處理常式會使用 **NotifyTransactionChangedEventArgs** ，來提供關於事件的詳細資料。 它包含指定變更類型的 Action 屬性 (例如， **NotifyTransactionChangedAction.Commit**)。 它也包含提供「指向變更之交易的參考」的交易屬性。

> [!NOTE]
> 現在，只有認可交易才會引發 **TransactionChanged** 事件。 此動作等於 **NotifyTransactionChangedAction.Commit**。 但是在未來，可能會有其他類型的交易狀態變更可以引發事件。 我們建議您檢查動作，並只在您預期的事件發生時處理事件。
> 
> 

以下是範例 **TransactionChanged** 事件處理常式。

```csharp
private void OnTransactionChangedHandler(object sender, NotifyTransactionChangedEventArgs e)
{
    if (e.Action == NotifyTransactionChangedAction.Commit)
    {
        this.lastCommitLsn = e.Transaction.CommitSequenceNumber;
        this.lastTransactionId = e.Transaction.TransactionId;

        this.lastCommittedTransactionList.Add(e.Transaction.TransactionId);
    }
}
```

**StateManagerChanged** 事件處理常式使用 **NotifyStateManagerChangedEventArgs** 來提供事件的相關詳細資料。
**NotifyStateManagerChangedEventArgs** 有兩個子類別︰**NotifyStateManagerRebuildEventArgs** 和 **NotifyStateManagerSingleEntityChangedEventArgs**。
您使用 **NotifyStateManagerChangedEventArgs** 中的 Action 屬性將 **NotifyStateManagerChangedEventArgs** 轉換為正確的子類別︰

* **NotifyStateManagerChangedAction.Rebuild**：**NotifyStateManagerRebuildEventArgs**
* **NotifyStateManagerChangedAction.Add** 和 **NotifyStateManagerChangedAction.Remove**：**NotifyStateManagerSingleEntityChangedEventArgs**

以下是範例 **StateManagerChanged** 通知處理常式。

```csharp
public void OnStateManagerChangedHandler(object sender, NotifyStateManagerChangedEventArgs e)
{
    if (e.Action == NotifyStateManagerChangedAction.Rebuild)
    {
        this.ProcessStateManagerRebuildNotification(e);

        return;
    }

    this.ProcessStateManagerSingleEntityNotification(e);
}
```

## <a name="reliable-dictionary-notifications"></a>可靠的字典通知
可靠的字典提供下列事件的通知︰

* 重建︰在 **ReliableDictionary** 從過去復原或複製的本機狀態或備份，復原其狀態時呼叫。
* 清除︰在透過 **ClearAsync** 方法清除 **ReliableDictionary** 的狀態時呼叫。
* 加入︰在已將項目加入至 **ReliableDictionary**時呼叫。
* 更新︰在已更新 **IReliableDictionary** 中的項目時呼叫。
* 移除︰在已刪除 **IReliableDictionary** 中的項目時呼叫。

若要註冊可靠的字典通知，使用者必須在 **IReliableDictionary** 上註冊事件處理常式 **DictionaryChanged**。 註冊這些事件處理常式的常見位置是在 **ReliableStateManager.StateManagerChanged** 加入通知中。
在將 **IReliableDictionary** 加入 **IReliableStateManager** 時註冊，可確保您不會錯過任何通知。

```csharp
private void ProcessStateManagerSingleEntityNotification(NotifyStateManagerChangedEventArgs e)
{
    var operation = e as NotifyStateManagerSingleEntityChangedEventArgs;

    if (operation.Action == NotifyStateManagerChangedAction.Add)
    {
        if (operation.ReliableState is IReliableDictionary<TKey, TValue>)
        {
            var dictionary = (IReliableDictionary<TKey, TValue>)operation.ReliableState;
            dictionary.RebuildNotificationAsyncCallback = this.OnDictionaryRebuildNotificationHandlerAsync;
            dictionary.DictionaryChanged += this.OnDictionaryChangedHandler;
        }
    }
}
```

> [!NOTE]
> **ProcessStateManagerSingleEntityNotification** 是前述 **OnStateManagerChangedHandler** 範例所呼叫的範例方法。
> 
> 

上述程式碼會設定 **IReliableNotificationAsyncCallback** 介面以及 **DictionaryChanged**。 由於 **NotifyDictionaryRebuildEventArgs** 包含需要以非同步方式列舉的 **IAsyncEnumerable** 介面，因此會透過 **RebuildNotificationAsyncCallback** 而不是 **OnDictionaryChangedHandler** 來引發重建通知。

```csharp
public async Task OnDictionaryRebuildNotificationHandlerAsync(
    IReliableDictionary<TKey, TValue> origin,
    NotifyDictionaryRebuildEventArgs<TKey, TValue> rebuildNotification)
{
    this.secondaryIndex.Clear();

    var enumerator = e.State.GetAsyncEnumerator();
    while (await enumerator.MoveNextAsync(CancellationToken.None))
    {
        this.secondaryIndex.Add(enumerator.Current.Key, enumerator.Current.Value);
    }
}
```

> [!NOTE]
> 在上述程式碼中，於處理重建通知的過程中，會先清除所維護的彙總狀態。 因為正在利用新狀態重建可靠的集合，因此與先前的所有通知無關。
> 
> 

**DictionaryChanged** 事件處理常式會使用 **NotifyDictionaryChangedEventArgs** 來提供關於事件的詳細資料。
**NotifyDictionaryChangedEventArgs** 有五個子類別。 使用 **NotifyDictionaryChangedEventArgs** 中的 Action 屬性將 **NotifyDictionaryChangedEventArgs** 轉換為正確的子類別：

* **NotifyDictionaryChangedAction.Rebuild**：**NotifyDictionaryRebuildEventArgs**
* **NotifyDictionaryChangedAction.Clear**：**NotifyDictionaryClearEventArgs**
* **NotifyDictionaryChangedAction.Add** 和 **NotifyDictionaryChangedAction.Remove**：**NotifyDictionaryItemAddedEventArgs**
* **NotifyDictionaryChangedAction.Update**：**NotifyDictionaryItemUpdatedEventArgs**
* **NotifyDictionaryChangedAction.Remove**：**NotifyDictionaryItemRemovedEventArgs**

```csharp
public void OnDictionaryChangedHandler(object sender, NotifyDictionaryChangedEventArgs<TKey, TValue> e)
{
    switch (e.Action)
    {
        case NotifyDictionaryChangedAction.Clear:
            var clearEvent = e as NotifyDictionaryClearEventArgs<TKey, TValue>;
            this.ProcessClearNotification(clearEvent);
            return;

        case NotifyDictionaryChangedAction.Add:
            var addEvent = e as NotifyDictionaryItemAddedEventArgs<TKey, TValue>;
            this.ProcessAddNotification(addEvent);
            return;

        case NotifyDictionaryChangedAction.Update:
            var updateEvent = e as NotifyDictionaryItemUpdatedEventArgs<TKey, TValue>;
            this.ProcessUpdateNotification(updateEvent);
            return;

        case NotifyDictionaryChangedAction.Remove:
            var deleteEvent = e as NotifyDictionaryItemRemovedEventArgs<TKey, TValue>;
            this.ProcessRemoveNotification(deleteEvent);
            return;

        default:
            break;
    }
}
```

## <a name="recommendations"></a>建議
*  完成通知事件。
*  執行任何耗費資源的作業 (例如 IO 作業) 做為同步事件的一部分。
*  Action 類型。 未來可能會加入新的 Action 類型。

以下是要牢記在心的一些事項：

* 通知會在執行作業過程中引發。 例如，在還原作業的最後一個步驟引發還原通知。 處理通知事件之前，不會完成還原。
* 因為通知是在套用作業過程中引發，因此，用戶端只會看見本機認可作業的通知。 而且因為作業只保證會在本機認可 (亦即記錄)，所以它們不一定可在未來復原。
* 在取消復原路徑上，會針對每個套用的作業引發單一通知。 這表示，如果交易 T1 包含 Create(X)、Delete(X)、Create(X)，您將依序得到一個針對 X 建立的通知、一個針對刪除的通知，然後再收到一個建立的通知。
* 如果是包含多個作業的交易，作業將依使用者在主要本上收到它們的順序套用。
* 在處理錯誤的進度過程中，某些作業可能會復原。 通知會針對這類復原作業加以引發，將複本狀態輪換回到可靠的時間點。 復原通知的一個重要差異，是使用重複索引鍵的事件會彙總在一起。 例如，如果復原上述的 T1，使用者將會看到 Delete(X) 的單一通知。

## <a name="next-steps"></a>後續步驟
* [可靠的集合](service-fabric-work-with-reliable-collections.md)
* [Reliable Services 快速入門](service-fabric-reliable-services-quick-start.md)
* [備份與還原 Reliable Services (災害復原)](service-fabric-reliable-services-backup-restore.md)
* [可靠的集合的開發人員參考資料](https://msdn.microsoft.com/library/azure/microsoft.servicefabric.data.collections.aspx)

