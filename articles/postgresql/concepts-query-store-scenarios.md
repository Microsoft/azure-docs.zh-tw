---
title: 查詢存放區使用案例 (適用於 PostgreSQL 的 Azure 資料庫)
description: 本文描述在適用於 PostgreSQL 的 Azure 資料庫中使用查詢存放區的一些案例。
services: postgresql
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/26/2018
ms.openlocfilehash: fc354a9545f3073a858c2b669d428d1fa4185073
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49377465"
---
# <a name="usage-scenarios-for-query-store"></a>查詢存放區的使用案例

**適用對象：** 適用於 PostgreSQL 的 Azure 資料庫 9.6 和 10

> [!IMPORTANT]
> 查詢存放區功能處於公開預覽狀態。

您可以在各種不同的案例中使用查詢存放區，在這些案例中追蹤和維護可預測的工作負載效能至關重要。 請思考一下以下範例： 
- 識別並調整前幾個成本昂貴的查詢 
- A/B 測試 
- 在升級期間保持效能穩定 
- 識別並改善臨機操作工作負載 

## <a name="identify-and-tune-expensive-queries"></a>識別並調整成本昂貴的查詢 

### <a name="identify-longest-running-queries"></a>識別長時間執行的查詢 
使用 Azure 入口網站中的[查詢效能見解](concepts-query-performance-insight.md)檢視，即可快速識別執行時間最長的查詢。 這些查詢通常會消耗大量資源。 最佳化執行時間最長的問題可提高效能，方法是釋放資源以供系統上執行的其他查詢使用。 

### <a name="target-queries-with-performance-deltas"></a>使用效能差異鎖定查詢 
查詢存放區會將效能資料分割成時間範圍，讓您能夠追蹤一段時間內的查詢效能。 這可協助您確切識別哪些查詢會增加所花費的整體時間。 如此一來，您就可以對工作負載進行具有針對性的疑難排解。

### <a name="tuning-expensive-queries"></a>調整成本昂貴的查詢 
當您識別效能欠佳的查詢時，您要採取的動作取決於問題的本質： 
- 使用[效能建議](concepts-performance-recommendations.md)來判斷是否有任何建議的索引。 若有，請建立索引，然後在建立索引之後使用查詢存放區來評估查詢效能。 
- 針對查詢所使用的基礎資料表，確定其中的統計資料為最新。
- 請考慮重新撰寫成本昂貴的查詢。 例如，利用查詢參數化並減少使用動態 SQL。 在讀取資料時實作最佳邏輯，例如在資料庫端上套用資料篩選，而不是在應用程式端上套用。 


## <a name="ab-testing"></a>A/B 測試 
使用查詢存放區，可比較您計劃引進應用程式變更之前和之後的工作負載效能。 使用查詢存放區來評估環境或應用程式變更對工作負載效能所造成影響的案例範例如下： 
- 推出新版本的應用程式。 
- 在伺服器上新增額外的資源。 
- 在成本昂貴查詢所參考資料表上建立遺漏的索引。 
 
在上述任一案例中，套用下列工作流程： 
1. 在計劃性變更之前使用查詢存放區執行您的工作負載，以產生效能基準。 
2. 在受控制的時間點套用應用程式變更。 
3. 在變更之後繼續執行工作負載足夠長的時間，以產生系統的效能影像。 
4. 比較變更之前和之後的結果。 
5. 決定是要保留變更還是復原。 


## <a name="identify-and-improve-ad-hoc-workloads"></a>識別並改善臨機操作工作負載 
某些工作負載沒有可調整的主控查詢，無法提高整體的應用程式效能。 這些工作負載通常具有相對大量的唯一查詢，各個查詢都會耗用一部分的系統資源。 每個唯一查詢都不會經常執行，因此其個別執行階段耗用量並不重要。 另一方面，假設應用程式一直產生新的查詢，則絕大部分的系統資源會花費在查詢編譯，這不是最佳狀況。 通常，如果您的應用程式會產生查詢 (而不是使用預存程序或參數化查詢)，或者它依賴於預設會產生查詢的物件關聯式對應架構，就會發生這種情況。 
 
如果您能夠掌控應用程式程式碼，則可以考慮重新撰寫資料存取層來使用預存程序或參數化查詢。 不過，您也可以改善這種情況而無需變更應用程式，只要使用相同的查詢雜湊，針對整個資料庫 (所有查詢) 或個別查詢範本強制執行查詢參數化即可。 

## <a name="next-steps"></a>後續步驟
- 深入了解[使用查詢存放區的最佳做法](concepts-query-store-best-practices.md)