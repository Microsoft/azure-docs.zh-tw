---
title: Azure Service Fabric Reliable Collections 中的交易和鎖定模式 | Microsoft Docs
description: Azure Service Fabric Reliable State Manager 和 Reliable Collections 交易和鎖定。
services: service-fabric
documentationcenter: .net
author: mcoskun
manager: timlt
editor: masnider,rajak
ms.assetid: 62857523-604b-434e-bd1c-2141ea4b00d1
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 5/1/2017
ms.author: mcoskun
ms.openlocfilehash: 79be861a70abb0331d971b00e753691e77642637
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34207361"
---
# <a name="transactions-and-lock-modes-in-azure-service-fabric-reliable-collections"></a>Azure Service Fabric Reliable Collections 中的交易和鎖定模式

## <a name="transaction"></a>交易
交易就是以單一工作邏輯單元執行的一連串作業。
交易必須顯現下列 ACID 屬性。 (請參閱：https://technet.microsoft.com/library/ms190612)
* **不可部分完成性**︰交易必須是不可部分完成的工作單位。 換句話說，執行其所有資料修改，或完全不執行。
* **一致性**︰交易完成時，所有資料必須維持一致的狀態。 所有內部資料結構在交易結束時必須是正確的。
* **隔離**︰並行交易所做的修改，必須與任何其他並行交易所做的修改隔離。 ITransaction 內的作業所用的隔離等級是由執行此作業的 IReliableState 所決定。
* **耐久性**：交易完成之後，其作用會永久存在系統中。 即使發生系統失敗仍會保存修改。

### <a name="isolation-levels"></a>隔離層級
隔離層級定義交易必須與其他交易所做的修改隔離的程度。
可靠的集合支援兩種隔離等級：

* **可重複讀取**：指定陳述式無法讀取已經修改但尚未由其他交易確認的資料，以及指定在目前的交易完成之前，任何其他交易都不能修改已經由目前交易讀取的資料。 如需詳細資料，請參閱 [https://msdn.microsoft.com/library/ms173763.aspx](https://msdn.microsoft.com/library/ms173763.aspx)。
* **快照集**：指定交易中任何陳述式所讀取的資料都會與交易開始時就存在的資料版本一致。
  交易只能辨識交易開始之前所認可的資料修改。
  在目前交易中執行的陳述式無法看到在目前交易開始之後，其他交易所進行的資料修改。
  效果就如同交易中的陳述式會取得認可資料的快照集，因為這項資料於交易開始時就存在。
  可靠的集合的快照集都是一致的。
  如需詳細資料，請參閱 [https://msdn.microsoft.com/library/ms173763.aspx](https://msdn.microsoft.com/library/ms173763.aspx)。

可靠的集合會依據交易建立時的作業和複本角色，自動選擇要用於指定讀取作業的隔離層級。
下表說明可靠的字典和佇列作業的隔離等級預設值。

| 作業 \ 角色 | 主要 | 次要 |
| --- |:--- |:--- |
| 單一實體讀取 |可重複讀取 |快照 |
| 列舉、計數 |快照 |快照 |

> [!NOTE]
> 單一實體作業的常見範例包括 `IReliableDictionary.TryGetValueAsync`、`IReliableQueue.TryPeekAsync`。
> 

可靠的字典和可靠的佇列皆支援「讀寫一致性」(Read Your Writes)。
換句話說，在屬於相同交易的下列讀取作業皆可看到交易內的任何寫入作業。

## <a name="locks"></a>鎖定
在 Reliable Collections 中，所有交易都會實作嚴格的兩階段鎖定：在以中止或認可來終止交易之前，交易不會釋放它所取得的鎖定。

可靠的字典會針對所有單一實體作業使用資料列層級鎖定。
可靠的佇列則會針對嚴格交易的 FIFO 屬性交換並行。
可靠的佇列會使用作業層級的鎖定，允許一次有 `TryPeekAsync` 和/或 `TryDequeueAsync` 的一個交易，以及有 `EnqueueAsync` 的一個交易。
請注意，為維持 FIFO，如果 `TryPeekAsync` 或 `TryDequeueAsync` 曾觀察到可靠的佇列是空的，則它們也會鎖定 `EnqueueAsync`。

寫入作業一律會採取「獨佔」鎖定。
讀取作業的鎖定則取決於一些因素。
任何使用快照隔離所完成的讀取作業都是無鎖定的。
任何可重複讀取作業預設都會採用共用鎖定。
不過，針對支援可重複讀取的任何讀取作業，使用者可以要求更新鎖定，而不是共用鎖定。
更新鎖定是一種非對稱式鎖定，用來避免常見的死結，這些死結通常會在多個交易鎖定資源稍後進行可能更新時發生。

下表中可找到鎖定相容性矩陣：

| 要求 \ 授與 | None | 共用 | 更新 | 獨佔 |
| --- |:--- |:--- |:--- |:--- |
| 共用 |無衝突 |無衝突 |衝突 |衝突 |
| 更新 |無衝突 |無衝突 |衝突 |衝突 |
| 獨佔 |無衝突 |衝突 |衝突 |衝突 |

Reliable Collections API 中的逾時引數是用來進行死結偵測。
例如，有兩筆交易 (T1 和 T2) 嘗試讀取和更新 K1。
這樣很可能會形成死結，因為它們最後都會有共用鎖定。
在這種情況下，其中一個或兩個作業將會逾時。

此死結案例就是更新鎖定可如何防止死結的絕佳範例。

## <a name="next-steps"></a>後續步驟
* [使用可靠的集合](service-fabric-work-with-reliable-collections.md)
* [Reliable Services 通知](service-fabric-reliable-services-notifications.md)
* [備份與還原 Reliable Services (災害復原)](service-fabric-reliable-services-backup-restore.md)
* [Reliable State Manager 組態](service-fabric-reliable-services-configuration.md)
* [可靠的集合的開發人員參考資料](https://msdn.microsoft.com/library/azure/microsoft.servicefabric.data.collections.aspx)

