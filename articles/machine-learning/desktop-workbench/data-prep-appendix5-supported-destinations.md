---
title: 可供 Azure Machine Learning 資料準備使用的支援資料目的地和輸出 | Microsoft Docs
description: 本文件提供 Azure Machine Learning 資料準備可使用的支援目的地和輸出完整清單
services: machine-learning
author: euangMS
ms.author: euang
manager: lanceo
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: ''
ms.devlang: ''
ms.topic: article
ms.date: 02/01/2018
ROBOTS: NOINDEX
ms.openlocfilehash: f3d7d10db8cdeb4a1a409650f17d2fc39bf7971f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46969479"
---
# <a name="supported-data-exports-for-this-preview"></a>適用於此預覽的支援資料匯出 

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


您可匯出成幾個不同的格式。 可以先使用這些格式來保留資料準備的中繼結果，然後再將結果整合至其餘 Machine Learning 工作流程。

## <a name="types"></a>類型 
### <a name="csv-file"></a>CSV 檔案 
將逗號分隔值檔案寫入儲存體。

#### <a name="options"></a>選項
- 行尾結束符號
- 將 Null 取代為
- 將錯誤取代為 
- 分隔符號


### <a name="parquet"></a>Parquet 
將資料集寫入儲存體作為 Parquet。

Parquet 格式在儲存體中可以有各種形式。 對於較小的資料集，有時會用單一 .parquet 檔案。 各種 Python 程式庫均支援讀取和寫入至單一 .parquet 檔案。 

目前，Azure Machine Learning Workbench 會依賴 PyArrow Python 程式庫，以便在本機互動使用期間寫出 Parquet。 這表示單一檔案 Parquet 是本機互動使用期間，目前唯一支援的 Parquet 輸出格式。

(在 Spark 上) 向外延展執行期間，Azure Machine Learning Workbench 依賴 Spark 的 Parquet 讀取和寫入功能。 適用於 Parquet 的 Spark 預設輸出格式 (目前唯一支援) 與 Hive 資料集的結構相似。 這表示資料夾包含許多 .parquet 檔案，每個檔案都是較大資料集的較小分割。 

#### <a name="caveats"></a>需要注意的事項 
Parquet 格式相對比較年輕，並且在不同程式庫之間存在一些不一致的實作方法。 例如，Spark 在寫出至 Parquet 時，會在「資料行名稱」中的有效字元上設定限制。 PyArrow 不這麼做。 下列字元不能存在資料行名稱中： 
- ，
- ;
- {}
- ()
- \\n
- \\t
- =

>[!NOTE]
>- 為了確保與 Spark 相容，每當寫入資料至 Parquet 時，「資料行名稱」中出現的這些字元會以底線 ( _ ) 取代。
>- 為了確保在本機和向外延展執行時，寫入 Parquet 之任何資料之間的一致性 (透過應用程式、Python 或 Spark)，會將其「資料行名稱」妥善處理，以確保 Spark 相容性。 為了確保在 Spark 中寫入 Parquet 字元時產生預期的「資料行名稱」，請在寫出之前，先移除無效集合。



## <a name="locations"></a>位置 
### <a name="local"></a>本機 
本機硬碟或對應的網路儲存位置。

### <a name="azure-blob-storage"></a>Azure Blob 儲存體
Azure Blob 儲存體需要 Azure 訂用帳戶。

