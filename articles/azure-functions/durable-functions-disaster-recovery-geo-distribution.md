---
title: Durable Functions 中的災難復原和地理分散 - Azure
description: 了解 Durable Functions 中的災難復原和地理分散。
services: functions
author: MS-Santi
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 04/25/2018
ms.author: azfuncdf
ms.openlocfilehash: bbfbf351b8976f4140b6dd98a9a54ba982c3d865
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39399392"
---
# <a name="disaster-recovery-and-geo-distribution"></a>災難復原和異地複寫

## <a name="overview"></a>概觀
在 Azure Durable Functions 中，所有狀態會持續保留在 Azure 儲存體中。 [工作中樞](durable-functions-task-hubs.md)是協調流程所使用之 Azure 儲存體資源的邏輯容器。 協調器和活動函式必須屬於相同的工作中樞，才能彼此互動。
所述的案例會提議部署選項，以在災難復原活動期間提升可用性和將停機時間降至最低。
請務必注意，這些案例是由 Azure 儲存體的使用量所支配，因此是以主動-被動組態為基礎。 此模式包含將備份 (被動) 函式應用程式部署至不同的區域。 流量管理員會監視主要 (主動) 函式應用程式的可用性。 它會在主要函式應用程式失敗時，容錯移轉至備份函式應用程式。 如需詳細資訊，請參閱[流量管理員](https://azure.microsoft.com/services/traffic-manager/)的[優先順序流量路由方法](../traffic-manager/traffic-manager-routing-methods.md#a-name--priorityapriority-traffic-routing-method)。


>[!NOTE]
>- 提議的主動-被動組態可確保用戶端永遠能透過 HTTP 觸發新的協調流程。 不過，由於讓兩個函式應用程式共用相同的儲存體，所以背景處理會分散於兩者之間，並且競相處理相同佇列上的訊息。 此組態會讓第二個函式應用程式產生附加輸出成本。
>- 基礎儲存體帳戶和工作中樞均建立於主要區域中，並由兩個函式應用程式所共用。
>- 在透過 HTTP 啟動的情況下，所有重複部署的函式應用程式都必須共用相同的函式存取金鑰。 Functions 執行階段會公開[管理 API](https://github.com/Azure/azure-functions-host/wiki/Key-management-API) ，讓取用者能以程式設計方式新增、刪除和更新功能鍵。

## <a name="scenario-1---load-balanced-compute-with-shared-storage"></a>案例 1 - 具有共用儲存體的負載平衡計算
如果 Azure 中的計算基礎結構失敗，函式應用程式可能會無法使用。 為了將這類停機的可能性降到最低，此案例會使用部署到不同區域的兩個函式應用程式。 流量管理員已設定為要偵測主要函式應用程式中的問題，並自動將流量重新導向至次要區域中的函式應用程式。 此函式應用程式會共用相同的 Azure 儲存體帳戶和工作中樞。 因此，函式應用程式的狀態並未遺失，而且工作可以正常繼續。 一旦健康情況還原到主要區域，Azure 流量管理員就會自動啟動對該函式應用程式的路由要求。


![顯示案例 1 的圖表。](media/durable-functions-disaster-recovery-geo-distribution/durable-functions-geo-scenario01.png)

使用此部署案例時有幾個好處：
- 如果計算基礎結構失敗，工作可以在容錯移轉區域中繼續，而不會遺失狀態。
- 流量管理員會自動負責自動容錯移轉至狀況良好的函式應用程式。
- 流量管理員會在中斷情況解決之後，自動重新建立對主要函式應用程式的流量。

不過，使用此案例時，請考慮：
- 如果使用專用的 App Service 方案部署函式應用程式，則在容錯移轉資料中心複寫計算基礎結構會增加成本。
- 此案例涵蓋計算基礎結構的中斷，但儲存體帳戶仍然會是函式應用程式的單一失敗點。 如果沒有儲存體中斷，則應用程式會遭遇停機狀況。
- 如果已容錯移轉函式應用程式，則會增加延遲時間，因為它會跨區域存取其儲存體帳戶。
- 從不同的所在區域存取儲存體服務，會因為網路傳出流量而造成較高的成本。
- 此案例取決於流量管理員。 考量到[流量管理員的運作方式](../traffic-manager/traffic-manager-how-it-works.md)，在取用 Durable Function 的用戶端應用程式需要從流量管理員再次查詢函式應用程式位址之前，可能會經過一些時間。 


## <a name="scenario-2---load-balanced-compute-with-regional-storage"></a>案例 2 - 具有區域性儲存體的負載平衡計算
先前的案例只涵蓋計算基礎結構中的失敗案例。 如果儲存體服務失敗，它會導致函式應用程式中斷。
為了確保 Durable Functions 能夠連續作業，此案例會使用已部署函式應用程式的每個區域的本機儲存體帳戶。

![顯示案例 2 的圖表。](media/durable-functions-disaster-recovery-geo-distribution/durable-functions-geo-scenario02.png)

這種方法可改善前一個案例：
- 如果函式應用程式失敗，則流量管理員會負責容錯移轉到次要區域。 不過，因為函式應用程式依賴自己的儲存體帳戶，所以 Durable Functions 會繼續運作。
- 在容錯移轉期間，容錯移轉區域沒有額外的延遲，因為函式應用程式與儲存體帳戶位於相同位置。
- 儲存層失敗會導致 Durable Functions 發生失敗，進而會觸發重新導向至容錯移轉區域。 同樣地，因為函式應用程式和儲存體依區域隔離，所以 Durable Functions 會繼續運作。
 
此案例的重要考量：
- 如果使用專用的 App Service 方案部署函式應用程式，則在容錯移轉資料中心複寫計算基礎結構會增加成本。
- 目前狀態並未容錯移轉，這表示執行和檢查點函式將會失敗。 由用戶端應用程式決定是否重試/重新啟動工作。

## <a name="scenario-3---load-balanced-compute-with-grs-shared-storage"></a>案例 3 - 具有 GRS 共用儲存體的負載平衡計算
此案例是修改自第一個案例，並實作共用儲存體帳戶。 主要差異在於建立的儲存體帳戶已啟用異地複寫。
在功能上，此案例提供與案例 1 相同的優點，但它還有額外的資料復原優點：
- 異地備援儲存體 (GRS) 和讀取權限異地備援儲存體 (RA-GRS) 可為儲存體帳戶提供最大的可用性。
- 如果儲存體服務發生區域中斷，其中一個可能值是資料中心作業判斷儲存體必須容錯移轉到次要區域。 在此情況下，儲存體帳戶存取權會以透明方式重新導向至儲存體帳戶的異地複寫副本，而不需使用者介入。
- 在此情況下，Durable Functions 的狀態將會保留到儲存體帳戶的最後一次複寫 (每隔幾分鐘發生一次)。
如同其他案例，此案例有一些重要考量：
- 容錯移轉至複本是由資料中心操作員進行，可能需要一些時間。 在這段時間內，函式應用程式會發生中斷。
- 使用異地複寫儲存體帳戶的成本於是增加。
- GRS 會以非同步方式發生。 有些最新交易可能會因為複寫程序延遲而遺失。

![顯示案例 3 的圖表。](media/durable-functions-disaster-recovery-geo-distribution/durable-functions-geo-scenario03.png)


## <a name="next-steps"></a>後續步驟

您可以深入了解如何[使用 RA-GRS 設計高可用性應用程式](../storage/common/storage-designing-ha-apps-with-ragrs.md)
