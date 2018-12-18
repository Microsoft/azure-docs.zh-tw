---
title: Azure Cosmos DB 資料表 API 簡介 | Microsoft Docs
description: 了解如何透過受歡迎的 OSS MongoDB API，使用 Azure Cosmos DB 來儲存及查詢大量機碼值資料 (低延遲)。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: na
ms.topic: overview
ms.date: 11/20/2017
ms.author: sngun
ms.openlocfilehash: 225811195ffa01ce26f51fdbb78ee567c747c3d1
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43282961"
---
# <a name="introduction-to-azure-cosmos-db-table-api"></a>Azure Cosmos DB 簡介：資料表 API

[Azure Cosmos DB](introduction.md) 提供的表格 API 適用於針對 Azure 資料表儲存體所撰寫、且需要進階功能的應用程式，例如：

* [周全的全域發佈](distribute-data-globally.md)。
* 全球[專用的輸送量](partition-data.md)。
* 99 百分位數的單一數字毫秒延遲。
* 保證高可用性。
* [自動次要索引](http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf)。

針對 Azure 資料表儲存體所撰寫的應用程式可使用資料表 API (不變更程式碼) 來移轉至 Azure Cosmos DB，並且利用進階功能。 資料表 API 具有適用於 .NET、Java、Python 和 Node.js 的用戶端 SDK。

## <a name="table-offerings"></a>表格供應項目
如果您目前是使用 Azure 資料表儲存體，移至 Azure Cosmos DB 資料表 API 可獲得下列好處：

| | Azure 資料表儲存體 | Azure Cosmos DB 資料表 API |
| --- | --- | --- |
| Latency | 快速，但延遲沒有上限。 | 一位數毫秒的讀取和寫入延遲，並在世界各地支援任何規模的 <10 毫秒延遲讀取和 <15 毫秒延遲寫入 (第 99 個百分位數)。 |
| Throughput | 變數輸送量模型。 資料表每秒 20,000 個作業的延展性限制。 | 高延展性且[每個資料表都有專用的保留輸送量](request-units.md) (由 SLA 支援)。 帳戶沒有輸送量上限，而且支援每個資料表每秒 > 1 千萬個作業。 |
| 全球發佈 | 具有一個選擇性高可用性可讀取次要讀取區域的單一區域。 您無法起始容錯移轉。 | [周全的全域發佈](distribute-data-globally.md)介於 1 到 30+ 個區域。 隨時隨地在世界各地支援[自動和手動容錯移轉](regional-failover.md)。 |
| 編製索引 | PartitionKey 和 RowKey 只有主要索引。 沒有次要索引。 | 對所有屬性自動執行完整的編製索引，但沒有索引管理。 |
| 查詢 | 查詢執行作業會使用主索引鍵的索引，要不然會進行掃描。 | 查詢可以利用自動編製屬性的索引，加快查詢速度。 |
| 一致性 | 主要區域內的強式。 次要區域內的事件式。 | [五個定義完善的一致性層級](consistency-levels.md)，可以您應用程式的需求作為基礎，進行可用性、延遲、輸送量及一致性的取捨。 |
| 價格 | 儲存體最佳化。 | 輸送量最佳化。 |
| SLA | 99.99% 可用性。 | 99.99% 可用性 SLA 適用於一致性很寬鬆的所有單一區域帳戶和所有多重區域帳戶，而所有多重區域資料庫帳戶有 99.999% 的讀取可用性[領先業界的全方位 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/) (公開上市)。 |

## <a name="get-started"></a>開始使用

在 [Azure 入口網站](https://portal.azure.com)中建立 Azure Cosmos DB 帳戶。 然後開始使用我們的[使用 .NET 快速入門資料表 API](create-table-dotnet.md)。 

> [!IMPORTANT]
> 如果您在預覽期間建立了資料表 API 帳戶，請建立[新表格 API 帳戶](create-table-dotnet.md#create-a-database-account)以使用正式推出的資料表 API SDK。
>

## <a name="next-steps"></a>後續步驟

以下是可讓您快速入門的一些指標：
* [使用資料表 API 來建置 .NET 應用程式](create-table-dotnet.md)
* [在 .NET 中利用資料表 API 進行開發](tutorial-develop-table-dotnet.md)
* [使用資料表 API 來查詢資料表資料](tutorial-query-table.md)
* [了解如何使用資料表 API 來設定 Azure Cosmos DB 全域發佈](tutorial-global-distribution-table.md)
* [Azure Cosmos DB 資料表 .NET API](table-sdk-dotnet.md)
* [Azure Cosmos DB 資料表 Java API](table-sdk-java.md)
* [Azure Cosmos DB 資料表 Node.js API](table-sdk-nodejs.md)
* [適用於 Python 的 Azure Cosmos DB 資料表](table-sdk-python.md)

