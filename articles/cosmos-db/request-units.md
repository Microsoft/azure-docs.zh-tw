---
title: Azure Cosmos DB 中的輸送量和效能貨幣要求單位
description: 了解如何在 Azure Cosmos DB 中指定和預估要求單位需求
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/19/2020
ms.openlocfilehash: 6831cb3f39c25eb69d16300156f456980cf57fa0
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/19/2020
ms.locfileid: "88604832"
---
# <a name="request-units-in-azure-cosmos-db"></a>Azure Cosmos DB 中的要求單位

Azure Cosmos DB 支援許多 API，例如 SQL、MongoDB、Cassandra、Gremlin 和資料表。 每個 API 都有它自己的一組資料庫作業。 這些作業的範圍可從簡單的點讀取和寫入到複雜查詢。 每個資料庫作業都會根據作業的複雜度來取用系統資源。

所有資料庫作業的成本會由 Azure Cosmos DB 進行正規化，並以 *要求單位* （縮寫為 ru ）。 您可以將 ru 視為效能貨幣，以抽象化執行 Azure Cosmos DB 所支援之資料庫作業所需的系統資源（例如 CPU、IOPS 和記憶體）。

進行點讀取的成本 (亦即，1 KB 專案的識別碼和分割區索引鍵值) 的單一專案是1個要求單位 (或 1 RU) 。 所有其他資料庫作業同樣會使用 RU 來指派成本。 無論您使用哪種 API 與 Azure Cosmos 容器互動，成本始終由 RU 測量。 無論資料庫作業是寫入、點讀取或查詢，成本一律會以 ru 來測量。

下圖顯示 RU 的高階概念：

:::image type="content" source="./media/request-units/request-units.png" alt-text="資料庫作業會取用要求單位" border="false":::

為了管理和規劃容量，Azure Cosmos DB 會確保指定資料集上指定資料庫作業的 RU 數目具有決定性。 您可以檢查回應標頭來追蹤任何資料庫作業所取用的 RU 數目。 當您瞭解 [影響 RU 費用的因素](request-units.md#request-unit-considerations) 以及應用程式的輸送量需求時，您可以有效地執行您的應用程式成本。

您所使用的 Azure Cosmos 帳戶類型會決定取用 ru 的計費方式：

- 在 [布建的輸送量](set-throughput.md) 模式中，您會以每秒為單位布建應用程式的 ru 數目，以每秒 100 ru 為單位遞增。 若要調整應用程式的布建輸送量，您可以隨時增加或減少 ru 數目（以 100 ru 的遞增或遞減）。 您可藉由程式設計方式或使用 Azure 入口網站來進行變更。 您會針對每秒布建的 ru 量，以每小時為單位計費。 您可以在兩個不同的資料粒度上佈建輸送量：
  - **容器**：如需詳細資訊，請參閱 [在 Azure Cosmos 容器上](how-to-provision-container-throughput.md)布建輸送量。
  - **資料庫**：如需詳細資訊，請參閱 [在 Azure Cosmos 資料庫上](how-to-provision-database-throughput.md)布建輸送量。
- 在 [無伺服器](serverless.md) 模式中，在 Azure Cosmos 帳戶中建立資源時，您不需要布建任何輸送量。 在您的計費期間結束時，您會收到資料庫作業所耗用的要求單位數量的費用。

## <a name="request-unit-considerations"></a>要求單位的考量

當您估計工作負載所耗用的 ru 數目時，請考慮下列因素：

* **項目大小**：當項目大小增加時，讀取或寫入項目所耗用的 RU 數目也會增加。

* **項目索引**：根據預設，每個項目都會自動編製索引。 如果您選擇不要編製容器中部分項目的索引，則會耗用較少的 RU。

* **項目屬性計數**：假設所有屬性上都有預設編製索引，則取用來寫入項目的 RU 數目會隨著項目屬性計數增加而增加。

* **已編製索引的屬性**：每個容器上之索引原則會判定預設要編製哪些索引的屬性。 若要減少寫入作業的 RU 耗用量，請限制已編製索引的屬性數目。

* **資料一致性**：強式和限定過期一致性層級在執行讀取作業時，會耗用大約兩倍以上的 ru （相較于其他寬鬆一致性層級）。

* **讀取的類型**：點讀取成本明顯比查詢更少的 ru。

* **查詢模式**：查詢的複雜性會影響針對作業所耗用的要求單位數目。 影響查詢作業成本的因素包括： 
    
    - 查詢結果數目
    - 述詞數目
    - 述詞性質
    - 使用者定義函式數目
    - 來源資料的大小
    - 結果集的大小
    - 投影

  Azure Cosmos DB 保證相同資料上相同查詢的成本一律會與重複執行上 RU 數目相同。

* **腳本使用**方式：如同查詢，預存程式和觸發程式會根據所執行作業的複雜度來取用 ru。 開發應用程式時，請檢查[要求費用標頭](optimize-cost-queries.md#evaluate-request-unit-charge-for-a-query)，以便進一步了解每個作業要取用多少 RU 容量。

## <a name="next-steps"></a>後續步驟

* 深入了解如何[在 Azure Cosmos 容器和資料庫上佈建輸送量](set-throughput.md)。
* 深入瞭解 [無伺服器的 Azure Cosmos DB](serverless.md)。
* 深入了解[邏輯分割區](partition-data.md)。
* 深入了解如何[全域調整佈建的輸送量](scaling-throughput.md)。
* 了解如何[在 Azure Cosmos 容器上佈建輸送量](how-to-provision-container-throughput.md)。
* 了解如何[在 Azure Cosmos 資料庫上佈建輸送量](how-to-provision-database-throughput.md)。
* 瞭解如何 [找出作業的要求單位費用](find-request-unit-charge.md)。
* 瞭解如何 [在 Azure Cosmos DB 中將布建的輸送量成本優化](optimize-cost-throughput.md)。
* 瞭解如何將 [Azure Cosmos DB 中的讀取和寫入成本優化](optimize-cost-reads-writes.md)。
* 瞭解如何將 [Azure Cosmos DB 中的查詢成本優化](optimize-cost-queries.md)。
* 瞭解如何 [使用計量來監視輸送量](use-metrics.md)。
