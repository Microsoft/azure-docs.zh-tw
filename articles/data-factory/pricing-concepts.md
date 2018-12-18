---
title: 透過範例了解 Azure Data Factory 定價 | Microsoft Docs
description: 此文章透過詳細範例說明及示範 Azure Data Factory 的定價模型
documentationcenter: ''
author: shlo
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/25/2018
ms.author: shlo
ms.openlocfilehash: fae74d3912bb608afec93b7aa587885f66af4e53
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49115054"
---
# <a name="understanding-data-factory-pricing-through-examples"></a>透過範例了解 Data Factory 定價

此文章透過詳細範例說明及示範 Azure Data Factory 的定價模型。

## <a name="copy-data-from-aws-s3-to-azure-blob-storage-hourly"></a>每小時將資料從 AWS S3 複製到 Azure Blob 儲存體

在此案例中，您希望依每小時的排程將資料從 AWS S3 複製到 Azure Blob 儲存體。

若要完成案例，您需要使用下列項目建立管線：

1. 複製活動，包含要從 AWS S3 複製之資料的輸入資料集。

2. Azure 儲存體上之資料的輸出資料集。

3. 每小時執行管線的排程觸發程序。

 ![案例 1](media/pricing-concepts/scenario1.png)

| **作業** | **類型與單位** |
| --- | --- |
| 建立連結的服務 | 2 個讀取/寫入實體  |
| 建立資料集 | 4 個讀取/寫入實體 (2 個用於建立資料集，2 個用於連結的服務參考) |
| 建立管線 | 3 個讀取/寫入實體 (1 個用於建立管線，2 個用於資料集參考) |
| 取得管線 | 1 個讀取/寫入實體 |
| 執行管線 | 2 個活動執行 (1 個用於觸發程序執行，1 個用於活動執行) |
| 複製資料假設︰執行時間 = 10 分鐘 | 10 \* 4 Azure 整合執行階段 (預設 DIU 設定 = 4) 如需有關資料整合單位和最佳化複製效能的詳細資訊，請參閱[此文章](copy-activity-performance.md) |
| 監視管線假設：僅發生 1 次執行 | 2 個重試的監視執行記錄 (1 個用於管線執行，1 個用於活動執行) |

**總案例定價：$0.16811**

- Data Factory 作業 = **$0.0001**
  - 讀取/寫入 = 10\*00001 = $0.0001 [1 讀取/寫入 = $0.50/50000 = 0.00001]
  - 監視 = 2\*000005 = $0.00001 [1 監視 = $0.25/50000 = 0.000005]
- 管線協調流程 &amp; 執行 = **$0.168**
  - 活動執行 = 001\*2 = 0.002 [1 執行 = $1/1000 = 0.001]
  - 資料移動活動 = $0.166 (依比例分配 10 分鐘的執行時間。 Azure 整合執行階段上每小時 0.25 美元)

## <a name="copy-data-and-transform-with-azure-databricks-hourly"></a>每小時複製資料，並使用 Azure Databricks 轉換

在此案例中，您希望依每小時的排程將資料從 AWS S3 複製到 Azure Blob 儲存體，並使用 Azure Databricks 轉換。

若要完成案例，您需要使用下列項目建立管線：

1. 一個複製活動，包含要從 AWS S3 複製資料的輸入資料集，以及 Azure 儲存體上之資料的輸出資料集。
2. 一個用於資料轉換的 Azure Databricks 活動。
3. 一個每小時執行管線的排程觸發程序。

![案例 2](media/pricing-concepts/scenario2.png)

| **作業** | **類型與單位** |
| --- | --- |
| 建立連結的服務 | 3 個讀取/寫入實體  |
| 建立資料集 | 4 個讀取/寫入實體 (2 個用於建立資料集，2 個用於連結的服務參考) |
| 建立管線 | 3 個讀取/寫入實體 (1 個用於建立管線，2 個用於資料集參考) |
| 取得管線 | 1 個讀取/寫入實體 |
| 執行管線 | 3 個活動執行 (1 個用於觸發程序執行，2 個用於活動執行) |
| 複製資料假設︰執行時間 = 10 分鐘 | 10 \* 4 Azure 整合執行階段 (預設 DIU 設定 = 4) 如需有關資料整合單位和最佳化複製效能的詳細資訊，請參閱[此文章](copy-activity-performance.md) |
| 監視管線假設：僅發生 1 次執行 | 3 個重試的監視執行記錄 (1 個用於管線執行，2 個用於活動執行) |
| 執行 Databricks 活動假設：執行時間 = 10 分鐘 | 10 分鐘外部管線活動執行 |

