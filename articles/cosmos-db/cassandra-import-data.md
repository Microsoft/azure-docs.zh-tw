---
title: 將您的資料移轉至 Azure Cosmos DB Cassandra API 帳戶
description: 了解如何使用 CQL Copy 命令和 Spark 將資料從 Apache Cassandra 複製到 Azure Cosmos DB Cassandra API。
services: cosmos-db
author: kanshiG
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.author: govindk
ms.topic: tutorial
ms.date: 09/24/2018
ms.reviewer: sngun
ms.openlocfilehash: f73a201a25bb2f975e8a261a6c21aa7b066c3a7c
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48247845"
---
# <a name="migrate-your-data-to-azure-cosmos-db-cassandra-api-account"></a>將您的資料移轉至 Azure Cosmos DB Cassandra API 帳戶

本教學課程提供有關如何將 Apache Cassandra 資料移轉至 Azure Cosmos DB Cassandra API 的指示。 

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 為移轉做規劃
> * 移轉的必要條件
> * 使用 cqlsh COPY 命令來移轉資料
> * 使用 Spark 來移轉資料 

## <a name="plan-for-migration"></a>為移轉做規劃

將資料移轉至 Azure Cosmos DB Cassandra API 之前，您應該預估您工作負載的輸送量需求。 一般而言，建議您從 CRUD 作業所需的平均輸送量開始著手，然後再納入「擷取、轉換、載入」(ETL) 或棘手作業所需的額外輸送量。 您需要有下列詳細資料，才能為移轉進行規劃： 

* **現有資料大小或預估資料大小：** 定義最基本的資料庫大小和輸送量需求。 如果您要為新應用程式預估資料大小，您可以假設資料平均分散至所有資料列，然後藉由乘以資料大小來預估值。 

* **所需的輸送量：** 大約的讀取 (查詢/取得) 和寫入 (更新/刪除/插入) 輸送量速度。 必須要有此值，才能計算所需要求單位及穩定的狀態資料大小。  

* **取得結構描述：** 透過 cqlsh 連線至您現有的 Cassandra 叢集，然後從 Cassandra 匯出結構描述： 

  ```bash
  cqlsh [IP] "-e DESC SCHEMA" > orig_schema.cql
  ```

在識別現有工作負載的需求之後，您應該根據所收集的輸送量需求，建立 Azure Cosmos DB 帳戶、資料庫及容器。  

* **判斷作業的 RU 費用：** 您可以使用所選擇的 Azure Cosmos DB Cassandra API SDK 來判斷 RU。 此範例會示範取得 RU 費用的 .NET 版本。

  ```csharp
  var tableInsertStatement = table.Insert(sampleEntity);
  var insertResult = await tableInsertStatement.ExecuteAsync();

  foreach (string key in insertResult.Info.IncomingPayload)
    {
       byte[] valueInBytes = customPayload[key];
       string value = Encoding.UTF8.GetString(valueInBytes);
       Console.WriteLine($"CustomPayload:  {key}: {value}");
    }
  ```

* **配置所需的輸送量：** Azure Cosmos DB 可以依據您的需求成長，自動調整儲存體和輸送量。 您可以使用 [Azure Cosmos DB 要求單位計算機](https://www.documentdb.com/capacityplanner)來評估您的輸送量需求。 

## <a name="prerequisites-for-migration"></a>移轉的必要條件

* **在 Azure Cosmos DB Cassandra API 帳戶中建立資料表：** 在您開始移轉資料之前，請先從 Azure 入口網站或從 cqlsh 預先建立所有資料表。 如果您要遷移至具有資料庫層級輸送量的 Azure Cosmos DB 帳戶，請務必在建立 Azure Cosmos DB 容器時提供分割區索引鍵。

* **增加輸送量︰** 資料移轉持續時間取決於您為 Azure Cosmos DB 中資料表佈建的輸送量多寡。 請針對移轉持續時間增加輸送量。 藉由較高的輸送量，您可以避免速率限制，並花費較少的時間進行移轉。 完成移轉之後，再降低輸送量以節省成本。 如需在有關增加輸送量的詳細資訊，請參閱為 Azure Cosmos DB 容器[設定輸送量](set-throughput.md)。 此外，也建議您使用相同區域中的 Azure Cosmos DB 帳戶作為您的來源資料庫。 

* **啟用 SSL：** Azure Cosmos DB 具有嚴格的安全性需求和標準。 與您的帳戶互動時，請務必啟用 SSL。 當您透過 SSH 使用 CQL 時，會有選項要您提供 SSL 資訊。

## <a name="options-to-migrate-data"></a>移轉資料的選項

您可以使用下列選項，將資料從現有的 Cassandra 工作負載移至 Azure Cosmos DB：

* [使用 cqlsh COPY 命令](#using-cqlsh-copy-command)  
* [使用 Spark](#using-spark) 

## <a name="migrate-data-using-cqlsh-copy-command"></a>使用 cqlsh COPY 命令來移轉資料

[CQL COPY 命令](http://cassandra.apache.org/doc/latest/tools/cqlsh.html#cqlsh)可用來將本機資料複製到 Azure Cosmos DB Cassandra API 帳戶。 請使用下列步驟來複製資料：

1. 取得您 Cassandra API 帳戶的連接字串資訊：

   * 登入 [Azure 入口網站](https://portal.azure.com)，然後瀏覽至您的 Azure Cosmos DB 帳戶。

   * 開啟 [連接字串] 窗格，其中包含從 cqlsh 連線至 Cassandra API 帳戶所需的一切資訊。

2. 使用入口網站中的連線資訊來登入 cqlsh。

3. 使用 CQL COPY 命令將本機資料複製到 Cassandra API 帳戶。

   ```bash
   COPY exampleks.tablename FROM filefolderx/*.csv 
   ```

## <a name="migrate-data-using-spark"></a>使用 Spark 來移轉資料 

搭配 Spark 使用下列步驟，將資料移轉至 Azure Cosmos DB Cassandra API：

- 佈建一個 [Azure Databricks](cassandra-spark-databricks.md) 或 [HDInsight 叢集](cassandra-spark-hdinsight.md) 

- 使用[資料表複製作業](cassandra-spark-table-copy-ops.md)將資料移至目的地 Cassandra API 端點 

如果您的資料位於 Azure 虛擬機器或任何其他雲端的現有叢集中，則建議使用 Spark 作業來移轉資料。 這需要將 Spark 設定為一次性或定期擷取的中繼項目。 您可以使用內部部署環境與 Azure 之間的 Express Route 連線，來加速這項移轉。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何將資料移轉至 Azure Cosmos DB Cassandra API 帳戶。 您現在可以繼續進行＜概念＞一節，了解有關 Azure Cosmos DB 的詳細資訊。 

> [!div class="nextstepaction"]
> [Azure Cosmos DB 中的 Tunable 資料一致性層級](../cosmos-db/consistency-levels.md)


