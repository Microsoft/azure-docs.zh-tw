---
title: 包含檔案
description: 包含檔案
services: storage
author: MarkMcGeeAtAquent
ms.service: storage
ms.topic: include
ms.date: 04/06/2018
ms.author: mimig
ms.custom: include file
ms.openlocfilehash: 76e1a22e02ad6b8161ec18f3806c88ffc54fe123
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "34665373"
---
如果您目前是使用 Azure 資料表儲存體，移至 Azure Cosmos DB 資料表 API 可獲得下列好處：

| | Azure 資料表儲存體 | Azure Cosmos DB 資料表 API |
| --- | --- | --- |
| Latency | 快速，但延遲沒有上限。 | 一位數毫秒的讀取和寫入延遲，並在世界各地支援任何規模的 <10 毫秒延遲讀取和 <15 毫秒延遲寫入 (第 99 個百分位數)。 |
| Throughput | 變數輸送量模型。 資料表每秒 20,000 個作業的延展性限制。 | 高延展性且[每個資料表都有專用的保留輸送量](../articles/cosmos-db/request-units.md) (由 SLA 支援)。 帳戶沒有輸送量上限，而且支援每個資料表每秒 > 1 千萬個作業。 |
| 全球發佈 | 具有一個選擇性高可用性可讀取次要讀取區域的單一區域。 您無法起始容錯移轉。 | [周全的全域發佈](../articles/cosmos-db/distribute-data-globally.md)介於 1 到 30+ 個區域。 隨時隨地在世界各地支援[自動和手動容錯移轉](../articles/cosmos-db/regional-failover.md)。 |
| 編製索引 | PartitionKey 和 RowKey 只有主要索引。 沒有次要索引。 | 對所有屬性自動執行完整的編製索引，但沒有索引管理。 |
| 查詢 | 查詢執行作業會使用主索引鍵的索引，要不然會進行掃描。 | 查詢可以利用自動編製屬性的索引，加快查詢速度。 |
| 一致性 | 主要區域內的強式。 次要區域內的事件式。 | [五個定義完善的一致性層級](../articles/cosmos-db/consistency-levels.md)，可以您應用程式的需求作為基礎，進行可用性、延遲、輸送量及一致性的取捨。 |
| 價格 | 儲存體最佳化。 | 輸送量最佳化。 |
| SLA | 99.99% 可用性。 | 99.99% 可用性 SLA 適用於一致性很寬鬆的所有單一區域帳戶和所有多重區域帳戶，而所有多重區域資料庫帳戶有 99.999% 的讀取可用性[領先業界的全方位 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/) (公開上市)。 |