**總案例定價：$0.16916**

- Data Factory 作業 = **$0.00012**
  - 讀取/寫入 = 11\*00001 = $0.00011 [1 R/W = $0.50/50000 = 0.00001]
  - 監視 = 3\*000005 = $0.00001 [1 監視 = $0.25/50000 = 0.000005]
- 管線協調流程 &amp; 執行 = **$0.16904**
  - 活動執行 = 001\*3 = 0.003 [1 執行 = $1/1000 = 0.001]
  - 資料移動活動 = $0.166 (依比例分配 10 分鐘的執行時間。 Azure 整合執行階段上每小時 0.25 美元)
  - 外部管線活動 = $0.000041 (依比例分配 10 分鐘的執行時間。 Azure 整合執行階段上每小時 0.00025 美元)

## <a name="copy-data-and-transform-with-dynamic-parameters-hourly"></a>每小時複製資料並使用動態參數進行轉換

在此案例中，您希望依每小時的排程將資料從 AWS S3 複製到 Azure Blob 儲存體，並使用 Azure Databricks (透過指令碼中的動態參數) 轉換。

若要完成案例，您需要使用下列項目建立管線：

1. 一個複製活動，包含要從 AWS S3 複製資料的輸入資料集、Azure 儲存體上之資料的輸出資料集。
2. 一個 Lookup 活動，用於以動態方式將參數傳遞到轉換指令碼。
3. 一個用於資料轉換的 Azure Databricks 活動。
4. 一個每小時執行管線的排程觸發程序。

![案例 3](media/pricing-concepts/scenario3.png)

| **作業** | **類型與單位** |
| --- | --- |
| 建立連結的服務 | 3 個讀取/寫入實體  |
| 建立資料集 | 4 個讀取/寫入實體 (2 個用於建立資料集，2 個用於連結的服務參考) |
| 建立管線 | 3 個讀取/寫入實體 (1 個用於建立管線，2 個用於資料集參考) |
| 取得管線 | 1 個讀取/寫入實體 |
| 執行管線 | 4 個活動執行 (1 個用於觸發程序執行，3 個用於活動執行) |
| 複製資料假設︰執行時間 = 10 分鐘 | 10 \* 4 Azure 整合執行階段 (預設 DIU 設定 = 4) 如需有關資料整合單位和最佳化複製效能的詳細資訊，請參閱[此文章](copy-activity-performance.md) |
| 監視管線假設：僅發生 1 次執行 | 4 個重試的監視執行記錄 (1 個用於管線執行，3 個用於活動執行) |
| 執行 Lookup 活動假設：執行時間 = 1 分鐘 | 1 分鐘管線活動執行 |
| 執行 Databricks 活動假設：執行時間 = 10 分鐘 | 10 分鐘外部管線活動執行 |

**總案例定價：$0.17020**

- Data Factory 作業 = **$0.00013**
  - 讀取/寫入 = 11\*00001 = $0.00011 [1 R/W = $0.50/50000 = 0.00001]
  - 監視 = 4\*000005 = $0.00002 [1 監視 = $0.25/50000 = 0.000005]
- 管線協調流程 &amp; 執行 = **$0.17007**
  - 活動執行 = 001\*4 = 0.004 [1 執行 = $1/1000 = 0.001]
  - 資料移動活動 = $0.166 (依比例分配 10 分鐘的執行時間。 Azure 整合執行階段上每小時 0.25 美元)
  - 管線活動 = $0.00003 (依比例分配 1 分鐘的執行時間。 Azure 整合執行階段上每小時 $0.002 美元)
  - 外部管線活動 = $0.000041 (依比例分配 10 分鐘的執行時間。 Azure 整合執行階段上每小時 0.00025 美元)

## <a name="next-steps"></a>後續步驟

既然您了解 Azure Data Factory 的定價，您可以立即開始！

- [使用 Azure Data Factory UI 建立資料處理站](quickstart-create-data-factory-portal.md)

- [Azure Data Factory 簡介](introduction.md)

- [Azure Data Factory 中的視覺化撰寫](author-visually.md)