---
title: Azure 搜尋中的服務限制 | Microsoft Docs
description: 容量計劃中使用的服務限制，以及 Azure 搜尋服務要求和回應的最大限制。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/24/2018
ms.author: heidist
ms.openlocfilehash: c24cccde507873424e3c51d584f5cd094df2b876
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34641164"
---
# <a name="service-limits-in-azure-search"></a>Azure 搜尋中的服務限制
儲存體與工作負載的最大限制，以及索引、文件和其他物件的數量上限，皆取決於您是在**免費**、**基本**，還是**標準**定價層中[佈建 Azure 搜尋服務](search-create-service-portal.md)。

+ **免費** 的是 Azure 訂用帳戶隨附的多租用戶共用服務。

+ **基本**會針對生產環境工作負載提供較小規模的專用計算資源。

+ **標準**是在專用的機器上執行，在各層級具有更多的儲存和處理容量。 標準共有四個等級︰S1、S2、S3 及 S3 HD。

  S3 高密度 (S3 HD) 是針對特定工作負載所設計：[多租用戶](search-modeling-multitenant-saas-applications.md)和大量的小型索引 (每個索引有 1 百萬份文件，每個服務有三千個索引)。 這一層不提供[索引子功能](search-indexer-overview.md)。 在 S3 HD 上，資料擷取必須利用推送方法，使用 API 呼叫以將資料從來源推送到索引。 

> [!NOTE]
> 服務會佈建在特定層。 跨層以取得容量牽涉到佈建新服務 (未提供就地升級)。 如需詳細資訊，請參閱[選擇 SKU 或階層](search-sku-tier.md)。 若要深入了解如何在已佈建的服務內調整容量，請參閱[調整適用於查詢和編製索引工作負載的資源等級](search-capacity-planning.md)。
>

## <a name="subscription-limits"></a>訂用帳戶限制
[!INCLUDE [azure-search-limits-per-subscription](../../includes/azure-search-limits-per-subscription.md)]

## <a name="storage-limits"></a>儲存體限制
[!INCLUDE [azure-search-limits-per-service](../../includes/azure-search-limits-per-service.md)]

<a name="index-limits"></a>

## <a name="index-limits"></a>索引限制

