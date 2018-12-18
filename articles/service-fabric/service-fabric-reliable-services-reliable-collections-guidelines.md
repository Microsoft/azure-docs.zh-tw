---
title: Azure Service Fabric 中 Reliable Collection 的指導方針與建議 | Microsoft Docs
description: 使用 Service Fabric Reliable Collection 的指導方針與建議
services: service-fabric
documentationcenter: .net
author: mcoskun
manager: timlt
editor: masnider,rajak,zhol
ms.assetid: 62857523-604b-434e-bd1c-2141ea4b00d1
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 12/10/2017
ms.author: mcoskun
ms.openlocfilehash: 72091d592475e41f254ef7c3882e3d3fee0c491b
ms.sourcegitcommit: e3d5de6d784eb6a8268bd6d51f10b265e0619e47
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39389526"
---
# <a name="guidelines-and-recommendations-for-reliable-collections-in-azure-service-fabric"></a>Azure Service Fabric 中 Reliable Collection 的指導方針與建議
本節提供使用 Reliable State Manager 和 Reliable Collection 的指導方針。 目標是要協助使用者避開常見的陷阱。

指導方針會編排成簡單的建議，前面再加上「請」、「請考慮」、「請避免」或「請不要」字詞。

* 請不要修改讀取作業所傳回自訂類型的物件 (例如 `TryPeekAsync` 或 `TryGetValueAsync`)。 可靠的集合就像並行的集合一樣，會傳回物件參考而不是複本。
* 請不要在未經修改之前，就深層複製傳回的自訂類型物件。 由於結構和內建類型都是傳值 (pass-by-value)，因此您不需要在其上執行深層複製，除非其包含您想要修改的參考類型欄位或屬性。
* 請不要針對逾時使用 `TimeSpan.MaxValue` 。 逾時應該用來偵測死結。
* 在認可、中止或處置交易之後，請勿使用該交易。
* 請不要在其所建立的交易範圍之外使用列舉。
* 請不要在另一個交易的 `using` 陳述式內建立交易，因為它會造成死結。
* 務必確保 `IComparable<TKey>` 實作是正確的。 系統會對 `IComparable<TKey>` 採取相依性以合併檢查點與資料列。
* 請不要在讀取需更新的項目時使用更新鎖定，以避免發生特定類別的死結。
* 請考慮將每個分割區可靠的集合數目保持小於 1000。 偏好具有較多項目可靠集合，勝過於具有較少項目的可靠集合。
* 請考慮將項目 (例如「可靠的字典」的 Tkey 和 TValue) 保持低於 80 KB (越小越好)。 這會減少大型物件堆積的使用量，並降低磁碟和網路 IO 需求。 通常這會在只更新一小部分的值時，減少重複資料的複寫次數。 在「可靠的字典」中達成此目的的常用方式，是將資料列分成多個資料列。
* 請考慮使用備份和還原功能以擁有災害復原。
* 避免在同一個交易中因為不同的隔離等級混用單一實體作業和多實體作業 (例如 `GetCountAsync`、`CreateEnumerableAsync`)。
* 請務必處理 InvalidOperationException。 系統有可能因為各種原因而中止使用者交易。 例如，當 Reliable State Manager 正由「主要」角色切換，或長時間執行的交易封鎖交易記錄檔的截斷時。 在這種情況下，使用者可能會收到 InvalidOperationException，指出他們的交易已終止。 假設終止交易不是使用者所要求，處理這個例外狀況的最佳方式是處置交易，請檢查是否已通知取消權杖 (或已變更複本的角色)，如果沒有，請建立新的交易並重試。  

以下是要牢記在心的一些事項：

* 所有可靠的集合 API 的預設逾時都是四秒。 大部分使用者應該使用的預設逾時。
* 在所有可靠的集合 API 中，預設的取消權杖為 `CancellationToken.None` 。
* 可靠字典的索引鍵類型參數 (*TKey*) 必須正確實作 `GetHashCode()` 和 `Equals()`。 索引鍵必須是不可變的。
* 若要讓可靠的集合達到高度可用性，每個服務應至少有一個目標和大小為 3 的最小複本集。
* 次要複本上的讀取作業可能會讀取未認可的仲裁版本。
  這表示從單一次要複本讀取的資料版本進度可能有誤。
  從主要複本讀取一定最穩定︰進度一定不會有誤。
* 應用程式保存在可靠集合中的資料會有何種安全性/隱私權將由您決定，且會受到儲存體管理的保護；也就是說， 作業系統磁碟加密可用來保護待用資料。  

### <a name="next-steps"></a>後續步驟
* [使用 Reliable Collections](service-fabric-work-with-reliable-collections.md)
* [交易和鎖定](service-fabric-reliable-services-reliable-collections-transactions-locks.md)
* 管理資料
  * [備份與還原](service-fabric-reliable-services-backup-restore.md)
  * [通知](service-fabric-reliable-services-notifications.md)
  * [序列化與升級](service-fabric-application-upgrade-data-serialization.md)
  * [Reliable State Manager 組態](service-fabric-reliable-services-configuration.md)
* 其他
  * [Reliable Services 快速入門](service-fabric-reliable-services-quick-start.md)
  * [可靠的集合的開發人員參考資料](https://msdn.microsoft.com/library/azure/microsoft.servicefabric.data.collections.aspx)
