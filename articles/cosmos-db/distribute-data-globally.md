---
title: 使用 Azure Cosmos DB 全域散發資料 | Microsoft Docs
description: 了解如何從 Azure Cosmos DB (全域散發的多模型資料庫服務)，使用全域資料庫進行全球規模的異地複寫、多重主機、容錯移轉及資料復原。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: mjbrown
ms.openlocfilehash: 227c243d82665dc533e3bfa6a1fe3e9bb775a262
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47408890"
---
# <a name="global-data-distribution-with-azure-cosmos-db"></a>透過 Azure Cosmos DB 進行全域資料散發

Azure 無所不在，它跨 50 多個地理區域，遍佈全球並持續擴充中。 遍佈全球且具有多重主機支援的 Azure 提供給開發人員的其中一項獨特功能，就是能夠輕鬆地建置、部署及管理分散在世界各地的應用程式。

[Azure Cosmos DB](../cosmos-db/introduction.md) 是 Microsoft 全域發佈的多模型資料庫服務，適用於任務關鍵性應用程式。 Azure Cosmos DB 提供周全的全域散發、全球[可彈性調整的輸送量和儲存體](../cosmos-db/partition-data.md)、讀取和寫入在 99% 的情況下延遲均僅有幾毫秒、[定義完善的一致性模型](consistency-levels.md)，以及保證的高可用性，全部都由[領先業界的全方位 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/) 所支援。 Azure Cosmos DB 會[自動編製所有資料的索引](http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf) \(英文\)，您不需要處理結構描述或管理索引。

## <a name="global-distribution-with-multi-master"></a>全域散發與多重主機

作為雲端服務，Azure Cosmos DB 經由縝密的設計，可為文件、索引鍵/值、圖形和資料行系列資料模型提供多租用戶、全域散發和多重主機的支援。

![分割並散發在三個區域的 Azure Cosmos DB 容器](./media/distribute-data-globally/global-apps.png)

**分割並散發在多個 Azure 區域的單一 Azure Cosmos DB 容器**

如我們在建置 Azure Cosmos DB 時所知，全域散發功能並無法於事後添加。 它無法「另外加裝」在「單一網站」的資料庫系統上。 全域分散式資料庫提供的功能遠遠超越「單一網站」資料庫具備的傳統地理災害復原 (Geo-DR) 所提供的功能。 提供 Geo-DR 功能的單一網站資料庫是分散在世界各地的資料庫嚴格子集。

藉著 Azure Cosmos DB 周全的全域散發功能，有些開發人員會為資料庫記錄採用 Lambda 模式，或者在多個區域間「重複寫入」，藉此省去自行建置複寫結構的步驟。 但這類方法並不能確保正確性，也無法提供健全的 SLA，因此我們不建議使用。

## <a id="Next Steps"></a>後續步驟

* [Azure Cosmos DB 如何啟用周全的全域散發](distribute-data-globally-turnkey.md)

* [Azure Cosmos DB 全域散發主要優點](distribute-data-globally-benefits.md)

* [如何設定 Azure Cosmos DB 全域資料庫複寫](tutorial-global-distribution-sql-api.md)

* [如何為 Azure Cosmos DB 帳戶啟用多重主機功能](enable-multi-master.md)

* [自動化與手動容錯移轉在Azure Cosmos DB 中的運作方式](regional-failover.md)

* [了解 Azure Cosmos DB 中的衝突解決方法](multi-master-conflict-resolution.md)

* [搭配開放原始碼 NoSQL 資料庫使用多重主機](multi-master-oss-nosql.md)
