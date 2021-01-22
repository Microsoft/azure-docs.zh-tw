---
author: baanders
description: 包含 Azure 數位 Twins 限制的檔案
ms.service: digital-twins
ms.topic: include
ms.date: 6/9/2020
ms.author: baanders
ms.openlocfilehash: 08ae189253c6141a6b84035b71016ff5099ec162
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98689855"
---
### <a name="functional-limits"></a>功能限制

下表列出 Azure 數位 Twins 的功能限制。

| 區域 | 功能 | 預設限制 | 調？ |
| --- | --- | --- | --- |
| Azure 資源 | 每個訂用帳戶的區域中的 Azure 數位 Twins 實例數目 | 10 | 是 |
| Digital Twins | Azure 數位 Twins 實例中的 twins 數目 | 200,000 | Yes |
| Digital Twins | 單一對應項的連入關聯性數目 | 5,000 | No |
| Digital Twins | 來自單一對應項的傳出關聯性數目 | 5,000 | No |
| Digital Twins | 單一對應項的 PUT 或 PATCH 要求) 中，JSON 主體的大小上限 ( | 32 KB | No |
| Digital Twins | 要求承載大小上限 | 32 KB | No | 
| 路由 | 單一 Azure 數位 Twins 實例的端點數目 | 6 | No |
| 路由 | 單一 Azure 數位 Twins 實例的路由數目 | 6 | 是 |
| 模型 | 單一 Azure 數位 Twins 實例內的模型數目 | 10,000 | Yes |
| 模型 | 可在單一 API 呼叫中上傳的模型數目 | 250 | No |
| 模型 | 在單一頁面中傳回的專案數 | 100 | No |
| 查詢 | 在單一頁面中傳回的專案數 | 100 | 是 |
| 查詢 | `AND`  /  `OR` 查詢中的運算式數目 | 50 | 是 |
| 查詢 | 子句中的陣列專案數目 `IN`  /  `NOT IN` | 50 | 是 |
| 查詢 | 查詢中的字元數 | 8,000 | 是 |
| 查詢 | `JOINS`查詢中的數目 | 5 | 是 |

### <a name="rate-limits"></a>速率限制

下表反映不同 Api 的速率限制。

| API | 功能 | 預設限制 | 調？ |
| --- | --- | --- | --- |
| 模型 API | 每秒要求數 | 100 | 是 |
| 數位 Twins API | 每秒要求數 | 2,000 | Yes |
| 數位 Twins API | 每秒在 **所有 twins 和關聯** 性中的建立/刪除作業數目 | 50 | 是 |
| 數位 Twins API | **單一** 對應項或其關聯性每秒的建立/更新/刪除作業數目 | 10 | 否 |
| 查詢 API | 每秒要求數 | 500 | Yes |
| 查詢 API | 每秒[查詢單位](../articles/digital-twins/concepts-query-units.md)數 | 4,000 | Yes |
| 事件路由 API | 每秒要求數 | 100 | 是 |

### <a name="other-limits"></a>其他限制

您可以在 GitHub： [*數位 Twins 定義語言 (DTDL) -第2版*](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md)中，找到 Azure 數位 TWINS 模型 DTDL 檔中的資料類型和欄位限制。
 
查詢延遲詳細資料和其他查詢限制可在 how [*to：查詢*](../articles/digital-twins/how-to-query-graph.md)對應項圖形中找到。
