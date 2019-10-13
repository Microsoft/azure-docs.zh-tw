---
title: 包含檔案
description: 包含檔案
services: data-factory
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 5/30/2019
ms.author: jingwang
ms.custom: include file
ms.openlocfilehash: 6dc846ab6af0ed4d6d48c6f6db0abbce06427c36
ms.sourcegitcommit: 11265f4ff9f8e727a0cbf2af20a8057f5923ccda
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2019
ms.locfileid: "72035878"
---
Azure Data Factory 是多租使用者服務，具有下列預設限制，以確保客戶訂用帳戶會受到彼此的工作負載保護。 若要提高訂用帳戶的最大限制，請聯絡支援人員。

### <a name="version-2"></a>第 2 版

| Resource | 預設限制 | 上限 |
| -------- | ------------- | ------------- |
| Azure 訂用帳戶中的Data Factory數量 | 50 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 資料處理站內的實體（例如管線、資料集、觸發程式、連結的服務和整合執行時間）總數 | 5,000 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 一個訂用帳戶下的 Azure SSIS 整合執行時間的總 CPU 核心數 | 256 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 每個資料處理站的並行管線執行會在 factory 中的所有管線之間共用 | 10,000  | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 每個訂用帳戶每個[Azure Integration Runtime 區域](../articles/data-factory/concepts-integration-runtime.md#integration-runtime-location)的並行外部活動執行<br><small>外部活動是在整合執行時間上管理，但在連結的服務上執行，包括 Databricks、預存程式、HDInsights 和其他。</small> | 3000 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 每個訂用帳戶每個[Azure Integration Runtime 區域](../articles/data-factory/concepts-integration-runtime.md#integration-runtime-location)的並行管線活動執行 <br><small>管線活動會在整合執行時間上執行，包括 Lookup、GetMetadata 和 Delete。</small>| 1000 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 每個訂用帳戶每個[Azure Integration Runtime 區域](../articles/data-factory/concepts-integration-runtime.md#integration-runtime-location)的並行撰寫作業<br><small>Including 測試連接、流覽資料夾清單和資料表清單、預覽資料。 | 200 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 並行資料整合單位<sup>1</sup>每個訂用帳戶每個[Azure Integration Runtime 區域](../articles/data-factory/concepts-integration-runtime.md#integration-runtime-location)的耗用量| 區域群組 1<sup>2</sup>：6000<br>區域群組 2<sup>2</sup>：3000<br>區域群組 3<sup>2</sup>：1500 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 每個管線的活動上限，包括容器的內部活動 | 40 | 40 |
| 可以針對單一自我裝載整合執行時間建立之連結整合執行時間的最大數目 | 100 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 每個管線的參數上限 | 50 | 50 |
| ForEach 項目 | 100,000 | 100,000 |
| ForEach 平行處理原則 | 20 | 50 |
| 每個運算式的字元數 | 8,192 | 8,192 |
| 最小輪轉視窗觸發程式間隔 | 15 分鐘 | 15 分鐘 |
| 管線活動執行的最大超時 | 7 天 | 7 天 |
| 管線物件的每個物件位元組<sup>3</sup> | 200 KB | 200 KB |
| 資料集和已連結服務物件的每個物件位元組數<sup>3</sup> | 100 KB | 2000 KB |
| 每次複製活動執行的資料整合單位<sup>1</sup> | 256 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 寫入 API 呼叫 | 1200/h<br/><br/> 這是由 Azure Resource Manager 所加諸的限制，而不是 Azure Data Factory。 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 讀取 API 呼叫 | 12500/h<br/><br/> 這是由 Azure Resource Manager 所加諸的限制，而不是 Azure Data Factory。 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 監視每分鐘查詢次數 | 1,000 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 每分鐘的實體 CRUD 作業 | 50 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 資料流程 debug 會話的最長時間 | 8小時 | 8小時 |
| 每個 factory 的並行資料流程數 | 50 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 每個處理站的每個使用者的資料流程多個資料流程 debug 會話數目 | 3 | 3 |
| 資料流程 Azure IR TTL 限制 | 4小時 | [連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |

<sup>1</sup>資料整合單位（DIU）用於雲端到雲端複製作業，請從[資料整合單位（第2版）](../articles/data-factory/copy-activity-performance.md#data-integration-units)深入瞭解。 如需計費的詳細資訊，請參閱[Azure Data Factory 定價](https://azure.microsoft.com/pricing/details/data-factory/)。

<sup>2</sup> [Azure Integration Runtime](../articles/data-factory/concepts-integration-runtime.md#azure-integration-runtime) [全域可用](https://azure.microsoft.com/global-infrastructure/services/)以確保資料合規性、效率，並降低網路輸出成本。 

| 區域群組 | Regions | 
| -------- | ------ |
| 區域群組1 | 美國中部、美國東部、東部美國2、北歐、西歐、美國西部、美國西部2 |
| 區域群組2 | 澳大利亞東部、澳大利亞東南部、巴西南部、印度中部、日本東部、Northcentral US、中南部 US、東南亞、美國中西部 |
| 區域群組3 | 加拿大中部、東亞、法國中部、韓國中部、英國南部 |

<sup>3</sup>管線、資料集和連結的服務物件代表您工作負載的邏輯群組。 這些物件的限制與您可以使用 Azure Data Factory 移動和處理的資料量無關。 Data Factory 的設計目的是要調整以處理數 pb 的資料。

### <a name="version-1"></a>第 1 版

| **Resource** | **預設限制** | **上限** |
| --- | --- | --- |
| Azure 訂用帳戶中的資料處理站 |50 |[連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 資料處理站內的管線 |2,500 |[連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| Data factory 內的資料集 |5,000 |[連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 每個資料集的並行配量 |10 |10 |
| 管線物件的每個物件位元組<sup>1</sup> |200 KB |200 KB |
| 資料集和連結服務物件的每個物件位元組數<sup>1</sup> |100 KB |2000 KB |
| Azure HDInsight 訂用帳戶中的隨選叢集核心<sup>2</sup> |60 |[連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 每次複製活動執行的雲端資料移動單位<sup>3</sup> |32 |[連絡客戶支援](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。 |
| 管線活動執行的重試計數 |1,000 |MaxInt (32 位元) |

<sup>1</sup>管線、資料集和連結的服務物件代表您工作負載的邏輯群組。 這些物件的限制與您可以使用 Azure Data Factory 移動和處理的資料量無關。 Data Factory 的設計目的是要調整以處理數 pb 的資料。

<sup>2</sup> 隨選 HDInsight 核心並非配置在包含資料處理站的訂用帳戶之中。 因此，先前的限制是隨選 HDInsight 核心的 Data Factory 強制核心限制。 不同于與您的 Azure 訂用帳戶相關聯的核心限制。

<sup>3 第</sup>1 版的雲端資料移動單位（DMU）用於雲端到雲端複製作業，請從[雲端資料移動單位（第1版）](../articles/data-factory/v1/data-factory-copy-activity-performance.md#cloud-data-movement-units)深入瞭解。 如需計費的詳細資訊，請參閱[Azure Data Factory 定價](https://azure.microsoft.com/pricing/details/data-factory/)。

| **Resource** | **預設下限** | **下限** |
| --- | --- | --- |
| 排程間隔 |15 分鐘 |15 分鐘 |
| 重試嘗試間的間隔 |1 秒 |1 秒 |
| 重試逾時值 |1 秒 |1 秒 |

#### <a name="web-service-call-limits"></a>Web 服務呼叫限制
Azure Resource Manager 有 API 呼叫限制。 您可使用 [Azure 資源管理員 API 限制](../articles/azure-subscription-service-limits.md#resource-group-limits)內的速率進行 API 呼叫。
