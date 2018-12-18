---
title: Azure 資料總管資料擷取
description: 了解您可以在 Azure 資料總管中擷取 (載入) 資料的不同方式
services: data-explorer
author: orspod
ms.author: v-orspod
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 94f96d949f2a05f71e9565fdcbc7b48ed2c2a5c5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46972653"
---
# <a name="azure-data-explorer-data-ingestion"></a>Azure 資料總管資料擷取

資料擷取是從一個或多個來源載入資料記錄所使用的程序，以建立或更新 Azure 資料總管中的資料表。 一旦擷取之後，資料就會變成可供查詢。 下圖顯示在 Azure 資料總管中工作的端對端流程，包括資料擷取 **(2)**。

![整體資料流程](media/ingest-data-overview/overall-data-flow.png)

Azure 資料總管資料管理服務負責資料擷取，並提供下列功能：

1. **資料提取**：從外部來源 (事件中樞) 提取資料，或從 Azure 佇列讀取擷取要求。

1. **批次處理**：批次處理流向相同資料庫和資料表的資料，將擷取輸送量最佳化。

1. **驗證**：初步驗證和格式轉換 (如有必要)。

1. **資料操作**：比對結構描述、組織、編製索引、編碼以及壓縮資料。

1. **擷取流程中的持續點**：管理引擎上的擷取負載，並在暫時性失敗時處理重試。

1. **認可資料擷取**：讓資料可供查詢。

> [!NOTE]
> 已擷取資料的有效保留原則衍生自資料庫的保留原則。 如需詳細資料，請參閱[保留原則](https://docs.microsoft.com/azure/kusto/concepts/retentionpolicy)。 擷取資料需要有**資料表擷取器**或**資料庫擷取器**權限。

## <a name="ingestion-methods"></a>擷取方法

Azure 資料總管支援數種擷取方法，每種方法都有自己的目標案例、優點和缺點。 Azure 資料總管會針對常用的服務，使用 SDK 的程式設計擷取，以及用於探索之引擎的直接存取，提供連接器。

### <a name="ingestion-using-connectors"></a>使用連接器擷取

Azure 資料總管目前支援事件中樞連接器，可以使用 Azure 入口網站中的管理精靈來管理。 如需詳細資訊，請參閱[快速入門：將資料從事件中樞內嵌至 Azure 資料總管](ingest-data-event-hub.md)。

### <a name="programmatic-ingestion"></a>程式設計擷取

Azure 資料總管會提供可用於查詢和資料擷取的 SDK。 程式設計擷取適合用於將擷取程序期間和之後的儲存體交易降至最低，藉此降低擷取成本 (COG)。

**可用的 SDK 及開放原始碼專案**：

Kusto 提供的用戶端 SDK 可用來搭配下列項目擷取及查詢資料：

* [Python SDK](https://docs.microsoft.com/azure/kusto/api/python/kusto-python-client-library)

* [.NET SDK](https://docs.microsoft.com/azure/kusto/api/netfx/about-the-sdk)

* [Java SDK](https://docs.microsoft.com/azure/kusto/api/java/kusto-java-client-library)

* [REST API](https://docs.microsoft.com/azure/kusto/api/netfx/kusto-ingest-client-rest)

**程式設計擷取技術**：

* 將資料直接擷取到 Azure 資料總管引擎 (最適合探索和原型設計)：

  * **內嵌擷取**：包含頻內資料的控制命令 (.ingest inline) 可供臨機操作測試之用。

  * **從查詢擷取**：指向查詢結果的控制命令 (.set、.set-or-append、.set-or-replace)，用來產生報表或小型暫存資料表。

  * **從儲存體擷取**：資料儲存在外部 (例如 Azure Blob 儲存體) 的控制命令 (.ingest into) 允許有效大量擷取資料。

* 透過 Azure 資料總管資料管理服務 (高輸送量且可靠的擷取) 擷取資料

  * [**批次擷取**](https://docs.microsoft.com/azure/kusto/api/netfx/kusto-ingest-queued-ingest-sample) (由 SDK 提供)：用戶端將資料上傳至 Azure Blob 儲存體 (Azure 資料總管資料管理服務所指定)，並將通知張貼到 Azure 佇列。 這是針對大量、可靠且不費力的資料擷取所建議的技術。

**不同方法的延遲**：

| 方法 | Latency |
| --- | --- |
| **內嵌擷取** | 立即 |
| **從查詢擷取** | 查詢時間 + 處理時間 |
| **從儲存體擷取** | 下載時間 + 處理時間 |
| **已排入佇列的擷取** | 批次處理時間 + 處理時間 |
| |

處理時間取決於資料大小，通常少於幾秒鐘。 批次處理時間預設為 5 分鐘。

## <a name="choosing-the-most-appropriate-ingestion-method"></a>選擇最適合的擷取方法

開始擷取資料之前，您應該自問下列問題。

* 我的資料在哪裡？ 
* 什麼是資料格式？可以變更嗎？ 
* 要查詢的必填欄位有哪些？ 
* 預期的資料量和速率為何？ 
* 預期有多少事件類型 (稱為資料表數目)？ 
* 預期變更事件結構描述頻率為何？ 
* 有多少個節點將會產生資料？ 
* 來源作業系統是什麼？ 
* 延遲需求為何？ 
* 可以使用其中一個現有的受控擷取管線嗎？ 

對於具有以傳訊服務 (如事件中樞) 為基礎之現有基礎架構的組織而言，使用連接器可能是最適當的解決方案。 已排入佇列的擷取適合大量資料。

## <a name="supported-data-formats"></a>支援的資料格式

若是從查詢擷取以外的其他所有擷取方法，資料格式必須是下列其中一種支援的資料格式，Azure 資料總管才可以進行剖析。

* CSV、TSV、PSV、SCSV、SOH
* JSON (以行分隔、多行)、Avro
* ZIP 和 GZIP 

> [!NOTE]
> 正在擷取資料時，會根據目標資料表資料行推斷資料類型。 如果記錄不完整或無法將欄位剖析為所需的資料類型，則會將對應的資料表資料行填入 null 值。

## <a name="schema-mapping"></a>結構描述對應

結構描述對應有助於以確定的方式，將來源資料欄位繫結至目的地資料表資料行。

* [CSV 對應](https://docs.microsoft.com/azure/kusto/management/mappings?branch=master#csv-mapping) (選擇性) 適用於所有以序數為基礎的格式，而且可以當作擷取命令參數傳遞，或[在資料表上預先建立](https://docs.microsoft.com/azure/kusto/management/tables?branch=master#create-ingestion-mapping)，並參考自擷取命令參數。
* [JSON 對應](https://docs.microsoft.com/azure/kusto/management/mappings?branch=master#json-mapping) (強制性) 和 [Avro 對應](https://docs.microsoft.com/azure/kusto/management/mappings?branch=master#avro-mapping) (強制性) 可以當作擷取命令參數傳遞，或[在資料表上預先建立](https://docs.microsoft.com/azure/kusto/management/tables#create-ingestion-mapping)，並參考自擷取命令參數。

## <a name="next-steps"></a>後續步驟

[快速入門：將資料從事件中樞內嵌至 Azure 資料總管](ingest-data-event-hub.md)

[快速入門：使用 Azure 資料總管 Python 程式庫擷取資料](python-ingest-data.md)