| 資源 | 免費 | 基本&nbsp;<sup>1</sup>  | S1 | S2 | S3 | S3&nbsp;HD |
| -------- | ---- | ------------------- | --- | --- | --- | --- |
| 索引上限 |3 |5 或 15 |50 |200 |200 |每個分割區 1000 個或每個服務 3000 個 |
| 每個索引的欄位上限 |1000 |100 |1000 |1000 |1000 |1000 |
| 每個索引的[建議工具](https://docs.microsoft.com/rest/api/searchservice/suggesters)上限 |1 |1 |1 |1 |1 |1 |
| 每個索引的[評分設定檔](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)上限 |100 |100 |100 |100 |100 |100 |
| 每個設定檔的函式上限 |8 |8 |8 |8 |8 |8 |

<sup>1</sup> 在 2017 年底之後建立的基本服務已將限制提高為 15 個索引、資料來源和索引子。 稍早建立的服務有 5 個。 基本層是唯一具有較低限制 (每個索引 100 個欄位) 的 SKU。

<a name="document-limits"></a>

## <a name="document-limits"></a>文件限制 

在大部分區域中，Azure 搜尋服務定價層 (基本、S1、S2、S3、S3 HD) 對於 2017 年 11 月/12 月以後建立的所有服務，具有無限的文件計數。 這一節會識別適用限制的區域，以及如何判斷您的服務是否受影響。 

若要判斷您的服務是否有文件限制，請檢查服務概觀頁面中的 [使用量] 圖格。 文件計數無限制，或受制於以層為基礎的限制。

  ![使用量圖格](media/search-limits-quotas-capacity/portal-usage-tile.png)

### <a name="regions-and-services-having-document-limits"></a>區域和服務具有文件限制

具有限制的服務是在 2017 年底以前建立，或是在使用較低容量叢集來裝載 Azure 搜尋服務的資料中心上執行。 受影響的資料中心位於下列區域：

+ 澳洲東部
+ 東亞
+ 印度中部
+ 日本西部
+ 美國中西部

受制於文件限制的服務適用於下列最大限制：

|  免費 | 基本 | S1 | S2 | S3 | S3&nbsp;HD |
|-------|-------|----|----|----|-------|
|  10,000 |100 萬 |每個分割區 1500 萬個或每個服務 1 億 8 千萬個 |每個分割區 6000 萬個或每個服務 7 億 2 千萬個 |每個分割區 1 億 2 千萬個或每個服務 14 億個 |每個索引 1 百萬個或每個分割區 2 億個 |

> [!Note] 
> 對於 2017 年底之後建立的 S3 高密度服務，已移除每個分割區 2 億份文件的限制，但保留每個索引 100 萬份文件的限制。


### <a name="document-size-limits-per-api-call"></a>每個 API 呼叫的文件大小限制

呼叫「索引 API」時的文件大小上限大約是 16 MB。

文件大小是索引 API 要求主體大小的實際限制。 由於您可以一次將多份文件整批傳遞給「索引 API」，因此大小限制實際上取決於批次中的文件數量。 針對具有單一文件的批次，文件大小上限為 16 MB 的 JSON。

為了降低文件大小，請記得從要求中排除不可查詢的資料。 影像和其他二進位資料無法執行查詢，而且不應該儲存於索引中。 若要將不可搜尋的資料整合到搜尋結果，請定義不可搜尋的欄位，將 URL 參考儲存於資源中。

## <a name="indexer-limits"></a>索引子限制

在 2017 年底之後建立的基本服務已將限制提高為 15 個索引、資料來源、技能集和索引子。

需要大量資源的作業 (例如 Azure Blob 索引中的映像分析，或認知搜尋中的自然語言處理) 有較短的執行時間上限，因此可以容納其他索引作業。 如果索引作業無法在允許的時間上限內完成，請按照排程嘗試執行索引作業。 排程器會追蹤索引狀態。 如果排定的索引作業因故中斷，索引子可以在下次排定的執行時間繼續從上次停止處進行。

| 資源 | 免費&nbsp;<sup>1</sup> | 基本&nbsp;<sup>2</sup>| S1 | S2 | S3 | S3&nbsp;HD&nbsp;<sup>3</sup>|
| -------- | ----------------- | ----------------- | --- | --- | --- | --- |
| 索引子上限 |3 |5 或 15|50 |200 |200 |N/A |
| 資料來源上限 |3 |5 或 15 |50 |200 |200 |N/A |
| 技能集上限為 <sup>4</sup> |3 |5 或 15 |50 |200 |200 |N/A |
| 每次叫用的索引編製負載上限 |10,000 份文件 |僅限制文件上限 |僅限制文件上限 |僅限制文件上限 |僅限制文件上限 |N/A |
| 執行時間上限 <sup>5</sup> | 1-3 分鐘 |24 小時 |24 小時 |24 小時 |24 小時 |N/A  |
| 影像分析的認知搜尋技能集或 Blob 索引適用的執行時間上限 <sup>5</sup> | 3-10 分鐘 |2 小時 |2 小時 |2 小時 |2 小時 |N/A  |
| Blob 索引子︰Blob 大小上限，MB |16 |16 |128 |256 |256 |N/A  |
| Blob 索引子︰從 Blob 擷取的內容字元數上限 |32,000 |64,000 |4 百萬 |4 百萬 |4 百萬 |N/A |

<sup>1</sup> 免費服務有索引子執行時間上限，針對 Blob 來源為 3 分鐘，針對其他所有資料來源為 1 分鐘。

<sup>2</sup> 在 2017 年底之後建立的基本服務已將限制提高為 15 個索引、資料來源和索引子。 稍早建立的服務有 5 個。

<sup>3</sup> S3 HD 服務不包含索引子支援。

<sup>4</sup> 每個技能集上限為 30 個技術。

<sup>5</sup> 在執行時間方面，Azure Blob 索引中的認知搜尋工作負載和映像分析較一般文字索引短。 影像分析和自然語言處理會耗用大量運算資源，並且需要大量的可用處理能力。 執行時間已減少，佇列中的其他作業才有機會執行。  

## <a name="queries-per-second-qps"></a>每秒查詢數目 (QPS)

每個客戶必須獨立開發 QPS 估計值。 索引大小和複雜性、查詢大小和複雜性、流量，這三者是 QPS 的主要決定因素。 不知道這些因素，便無法提供有意義的估計值。

計算在專用資源 (基本和標準層) 上執行的服務，更容易預測估計值。 由於可控制較多的參數，所以能更準確地估計 QPS。 如需有關如何進行估計的指引，請參閱 [Azure 搜尋服務的效能與最佳化](search-performance-optimization.md)。

## <a name="api-request-limits"></a>API 要求限制
* 每個要求最多 16 MB <sup>1</sup>
* 最長 8 KB 的 URL 長度
* 每個索引上傳、合併、或刪除批次最多包含 1000 個文件
* $orderby 子句中最多 32 個欄位
* 最大搜尋詞彙的大小是 utf-8 編碼文字的 32,766 個位元組 (32 KB 減 2 個位元組)

<sup>1</sup> 在「Azure 搜尋服務」中，要求主體的上限是 16 MB，這會針對不受理論上限制約束之個別欄位或集合的內容強加實際限制 (如需有關欄位組合和限制的詳細資訊，請參閱[支援的資料類型](https://docs.microsoft.com/rest/api/searchservice/supported-data-types))。

## <a name="api-response-limits"></a>API 回應限制
* 每一頁搜尋結果最多傳回 1000 個文件
* 每個建議 API 要求最多傳回 100 個建議

## <a name="api-key-limits"></a>API 金鑰限制
API 金鑰可用於服務驗證。 有兩種類型。 系統管理金鑰是在要求標頭中指定，並會授與完整的服務讀寫存取。 查詢金鑰為唯讀並在 URL 上指定，且通常會發佈到用戶端應用程式。

* 每個服務最多 2 個系統管理金鑰
* 每個服務最多 50 個查詢金鑰
