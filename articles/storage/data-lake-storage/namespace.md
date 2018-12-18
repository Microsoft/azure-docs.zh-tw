---
title: Azure Data Lake Storage Gen2 預覽版階層式命名空間
description: 說明 Azure Data Lake Storage Gen2 預覽版階層式命名空間的概念
services: storage
author: jamesbak
ms.service: storage
ms.topic: article
ms.date: 06/27/2018
ms.author: jamesbak
ms.component: data-lake-storage-gen2
ms.openlocfilehash: 44eec21f4687d2df64c59d41cdb02c6ef2268f82
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39528692"
---
# <a name="azure-data-lake-storage-gen2-preview-hierarchical-namespace"></a>Azure Data Lake Storage Gen2 預覽版階層式命名空間

若要允許 Azure Data Lake Storage Gen2 預覽版以物件儲存體的規模和價格提供檔案系統效能，其關鍵機制在於新增**階層式命名空間**。 這會收集帳戶中的物件/檔案集合，並整理成階層式目錄和巢狀子目錄，此方式與整理電腦上的檔案系統方式相同。 啟用階層式命名空間後，Data Lake Storage Gen2 可擴展物件儲存體並符合成本效益，並提供分析引擎及架構所熟悉的檔案系統語意。

## <a name="the-benefits-of-the-hierarchical-namespace"></a>階層式命名空間的優勢

> [!NOTE]
> 在 Azure Data Lake Storage Gen2 公開預覽期間，以下所列部分功能的可用性可能會有所不同。 在預覽計畫期間發布新功能和區域時，將會透過我們專屬的 Yammer 群組傳達此資訊。  

以下優勢是關於在 blob 資料上執行階層式命名空間的檔案系統：

- **不可部分完成的目錄操作：** 物件採用在物件名稱中內嵌斜線 (/) 以表示路徑線段的慣例，儲存近似目錄階層的架構。 雖然此慣例適用於整理物件，但此慣例不提供移動、重新命名或刪除目錄等協助動作。 如果沒有真正的目錄，應用程式就必須處理可能數百萬個單獨的 blob 才能達到目錄層級的工作。 相較之下，階層式命名空間以更新單一項目 (父目錄) 的方式來處理這些工作。 

    這種大幅最佳化對於許多巨量資料分析架構來說尤為重要。 Hive 和 Spark 等工具經常將輸出寫入暫時位置，然後在作業結束時重新命名該位置。 如果沒有階層式命名空間，重新命名所花費的時間通常會比分析流程本身更長。 較低的作業延遲等於較低的分析工作負載擁有權總成本 (TCO)。

- **熟悉的介面樣式：** 開發人員和使用者都很熟悉檔案系統。 當您前往雲端時不需要學習新的儲存體範例，因為 Data Lake Storage Gen2 公開的檔案系統介面與大小和電腦使用的範例相同。

以往物件儲存不支援階層式命名空間的原因之一，是因為階層式命名空間限制了規模。 但是，Data Lake Storage Gen2 階層式命名空間能夠線性擴展，不會降低資料容量或效能。

## <a name="when-to-enable-the-hierarchical-namespace"></a>階層式命名空間啟用時機

對於針對操作目錄的檔案系統所設計的儲存體工作負載而言，建議啟用階層式命名空間。 這包括主要用來分析處理的所有工作負載。 需要有效組織的資料集也會因啟用階層式命名空間而受益。

啟用階層式命名空間的原因由 TCO 分析確定。 一般而言，若要改善因儲存體加速而導致的工作負載延遲，需要計算資源才能縮短時間。 由於階層式命名空間啟用的不可部分完成目錄操作，許多工作負載的延遲可能因此獲得改善。 在許多工作負載中，計算資源佔總成本的 85％ 以上，因此即使適度減少工作負載延遲也相當於大量節省 TCO。 即使啟用階層式命名空間會提高儲存體成本，但由於計算成本降低，TCO 仍會降低。

## <a name="when-to-disable-the-hierarchical-namespace"></a>階層式命名空間停用時機

某些物件儲存工作負載可能無法因啟用階層式命名空間而受益。 這些工作負載範例包括備份、影像儲存及其他應用程式，其中物件組織與物件本身會分開儲存 (*例如*在單獨的資料庫中)。

> [!NOTE]
> 在預覽版本中，如果啟用階層式命名空間，則 Blob 和 Data Lake Storage Gen2 REST API 之間不會互通資料或作業。 這項功能將會在預覽期間加入。

## <a name="next-steps"></a>後續步驟

- [建立儲存體帳戶](./quickstart-create-account.md)