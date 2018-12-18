---
title: Azure Cosmos DB 全域散發的主要優點
description: 深入了解 Azure Cosmos DB 多重主機、異地複寫提供的主要優點、它能發揮效用的多重主機和使用案例。
services: cosmos-db
author: markjbrown
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: mjbrown
ms.reviewer: sngun
ms.openlocfilehash: 11d70a648bfc1882753688e5352835b04cee6b9f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46968336"
---
# <a name="distribute-data-globally-with-azure-cosmos-db"></a>使用 Azure Cosmos DB 全域散發資料

此文章說明將資料散發到全域的主要優點，以及一些需要全域資料散發的即時案例。

## <a name="key-benefits"></a>主要優點

下列是利用 Azure Cosmos DB 全域資料散發功能之帳戶可取得的主要優點：

* **一位數的延遲** - Azure Cosmos DB 提供在第 99 百分位數小於 10 毫秒的讀取和寫入延遲，由[含補償方案的 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/) 擔保。

* **5-9 的可用性** - Azure Cosmos DB 提供 99.999% 的讀取和寫入可用性，由[含補償方案的 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/) 擔保。

* **彈性的衝突解決** - Azure Cosmos DB 為利用多重主機功能的客戶提供三種處理衝突的模式，以確保全域資料的完整性與一致性。

* **可調式一致性** - Azure Cosmos DB 支援 5 個將全域分散納入考量的不同[一致性層級](consistency-levels.md)，針對有多重主機功能的帳戶不但全部支援，而且還是使用強式一致性。

* **隱含容錯**  - 透過跨多個區域複寫的資料，應用程式可以享有對區域性中斷的高容錯。

## <a name="use-cases"></a>使用案例

以下只是可以利用全域散發功能 Azure Cosmos DB 的一小部分案例。

* **社交媒體應用程式** - 社交媒體應用程式需要低延遲、高可用性與可調整性，以提供全球使用者絕佳體驗。

* **IoT**  - 地理位置分散邊緣部署通常需要追蹤來自多個位置的時間序列資料。 每個裝置都可以設定到最接近它的區域。 當裝置移動到其他區域時，那些裝置可重新設定成寫入到最接近的可用區域。

* **電子商務**  - 電子商務需要非常低的延遲與高可用性。 含讀取和寫入的資料異地分散會將資料放置到最接近使用者的位置，以增加應用程式的回應性。 在多個區域都能讀取和寫入的能力提供更好的可用性。

* **詐騙/異常偵測**  - 監視使用者活動或帳戶活動的應用程式通常都是全域散發且必須同時追蹤數個事件，以更新分數來維持風險計量。

* **計量**  - 使用量 (例如 API 呼叫、每秒交易數、使用分鐘數) 的計算和控管，可以使用 Azure Cosmos DB 多重主機輕鬆地全域實作。 內建的衝突解決功能可即時兼顧計數和法規的正確性。

## <a name="next-steps"></a>後續步驟  

在此文章中，您已了解 Azure Cosmos DB 的全域資料散發功能的主要優點與使用案例。 接著，請參閱下列資源：

* [Azure Cosmos DB 如何啟用周全的全域散發](distribute-data-globally-turnkey.md)

* [如何設定 Azure Cosmos DB 全域資料庫複寫](tutorial-global-distribution-sql-api.md)

* [如何為 Azure Cosmos DB 帳戶啟用多重主機功能](enable-multi-master.md)

* [自動化與手動容錯移轉在 Azure Cosmos DB 中的運作方式](regional-failover.md)

* [了解 Azure Cosmos DB 中的衝突解決方法](multi-master-conflict-resolution.md)

* [搭配開放原始碼 NoSQL 資料庫使用多重主機](multi-master-oss-nosql.md)
