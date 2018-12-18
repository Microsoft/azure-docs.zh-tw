---
title: Azure Service Fabric 中的 ReliableConcurrentQueue
description: ReliableConcurrentQueue 是高輸送量佇列，可進行平行加入佇列以及清除佇列。
services: service-fabric
documentationcenter: .net
author: sangarg
manager: timlt
editor: raja,tyadam,masnider,vturecek
ms.assetid: 62857523-604b-434e-bd1c-2141ea4b00d1
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 5/1/2017
ms.author: sangarg
ms.openlocfilehash: e04123f7870921a2979564d0f6c68424d4d7711c
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34206572"
---
# <a name="introduction-to-reliableconcurrentqueue-in-azure-service-fabric"></a>Azure Service Fabric 中的 ReliableConcurrentQueue 簡介
可靠的並行佇列是非同步、交易式和複寫的佇列，特徵是加入佇列與清除佇列作業的高並行存取。 它旨在提供高輸送量和低延遲，方法是將[可靠的佇列](https://msdn.microsoft.com/library/azure/dn971527.aspx)所提供的嚴格 FIFO 順序放寬，並改為提供最佳的順序。

## <a name="apis"></a>API

|並行佇列                |可靠的並行佇列                                         |
|--------------------------------|------------------------------------------------------------------|
| void Enqueue(T item)           | Task EnqueueAsync(ITransaction tx, T item)                       |
| bool TryDequeue(out T result)  | Task< ConditionalValue < T > > TryDequeueAsync(ITransaction tx)  |
| int Count()                    | long Count()                                                     |

## <a name="comparison-with-reliable-queuehttpsmsdnmicrosoftcomlibraryazuredn971527aspx"></a>與[可靠的佇列](https://msdn.microsoft.com/library/azure/dn971527.aspx)進行比較

可靠的並行佇列會以[可靠的佇列](https://msdn.microsoft.com/library/azure/dn971527.aspx)替代形式提供。 它應用於不需要嚴格 FIFO 順序的情況，因為保證 FIFO 使用並行存取時需要有所取捨。  [可靠的佇列](https://msdn.microsoft.com/library/azure/dn971527.aspx)會使用鎖定來強制使用 FIFO 順序，並且一次最多允許一個交易加入佇列，以及一次最多允許一個交易清除佇列。 相較之下，可靠的並行佇列會放寬排序條件約束，並允許任何數目的並行交易可交錯其加入佇列及清除佇列作業。 會提供最佳順序，不過，一律無法保證可靠並行佇列中兩個值的相對順序。

每當多個並行交易執行加入佇列或清除佇列時，可靠的並行佇列會提供比[可靠的佇列](https://msdn.microsoft.com/library/azure/dn971527.aspx)更高的輸送量和更低的延遲。

ReliableConcurrentQueue 的範例使用情況為[訊息佇列](https://en.wikipedia.org/wiki/Message_queue)案例。 在此案例中，一個或多個訊息生產者會建立項目並加以新增至佇列，而一個或多個訊息取用者則會從佇列中提取訊息並加以處理。 多個產生者和取用者可以獨立運作，使用並行交易以處理佇列。

## <a name="usage-guidelines"></a>使用方針
* 佇列會預期佇列中的項目具有低保留期限。 也就是項目在佇列中不會長時間停留。
* 佇列並不保證嚴格的 FIFO 順序。
* 佇列不會閱讀它自己的寫入。 如果項目在交易內加入佇列，則同一個交易內的清除佇列者將看不到它。
* 清除佇列不會彼此互相隔離。 如果已在交易 txnA 中將項目 A 清除佇列，則即使 txnA 不認可，並行交易 txnB 也看不到項目 A。  如果 txnA 中止，txnB 就可立即看到 A。
* 可將 TryPeekAsync 行為加以實作，方法是使用 TryDequeueAsync，然後中止交易。 程式設計模式一節中可以找到這個範例。
* 計數為非交易式。 它可用來了解佇列中元素的數目，但會以時間點表示且無法依賴。
* 交易使用中時，不應在清除佇列的項目上執行昂貴的處理，以避免長時間執行交易可能會對系統造成的效能影響。

## <a name="code-snippets"></a>程式碼片段
讓我們看看幾個程式碼片段，和其預期的輸出。 本節中會忽略例外狀況處理。

### <a name="enqueueasync"></a>EnqueueAsync
以下是使用 EnqueueAsync 的幾個程式碼片段，隨後是其預期的輸出。

- 案例 1︰單一加入佇列工作

```
using (var txn = this.StateManager.CreateTransaction())
{
    await this.Queue.EnqueueAsync(txn, 10, cancellationToken);
    await this.Queue.EnqueueAsync(txn, 20, cancellationToken);

    await txn.CommitAsync();
}
```

假設已順利完成工作，且沒有修改佇列的並行交易。 使用者可以預期佇列會以下列任何一個順序來包含項目︰

> 10, 20

> 20, 10


- 案例 2︰平行加入佇列工作

```
// Parallel Task 1
using (var txn = this.StateManager.CreateTransaction())
{
    await this.Queue.EnqueueAsync(txn, 10, cancellationToken);
    await this.Queue.EnqueueAsync(txn, 20, cancellationToken);

    await txn.CommitAsync();
}

// Parallel Task 2
using (var txn = this.StateManager.CreateTransaction())
{
    await this.Queue.EnqueueAsync(txn, 30, cancellationToken);
    await this.Queue.EnqueueAsync(txn, 40, cancellationToken);

    await txn.CommitAsync();
}
```

假設已順利完成工作、工作以平行方式執行，且沒有其他修改佇列的並行交易。 無法推斷佇列中的項目順序。 此程式碼片段中，項目會以下列 4 個之一出現！ 可能的排序。  佇列會嘗試以原始順序保留項目 (佇列)，但可能會因並行作業或錯誤而強制將它們重新排序。


### <a name="dequeueasync"></a>DequeueAsync
以下是使用 TryDequeueAsync 的幾個程式碼片段，隨後是預期的輸出。 假設已將佇列中的下列項目填入佇列︰
> 10, 20, 30, 40, 50, 60

- 案例 1︰單一清除佇列工作

```
using (var txn = this.StateManager.CreateTransaction())
{
    await this.Queue.TryDequeueAsync(txn, cancellationToken);
    await this.Queue.TryDequeueAsync(txn, cancellationToken);
    await this.Queue.TryDequeueAsync(txn, cancellationToken);

    await txn.CommitAsync();
}
```

假設已順利完成工作，且沒有修改佇列的並行交易。 由於無法推斷佇列中的項目順序，任何三個項目都可能會以任何順序加以清除佇列。 佇列會嘗試以原始順序保留項目 (佇列)，但可能會因並行作業或錯誤而強制將它們重新排序。  

- 案例 2︰平行清除佇列工作

```
// Parallel Task 1
List<int> dequeue1;
using (var txn = this.StateManager.CreateTransaction())
{
    dequeue1.Add(await this.Queue.TryDequeueAsync(txn, cancellationToken)).val;
    dequeue1.Add(await this.Queue.TryDequeueAsync(txn, cancellationToken)).val;

    await txn.CommitAsync();
}

// Parallel Task 2
List<int> dequeue2;
using (var txn = this.StateManager.CreateTransaction())
{
    dequeue2.Add(await this.Queue.TryDequeueAsync(txn, cancellationToken)).val;
    dequeue2.Add(await this.Queue.TryDequeueAsync(txn, cancellationToken)).val;

    await txn.CommitAsync();
}
```

假設已順利完成工作、工作以平行方式執行，且沒有其他修改佇列的並行交易。 由於無法推斷佇列中的項目順序，dequeue1 和 dequeue2 都會以任何順序包含任何兩個項目。

相同項目不會同時出現在兩份清單中。 因此，如果 dequeue1 包含 10、*30*，dequeue2 就會包含 *20*、*40*。

- 案例 3︰使用交易中止來清除佇列排序

使用執行中的清除佇列將交易中止，會將項目放回佇列的前端。 無法保證將項目放回佇列前端的順序。 我們來看看下面的程式碼：

```
using (var txn = this.StateManager.CreateTransaction())
{
    await this.Queue.TryDequeueAsync(txn, cancellationToken);
    await this.Queue.TryDequeueAsync(txn, cancellationToken);

    // Abort the transaction
    await txn.AbortAsync();
}
```
假設已依下列順序將項目清除佇列︰
> 10, 20

當我們將交易中止時，會依下列任何一個順序將項目新增回佇列的前端︰
> 10, 20

> 20, 10

這也適用於交易未成功認可的所有情況。

## <a name="programming-patterns"></a>程式設計模式
在本節中，我們來看看幾個可能有助於使用 ReliableConcurrentQueue 的程式設計模式。

### <a name="batch-dequeues"></a>批次清除佇列
建議的程式設計模式是使取用者工作以批次方式清除佇列，而不是一次執行一個清除佇列。 使用者可以選擇節流處理每個批次或批次大小之間的延遲。 下列程式碼片段會示範此程式設計模型。  請注意，在此範例中，交易認可後即完成處理，因此如果在處理時發生錯誤，未處理的項目就會遺失而未經處理。  或者，可以在交易的範圍內完成處理，不過，這可能會對效能造成負面影響，而且需要處理已經處理的項目。

```
int batchSize = 5;
long delayMs = 100;

while(!cancellationToken.IsCancellationRequested)
{
    // Buffer for dequeued items
    List<int> processItems = new List<int>();

    using (var txn = this.StateManager.CreateTransaction())
    {
        ConditionalValue<int> ret;

        for(int i = 0; i < batchSize; ++i)
        {
            ret = await this.Queue.TryDequeueAsync(txn, cancellationToken);

            if (ret.HasValue)
            {
                // If an item was dequeued, add to the buffer for processing
                processItems.Add(ret.Value);
            }
            else
            {
                // else break the for loop
                break;
            }
        }

        await txn.CommitAsync();
    }

    // Process the dequeues
    for (int i = 0; i < processItems.Count; ++i)
    {
        Console.WriteLine("Value : " + processItems[i]);
    }

    int delayFactor = batchSize - processItems.Count;
    await Task.Delay(TimeSpan.FromMilliseconds(delayMs * delayFactor), cancellationToken);
}
```

### <a name="best-effort-notification-based-processing"></a>以通知作為基礎的最佳處理
另一個有趣的程式設計模式是使用計數 API。 在這裡，我們可以針對佇列實作以通知作為基礎的最佳處理。 佇列計數可以用來將加入佇列或清除佇列工作進行節流處理。  請注意，如同先前的範例，因為處理發生於交易外部，如果在處理期間發生錯誤，未處理的項目可能就會遺失。

```
int threshold = 5;
long delayMs = 1000;

while(!cancellationToken.IsCancellationRequested)
{
    while (this.Queue.Count < threshold)
    {
        cancellationToken.ThrowIfCancellationRequested();

        // If the queue does not have the threshold number of items, delay the task and check again
        await Task.Delay(TimeSpan.FromMilliseconds(delayMs), cancellationToken);
    }

    // If there are approximately threshold number of items, try and process the queue

    // Buffer for dequeued items
    List<int> processItems = new List<int>();

    using (var txn = this.StateManager.CreateTransaction())
    {
        ConditionalValue<int> ret;

        do
        {
            ret = await this.Queue.TryDequeueAsync(txn, cancellationToken);

            if (ret.HasValue)
            {
                // If an item was dequeued, add to the buffer for processing
                processItems.Add(ret.Value);
            }
        } while (processItems.Count < threshold && ret.HasValue);

        await txn.CommitAsync();
    }

    // Process the dequeues
    for (int i = 0; i < processItems.Count; ++i)
    {
        Console.WriteLine("Value : " + processItems[i]);
    }
}
```

### <a name="best-effort-drain"></a>盡可能清空
由於資料結構的並行本質，無法保證可將佇列清空。  即使在佇列上沒有進行中的使用者作業，特定的 TryDequeueAsync 呼叫可能不會傳回先前已加入佇列並且受到認可的項目。  清除佇列最終保證可看到加入佇列的項目，不過，沒有超出訊號範圍通訊機制的獨立取用者，無法得知佇列已觸達穩定狀態，即使已將所有的產生者停止，且不允許任何新的加入佇列作業亦然。 因此，已盡可能清空作業，實作如下。

使用者應該將所有進一步的生產者和取用者工作停止，並在嘗試清空佇列之前，等待任何進行中的交易加以認可或中止。  如果使用者知道佇列中預期的項目數目，就可以設定通知，發出所有項目皆已清除佇列的訊號。

```
int numItemsDequeued;
int batchSize = 5;

ConditionalValue ret;

do
{
    List<int> processItems = new List<int>();

    using (var txn = this.StateManager.CreateTransaction())
    {
        do
        {
            ret = await this.Queue.TryDequeueAsync(txn, cancellationToken);

            if(ret.HasValue)
            {
                // Buffer the dequeues
                processItems.Add(ret.Value);
            }
        } while (ret.HasValue && processItems.Count < batchSize);

        await txn.CommitAsync();
    }

    // Process the dequeues
    for (int i = 0; i < processItems.Count; ++i)
    {
        Console.WriteLine("Value : " + processItems[i]);
    }
} while (ret.HasValue);
```

### <a name="peek"></a>預覽
ReliableConcurrentQueue 不會提供 TryPeekAsync API。 使用者可以預覽語意，方法是使用 TryDequeueAsync 然後將交易中止。 在此範例中，僅在項目大於 10 時，才會處理清除佇列。

```
using (var txn = this.StateManager.CreateTransaction())
{
    ConditionalValue ret = await this.Queue.TryDequeueAsync(txn, cancellationToken);
    bool valueProcessed = false;

    if (ret.HasValue)
    {
        if (ret.Value > 10)
        {
            // Process the item
            Console.WriteLine("Value : " + ret.Value);
            valueProcessed = true;
        }
    }

    if (valueProcessed)
    {
        await txn.CommitAsync();    
    }
    else
    {
        await txn.AbortAsync();
    }
}
```

## <a name="must-read"></a>必須閱讀
* [Reliable Services 快速入門](service-fabric-reliable-services-quick-start.md)
* [使用可靠的集合](service-fabric-work-with-reliable-collections.md)
* [Reliable Services 通知](service-fabric-reliable-services-notifications.md)
* [備份與還原 Reliable Services (災害復原)](service-fabric-reliable-services-backup-restore.md)
* [Reliable State Manager 設定](service-fabric-reliable-services-configuration.md)
* [開始使用 Service Fabric Web API 服務](service-fabric-reliable-services-communication-webapi.md)
* [Reliable Services 程式設計模型的進階用法](service-fabric-reliable-services-advanced-usage.md)
* [可靠集合的開發人員參考資料](https://msdn.microsoft.com/library/azure/microsoft.servicefabric.data.collections.aspx)
