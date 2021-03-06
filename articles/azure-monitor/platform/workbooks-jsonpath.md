---
title: Azure 監視器活頁簿-使用 JSONPath 轉換 JSON 資料
description: 如何使用 Azure 監視器活頁簿中的 JSONPath，將所查詢端點所傳回的 JSON 資料結果轉換成您想要的格式。
services: azure-monitor
author: lgayhardt
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 05/06/2020
ms.author: lagayhar
ms.openlocfilehash: a2411d9257b1083cb2bcbfcad289813a6c062dff
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143579"
---
# <a name="how-to-use-jsonpath-to-transform-json-data-in-workbooks"></a>如何使用 JSONPath 轉換活頁簿中的 JSON 資料

活頁簿可以查詢來自許多來源的資料。 某些端點（例如 [Azure Resource Manager](../../azure-resource-manager/management/overview.md) 或自訂端點）可以傳回 JSON 格式的結果。 如果查詢的端點所傳回的 JSON 資料未使用您想要的格式進行設定，則可以使用 JSONPath 來轉換結果。

JSONPath 是 JSON 的查詢語言，類似于 XML 的 XPath。 就像 XPath 一樣，JSONPath 可讓您從 JSON 結構中解壓縮和篩選資料。

藉由使用 JSONPath 轉換，活頁簿作者可以將 JSON 轉換成資料表結構。 然後，您可以使用該資料表來繪製活頁 [簿視覺效果](./workbooks-overview.md#visualizations)。

## <a name="using-jsonpath"></a>使用 JSONPath

1. 按一下 [ *編輯* ] 工具列專案，將活頁簿切換至編輯模式。
2. 使用 [*加入*  >  *加入查詢*] 連結，將查詢控制項加入至活頁簿。
3. 將資料來源選取為 *JSON*。
4. 使用 JSON 編輯器來輸入下列 JSON 程式碼片段
    ```json
    { "store": {
        "books": [ 
          { "category": "reference",
            "author": "Nigel Rees",
            "title": "Sayings of the Century",
            "price": 8.95
          },
          { "category": "fiction",
            "author": "Evelyn Waugh",
            "title": "Sword of Honour",
            "price": 12.99
          },
          { "category": "fiction",
            "author": "Herman Melville",
            "title": "Moby Dick",
            "isbn": "0-553-21311-3",
            "price": 8.99
          },
          { "category": "fiction",
            "author": "J. R. R. Tolkien",
            "title": "The Lord of the Rings",
            "isbn": "0-395-19395-8",
            "price": 22.99
          }
        ],
        "bicycle": {
          "color": "red",
          "price": 19.95
        }
      }
    }
    ```  

假設我們將上述 JSON 物件指定為商店清查的標記法。 我們的工作是藉由列出商店的標題、作者和價格，來建立商店可用書籍的表格。

1. 選取 [ *結果設定* ] 索引標籤，並將結果格式切換至 [ *JSON 路徑*]。
2. 套用下列 JSON 路徑設定：

    JSON 路徑表： `$.store.books` 。 此欄位代表資料表根目錄的路徑。 在此情況下，我們在意商店的書籍清查。 資料表路徑會將 JSON 篩選至書籍資訊。

   | 資料行識別碼 | 資料行 JSON 路徑 |
   |:-----------|:-----------------|
   | 標題      | `$.title`        |
   | 作者     | `$.author`       |
   | 價格      | `$.price`        |

    資料行識別碼將會是資料行標頭。 [資料行 JSON 路徑] 欄位代表從資料表根目錄到資料行值的路徑。

1. 按一下 [*執行查詢*] 以套用上述設定

![ 使用 JSON 資料來源和 JSON 路徑結果格式編輯查詢專案](./media/workbooks-jsonpath/query-jsonpath.png)

## <a name="next-steps"></a>後續步驟
- [活頁簿總覽](workbooks-overview.md)
- [Azure 監視器活頁簿中的群組](workbooks-groups.md)